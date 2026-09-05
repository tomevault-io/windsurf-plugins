---
trigger: always_on
description: Contexto para quem (ou qual assistente) for mexer neste repositório depois.
---

# CLAUDE.md

Contexto para quem (ou qual assistente) for mexer neste repositório depois.

## O que é

Site do chá de bebê da Julieta. Funciona como site de casamento: o convidado
escolhe um presente da lista e recebe um **Pix copia e cola** já com o valor
preenchido, para colar no app do banco. Depois de copiar, pode deixar um
recado — que vai para uma planilha privada, nunca para o site.

Feito para a Ana (mãe). A conta que recebe é do André (pai), no C6.
Vai haver mais de um chá, em cidades diferentes: **o site não tem data nem
local**, de propósito.

## Decisões de arquitetura

- **Site 100% estático.** HTML, CSS e JS puros, sem framework, sem build, sem
  dependência instalada. Hospedagem gratuita para sempre, nada para manter no ar.
- **O Pix é gerado no navegador do convidado.** `assets/pix.js` monta o BR Code
  (EMV®QRCPS do Banco Central) e calcula o CRC16-CCITT-FALSE.
- **O formato foi calibrado contra um Pix real do C6.** A primeira versão foi
  recusada pelo banco; o gabarito revelou duas diferenças: faltava o campo
  `01 = 11` (QR reutilizável) e a cidade cadastrada é SÃO PAULO, não Campinas.
  O teste `tests/testar_pix.js` compara campo a campo com esse código real —
  **se mexer no gerador, rode esse teste antes de qualquer coisa.**
- **O identificador da transação é `***`.** Um txid inventado é o único campo
  que pode fazer um banco recusar o código, e não é necessário: como cada
  presente tem um **valor único**, o valor já identifica o presente no extrato.
- **Cada presente aceita um `codigo` fixo** no catálogo. Se preenchido, o site
  usa esse código em vez de montar o dele — escotilha de emergência caso algum
  banco implique com o código gerado.
- **Recados vão para o Google Sheets** via Apps Script (`apps-script/Codigo.gs`).
  O envio usa `fetch` com `mode: 'no-cors'` e `Content-Type: text/plain`, que
  evita o preflight CORS que o Apps Script não responde. Como a resposta é
  opaca, **o site não sabe se deu certo** — por isso o Apps Script manda um
  e-mail de confirmação opcional.
- **Dez presentes.** A lista começou com vinte e foi enxugada: lista longa
  cansa quem escolhe. As pinturas dos dez que saíram continuam em `arte/`.
- **Presentes podem se repetir.** Sem controle de estoque, decisão da Ana. É o
  que mantém o site sem estado e sem backend.
- **Nada de localStorage nem cookie.**

## Estrutura

```
index.html               markup da página inteira, inclusive o modal
assets/styles.css        paleta e layout
assets/pix.js            gerador do BR Code (também roda no Node, nos testes)
assets/app.js            CONFIG, CATALOGO, montagem da página, modal, recados
assets/ilustracao.jpg    a aquarela do save the date
assets/bichos/*.png      bichos recortados da aquarela (fundo transparente)
assets/presentes/*.png   os 10 ícones de presente, pintados por código
arte/icones.py           20 pinturas (as 10 em uso + 10 de reserva)
arte/base.py             primitivas de apoio (corpo, sombra, linha, arred)
apps-script/Codigo.gs    web app que grava os recados na planilha
tests/                   três suítes, descritas abaixo
```

**Ponto de entrada para mudança de conteúdo: `CONFIG` e `CATALOGO`, no topo do
`assets/app.js`.**

## Identidade visual

Vem inteira da aquarela do save the date. Nada foi baixado da internet.

**Os bichos** (`assets/bichos/`) são recortes da própria ilustração. Os que
estão sobre papel branco viram recorte limpo (beija-flor e joaninha); os que estão sobre folhagem viram medalhão de borda esfumada
(tucano, maritaca, sagui). Uso: maritaca e sagui encabeçam
as duas seções; joaninha é o ornamento das divisas; tucano aparece no modal
depois de copiar; beija-flor fecha a página.

**Os ícones dos presentes** foram pintados por código com a skill `paint`
(`/mnt/skills/examples/paint`), em aquarela, na mesma paleta. Não são fotos:
não havia como baixar imagens, e banco de imagem traria problema de licença.
Se um dia quiser fotos de verdade, é só trocar os arquivos em
`assets/presentes/` mantendo os nomes.

Cores amostradas da ilustração:

| Token | Hex | De onde veio |
|---|---|---|
| `--papel` | `#FFFFFF` | fundo da aquarela |
| `--creme` | `#FBF4E6` | papel da ilustração |
| `--halo` | `#F7E6C9` | a aguada pêssego atrás do moisés |
| `--granada` | `#852E27` | as flores vinho |
| `--musgo` | `#425833` | as samambaias |
| `--casca` | `#7A5A42` | os troncos |

Tipos: **Gilda Display** (títulos, valores) e **Alegreya Sans** (texto), com
reservas do sistema. Cantos assimétricos (`--curva: 2px 14px 2px 14px`) para
lembrar borda de papel molhado.

Elemento assinatura: a **florada** — ao copiar o código, uma aguada pêssego se
espalha pela caixa, como tinta em papel úmido (`@keyframes florada`). É o único
efeito exuberante; o resto é deliberadamente quieto. Respeita
`prefers-reduced-motion`.

## Repintar um ícone

```bash
cd arte && CANVAS_WORKERS=0 python3 icones.py    # renderiza em arte/saida/
```

Depois, exportar para o site (recorte na caixa do desenho, quadrado, 300px,
paleta de 110 cores — fica em ~7 KB cada):

```python
from PIL import Image; import numpy as np, os
for f in sorted(os.listdir('saida')):
    arr = np.array(Image.open(f'saida/{f}').convert('RGB')).astype(float)
    alpha = np.clip((252 - arr.min(axis=2)) / 22.0, 0, 1)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andred-br/Cha-juju](https://github.com/andred-br/Cha-juju) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
