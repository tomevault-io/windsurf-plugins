---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Idioma: o domínio (boletos, CNAB, bancos brasileiros), a documentação e os commits são em português. Escreva comentários, mensagens de validação e commits em português, como no restante do código.

## O que é

Gem `brcobranca` — emissão de boletos bancários e geração/leitura de arquivos CNAB (remessa e retorno) para bancos brasileiros. Biblioteca pura, sem Rails: **não depende de ActiveSupport nem de ActiveModel** (ver "Sem ActiveSupport" abaixo).

## Comandos

```bash
bundle exec rake                              # suíte completa (é o que o CI roda)
bundle exec rspec                             # idem
bundle exec rspec spec/brcobranca/boleto/itau_spec.rb          # um arquivo
bundle exec rspec spec/brcobranca/boleto/itau_spec.rb:42       # um exemplo (linha)
bundle exec rspec -e "nosso numero"           # por descrição
bundle exec rubocop --parallel                # lint (CI usa --fail-level E)
bundle exec rubocop -a                        # autocorreção segura
rake build / rake install / rake release      # tarefas do bundler/gem_tasks
```

### Ambiente

- **rbenv**; `.ruby-version` pede **3.4.3**, que pode não estar instalado localmente (há 3.4.9 e 4.0.x). Se `ruby -v` reclamar, use `RBENV_VERSION=3.4.9 bundle exec ...` ou instale a versão pedida — não altere `.ruby-version` sem combinar.
- **GhostScript é obrigatório** para gerar boletos (PDF/PNG/…) e os specs de boleto usam. macOS: `brew install ghostscript` (`gs`).
- Cobertura via SimpleCov sai em `coverage/` a cada `rspec`.

### Compatibilidade de sintaxe (importante)

O CI roda a matriz **Ruby 3.3, 3.4, 4.0 + head**, o gemspec exige `>= 3.3.0` e o RuboCop tem `TargetRubyVersion: 3.3` — as versões anteriores saíram junto com o EOL upstream. Sintaxe de 3.3 é permitida; nada de 3.4+ exclusivo. `# frozen_string_literal: true` é obrigatório em todo arquivo.

## Arquitetura

Três subsistemas independentes sob `lib/brcobranca/`, todos registrados por `autoload` em `lib/brcobranca.rb`:

| Subsistema | Entrada | Saída |
| --- | --- | --- |
| `Boleto::*` | atributos do título | arquivo PDF/PNG/… + código de barras/linha digitável |
| `Remessa::*` | objetos `Pagamento` | string do arquivo CNAB 240/400/444 enviado ao banco |
| `Retorno::*` | arquivo CNAB do banco | array de objetos com os campos parseados |

**`lib/brcobranca.rb` é o registro central.** Toda classe nova (banco, remessa, retorno) precisa de uma linha `autoload` lá, senão simplesmente não existe. É também onde ficam as exceções (`NaoImplementado`, `BoletoInvalido`, `RemessaInvalida`, `ValorInvalido`) e o `Brcobranca.configuration` (`gerador`, `formato`, `resolucao`, `external_encoding`).

### Sem ActiveSupport / ActiveModel

Reimplementações próprias que se parecem com Rails mas não são:

- `Brcobranca::Validations` (`lib/brcobranca/validations.rb`) — reimplementa `validates_presence_of`, `validates_length_of`, `validates_numericality_of`, `validates_inclusion_of`, `validates_format_of`, `validates_each`, `with_options`, além de `valid?`/`invalid?`/`errors.full_messages`. Suporta só o que está implementado ali; ao precisar de uma validação nova, estenda esse módulo.
- `lib/brcobranca/util/date.rb` — define `Date.current` / `Time.current` (usa `Time.zone` se existir). Use sempre `Date.current`, nunca `Date.today`, para que o Timecop e o fuso do host funcionem nos specs.
- **Monkey patches em `String`/`Integer`/`Date`** — é por isso que `'341'.modulo11` e `valor.somente_numeros` funcionam:
  - `Calculo`: `modulo10`, `modulo11`, `duplo_digito`, `soma_digitos`, `multiplicador`
  - `Formatacao`: `somente_numeros`, `linha_digitavel`, `to_br_cpf/cnpj/cep`, `formata_documento`
  - `FormatacaoString`: `format_size(n)` (ljust/truncate), `truncate`, `remove_accents`
  - `CalculoData`: `fator_vencimento`, `to_s_br`, `to_juliano`

### Boleto

`Boleto::Base` concentra atributos, validações e a montagem genérica do código de barras: `codigo_barras` = `codigo_barras_primeira_parte` (18 díg.) + `codigo_barras_segunda_parte` (25 díg.) + DV módulo 11 inserido na 5ª posição = 44 dígitos.

Cada banco herda de `Base` e tipicamente sobrescreve:

- `banco` (código de 3 dígitos), `nosso_numero_dv`, `nosso_numero_boleto`, `agencia_conta_boleto`
- `codigo_barras_segunda_parte` — **o único método realmente obrigatório** além de `banco`
- setters (`agencia=`, `conta_corrente=`, `nosso_numero=`, `carteira=`) que fazem `rjust(n, '0')`, garantindo o tamanho fixo
- validações de tamanho específicas da carteira


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kivanio/brcobranca](https://github.com/kivanio/brcobranca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
