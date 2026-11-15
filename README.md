# acervo-rank

Comparação estatística entre dois modelos de busca, TF-IDF e BM25, para decidir qual ordena melhor os artigos de um acervo.

> Projeto 2 de Estatística e Probabilidade para Computação (CIn-UFPE, 2026.2). </br>
> **Equipe:** Arthur Dias, Eduardo Henrique, Felipe Augusto, Gabriel André e Philip Santiago.

> **Status:** em construção. A documentação e os dados estão prontos; o notebook ainda não foi escrito.

## Ideia geral

### O cenário: a biblioteca

Uma biblioteca universitária quer melhorar a busca do seu acervo de artigos. Hoje, o sistema ordena os resultados com **TF-IDF**, um modelo que mede a semelhança entre a consulta e cada documento pelas palavras que eles têm em comum. A proposta é trocar pelo **BM25**, um modelo probabilístico usado em motores de busca como Lucene e Elasticsearch.

Trocar de sistema tem custo. Por isso, a biblioteca só quer fazer a troca se houver **evidência estatística** de que os usuários seriam mais bem atendidos.

### Pergunta central

> Com o BM25, os pesquisadores encontram os artigos certos mais perto do topo da lista do que com o TF-IDF, e essa melhora é estatisticamente significativa?

### Como vamos responder

1. Exploramos os dados (EDA) para entender a coleção e testar se o tamanho dos artigos influencia a relevância.
2. Rodamos os dois modelos sobre as mesmas consultas.
3. Para cada consulta, medimos o quanto cada modelo coloca os artigos certos perto do topo.
4. Comparamos os dois modelos consulta a consulta, com um teste estatístico, para saber se a diferença é real ou pode ser acaso.

As hipóteses, os testes e todas as decisões estão em [docs/PROJETO.md](docs/PROJETO.md).

## A base de dados: CISI

Um acervo real não tem gabarito, ou seja, não sabemos quais artigos respondem cada pergunta. Por isso, a coleção **CISI** faz o papel do acervo da biblioteca. Ela traz resumos reais de artigos de ciência da informação, pedidos de busca reais feitos por pesquisadores e o gabarito de quais artigos são relevantes para cada pergunta.

| Item | Valor |
| --- | --- |
| Documentos (artigos) | 1.460 |
| Consultas (perguntas) | 112 |
| Consultas que têm gabarito | 76 |
| Tipo de relevância | Binária: o artigo é relevante ou não é |

São três arquivos de texto:

| Arquivo | O que contém |
| --- | --- |
| `CISI.ALL` | Os artigos: identificador, título, autores, resumo e referências cruzadas |
| `CISI.QRY` | As consultas: identificador e texto da pergunta |
| `CISI.REL` | O gabarito: cada linha liga uma consulta a um artigo relevante para ela |

Os três arquivos já vêm no repositório, em `data/raw/`. Detalhes sobre cada um estão em [data/README.md](data/README.md).

## Arquitetura

O projeto é feito em **R**, num **notebook do Google Colab**, como pede a [especificação](docs/especificacoes-projeto-2.pdf). Todo o código fica dentro do notebook, que funciona como relatório: cada etapa tem texto explicando o raciocínio e código executando a análise.

Este repositório é a base do notebook: guarda os dados, a documentação, a versão oficial do notebook e os gráficos exportados. O notebook lê os dados direto daqui.

### Estrutura do repositório

```
acervo-rank/
├── data/
│   ├── raw/             # os três arquivos do CISI
│   └── README.md        # explicação da base de dados
├── docs/
│   ├── PROJETO.md                    # documento do projeto: hipóteses, decisões e cronograma
│   ├── especificacoes-projeto-2.pdf  # o que o professor pede
│   └── apresentacao-projeto-1.pdf    # o plano apresentado no Projeto 1
├── notebooks/
│   ├── analise.ipynb    # o notebook-relatório (entregável)
│   └── rascunhos/       # notebooks de trabalho de cada fase
├── results/
│   ├── tf-idf/          # gráficos e tabelas só do TF-IDF
│   ├── bm25/            # gráficos e tabelas só do BM25
│   ├── comparison/      # os dois lado a lado: onde cada um foi melhor e teste estatístico
│   └── conclusao.md     # resumo da resposta final: a troca vale a pena?
├── CONTRIBUTING.md      # como contribuir com o projeto
├── LICENSE              # licença MIT
└── README.md            # este arquivo
```

### Fluxo de dados

Os dados passam por sete etapas, sempre nesta ordem. Nas setas está o que passa de uma etapa para a outra:

```
    CISI.ALL               CISI.QRY               CISI.REL
    1.460 artigos          112 consultas          gabarito
        │                      │                      │
        └──────────────────────┼──────────────────────┘
                               ▼
 ┌───────────────────────────────────────────────────────────┐
 │ 1. LEITURA E LIMPEZA                                      │
 │    organiza os três arquivos e mantém só as 76 consultas  │
 │    que têm gabarito                                       │
 └─────────────────────────────┬─────────────────────────────┘
                               │ artigos, consultas e gabarito
                               ▼
 ┌───────────────────────────────────────────────────────────┐
 │ 2. ANÁLISE EXPLORATÓRIA (EDA)                             │
 │    tamanhos, relevantes por consulta, tipos de consulta   │
 │    teste: artigos longos são mais vezes relevantes?       │
 └─────────────┬─────────────────────────────────┬───────────┘
               │ artigos e consultas             │ gabarito
               ▼                                 │
 ┌─────────────────────────────────┐             │
 │ 3. PRÉ-PROCESSAMENTO            │             │
 │    o mesmo tratamento de texto  │             │
 │    para os dois modelos         │             │
 └───────┬─────────────────┬───────┘             │
         │ palavras        │ palavras            │
         ▼                 ▼                     │
 ┌───────────────┐ ┌───────────────┐             │
 │ 4a. TF-IDF    │ │ 4b. BM25      │             │
 │ ordena os     │ │ ordena os     │             │
 │ artigos       │ │ artigos       │             │
 └───────┬───────┘ └───────┬───────┘             │
         │ ranking         │ ranking             │
         ▼                 ▼                     ▼
 ┌───────────────────────────────────────────────────────────┐
 │ 5. AVALIAÇÃO                                              │
 │    compara cada ranking com o gabarito, consulta a        │
 │    consulta (P@10, MAP, nDCG@10)                          │
 └─────────────────────────────┬─────────────────────────────┘
                               │ notas dos dois modelos em cada consulta
                               ▼
 ┌───────────────────────────────────────────────────────────┐
 │ 6. TESTE PRINCIPAL                                        │
 │    t pareado: o BM25 é melhor ou foi acaso?               │
 └─────────────────────────────┬─────────────────────────────┘
                               │ diferença média, valor-p
                               ▼
 ┌───────────────────────────────────────────────────────────┐
 │ 7. RESULTADOS E CONCLUSÃO                                 │
 │    tabelas, gráficos e a recomendação para a biblioteca   │
 └──────┬──────────────┬──────────────┬──────────────┬───────┘
        ▼              ▼              ▼              ▼
    results/       results/       results/       results/
    tf-idf/        bm25/          comparison/    conclusao.md
```

| Etapa | Seção do notebook | O que entra | O que faz | O que sai |
| --- | --- | --- | --- | --- |
| 1. Leitura e limpeza | 1 e 2.1 | Os três arquivos do CISI | Lê os arquivos, trata inconsistências e separa as consultas com gabarito | Artigos, consultas e gabarito em tabelas do R |
| 2. EDA | 2.2 a 3 | Artigos, consultas e gabarito | Descreve a coleção com gráficos e testa se o tamanho do artigo influencia a relevância | Gráficos comentados, resultado do teste da EDA |
| 3. Pré-processamento | 4.1 | Texto dos artigos e das consultas | Minúsculas, separação em palavras, remoção de stopwords e stemming | Lista de palavras de cada artigo e de cada consulta |
| 4. Ranqueamento | 4.2 | Palavras dos artigos e das consultas | Cada modelo dá uma nota a cada artigo para cada consulta e ordena do mais para o menos relevante | Uma lista ordenada de artigos por consulta, para cada modelo |
| 5. Avaliação | 5.1 | Listas ordenadas e gabarito | Mede, consulta a consulta, o quanto os artigos certos aparecem perto do topo | Notas de cada modelo em cada consulta |
| 6. Teste principal | 5.1 e 5.2 | Notas por consulta dos dois modelos | Verifica se o BM25 é significativamente melhor | Diferença média, valor-p e interpretação |
| 7. Resultados e conclusão | 5 e 6 | Notas e resultado dos testes | Monta tabelas e gráficos e responde à pergunta central | Seção 6 do notebook e arquivos em `results/` |

Um cuidado importante: **o pré-processamento tem que ser o mesmo para os dois modelos**. Se cada modelo tratasse o texto de um jeito, não daria para saber se a diferença veio do modelo ou do tratamento.

### Pacotes R

Os testes estatísticos (`t.test`, `wilcox.test`, `shapiro.test`) já vêm no R. Os modelos e as métricas são implementados no próprio notebook. Os pacotes extras previstos são:

| Pacote | Para que serve |
| --- | --- |
| `SnowballC` | Stemming de Porter |
| `stopwords` | Lista de stopwords em inglês |
| `ggplot2` | Gráficos |
| `dplyr` e `tidyr` | Manipulação de tabelas |

Os pacotes que não vêm instalados no Colab são instalados na primeira célula do notebook.

## Como usar o repositório

### Abrir e rodar o notebook no Colab

1. Abra o notebook direto pelo link:
   https://colab.research.google.com/github/philipsantiagoo/acervo-rank/blob/main/notebooks/analise.ipynb
2. Confira se o ambiente é R: **Ambiente de execução → Alterar o tipo de ambiente de execução → R**.
3. Rode tudo: **Ambiente de execução → Executar tudo**.

Não é preciso baixar nada: o notebook lê os dados do CISI direto deste repositório no GitHub.

### Salvar alterações no GitHub

No Colab: **Arquivo → Salvar uma cópia no GitHub**, escolha este repositório, a branch e o caminho do notebook, e escreva uma mensagem de commit. O passo a passo da equipe está no [CONTRIBUTING.md](CONTRIBUTING.md).

### Rodar localmente (rodada final)

O desenvolvimento é todo no Colab. Mas os gráficos e tabelas de `results/` só são gravados no repositório quando o notebook roda no próprio computador. Por isso, no fim do projeto, uma pessoa faz uma **rodada final local**: instala o R, o Jupyter e o kernel de R para o Jupyter (`IRkernel`), roda o notebook e faz o commit de `results/`. O passo a passo está no [CONTRIBUTING.md](CONTRIBUTING.md#rodada-final-gerando-results).

## Como contribuir

Veja o [CONTRIBUTING.md](CONTRIBUTING.md).

## Mais detalhes

Contexto, hipóteses, decisões, métodos, estrutura do notebook e cronograma estão em [docs/PROJETO.md](docs/PROJETO.md).

## Licença

[MIT](LICENSE). A licença vale para o código deste repositório, não para os dados do CISI.
