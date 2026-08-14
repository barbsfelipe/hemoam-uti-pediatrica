# HEMOAM — UTI Pediátrica

Painel único de ferramentas internas da UTI Pediátrica do HEMOAM:

- [ISBAR](https://barbsfelipe.github.io/hemoam-uti-pediatrica/isbar/) — comunicação estruturada de passagem de plantão
- [Visita Multiprofissional](https://barbsfelipe.github.io/hemoam-uti-pediatrica/visita-multi/) — checklist da visita multi por leito
- [Admissão](https://barbsfelipe.github.io/hemoam-uti-pediatrica/admissao/) — ferramenta de admissão e evolução

Cada ferramenta é uma cópia independente e autocontida (sem backend, sem
dependências externas). Editar uma ferramenta aqui não afeta os
repositórios originais (`sbar-hemoam` e `admissao-uti-pediatrica-hemoam`),
e vice-versa.

## Nota sobre armazenamento local

Os rascunhos de cada ferramenta são salvos no navegador (localStorage/
IndexedDB), não em um servidor. Como todas as páginas do GitHub Pages da
conta `barbsfelipe` compartilham a mesma origem do navegador, um rascunho
salvo na versão original de uma ferramenta (ex: `sbar-hemoam`) também
aparece na cópia deste painel, e vice-versa — dentro do mesmo navegador,
nada trafega pela rede, então isso não é um vazamento de dados. Ainda
assim, se as duas versões (original e cópia) divergirem no futuro em
formato de dados, isso pode causar sobrescrita silenciosa de rascunhos;
vale revisitar (renomear as chaves de armazenamento) se isso vier a
acontecer.

## Adicionar uma nova ferramenta

1. Criar uma subpasta na raiz com um `index.html` autocontido.
2. Duplicar um bloco `<a class="card">...</a>` em `index.html` (o
   dashboard) apontando para a nova subpasta.
3. Adicionar ao `<head>` do novo `index.html` uma regra `.hub-back` (fundo
   `#1f3864`, ver os outros `index.html` desta pasta como referência) e ao
   `<body>` uma barra `<div class="hub-back no-print"><a href="../">← Painel
   HEMOAM UTI Pediátrica</a></div>`. IMPORTANTE: acrescente a regra
   `.hub-back` dentro do bloco `<style>` já existente — nunca abra um
   segundo `<style>`, pois o navegador trata o texto literal como
   conteúdo CSS dentro do bloco ainda aberto e a regra é descartada.
