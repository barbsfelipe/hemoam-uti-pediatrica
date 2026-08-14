# Site Unificado UTI Pediátrica HEMOAM Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Criar um novo site estático (`hemoam-uti-pediatrica`) que reúne cópias do ISBAR, do Checklist de Visita Multiprofissional e da ferramenta de Admissão atrás de um dashboard único, publicado via GitHub Pages, sem alterar os dois repositórios/sites originais.

**Architecture:** Repositório Git único com um `index.html` (dashboard) na raiz e uma subpasta por ferramenta (`isbar/`, `visita-multi/`, `admissao/`), cada uma contendo uma cópia fiel do HTML original mais uma pequena barra de navegação de volta ao dashboard. Navegação por link `<a href>` comum, sem iframe. Publicado como GitHub Pages a partir da branch `main`.

**Tech Stack:** HTML/CSS/JS estático puro (sem build step), Python 3 + Pillow (só para o pré-processamento único da logo), `gh` CLI para criar o repositório remoto e ativar o Pages.

## Global Constraints

- Os repositórios originais (`barbsfelipe/sbar-hemoam` e `barbsfelipe/admissao-uti-pediatrica-hemoam`) não são modificados — este é um repositório novo e independente, criado a partir de cópias.
- Navegação entre dashboard e ferramentas é feita por `<a href>` normal — nunca por `<iframe>`.
- Fora da barra "← Painel HEMOAM UTI Pediátrica" inserida no topo, nenhum outro trecho do HTML/CSS/JS original de cada ferramenta é alterado — cópia fiel.
- A barra de retorno usa a classe `no-print` (reaproveitando a convenção já existente no ISBAR/Checklist; adicionada explicitamente via `@media print` na Admissão, que não tinha essa regra) para sumir na impressão.
- Paleta: `#1f3864` (mesmo azul-marinho já usado como `--header-bg` no ISBAR/Checklist) para cabeçalho do dashboard, cartões e barra de retorno.
- Cartões do dashboard são blocos HTML estáticos e repetidos — não gerados via JSON/JS — para que adicionar uma ferramenta nova no futuro seja "duplicar um bloco + criar uma subpasta".
- Logo (`assets/logo-hemoam.png`) precisa ter fundo transparente, recortada rente ao conteúdo (sem margens em branco).

---

## File Structure

```
hemoam-uti-pediatrica/
├── docs/superpowers/specs/2026-08-13-dashboard-unificado-design.md   (já existe)
├── docs/superpowers/plans/2026-08-13-dashboard-unificado.md          (este arquivo)
├── assets/
│   └── logo-hemoam.png       # Task 1 — logo extraída do PDF, fundo transparente
├── isbar/
│   └── index.html            # Task 2 — cópia de ../site/index.html + barra de retorno
├── visita-multi/
│   └── index.html            # Task 3 — cópia de ../site/checklist-visita-multi.html + barra de retorno
├── admissao/
│   └── index.html            # Task 4 — cópia de "../Admissão e evolução/index.html" + barra de retorno
├── index.html                 # Task 5 — dashboard com os 3 cartões
└── README.md                  # Task 6 — descrição curta do repositório
```

Caminhos de origem (fora deste repositório, pastas irmãs de `hemoam-uti-pediatrica/` dentro de `/Users/felipebarbosa/Desktop/Claude/Hemoam/`):

- `../site/index.html` (ISBAR)
- `../site/checklist-visita-multi.html` (Checklist)
- `../Admissão e evolução/index.html` (Admissão)
- `../LOGO DO HOSPITAL DO SANGUE.pdf` (logo, página 1)

Todos os comandos abaixo assumem `cwd = /Users/felipebarbosa/Desktop/Claude/Hemoam/hemoam-uti-pediatrica`.

---

### Task 1: Extrair a logo do PDF como PNG transparente

**Files:**
- Create: `assets/logo-hemoam.png`

**Interfaces:**
- Produces: `assets/logo-hemoam.png` — PNG RGBA, fundo transparente, usado pelo `<header>` do dashboard na Task 5.

- [ ] **Step 1: Verificar que o arquivo ainda não existe (red)**

Run: `test -f assets/logo-hemoam.png && echo EXISTS || echo MISSING`
Expected: `MISSING`

- [ ] **Step 2: Extrair e converter a logo**

Run:
```bash
set -e
RAWDIR=$(mktemp -d)
qlmanage -t -s 1600 -o "$RAWDIR" "/Users/felipebarbosa/Desktop/Claude/Hemoam/LOGO DO HOSPITAL DO SANGUE.pdf"
mkdir -p assets
python3 -c "
from PIL import Image
src = '$RAWDIR/LOGO DO HOSPITAL DO SANGUE.pdf.png'
img = Image.open(src).convert('RGBA')
newData = []
for r, g, b, a in img.getdata():
    if r > 240 and g > 240 and b > 240:
        newData.append((r, g, b, 0))
    else:
        newData.append((r, g, b, a))
img.putdata(newData)
cropped = img.crop(img.getbbox())
cropped.save('assets/logo-hemoam.png')
print('saved', cropped.size)
"
rm -rf "$RAWDIR"
```
Expected: última linha `saved (721, 234)`

- [ ] **Step 3: Verificar transparência e existência (green)**

Run:
```bash
python3 -c "
from PIL import Image
img = Image.open('assets/logo-hemoam.png')
print(img.size, img.mode, img.getpixel((0,0)))
"
```
Expected: `(721, 234) RGBA (255, 255, 255, 0)` (canto transparente, alpha 0)

- [ ] **Step 4: Commit**

```bash
git add assets/logo-hemoam.png
git commit -m "Adiciona logo do Hospital do Sangue (PNG transparente)

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>"
```

---

### Task 2: Copiar o ISBAR para `isbar/index.html` com barra de retorno

**Files:**
- Create: `isbar/index.html` (cópia de `../site/index.html`)

**Interfaces:**
- Consumes: nenhuma dependência de tasks anteriores.
- Produces: `isbar/index.html`, acessível como `/isbar/`. O cartão "ISBAR" do dashboard (Task 5) linka para `isbar/`.

- [ ] **Step 1: Copiar o arquivo original**

Run:
```bash
mkdir -p isbar
cp "../site/index.html" "isbar/index.html"
```

- [ ] **Step 2: Verificar que a barra de retorno ainda não existe (red)**

Run: `grep -q "hub-back" isbar/index.html && echo FOUND || echo MISSING`
Expected: `MISSING`

- [ ] **Step 3: Inserir CSS e barra de retorno**

Em `isbar/index.html`, substituir:

```
</style>
</head>
<body>

<div class="toolbar no-print">
```

por:

```
<style>
  .hub-back { background: var(--header-bg); padding: 6px 16px; font-family: Arial, Helvetica, sans-serif; }
  .hub-back a { color: #fff; text-decoration: none; font-size: 13px; font-weight: bold; }
  .hub-back a:hover { text-decoration: underline; }
</style>
</head>
<body>

<div class="hub-back no-print">
  <a href="../">← Painel HEMOAM UTI Pediátrica</a>
</div>

<div class="toolbar no-print">
```

- [ ] **Step 4: Corrigir o link cruzado para o Checklist**

Em `isbar/index.html`, substituir:

```
    <a class="tool-link" href="checklist-visita-multi.html">Checklist de Visita Multiprofissional →</a>
```

por:

```
    <a class="tool-link" href="../visita-multi/">Checklist de Visita Multiprofissional →</a>
```

- [ ] **Step 5: Verificar tudo (green)**

Run:
```bash
grep -q 'hub-back no-print' isbar/index.html && \
grep -q 'Painel HEMOAM UTI Pediátrica' isbar/index.html && \
grep -q 'href="../visita-multi/"' isbar/index.html && \
! grep -q 'href="checklist-visita-multi.html"' isbar/index.html && \
grep -q '<title>ISBAR UTI Pediátrica - HEMOAM</title>' isbar/index.html && \
echo PASS || echo FAIL
```
Expected: `PASS`

- [ ] **Step 6: Commit**

```bash
git add isbar/index.html
git commit -m "Copia ISBAR para isbar/ com barra de retorno ao painel

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>"
```

---

### Task 3: Copiar o Checklist de Visita Multi para `visita-multi/index.html`

**Files:**
- Create: `visita-multi/index.html` (cópia de `../site/checklist-visita-multi.html`)

**Interfaces:**
- Consumes: nenhuma dependência de tasks anteriores.
- Produces: `visita-multi/index.html`, acessível como `/visita-multi/`. O cartão "Visita Multiprofissional" do dashboard (Task 5) linka para `visita-multi/`.

- [ ] **Step 1: Copiar o arquivo original**

Run:
```bash
mkdir -p visita-multi
cp "../site/checklist-visita-multi.html" "visita-multi/index.html"
```

- [ ] **Step 2: Verificar que a barra de retorno ainda não existe (red)**

Run: `grep -q "hub-back" visita-multi/index.html && echo FOUND || echo MISSING`
Expected: `MISSING`

- [ ] **Step 3: Inserir CSS e barra de retorno**

Em `visita-multi/index.html`, substituir:

```
</style>
</head>
<body>

<div class="toolbar no-print">
```

por:

```
  .hub-back { background: var(--header-bg); padding: 6px 16px; font-family: Arial, Helvetica, sans-serif; }
  .hub-back a { color: #fff; text-decoration: none; font-size: 13px; font-weight: bold; }
  .hub-back a:hover { text-decoration: underline; }
</style>
</head>
<body>

<div class="hub-back no-print">
  <a href="../">← Painel HEMOAM UTI Pediátrica</a>
</div>

<div class="toolbar no-print">
```

**Nota (corrigida durante execução da Task 2):** o padrão original deste passo abria um novo bloco `<style>` no lugar do `</style>` original, o que faz o navegador tratar o texto `<style>` literal como conteúdo CSS dentro do bloco ainda aberto (HTML trata `<style>...</style>` como raw text) — a regra `.hub-back` era descartada pelo parser CSS por seletor inválido, deixando a barra sem fundo navy. A substituição acima já vem corrigida: acrescenta as regras `.hub-back` dentro do MESMO bloco `<style>` existente, fechando com um único `</style>`.

- [ ] **Step 4: Corrigir o link cruzado para o ISBAR**

Em `visita-multi/index.html`, substituir:

```
    <a class="tool-link" href="index.html">ISBAR (passagem de plantão) →</a>
```

por:

```
    <a class="tool-link" href="../isbar/">ISBAR (passagem de plantão) →</a>
```

- [ ] **Step 5: Verificar tudo (green)**

Run:
```bash
grep -q 'hub-back no-print' visita-multi/index.html && \
grep -q 'Painel HEMOAM UTI Pediátrica' visita-multi/index.html && \
grep -q 'href="../isbar/"' visita-multi/index.html && \
! grep -q 'href="index.html"' visita-multi/index.html && \
grep -q '<title>Checklist de Visita Multiprofissional - HEMOAM</title>' visita-multi/index.html && \
echo PASS || echo FAIL
```
Expected: `PASS`

- [ ] **Step 6: Commit**

```bash
git add visita-multi/index.html
git commit -m "Copia Checklist de Visita Multi para visita-multi/ com barra de retorno ao painel

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>"
```

---

### Task 4: Copiar a Admissão para `admissao/index.html`

**Files:**
- Create: `admissao/index.html` (cópia de `../Admissão e evolução/index.html`)

**Interfaces:**
- Consumes: nenhuma dependência de tasks anteriores.
- Produces: `admissao/index.html`, acessível como `/admissao/`. O cartão "Admissão" do dashboard (Task 5) linka para `admissao/`.

- [ ] **Step 1: Copiar o arquivo original**

Run:
```bash
mkdir -p admissao
cp "../Admissão e evolução/index.html" "admissao/index.html"
```

- [ ] **Step 2: Verificar que a barra de retorno ainda não existe (red)**

Run: `grep -q "hub-back" admissao/index.html && echo FOUND || echo MISSING`
Expected: `MISSING`

- [ ] **Step 3: Inserir CSS (com regra de impressão própria, pois este arquivo não tem `@media print`) e barra de retorno**

Em `admissao/index.html`, substituir:

```
  @media (max-width: 420px) { .grid-2 { grid-template-columns: 1fr; } }
</style>
</head>
<body>

<h1>Admissão UTI Pediátrica</h1>
```

por:

```
  @media (max-width: 420px) { .grid-2 { grid-template-columns: 1fr; } }
  .hub-back { background: #1f3864; padding: 6px 16px; font-family: Arial, Helvetica, sans-serif; }
  .hub-back a { color: #fff; text-decoration: none; font-size: 13px; font-weight: bold; }
  .hub-back a:hover { text-decoration: underline; }
  @media print { .no-print { display: none; } }
</style>
</head>
<body>

<div class="hub-back no-print">
  <a href="../">← Painel HEMOAM UTI Pediátrica</a>
</div>

<h1>Admissão UTI Pediátrica</h1>
```

- [ ] **Step 4: Verificar tudo (green)**

Run:
```bash
grep -q 'hub-back no-print' admissao/index.html && \
grep -q 'Painel HEMOAM UTI Pediátrica' admissao/index.html && \
grep -q '@media print { .no-print { display: none; } }' admissao/index.html && \
grep -q '<title>Admissão UTI Pediátrica — HEMOAM</title>' admissao/index.html && \
echo PASS || echo FAIL
```
Expected: `PASS`

- [ ] **Step 5: Commit**

```bash
git add admissao/index.html
git commit -m "Copia Admissão para admissao/ com barra de retorno ao painel

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>"
```

---

### Task 5: Criar o dashboard (`index.html`)

**Files:**
- Create: `index.html`

**Interfaces:**
- Consumes: `assets/logo-hemoam.png` (Task 1), `isbar/` (Task 2), `visita-multi/` (Task 3), `admissao/` (Task 4).
- Produces: página inicial do site, servida em `/`.

- [ ] **Step 1: Verificar que o dashboard ainda não existe (red)**

Run: `test -f index.html && echo EXISTS || echo MISSING`
Expected: `MISSING`

- [ ] **Step 2: Criar `index.html`**

Conteúdo completo do arquivo:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>UTI Pediátrica · HEMOAM</title>
<style>
  :root {
    --header-bg: #1f3864;
    --header-fg: #ffffff;
    --paper: #ffffff;
    --ink: #1a1a1a;
    --muted: #555;
    --card-border: #e7e9ec;
  }
  * { box-sizing: border-box; }
  html, body {
    margin: 0;
    padding: 0;
    background: var(--paper);
    font-family: Arial, Helvetica, sans-serif;
    color: var(--ink);
  }
  header {
    background: var(--header-bg);
    color: var(--header-fg);
    display: flex;
    align-items: center;
    gap: 16px;
    padding: 16px 24px;
  }
  header img {
    height: 48px;
    width: auto;
  }
  header h1 {
    font-size: 20px;
    margin: 0;
    font-weight: bold;
  }
  main {
    max-width: 960px;
    margin: 0 auto;
    padding: 32px 24px;
  }
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 20px;
  }
  .card {
    display: block;
    text-decoration: none;
    color: inherit;
    border: 1px solid var(--card-border);
    border-radius: 10px;
    padding: 20px;
    transition: box-shadow 0.15s, transform 0.15s;
  }
  .card:hover {
    box-shadow: 0 4px 14px rgba(0,0,0,0.12);
    transform: translateY(-2px);
  }
  .card .icon {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    background: var(--header-bg);
    color: #fff;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 22px;
    margin-bottom: 12px;
  }
  .card h2 {
    font-size: 17px;
    margin: 0 0 6px 0;
    color: var(--header-bg);
  }
  .card p {
    font-size: 13px;
    color: var(--muted);
    margin: 0;
  }
  footer {
    text-align: center;
    font-size: 12px;
    color: var(--muted);
    padding: 24px;
  }
</style>
</head>
<body>

<header>
  <img src="assets/logo-hemoam.png" alt="Hospital do Sangue - HEMOAM">
  <h1>UTI Pediátrica · HEMOAM</h1>
</header>

<main>
  <div class="grid">
    <a class="card" href="isbar/">
      <div class="icon">🩺</div>
      <h2>ISBAR</h2>
      <p>Comunicação estruturada de passagem de plantão</p>
    </a>
    <a class="card" href="visita-multi/">
      <div class="icon">👥</div>
      <h2>Visita Multiprofissional</h2>
      <p>Checklist da visita multiprofissional por leito</p>
    </a>
    <a class="card" href="admissao/">
      <div class="icon">📋</div>
      <h2>Admissão</h2>
      <p>Ferramenta de admissão e evolução</p>
    </a>
  </div>
</main>

<footer>
  Hospital do Sangue · HEMOAM — Ferramentas internas UTI Pediátrica
</footer>

</body>
</html>
```

- [ ] **Step 3: Verificar estrutura do dashboard (green)**

Run:
```bash
grep -q 'href="isbar/"' index.html && \
grep -q 'href="visita-multi/"' index.html && \
grep -q 'href="admissao/"' index.html && \
grep -q 'src="assets/logo-hemoam.png"' index.html && \
echo PASS || echo FAIL
```
Expected: `PASS`

- [ ] **Step 4: Teste de integração local (todas as páginas respondem)**

Run:
```bash
python3 -m http.server 8791 --directory . >/tmp/hemoam-httpserver.log 2>&1 &
SERVER_PID=$!
sleep 1
for path in "" "isbar/" "visita-multi/" "admissao/" "assets/logo-hemoam.png"; do
  code=$(curl -s -o /dev/null -w "%{http_code}" "http://localhost:8791/$path")
  echo "$path -> $code"
done
kill $SERVER_PID
```
Expected: as 5 linhas mostram `-> 200`

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Adiciona dashboard com cartões para ISBAR, Visita Multi e Admissão

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>"
```

---

### Task 6: README e publicação no GitHub Pages

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: repositório local completo (Tasks 1–5).
- Produces: repositório remoto `barbsfelipe/hemoam-uti-pediatrica` no GitHub, publicado em `https://barbsfelipe.github.io/hemoam-uti-pediatrica/`.

- [ ] **Step 1: Preflight — confirmar que o `gh` CLI está instalado e autenticado**

Run: `gh auth status`
Expected: saída contendo `Logged in to github.com account barbsfelipe`. Se não estiver autenticado, rodar `gh auth login` interativamente antes de continuar (ação do usuário — não prosseguir sem isso).

- [ ] **Step 2: Criar `README.md`**

Conteúdo completo do arquivo:

```markdown
# HEMOAM — UTI Pediátrica

Painel único de ferramentas internas da UTI Pediátrica do HEMOAM:

- [ISBAR](https://barbsfelipe.github.io/hemoam-uti-pediatrica/isbar/) — comunicação estruturada de passagem de plantão
- [Visita Multiprofissional](https://barbsfelipe.github.io/hemoam-uti-pediatrica/visita-multi/) — checklist da visita multi por leito
- [Admissão](https://barbsfelipe.github.io/hemoam-uti-pediatrica/admissao/) — ferramenta de admissão e evolução

Cada ferramenta é uma cópia independente e autocontida (sem backend, sem
dependências externas). Editar uma ferramenta aqui não afeta os
repositórios originais (`sbar-hemoam` e `admissao-uti-pediatrica-hemoam`),
e vice-versa.

## Adicionar uma nova ferramenta

1. Criar uma subpasta na raiz com um `index.html` autocontido.
2. Duplicar um bloco `<a class="card">...</a>` em `index.html` (o
   dashboard) apontando para a nova subpasta.
```

- [ ] **Step 3: Commit do README**

```bash
git add README.md
git commit -m "Adiciona README com visão geral do repositório

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>"
```

- [ ] **Step 4: Criar o repositório remoto e publicar**

Run:
```bash
gh repo create barbsfelipe/hemoam-uti-pediatrica --public --source=. --remote=origin --push
```
Expected: saída confirmando a criação do repositório e o push da branch `main` (`branch 'main' set up to track 'origin/main'`).

- [ ] **Step 5: Ativar o GitHub Pages (branch `main`, raiz)**

Run:
```bash
gh api repos/barbsfelipe/hemoam-uti-pediatrica/pages -X POST -f "source[branch]=main" -f "source[path]=/"
```
Expected: resposta JSON com `"status":"building"` (ou `"queued"`) e um `"html_url"` igual a `https://barbsfelipe.github.io/hemoam-uti-pediatrica/`.

- [ ] **Step 6: Verificar a publicação**

Run (o primeiro build do Pages pode levar 1–2 minutos; repetir a cada ~20s até sair `200`):
```bash
curl -s -o /dev/null -w "%{http_code}\n" https://barbsfelipe.github.io/hemoam-uti-pediatrica/
```
Expected: `200` (pode retornar `404` nos primeiros segundos enquanto o Pages ainda está fazendo o build — não é falha, apenas espere e tente de novo)
