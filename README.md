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
