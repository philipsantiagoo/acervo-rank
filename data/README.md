# Dados: a coleção CISI

Esta pasta guarda os dados do projeto.

## O que é o CISI

O **CISI** é uma coleção de teste clássica da área de Recuperação de Informação, distribuída pela Universidade de Glasgow. Ela tem três partes:

- **artigos** de ciência da informação (título, autores e resumo);
- **consultas**: a maioria são perguntas escritas por pesquisadores, e as demais são resumos de artigos usados como consulta (detalhes em [`CISI.QRY`](#cisiqry-as-consultas));
- um **gabarito**, que diz quais artigos respondem cada consulta.

| Item | Valor |
| --- | --- |
| Artigos | 1.460 |
| Consultas | 112 |
| Consultas com gabarito | 76 |
| Pares (consulta, artigo relevante) | 3.114 |
| Artigos relevantes por consulta | de 1 a 155 |
| Tipo de relevância | Binária: o artigo é relevante ou não é |
| Idioma | Inglês |

## Por que o CISI

**Precisamos de um gabarito.** Para dizer qual modelo ordena melhor, precisamos saber quais artigos são as respostas certas de cada pergunta. Um acervo real de biblioteca não tem essa informação. O CISI tem, e os julgamentos foram feitos por pessoas.

**Combina com o cenário.** O projeto simula a busca de uma biblioteca universitária. O CISI é feito de resumos reais de artigos acadêmicos e de pedidos de busca reais de pesquisadores, ou seja, o mesmo tipo de conteúdo e de usuário do cenário. O tema dos artigos (ciência da informação, bibliotecas, indexação) também é próximo.

**É pequeno.** Com 1.460 artigos, dá para ordenar a coleção inteira para cada consulta e rodar tudo num computador comum, em pouco tempo.

**É conhecido.** Por ser uma coleção clássica, tem formato estável e é fácil de encontrar e de comparar com outros trabalhos.

**Limitações:** é uma coleção pequena e antiga, em inglês e de um domínio só. Os resultados valem para esse cenário, não para qualquer acervo.

## Por que três arquivos

Avaliar um sistema de busca exige três coisas, e cada arquivo traz uma:

1. **o que buscar:** os artigos (`CISI.ALL`);
2. **o que perguntar:** as consultas (`CISI.QRY`);
3. **qual é a resposta certa:** o gabarito (`CISI.REL`).

Sem qualquer um deles a avaliação não fecha. Sem os artigos, não há o que ordenar. Sem as consultas, não há o que buscar. Sem o gabarito, não há como dizer se a ordem ficou boa.

## Os arquivos

Os três são de texto puro. Em `CISI.ALL` e `CISI.QRY`, cada campo começa com um **marcador** no início da linha (`.I`, `.T` etc.), e o conteúdo do campo pode ocupar várias linhas, até o próximo marcador.

> **Atenção:** os três arquivos usam quebra de linha no padrão do Windows (`\r\n`). Quem for escrever a leitura precisa tratar o `\r` no fim das linhas.

### `CISI.ALL`: os artigos

Cada artigo começa com `.I` seguido do seu identificador.

| Marcador | Campo | Observação |
| --- | --- | --- |
| `.I` | Identificador | Número de 1 a 1.460, na mesma linha do marcador |
| `.T` | Título | Pode ter mais de uma linha |
| `.A` | Autores | Um artigo pode ter vários `.A`, um por autor |
| `.W` | Resumo | O texto principal do artigo |
| `.X` | Referências cruzadas | Ligações com outros artigos (três números por linha) |
| `.B` | Ano de publicação | Só em 24 artigos |
| `.K` | Palavras-chave | Só no artigo 321 |
| `.C` | Códigos de classificação | Só no artigo 321 |

**Cuidados na leitura:**

- Os marcadores `.B`, `.K` e `.C` são raros e não aparecem na documentação original da coleção. A leitura precisa encerrar o resumo (`.W`) em **qualquer** marcador, e não só nos conhecidos; senão, as palavras-chave e os códigos do artigo 321 entram no resumo.
- Em 15 linhas, o marcador vem seguido de espaço (`.T ` em vez de `.T`). Compare o marcador sem os espaços do fim da linha.
- O arquivo tem 43 linhas em branco no meio dos textos.

Exemplo (resumido):

```
.I 1
.T
18 Editions of the Dewey Decimal Classifications
.A
Comaromi, J.P.
.W
   The present study is a history of the DEWEY Decimal
Classification.  The first edition of the DDC was published
in 1876, ...
.X
1	5	1
92	1	1
...
```

No projeto, o texto de cada artigo é **título + resumo**, porque são os campos com o conteúdo do artigo. Autores e referências cruzadas não dizem do que o artigo trata (decisão registrada no [PROJETO.md](../docs/PROJETO.md)).

### `CISI.QRY`: as consultas

Cada consulta começa com `.I` seguido do seu identificador (de 1 a 112).

| Marcador | Campo | Observação |
| --- | --- | --- |
| `.I` | Identificador | Na mesma linha do marcador |
| `.W` | Texto da consulta | Presente em todas as 112 |
| `.T` | Título | Só em 55 consultas |
| `.A` | Autores | Só em 55 consultas |
| `.B` | Fonte bibliográfica | Só em 55 consultas, por exemplo `(JASIS, Vol. 31, No. 6, November 1980, pp. 438-444)` |

Existem dois tipos de consulta no arquivo:

**Perguntas (57 consultas, 50 com gabarito).** Só têm `.I` e `.W`, e o `.W` é uma pergunta escrita por um pesquisador:

```
.I 3
.W
What is information science?  Give definitions where possible.
```

**Artigos usados como consulta (55 consultas, 26 com gabarito).** Têm também `.T`, `.A` e `.B`. Aqui o `.W` não é uma pergunta: é o resumo de um artigo, e `.T`, `.A` e `.B` são o título, os autores e a fonte desse artigo. A ideia é "encontre artigos parecidos com este":

```
.I 58
.T
Directions in Library Networking
.A
Avram, H.D.
McCallum, S.H.
.W
    Bibliographic control before and after MARC is reviewed.  The capability
of keying into online systems brought an interdependence among libraries, ...
.B
(JASIS, Vol. 31, No. 6, November 1980, pp. 438-444)
```

Os dois tipos entram nas 76 consultas com gabarito (50 + 26). No projeto, usamos só o campo `.W` de todas as consultas, para tratar os dois tipos da mesma forma (decisão registrada no [PROJETO.md](../docs/PROJETO.md)).

### `CISI.REL`: o gabarito

Uma linha por par (consulta, artigo relevante), com quatro colunas separadas por espaços:

```
     1     28	0	0.000000
     1     35	0	0.000000
     2     29	0	0.000000
```

| Coluna | Significado |
| --- | --- |
| 1ª | Identificador da consulta |
| 2ª | Identificador do artigo relevante |
| 3ª e 4ª | Não são usadas (são sempre `0` e `0.000000`) |

Só aparecem pares relevantes: se um artigo não está listado para uma consulta, ele não é relevante para ela.

O gabarito cobre só **76 das 112 consultas**. As outras 36 não têm nenhum artigo marcado como relevante, então não dá para avaliar os modelos com elas.

## Onde estão os arquivos

Os três arquivos já vêm no repositório, em `data/raw/`:

```
data/
└── raw/
    ├── CISI.ALL
    ├── CISI.QRY
    └── CISI.REL
```

**Não edite esses arquivos.** Eles são a cópia original da coleção. Qualquer limpeza ou correção é feita pelo código.

## Origem

O CISI é distribuído publicamente pela Universidade de Glasgow, junto com outras coleções de teste clássicas de Recuperação de Informação:

- **Universidade de Glasgow (original):** https://ir.dcs.gla.ac.uk/resources/test_collections/cisi/ (arquivo `cisi.tar.gz`; lá, o arquivo de consultas se chama `cisi.que`)
- **Kaggle (cópia):** https://www.kaggle.com/datasets/dmaso01dsta/cisi-a-dataset-for-information-retrieval

Se precisar baixar de novo, use uma dessas fontes e salve com os nomes acima.

## Conferência rápida

Para conferir se os arquivos estão íntegros, rode na raiz do repositório:

```bash
grep -c '^\.I ' data/raw/CISI.ALL                  # deve dar 1460
grep -c '^\.I ' data/raw/CISI.QRY                  # deve dar 112
awk '{print $1}' data/raw/CISI.REL | sort -u | wc -l   # deve dar 76
```

## Por que os dados estão no repositório

O CISI é público e pequeno (cerca de 2,4 MB). Com os dados versionados junto do código, qualquer pessoa clona o repositório e roda a análise sem precisar baixar nada. Isso também garante que todos usam exatamente a mesma cópia.

A licença MIT do repositório vale só para o código. Os dados são da coleção CISI, distribuída pela Universidade de Glasgow.
