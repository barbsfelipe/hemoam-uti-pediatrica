# Design: Site unificado de ferramentas da UTI Pediátrica HEMOAM

**Data:** 2026-08-13
**Status:** Aprovado para planejamento

## Contexto

Hoje existem três ferramentas HTML standalone criadas para a UTI Pediátrica do
HEMOAM, publicadas em dois repositórios GitHub separados (dois sites
GitHub Pages distintos):

- **ISBAR** — `barbsfelipe/sbar-hemoam` → `site/index.html`
- **Checklist de Visita Multiprofissional** — `barbsfelipe/sbar-hemoam` →
  `site/checklist-visita-multi.html`
- **Admissão UTI Pediátrica** — `barbsfelipe/admissao-uti-pediatrica-hemoam` →
  `Admissão e evolução/index.html`

Cada ferramenta é um HTML autocontido (CSS/JS inline, sem dependências
externas), com sua própria lógica de armazenamento local, impressão e estado.
Não há hoje nenhum ponto de entrada único: o usuário precisa saber/guardar
duas URLs diferentes.

## Objetivo

Criar um **site novo e independente** que funcione como painel único de
entrada, com ícones/cartões que levam diretamente a cada ferramenta, mantendo:

- As URLs e sites atuais **inalterados e no ar** (não é uma migração — é uma
  cópia).
- Facilidade de adicionar novas ferramentas no futuro (ex.: ferramentas de
  Fisioterapia já cogitadas).

## Fora de escopo

- Sincronizar edições futuras entre a cópia nova e os repositórios antigos —
  a partir da cópia, cada ferramenta passa a evoluir de forma independente
  em cada lugar onde existir. Isso é uma escolha consciente do usuário.
- Autenticação/controle de acesso — site público, sem dados de pacientes
  armazenados no servidor (as ferramentas já usam localStorage no
  navegador do usuário).
- Unificar os repositórios antigos ou apagá-los.

## Arquitetura

Novo repositório Git independente, `hemoam-uti-pediatrica`, publicado via
GitHub Pages a partir da branch `main`:

```
hemoam-uti-pediatrica/
├── index.html              # Dashboard (painel de entrada)
├── assets/
│   └── logo-hemoam.png     # Logo do Hospital do Sangue (extraída do PDF)
├── isbar/
│   └── index.html          # Cópia de site/index.html
├── visita-multi/
│   └── index.html          # Cópia de site/checklist-visita-multi.html
└── admissao/
    └── index.html          # Cópia de "Admissão e evolução/index.html"
```

Cada ferramenta em sua própria subpasta gera uma URL limpa
(`.../isbar/`, `.../visita-multi/`, `.../admissao/`). A navegação do
dashboard para cada ferramenta é feita por link `<a href>` comum — não por
iframe — para não interferir com o JS, localStorage e CSS de impressão de
cada ferramenta, que continuam funcionando exatamente como hoje.

URL final: `https://barbsfelipe.github.io/hemoam-uti-pediatrica/`.

## Dashboard (`index.html`)

**Identidade visual:** reaproveita o azul-marinho já usado no ISBAR
(`#1f3864`), próximo ao azul institucional do site oficial da HEMOAM
(`#1d274b`) — mantém consistência com as ferramentas existentes sem copiar
o site institucional pixel a pixel.

**Cabeçalho:** barra azul-marinho de largura total, com a logo do Hospital
do Sangue (extraída de `LOGO DO HOSPITAL DO SANGUE.pdf`, fundo transparente)
à esquerda e o título "UTI Pediátrica · HEMOAM".

**Corpo:** fundo branco, grade de cartões responsiva (1 coluna em telas
estreitas/celular, 2–3 colunas em tablet/desktop — uso previsto à beira-leito
em dispositivos variados). Um cartão por ferramenta, seguindo o padrão
"ícone em círculo colorido + título em negrito + descrição de uma linha"
observado no site oficial da HEMOAM:

| Ferramenta | Título do cartão | Descrição |
|---|---|---|
| ISBAR | ISBAR | Comunicação estruturada de passagem de plantão |
| Visita Multi | Visita Multiprofissional | Checklist da visita multi por leito |
| Admissão | Admissão | Ferramenta de admissão e evolução |

Cartão inteiro clicável, com leve elevação/sombra no hover. Cada cartão é um
bloco HTML autocontido e repetido (sem geração via JSON/JS) — adicionar uma
ferramenta nova no futuro é: duplicar um bloco de cartão + criar a subpasta
correspondente. Prioriza simplicidade de edição manual sobre abstração.

**Rodapé:** discreto — "Hospital do Sangue · HEMOAM — Ferramentas internas
UTI Pediátrica".

## Ferramentas individuais

Cada `index.html` copiado recebe uma pequena barra no topo, fora da área
de impressão (usa a convenção `.no-print` já existente no ISBAR/Checklist;
para a Admissão, que não tem essa classe hoje, a regra de impressão é
adicionada localmente só para esse elemento), com o link:

> ← Painel HEMOAM UTI Pediátrica

Estilizada no mesmo azul-marinho do dashboard. Nenhuma outra alteração é
feita no HTML/CSS/JS original de cada ferramenta — cópia fiel além dessa
barra de navegação.

## Publicação

1. Extrair a logo do PDF (`LOGO DO HOSPITAL DO SANGUE.pdf`, página 1) como
   PNG de fundo transparente.
2. Copiar os três HTMLs para as subpastas correspondentes, ajustando apenas
   a barra de retorno.
3. Criar `index.html` do dashboard.
4. Commitar tudo no novo repositório local.
5. Criar o repositório `hemoam-uti-pediatrica` no GitHub (conta
   `barbsfelipe`, público) via `gh` CLI, dar push e ativar GitHub Pages
   (branch `main`, pasta raiz).

## Verificação

- Abrir o dashboard localmente e conferir que os três cartões levam à
  ferramenta correta.
- Em cada ferramenta copiada, conferir que o link de volta funciona e some
  na visualização de impressão.
- Testar o grid do dashboard em largura de celular e desktop.
- Após o deploy, conferir a URL pública do GitHub Pages carregando as três
  ferramentas.
