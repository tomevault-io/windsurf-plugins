---
trigger: always_on
description: Çip Tasarımı: RTL'den Tapeout'a Pratik Workflow. Verilog/SystemVerilog, açık kaynak EDA (OpenROAD, Yosys, OpenLane, Verilator), SKY130 PDK, RISC-V CPU tasarımı, FPGA prototyping ve tapeout süreçleri.
---


# CHİP-DESİGN — Çip Tasarımı Skill'i

Pratik çip tasarımı workflow'u: RTL'den tapeout'a kadar end-to-end süreç.

## ⚡ Hızlı Başlangıç

```
1. Konsept           → Belirle ne yapacaksın
2. RTL Kodlama       → Verilog/SystemVerilog
3. Simülasyon        → Verilator (ücretsiz) veya Icarus
4. Sentez            → Yosys (ücretsiz)
5. P&R               → OpenROAD veya OpenLane
6. Fiziksel Doğrulama → Magic (DRC/LVS)
7. GDSII Export      → Tapeout için hazır dosya
```

##  RTL Tasarımı

### Dil Seçimi

**Verilog (IEEE 1364):** Temel, yaygın, tüm araçlar destekler. 1995'ten beri standart.

**SystemVerilog (IEEE 1800):** Verilog'un superseti. Verification için zorunlu (class, interface, coverage, assertions, package). Üretim tasarımında da giderek yaygınlaşıyor.

**Chisel (Scala-based):** Yüksek seviyeli, OO + fonksiyonel. Rocket Chip, CVA6, SiFive çipleri bu dilde yazılmış. Kod üretimi çok daha kompakt.

**SpinalHDL (Scala-based):** Chisel'e alternatif. Daha güçlü type system, daha fazla kontrol. Tüm VHDL/Verilog'a compile ediyor.

**Amaranth HDL (Python-based):** Python DSL. Linux dünyasında popüler. Nmigen projesinin devamı.

**Clash (Haskell-based):** Haskell'in gücünü HDL'e taşıyor. Strong typing, compile-time synthesis.

**MyHDL (Python-based):** Python → Verilog dönüştürücü. Düşük bariyer giriş için iyi.

### Temel Yapılar

```verilog
// Kombinasyonel
assign y = a & b;
always @* begin
  case (sel)
    2'b00: out = a + b;
    2'b01: out = a - b;
    default: out = 0;
  endcase
end

// Senkron
always @(posedge clk or negedge rst_n) begin
  if (!rst_n) q <= 0;
  else q <= d;
end

// FSM (one-hot encoding)
typedef enum logic [2:0] {
  IDLE  = 3'b001,
  READ  = 3'b010,
  WRITE = 3'b100
} state_t;
```

### High-Level Synthesis (HLS)

C/C++/SystemC → RTL dönüşümü. Tasarımcı daha soyut seviyede tanımlama yapar.

**Komersiyel HLS Araçları:**
- **Vitis HLS** (AMD/Xilinx) — C/C++/SystemC → Verilog/VHDL, Xilinx FPGA hedefli
- **Intel HLS Compiler** — C++ → Verilog, Intel FPGA hedefli
- **Catapult HLS** (Siemens) — C++/SystemC → RTL, ASIC + FPGA
- **Stratus HLS** (Cadence) — C/SystemC → RTL, yüksek kalite

**Açık Kaynak HLS:**
- **LegUp** (Toronto Üniv.) — C → Verilog, LLVM tabanlı, FPGA
- **Bambu** (Politecnico di Milano) — C → Verilog, geniş input desteği
- **OpenArc** — AccelChip mirası, ticari desteği olan açık sistem

**Ne zaman HLS:** Algoritmik yoğun tasarımlar (signal processing, video codec), büyük veri yolu manipülasyonu. Manuel RTL daha fazla kontrol sağlar.

### Best Practices

**Clock Domain Crossing (CDC):**
- Tek bit: 2-flip-flop synchronizer (minimum)
- Çok bit: Gray code FIFO veya handshake protocol
- Asenkron FIFO:读写 pointer Gray encode, full/empty binary compare

**Reset:**
- Asenkron reset kullanıyorsan: recovery ve removal timing analiz et
- Senkron reset tercih et (daha az glitch riski)
- Reset tree synthesis: gated reset, reset buffering

**Pipeline:**
- Throughput × latency tradeoff: her pipeline stage ~1 clock cycle ekler
- Stall sinyal: upstream'e "durdur" sinyali ver
- Bubble: gereksiz stage'leri boş atla
- Handshake: valid/ready protokolü ile data transfer

**Coding style:**
- Tek boyutlu wire/reg
- `casez`/`casex` yerine `case` + if/else wildcards
- Synchronous reset her zaman tercih et
- Lint araçları (Verilator `--lint-only`) sürekli çalıştır

##  Açık Kaynak EDA Araçları

### Kurulum (Ubuntu 22.04+)

```bash
# Docker (OpenROAD için — önerilen)
sudo apt install -y docker.io
sudo usermod -aG docker $USER
docker pull theopenroadproject/openroad:latest

# Yosys
sudo apt install yosys

# Verilator (en güncel için: kaynak koddan derle)
sudo apt install verilator

# Magic (DRC/LVS)
sudo apt install magic

# Icarus Verilog
sudo apt install iverilog

# KLayout (GDS viewer)
sudo apt install klayout

# Netgen (LVS)
sudo apt install netgen
```

### OpenLane Workflow

OpenROAD'ın wrapper'ı. Design exploration ve hardening için.

```bash
# OpenLane indir
git clone --depth 1 https://github.com/The-OpenROAD-Project/OpenLane.git
cd OpenLane
make setup  # PDK'ları indirir (~5GB)

# İlk tasarım
cp -r designs/spm designs/my_chip
# designs/my_chip/config.tcl düzenle

# Akışı çalıştır
./flow.tcl -design my_chip -tag run_001

# Sonuçlar
ls designs/my_chip/runs/run_001/results/final/
```

### OpenROAD Komutları

```bash
# Interactive
openroad

# Script ile
openroad -quiet -skel scripts/floorplan.tcl
openroad -quiet scripts/pdn.tcl
openroad -quiet scripts/route.tcl

# Timing analizi
openroad -quiet -skel scripts/timing.tcl
```

### Yosys Sentez

```bash
# Basic synthesis (ekran çıktısı)
yosys -p "read_verilog design.v; synth -top top; stat"

# FPGA (Xilinx)
yosys -p "read_verilog design.v; synth_xilinx -top top -edif design.edif"

# ASIC (SKY130)
yosys -p "
  read_verilog rtl.v
  synth -top top
  dfflibmap -liberty $PDK_ROOT/sky130A/libs.ref/sky130_sram_sc_hd/lib/sky130_sram_1kbyte_1rw1r_32x256_8_hd.v
  abc -liberty $PDK_ROOT/sky130A/libs.ref/sky130_sram_sc_hd/lib/sky130_sram_1kbyte_1rw1r_32x256_8_hd.v
  stat
"
```

### Verilator Simülasyon

```bash
# Lint (syntax check)
verilator --lint-only --Wall design.sv

# Compile
verilator --cc --exe --build -j 0 \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yunusgungor/chip-design-skill](https://github.com/yunusgungor/chip-design-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
