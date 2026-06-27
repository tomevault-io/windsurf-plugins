---
trigger: always_on
description: `tests/fixtures/integration_annotated.vcf` is a pre-annotated (post-fastVEP) VCF designed to
---

# mafsmith — development notes for Claude

## Integration test fixture

`tests/fixtures/integration_annotated.vcf` is a pre-annotated (post-fastVEP) VCF designed to
cover every conversion scenario that has required a bug fix. Run it with `--skip-annotation`
to exercise conversion logic in isolation:

```bash
target/release/mafsmith vcf2maf \
  -i tests/fixtures/integration_annotated.vcf \
  -o /tmp/out.maf \
  --vcf-tumor-id test_tumor --tumor-id test_tumor \
  --vcf-normal-id test_normal --normal-id test_normal \
  --genome grch38 --skip-annotation
```

### Scenarios covered and why each was added

| Record | Variant_Classification | Variant_Type | What it tests / bug it covers |
|--------|----------------------|--------------|-------------------------------|
| 21:34792292 A>C | Missense_Mutation | SNP | Multi-transcript CSQ → canonical MANE transcript selected |
| 21:34880555 A>ACCTCTT | Splice_Site | INS | Insertion classified correctly |
| 21:34880650 T>TG | Frame_Shift_Ins | INS | Frameshift insertion |
| 21:41479182 GT>G | Frame_Shift_Del | DEL | Frameshift deletion |
| 21:41479219 C>T | Silent | SNP | Synonymous SNV |
| 21:44237111 C>T | Nonsense_Mutation | SNP | Stop gained |
| 21:34880300 G>A,C GT=0/0 | Missense_Mutation | SNP | Multi-allelic GVCF: picks highest-depth alt (C, 25 reads) not A (3 reads). Bug: mafsmith defaulted to ALT[0] instead of highest-depth alt for hom-ref GT calls |
| 1:1000000 `<DEL>` CHR2+END | Intron | INS | SV DEL emits secondary breakpoint row. Bug: only 1 row was emitted |
| 1:2000000 `<DUP:TANDEM>` | Intron | INS | ALT normalized to `<DUP>`. Bug: `<DUP:TANDEM>` was passed through verbatim |
| 1:3000000 `G]2:5000000]` BND CHR2=2 | Intron | INS | BND secondary row on chr2; HGVSc=`5000000]` (greedy last-colon strip). Bug: used first colon |
| 1:4000000 `T]2:6000000]` BND no CHR2/END in INFO | Intron | INS | BND secondary row parsed from ALT notation. Bug: only INFO was checked; fix: parse `chr:pos` from `]chr:pos]` or `[chr:pos[` in ALT when CHR2 absent |
| 1:5000000 `<INV>` CHR2+END | Intron | INS | INV secondary breakpoint row |
| 1:6000000 `<DEL>` multi-consequence | Splice_Site | INS | `feature_truncation&splice_acceptor_variant` → Splice_Site. Bug: first consequence short-circuited; fix: sort by severity |
| 21:45000000 GGCT>G | In_Frame_Del | DEL | 3 bp in-frame deletion |
| 21:45100000 G>GCAG | In_Frame_Ins | INS | 3 bp in-frame insertion |
| 21:45200000 T>C | Nonstop_Mutation | SNP | stop_lost |
| 21:45300000 A>G | Translation_Start_Site | SNP | start_lost |
| 21:45400000 A>G | Splice_Region | SNP | splice_region_variant (also tests multi-consequence sort) |
| 21:45500000 C>T | 3'UTR | SNP | 3_prime_UTR_variant |
| 21:45600000 G>A | 5'UTR | SNP | 5_prime_UTR_variant |
| 21:45700000 T>C | 5'Flank | SNP | upstream_gene_variant |
| 21:45800000 A>G | 3'Flank | SNP | downstream_gene_variant |
| 21:45900000 G>A | Intron | SNP | intron_variant (SNV, not SV) |
| 21:46000000 C>T | IGR | SNP | intergenic_variant |
| 21:46100000 G>A | RNA | SNP | non_coding_transcript_exon_variant (lncRNA biotype) |
| 21:46200000 AC>GT | Missense_Mutation | DNP | Dinucleotide polymorphism Variant_Type |
| 21:46300000 A>G GT=0/1 AD=2,18 | Missense_Mutation | SNP | Het call with VAF=0.9 ≥ 0.7: Tumor_Seq_Allele1 must stay REF. Bug: VAF override fired on explicit GT=0/1; fix: suppress override when GT has both ref (0) and alt (>0) indices |

### Edge case bugs found and fixed via real-VCF comparison

| Bug | Root cause | Fix | Verified on |
|-----|-----------|-----|-------------|
| Ion Torrent multi-allelic: `t_alt_count` wrong for GT=0/2 | `AO` is alt-only (no REF entry at index 0). For VCF allele index N, correct AO element is `AO[N-1]`. Code always took `AO[0]`. | `depth.rs` case 6: use `ao.split(',').nth(alt_vcf_idx.saturating_sub(1))` | syn20443868 (147→0 diffs) |
| `protein_altering_variant` mis-classified as Frame_Shift for in-frame indels with "-" allele | `inframe` computed as `"-".len() - alt.len() = 1 - 9 = 8; 8%3≠0` instead of `0 - 9 = 9; 9%3==0`. Normalized insertions use "-" placeholder (len=1) not empty string (len=0). | `consequence.rs`: treat `"-"` as len=0 in inframe check | syn5614682 (1 diff fixed) |
| Truncated AD alt-index OOB: `t_ref_count`/`t_alt_count` serialized as `''` instead of `'.'` | For multi-allelic records where AD has fewer values than alts (e.g. AD="0,2" with 3 alleles and effective_alt_vcf_idx=2), alt_count=None was serialized as `''`. vcf2maf.pl outputs `'.'`. | `vcf2maf.rs`: added `tumor_alt_oor`/`normal_alt_oor` flags when AD has real values but alt_vcf_idx is OOB; required `ad_len >= 2` to avoid false-positive on VarScan single-value AD | syn5555584, syn5553158 |
| VarScan somatic: all depth fields outputting `'.'`, all TSA1 defaulting to REF | Two bugs: (1) `ad_oor` check fired on VarScan's single-value AD (ad_len=1 <= alt_vcf_idx=1), masking VarScan depth extraction. (2) Depth-based hom_alt TSA1 fallback used `ad_vals.get(alt_vcf_idx)` which fails for VarScan's AD[0] = alt_count. | (1) `vcf2maf.rs` `ad_oor`: add `ad_len >= 2` guard. (2) Replace AD-index hom_alt logic with `extract_depth()` call that handles all callers | syn6840402 (933→0 diffs) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nf-osi/mafsmith](https://github.com/nf-osi/mafsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
