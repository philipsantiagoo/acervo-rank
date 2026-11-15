# acervo-rank: documento do projeto

> Projeto 2 de Estatística e Probabilidade para Computação (CIn-UFPE, 2026.2). </br>
> **Equipe:** Arthur Dias, Eduardo Henrique, Felipe Augusto, Gabriel André e Philip Santiago.

Este documento é o norte do projeto: o que vamos fazer, como, e quais decisões já foram tomadas. Quem for trabalhar no notebook deve ler antes.

**Documentos de referência (nesta pasta):**

- [especificacoes-projeto-2.pdf](especificacoes-projeto-2.pdf): o que o professor pede (estrutura obrigatória, prazos e avaliação).
- [apresentacao-projeto-1.pdf](apresentacao-projeto-1.pdf): o plano que apresentamos no Projeto 1. O Projeto 2 tem que segui-lo; desvios são permitidos, desde que justificados no notebook.

---

## 1. Entregas e prazos

| Entrega | Prazo |
| --- | --- |
| Link do notebook (Google Classroom) | **22/11, 23h59** |
| Link dos slides (Google Classroom) | **22/11, 23h59** |
| Avisar no Discord o dia da apresentação | Antes das apresentações |
| Apresentação em sala | 23/11, 25/11 ou 30/11 |

- **Linguagem:** R.
- **Ambiente:** Jupyter Notebook, Google Colab ou Kaggle.
- **Avaliação:** apresentação 4, organização e estrutura 3, relatório em Markdown 3.

## 2. Contexto

Uma biblioteca universitária quer melhorar a busca do seu acervo de artigos. Hoje, o sistema ordena os resultados com **TF-IDF**. A proposta é trocar pelo **BM25**, modelo probabilístico padrão em motores como Lucene e Elasticsearch.

Trocar de sistema tem custo, então a biblioteca só quer fazer a troca se houver **evidência estatística** de que os usuários seriam mais bem atendidos.

Como um acervo real não tem gabarito de relevância, a coleção **CISI** faz o papel do acervo (detalhes em [data/README.md](../data/README.md)).

## 3. Pergunta central

> Com o BM25, os pesquisadores encontram os artigos certos mais perto do topo da lista do que com o TF-IDF, e essa melhora é estatisticamente significativa?

## 4. Hipóteses

O projeto tem **dois testes de hipóteses**, porque a especificação exige um teste baseado na EDA (seção 3 do notebook) antes da modelagem.

### 4.1. Teste da EDA: o tamanho do artigo influencia a relevância?

**Ideia:** se artigos mais longos aparecem mais vezes como relevantes, o tamanho é um fator que o modelo de busca precisa tratar. Isso motiva o BM25, que normaliza pelo tamanho do documento (parâmetro b).

- **Grupos:** artigos relevantes para pelo menos uma consulta × artigos que nunca são relevantes.
- **Variável:** tamanho do artigo (número de palavras de título + resumo).
- **H0:** o tamanho médio é igual nos dois grupos.
- **H1:** o tamanho médio é diferente.
- **Teste:** t de Welch, bilateral, α = 0,05. Se a EDA mostrar distribuições muito assimétricas, usar Mann-Whitney e justificar no notebook.

### 4.2. Teste principal: o BM25 é melhor que o TF-IDF?

Para cada métrica (P@10, MAP e nDCG@10), seja *d* a diferença por consulta (BM25 − TF-IDF) nas 76 consultas com gabarito:

- **H0:** μ_d ≤ 0 (o BM25 não é melhor).
- **H1:** μ_d > 0 (o BM25 é melhor).
- **Teste:** t pareado, **unilateral**, α = 0,05.

**Por que pareado:** os dois modelos são avaliados nas mesmas consultas, e a dificuldade de cada consulta varia mais do que a diferença entre os modelos. Comparando consulta a consulta, essa dificuldade se cancela.

**Por que unilateral:** a hipótese do Projeto 1 é direcional ("P@10, MAP e nDCG maiores" com o BM25), e a biblioteca só troca de sistema se o BM25 for melhor.

**Suposição a validar:** o teste t pareado supõe que as diferenças têm distribuição aproximadamente normal. Verificar com histograma, QQ-plot e Shapiro-Wilk. Se a suposição falhar, rodar também o teste de Wilcoxon pareado e comparar.

## 5. Decisões

| Decisão | Valor | Motivo | Origem |
| --- | --- | --- | --- |
| Dataset | CISI | Tem gabarito de relevância e simula o acervo de uma biblioteca | Projeto 1 |
| Modelos | TF-IDF (sistema atual) × BM25 | Pergunta central | Projeto 1 |
| Pré-processamento | Minúsculas, tokenização, remoção de stopwords em inglês e stemming de Porter, **iguais para os dois modelos** | Se fosse diferente, não daria para saber se a diferença veio do modelo ou do tratamento | Projeto 1 |
| Métricas | P@10, MAP e nDCG@10 | Medem se os relevantes estão no topo; o corte em 10 combina com o P@10 | Projeto 1 (corte @10: fase 0) |
| Teste principal | t pareado, unilateral, α = 0,05 | Ver seção 4.2 | Projeto 1 (unilateral: fase 0) |
| Texto do artigo | Título + resumo | São os campos que dizem do que o artigo trata; autores e referências cruzadas ficam de fora | Fase 0 |
| Texto da consulta | Só o campo `.W`, para todas as consultas | Trata os dois tipos de consulta da mesma forma (ver seção 6) | Fase 0 |
| Consultas avaliadas | As 76 com gabarito | Sem gabarito não há como medir | Fase 0 |
| Parâmetros do BM25 | k1 = 1,2; b = 0,75 | Valores padrão da literatura; ajustar nas mesmas consultas da avaliação daria vantagem injusta ao BM25 | Fase 0 |
| TF-IDF | Peso tf × idf, com idf = log10(N / df), e similaridade do cosseno entre consulta e documento | Modelo vetorial clássico. O exemplo do slide 8 do Projeto 1 soma os pesos sem normalizar; o cosseno corrige o efeito do tamanho do vetor | Fase 0 |
| Tamanho do ranking | Todos os 1.460 artigos | O MAP precisa do ranking completo | Fase 0 |
| Teste da EDA | Tamanho do artigo × relevância | Ver seção 4.1 | Fase 0 |
| Extensão com embeddings | Fora do escopo; citada em "Próximos passos" | Prazo de uma semana e pouco suporte em R | Fase 0 |

## 6. Desvios em relação ao Projeto 1

A especificação permite pequenos desvios, desde que justificados no notebook.

1. **Tipos de consulta.** O Projeto 1 descreve as consultas como "escritas por pesquisadores". Na leitura dos dados, vimos que 55 das 112 consultas são, na verdade, **resumos de artigos usados como consulta** (26 delas têm gabarito). Usamos o campo `.W` em todas e mostramos essa diferença na EDA.
2. **Teste da EDA.** O Projeto 1 previa só o teste t pareado. A especificação do Projeto 2 exige um teste baseado na EDA antes da modelagem, então incluímos o teste da seção 4.1.

## 7. Métodos

### 7.1. Pré-processamento

A mesma função para artigos e consultas:

1. converter para minúsculas;
2. separar em palavras (tokenização), removendo pontuação e números;
3. remover stopwords em inglês;
4. aplicar o stemming de Porter (pacote `SnowballC`).

### 7.2. Modelos

**TF-IDF:** cada artigo e cada consulta vira um vetor de pesos tf × idf. A nota de um artigo para uma consulta é o cosseno entre os dois vetores.

**BM25:** para uma consulta *q* e um artigo *D*:

```
score(D, q) = Σ  idf(t) · f(t, D) · (k1 + 1) / ( f(t, D) + k1 · (1 − b + b · |D| / avgdl) )
             t∈q
```

- f(t, D): quantas vezes o termo *t* aparece em *D*;
- |D|: tamanho de *D*; avgdl: tamanho médio dos artigos;
- k1 = 1,2 (saturação); b = 0,75 (normalização pelo tamanho).

Os dois modelos são implementados em R, no próprio notebook.

### 7.3. Métricas

Calculadas por consulta, com relevância binária:

- **P@10:** fração de relevantes entre os 10 primeiros.
- **AP (precisão média):** média das precisões medidas em cada posição onde aparece um relevante; relevantes nunca recuperados contam como 0. O **MAP** é a média das APs.
- **nDCG@10:** ganho 1 para relevante e 0 para não relevante, descontado por log2(1 + posição), dividido pelo valor da ordem ideal.

**Conferência manual (obrigatória antes de usar as métricas):** para a lista [relevante, irrelevante, relevante, irrelevante, irrelevante, relevante], com 3 relevantes no total, a AP deve dar (1 + 2/3 + 3/6) / 3 ≈ 0,722.

## 8. Estrutura do notebook

O notebook segue a estrutura obrigatória da especificação:

| Seção | Conteúdo no nosso projeto |
| --- | --- |
| **1. Introdução** | |
| 1.1. Contextualização | Cenário da biblioteca (seção 2) |
| 1.2. Objetivo | Pergunta central (seção 3) |
| 1.3. Descrição dos dados | CISI, fonte e dicionário dos três arquivos |
| **2. EDA** | |
| 2.1. Carregamento e limpeza | Leitura dos arquivos, `\r` no fim das linhas, consultas sem gabarito, tipos de consulta, duplicados |
| 2.2. Análise univariada | Tamanho dos artigos e das consultas, relevantes por consulta, palavras mais frequentes |
| 2.3. Análise bivariada | Tamanho do artigo × relevância; tamanho da consulta × número de relevantes; tipo de consulta × número de relevantes |
| 2.4. Insights | Resumo do que a EDA mostrou e como isso conecta com o BM25 |
| **3. Teste de hipóteses** | Teste da EDA (seção 4.1) |
| **4. Pré-processamento e modelagem** | |
| 4.1. Preparação | Pré-processamento (seção 7.1) |
| 4.2. Modelagem | TF-IDF e BM25, com a justificativa dos parâmetros (seção 7.2) |
| **5. Resultados** | |
| 5.1. Apresentação | Métricas por modelo, teste t pareado (seção 4.2), gráficos de comparação |
| 5.2. Análise crítica | Em quais consultas o BM25 ganhou ou perdeu e por quê; tradução dos números para a biblioteca |
| **6. Conclusão** | |
| 6.1. Resumo | Resposta direta à pergunta central |
| 6.2. Recomendações | Trocar ou não o sistema, com 1 ou 2 ações concretas |
| 6.3. Limitações e próximos passos | Seção 11 deste documento; embeddings como próximo passo |

## 9. Fases e cronograma

| Fase | Período | O que fazer | Pronto quando |
| --- | --- | --- | --- |
| 0. Organização | 15–16/11 | Fechar decisões, preparar ambiente, dividir tarefas | Este documento aprovado pela equipe |
| 1. Leitura e limpeza | 16/11 | Seções 1 e 2.1 do notebook | Contagens batem: 1.460 artigos, 112 consultas, 76 com gabarito, 3.114 pares |
| 2. EDA e teste da EDA | 16–17/11 | Seções 2.2 a 3 | Gráficos comentados e teste interpretado |
| 3. Modelos e métricas | 17–19/11 | Seção 4 e métricas | Rankings completos para as 76 consultas; conferência manual da AP bate |
| 4. Teste principal | 19–20/11 | Seção 5 | Suposições verificadas e teste interpretado |
| 5. Conclusão | 20/11 | Seção 6 do notebook | Notebook responde à pergunta central |
| 6. Rodada final local | 20/11 | Rodar o notebook localmente para gerar `results/` e escrever o `conclusao.md` (ver [CONTRIBUTING](../CONTRIBUTING.md#rodada-final-gerando-results)) | Gráficos e tabelas em `results/` no GitHub |
| 7. Slides | 21/11 | Montar a apresentação com os gráficos de `results/` | Slides prontos |
| 8. Revisão e entrega | 22/11 | Revisar o texto do notebook e dos slides, enviar os links | Links enviados até 23h59 |

## 10. Divisão de tarefas

| Fase | Responsável |
| --- | --- |
| 1. Leitura e limpeza | |
| 2. EDA e teste da EDA | |
| 3. Modelos e métricas | |
| 4. Teste principal | |
| 5. Conclusão e slides | |
| 6. Revisão e entrega | Todos |

## 11. Limitações conhecidas

- O CISI é pequeno, antigo, em inglês e de um domínio só; os resultados valem para esse cenário, não para qualquer acervo.
- Os julgamentos de relevância são binários, então o nDCG perde parte da sua vantagem (graus de relevância).
- O cenário da biblioteca é simulado: o CISI faz o papel do acervo porque tem gabarito.
- São três testes (um por métrica) sobre as mesmas consultas, o que aumenta a chance de um falso positivo. A conclusão deve olhar as três métricas em conjunto.

## 12. Referências

- MANNING, C. D.; RAGHAVAN, P.; SCHÜTZE, H. *Introduction to Information Retrieval*. Cambridge University Press, 2008.
- ROBERTSON, S. E. The probability ranking principle in IR. *Journal of Documentation*, v. 33, n. 4, p. 294–304, 1977.
- ROBERTSON, S.; ZARAGOZA, H. The Probabilistic Relevance Framework: BM25 and Beyond. *Foundations and Trends in Information Retrieval*, v. 3, n. 4, p. 333–389, 2009.
- SMUCKER, M. D.; ALLAN, J.; CARTERETTE, B. A Comparison of Statistical Significance Tests for Information Retrieval Evaluation. *CIKM*, 2007.
- UNIVERSITY OF GLASGOW. CISI test collection. Disponível em: https://ir.dcs.gla.ac.uk/resources/test_collections/cisi/
