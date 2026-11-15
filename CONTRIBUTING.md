# Como contribuir

## Antes de começar

1. Leia o [README.md](README.md) para entender a ideia geral e a arquitetura.
2. Leia o [docs/PROJETO.md](docs/PROJETO.md) para os detalhes: decisões, estrutura do notebook, cronograma e divisão de tarefas.
3. Confira na [especificação](docs/especificacoes-projeto-2.pdf) o que o professor pede.

## Como o trabalho está organizado

Todo o código fica em notebooks do **Google Colab**, em **R**. São dois tipos de notebook:

| Notebook | Para que serve | Quem edita |
| --- | --- | --- |
| `notebooks/analise.ipynb` | O notebook-relatório que vai ser entregue | **Uma pessoa por vez** |
| `notebooks/rascunhos/<fase>.ipynb` | Onde cada pessoa desenvolve e testa a sua fase (ex.: `eda.ipynb`, `modelos.ipynb`) | O responsável pela fase |

Por que separar: notebooks são arquivos difíceis de juntar no git. Se duas pessoas editam o mesmo notebook ao mesmo tempo, uma das versões se perde. Com rascunhos separados, cada um trabalha sem atrapalhar o outro, e o notebook principal só recebe o que já está pronto.

### Fluxo de uma fase

1. **Desenvolva no seu rascunho.** Crie `notebooks/rascunhos/<fase>.ipynb` e trabalhe nele até a fase estar pronta (veja o critério de "pronto" no [cronograma](docs/PROJETO.md#9-fases-e-cronograma)).
2. **Avise no grupo** que vai mexer no `analise.ipynb`. Ninguém mais edita enquanto isso.
3. **Copie as células** do rascunho para a seção certa do `analise.ipynb` (a ordem das seções está em [docs/PROJETO.md](docs/PROJETO.md#8-estrutura-do-notebook)).
4. **Rode tudo** (*Ambiente de execução → Executar tudo*) e confira se o notebook funciona do início ao fim.
5. **Salve no GitHub** (veja abaixo) e avise no grupo que liberou o notebook.

## Salvando no GitHub

### Pelo Colab (notebooks)

1. Crie uma branch para a tarefa no GitHub, a partir da `development` (botão de branches na página do repositório).
2. No Colab: **Arquivo → Salvar uma cópia no GitHub**.
3. Escolha o repositório `acervo-rank`, a **sua branch** e o caminho do arquivo (ex.: `notebooks/analise.ipynb`).
4. Escreva a mensagem de commit e marque **Incluir um link para o Colab**.
5. No GitHub, abra um Pull Request da sua branch para a `development`.

### Pelo terminal (documentação e outros arquivos)

```bash
git checkout development
git pull
git checkout -b tipo/descricao-curta
# ... edite os arquivos ...
git add <arquivos>
git commit -m "Mensagem"
git push -u origin tipo/descricao-curta
```

Depois, abra um Pull Request para a `development` no GitHub.

### Nomes de branch

| Tipo | Quando usar | Exemplo |
| --- | --- | --- |
| `feat/` | Parte nova da análise | `feat/eda` |
| `fix/` | Correção de erro | `fix/contagem-consultas` |
| `docs/` | Só documentação | `docs/readme-dados` |

### Mensagens de commit

- Cada commit faz **uma** coisa.
- Mensagem em português, no imperativo, curta e clara:

  ```
  Adiciona leitura do CISI.QRY
  Corrige contagem de consultas com gabarito
  ```

### Pull Requests

- No título, diga o que o PR faz.
- Na descrição, explique o que mudou e por quê.
- Peça revisão de **pelo menos uma pessoa** da equipe.
- Só faça o merge depois da aprovação.

Ao revisar, confira se:

- o notebook roda do início ao fim sem erro;
- cada bloco de código tem uma célula de texto explicando o raciocínio;
- os arquivos do CISI em `data/raw/` não foram alterados.

Comentários de revisão são sobre o trabalho, não sobre a pessoa.

## Regras do notebook

- **Texto antes do código.** O notebook é um relatório: cada bloco de código vem depois de uma célula de texto (Markdown) dizendo o que vai ser feito e por quê, e os resultados são comentados logo depois.
- **Siga a estrutura obrigatória.** As seções e subseções da especificação (1.1, 1.2, ..., 6.3) têm que aparecer com esses números.
- **Funções com comentário.** Toda função começa com um comentário curto, em português, dizendo o que entra e o que sai.
- **Pré-processamento único.** Artigos e consultas passam pela mesma função, para os dois modelos.
- **Pacotes.** Todos os pacotes são instalados e carregados na primeira célula. Pacote novo só entra com combinação da equipe.
- **Dados.** Leia os arquivos do CISI pelo link do GitHub (`https://raw.githubusercontent.com/philipsantiagoo/acervo-rank/main/data/raw/...`), para o notebook funcionar em qualquer máquina.
- **Não edite os dados.** Os arquivos em `data/raw/` são a cópia original da coleção. Qualquer limpeza é feita no notebook.
- **Decisões novas** (mudar uma métrica, um parâmetro, um teste) são combinadas no grupo e registradas no [docs/PROJETO.md](docs/PROJETO.md) antes de virar código.

## Resultados

Os gráficos e tabelas são exportados pelo notebook e salvos em `results/`:

| Onde | O que vai |
| --- | --- |
| `results/tf-idf/` | Gráficos e tabelas só do TF-IDF |
| `results/bm25/` | Gráficos e tabelas só do BM25 |
| `results/comparison/` | Os dois modelos lado a lado: onde cada um foi melhor, diferenças por consulta e teste estatístico |
| `results/conclusao.md` | Resumo da resposta final: houve melhora significativa? A troca vale a pena? |

- Gráficos e tabelas **só são gerados pelo notebook**, nunca editados à mão.
- O `conclusao.md` resume a seção 6 do notebook e é escrito pela equipe.

### Como o notebook salva os arquivos

Toda vez que o notebook gera um gráfico ou tabela que vai para `results/`, ele salva com um caminho relativo à pasta `notebooks/`, criando a pasta se ela não existir:

```r
dir.create("../results/bm25", recursive = TRUE, showWarnings = FALSE)
ggsave("../results/bm25/map_por_consulta.png", grafico, width = 8, height = 5)
```

No Colab, esses arquivos são salvos na máquina do Google e descartados quando a sessão termina. Isso é esperado: lá eles só servem para conferir. Os arquivos oficiais saem da rodada final.

### Rodada final: gerando `results/`

Feita **uma vez**, por uma pessoa, quando o notebook estiver completo (ver o [cronograma](docs/PROJETO.md#9-fases-e-cronograma)).

1. Instale no computador: R, Jupyter e o kernel de R para o Jupyter:

   ```r
   install.packages("IRkernel")
   IRkernel::installspec()
   ```

2. Atualize o repositório e crie a branch:

   ```bash
   git checkout development
   git pull
   git checkout -b feat/resultados
   ```

3. Rode o notebook do início ao fim, a partir da pasta `notebooks/`:

   ```bash
   cd notebooks
   jupyter nbconvert --to notebook --execute --inplace analise.ipynb
   ```

4. Confira se os gráficos e tabelas apareceram em `results/tf-idf/`, `results/bm25/` e `results/comparison/`.
5. Escreva o `results/conclusao.md` com base na seção 6 do notebook.
6. Faça o commit de `results/` e do `analise.ipynb` (que agora tem as saídas atualizadas) e abra um PR para a `development`.

Se o notebook mudar depois disso, a rodada final precisa ser repetida.

## O que nunca vai para o git

- Checkpoints do Jupyter (`.ipynb_checkpoints/`).
- Arquivos temporários do R (`.Rhistory`, `.RData`, `.Rproj.user/`).
- Arquivos do sistema e de editores (`.DS_Store`, `.vscode/`, `.idea/`).

Antes de cada commit pelo terminal, confira com `git status` o que está sendo enviado.

## Dúvidas

Pergunte no grupo da equipe.
