# 5LTEP-L1: Ferramenta da Camada 1 (Contratos Estruturais) do 5L-TEP

[![Tests](https://github.com/lsp3cesarschool/5ltep-layer1/actions/workflows/tests.yml/badge.svg)](https://github.com/lsp3cesarschool/5ltep-layer1/actions/workflows/tests.yml) [![Camada 1](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2Flsp3cesarschool%2F5ltep-layer1%2Fmain%2Fdocs%2Fdata%2Fstatus.pt.json)](https://github.com/lsp3cesarschool/5ltep-layer1/actions/workflows/layer1.yml) [![Licença: MIT](https://img.shields.io/badge/Licen%C3%A7a-MIT-blue.svg)](LICENSE)

[English](README.md) · **Português**

**Verifica se um portal de dados abertos diz como seus arquivos deveriam ser, e se os arquivos são
assim.** Um levantamento de todos os conjuntos de dados de um portal CKAN numa escala de maturidade de
esquema, leitores para os dicionários de dados que o portal publica (inclusive em PDF, com um modelo de
IA local e revisão humana) e a validação de cada arquivo publicado contra o esquema declarado, com o
Table Schema do Frictionless.

| Recurso | O que você encontra |
|---|---|
| 📊 **Painel** | [lsp3cesarschool.github.io/5ltep-layer1](https://lsp3cesarschool.github.io/5ltep-layer1/?lang=pt): maturidade, achados de documentação, todos os conjuntos e arquivos |
| 🔀 **Deriva de esquema** | [![issues de deriva](https://img.shields.io/github/issues/lsp3cesarschool/5ltep-layer1/layer1?label=issues%20de%20deriva&color=0366d6)](https://github.com/lsp3cesarschool/5ltep-layer1/issues?q=is%3Aissue+label%3Alayer1): uma issue a cada vez que um arquivo, ou o esquema declarado para ele, muda de estrutura |
| 🧑‍⚖️ **Esquemas sugeridos** | [pull requests](https://github.com/lsp3cesarschool/5ltep-layer1/pulls?q=is%3Apr+schemas+suggested) com esquemas extraídos de dicionários em PDF, aguardando uma pessoa |
| 🧪 **Escolha do modelo** | [5ltep-layer1-modeltest](https://github.com/lsp3cesarschool/5ltep-layer1-modeltest): o benchmark que escolhe o LLM para os dicionários em PDF |
| 🔁 **Experimentos de controle** | [5ltep-layer1-aneel](https://github.com/lsp3cesarschool/5ltep-layer1-aneel) e [5ltep-layer1-recife](https://github.com/lsp3cesarschool/5ltep-layer1-recife): o mesmo código em outros dois portais |

> **Situação: demonstração de pesquisa.** Esta ferramenta faz parte de um projeto de mestrado e é
> mantida pelo autor. Não é um serviço oficial do IBAMA (nem da ANEEL, nem do Recife) e não pressupõe
> que algum órgão vá revisar seus resultados ou adotá-la. O fluxo completo, revisão humana incluída,
> funciona e está pronto para adoção; o autor não atua como revisor das saídas da própria ferramenta.

## Caso de uso em um parágrafo

Antes que alguém possa dizer que um valor de um conjunto de dados abertos está errado, alguém precisa
dizer como esse valor deveria ser: que colunas o arquivo tem, que tipo cada uma guarda, quantos
caracteres um código pode ter. Esse é o contrato do próprio publicador, definido antes de os dados
existirem, e é a primeira camada de confiança. Esta ferramenta faz duas perguntas a um portal inteiro.
**O portal publica um contrato verificável?** Alguns não publicam nenhum, alguns publicam um PDF que
só pessoas leem, alguns uma planilha que um programa lê. **Os arquivos o respeitam?** Um dicionário que
lista uma coluna que o arquivo não tem, uma data escrita num formato que ninguém declarou ou um código
maior que o tamanho declarado são encontrados automaticamente, toda semana, arquivo por arquivo. A
resposta ajuda uma equipe de dados a decidir onde o trabalho de documentação compensa.

## Termos-chave

| Termo | Significado aqui |
|---|---|
| **Dicionário de dados** | O que o publicador declara sobre um arquivo: campos, tipos, tamanhos e descrições, em qualquer formato (CSV, JSON, XLSX, PDF ou uma lista na descrição do recurso). |
| **Table Schema** | O padrão do [Frictionless Data](https://specs.frictionlessdata.io/table-schema/) para descrever uma tabela (nomes de campos, tipos, formatos, restrições), da mesma comunidade do CKAN. Todo esquema aqui é guardado nele. |
| **Esquema declarado** | O Table Schema construído a partir do que o portal declara. **Esquema observado**: o que o próprio arquivo mostra (cabeçalho, tipos inferidos, delimitador, codificação). |
| **Conformidade** | Um arquivo é conforme quando tem todos os campos declarados, não tem coluna não declarada e no máximo 1% das células verificadas viola o tipo ou as restrições declaradas. |
| **Nível de maturidade** | De 0 a 4: quão verificável é a estrutura que o portal declara (ver abaixo). |
| **Oráculo** | O próprio cabeçalho do arquivo publicado, usado para medir a qualidade da extração de um dicionário em PDF. |
| **Deriva de esquema** | Uma mudança na estrutura de um arquivo, ou no esquema declarado para ele, entre duas execuções. |

## Visão geral

A Camada 1 da Pirâmide de Engenharia da Confiança em Cinco Camadas (5L-TEP) é *verificação*, no
sentido de Boehm (1984) e da IEEE 1012: o produto está conforme a sua própria especificação? Ela fica
separada da Camada 2 (regras de domínio, escritas por especialistas *depois* de os dados existirem),
porque o dono e o momento de cada regra são diferentes: aqui a especificação é do publicador e é
anterior aos dados.

### Arquitetura

```
portal.json (um valor: a URL do portal)
        │
        ▼
 levantamento ── API CKAN: todos os conjuntos e recursos ── dicionários (CSV, JSON, XML, XLSX, PDF, descrição)
   │              tipos do DataStore, esquema anexado               │
   │                                                                ▼
   │                                          schemas/<conjunto>/<recurso>.declared.json
   ▼
 fila de trabalho (novo, alterado, esquema alterado, lido por um leitor mais antigo, rodízio mensal)
   │
   ▼
 validate ── lê cada arquivo em stream (nenhum dado bruto guardado) ── esquema observado + conformidade por campo
   │                                                                │
   │                                                                ▼
   │                                     deriva (arquivo ou esquema declarado mudou) → Issue no GitHub
   ▼
 dicionários em PDF: determinística → LLM local → pull request (o cabeçalho do arquivo é o oráculo)
   │
   ▼
 report ── results/layer1_summary.json (Camada 5) · achados de documentação · painel
```

## A escala de maturidade

Cada arquivo tabular (CSV, XLSX, XLS, ODS, Parquet, JSON, XML, ou um zip com qualquer um deles) recebe
um nível a partir do que o portal declara sobre ele:

| Nível | Critério (lido da API CKAN) |
|---|---|
| 0 | nenhum dicionário ou esquema associado ao arquivo |
| 1 | um dicionário legível só por pessoas (PDF, HTML ou uma lista de campos escrita na descrição), ou um legível por máquina que não pode ser processado (uma página HTML por trás do link, JSON malformado) |
| 2 | um dicionário legível por máquina (CSV, JSON, XML, XLSX) que a ferramenta lê |
| 3 | tipos expostos pela API numa forma padrão: um Table Schema anexado ao recurso, ou campos do DataStore com tipos reais (um DataStore com todos os campos `text` não declara nada) |
| 4 | nível 3, e o arquivo está conforme ao esquema declarado |

Um conjunto é tão verificável quanto o seu arquivo menos documentado. O nível descreve o **portal**:
ele não muda quando esta ferramenta extrai um esquema de um PDF. Essa extração torna a conformidade
verificável, o que é relatado separadamente.

## De onde vem o esquema declarado

Os leitores de dicionário são escolhidos pelo *conteúdo* do arquivo, não pelo portal, e por isso outro
portal não precisa de mudança no código quando seus dicionários se parecem com algum destes:

- uma tabela com uma linha por campo e um cabeçalho nomeando as colunas (`nome_atributo;datatype;descricao`,
  `Nome da variável | Tipo`...), em CSV ou XLSX (uma parte por aba);
- JSON com uma lista de objetos de campo (`metadados.campos[{codigo, tipo, tamanho}]`), que pode também
  nomear os recursos que descreve;
- XML com um elemento repetido por campo;
- uma lista de campos na descrição do recurso (`* SEQ_TAD – Chave...`, `- **UF**: Sigla. Formato: texto`);
- um PDF (próxima seção).

Cada dicionário é então ligado ao arquivo que descreve, pela evidência mais forte disponível, e o método
fica registrado: o dicionário nomeia o id do recurso; seus campos coincidem com o cabeçalho do arquivo;
seu nome corresponde ao nome do recurso; ou é o único dicionário do conjunto, com um nome genérico.
O cabeçalho de um arquivo nunca validado é lido no levantamento (a primeira linha; nada mais é guardado),
para que o arquivo seja ligado pelo cabeçalho, e validado contra o seu dicionário, já na primeira execução.

Os tipos declarados são escritos de muitos jeitos (`TEXTO (STRING)`, `Cadeia de caracteres`, `VARCHAR`,
`char`...). Eles são mapeados para tipos do Table Schema por palavras-chave, numa ordem fixa e auditável
([`src/types_map.py`](src/types_map.py)); um tipo não reconhecido vira `any` e é relatado, não adivinhado.
Um formato escrito no tipo (`DATA (DD/MM/AAAA)`) é mantido.

Quando um arquivo tem várias fontes, a validação usa a primeira de: Table Schema anexado, dicionário
legível por máquina, esquema extraído de PDF (depois de confirmado), lista na descrição, tipos do DataStore.

## Validação

Cada arquivo é lido em stream: CSV, JSON (uma lista de registros) e XML (elementos repetidos) direto da
resposta HTTP; zip, Parquet ou XLS a partir de um arquivo temporário, membro por membro, aba por aba ou
lote por lote. O formato é reconhecido pelos primeiros bytes, não pelo rótulo no portal. Um zip dentro
de um zip é aberto um nível abaixo. Nada é guardado além de contagens. Codificação e delimitador são
detectados nos formatos de texto; bytes que não decodificam são contados. Para cada arquivo:

- o **esquema observado** (cabeçalho, delimitador, codificação e o tipo mais estreito em que cabem as
  primeiras 5.000 linhas de cada coluna) é gravado em `schemas/<conjunto>/<recurso>.observed.json`;
- contra o esquema declarado: campos declarados ausentes do arquivo, colunas não declaradas, nomes que
  diferem só na grafia (`Nome/Razão Social` e `NOME_RAZAO_SOCIAL`), e cada célula das colunas
  correspondentes verificada com os leitores de célula do Frictionless (tipo, `maxLength`...).

Formatos que o dicionário não declara (uma data escrita `03/05/2022`, vírgula decimal) são tirados do
que a maioria dos valores da amostra segue e registrados como *inferidos*: a verificação é se os
valores são consistentes com o tipo declarado num formato, não se seguem a ISO 8601.

**Uma tabela, vários formatos.** Quando um conjunto publica a mesma tabela em vários formatos (o mesmo
nome com `CSV`, `XLSX`, `JSON`...), ela conta como uma tabela: a primeira na ordem CSV, ZIP, Parquet,
XLSX, ODS, XLS, JSON, XML é validada inteira, e as outras só têm as colunas comparadas (mesmas colunas,
ou quais faltam ou sobram). **Não é tabela:** um zip só com documentos ou mapas é listado à parte e fica
fora do universo tabular; um link que devolve uma página web ou um PDF no lugar da tabela é falha de
link. Formatos geográficos (shapefile, KML, GeoJSON, WMS/WFS) e tabelas em HTML ficam fora do escopo.

A primeira execução num portal valida todos os arquivos, em lotes que cabem no limite de tempo do
runner. As seguintes validam só o que é novo, alterado (URL, metadados do CKAN, ou ETag, Last-Modified
ou tamanho informados pelo servidor), validado contra um esquema que mudou, que falhou, ou que foi
verificado há mais de 28 dias.

## Dicionários em PDF: determinística, LLM, pessoas

Muitos portais publicam dicionários só em PDF. Eles viram esquemas em três etapas, a ordem que Al Hilmi
et al. (2026) acharam mais confiável para PDFs tabulares com modelos locais:

1. **Determinística:** o pdfplumber encontra as tabelas; cada linha é lida pelo *conteúdo* (um
   identificador é o nome, uma palavra de tipo é o tipo, um número é o tamanho), porque as células
   mudam de coluna entre as páginas.
2. **LLM local:** só onde a etapa 1 não achou nada ou discorda dos dados, um modelo executado com Ollama
   no runner lê o texto do PDF e devolve a lista de campos em JSON (temperatura 0, semente fixa).
   O modelo é o que o [benchmark de modelos da Camada 1](https://github.com/lsp3cesarschool/5ltep-layer1-modeltest)
   aprova para esta tarefa (`LLM_MODEL=auto`, lido do `recommendation.json` público a cada execução;
   `qwen3:8b` até o benchmark publicar); cada extração registra o modelo e o seu digest.
3. **Pessoas:** o que não for confirmado vai para um pull request, com status `suggested`. Só é usado
   depois que uma pessoa revisa e marca `"status": "verified"`.

O **oráculo** é o próprio cabeçalho do arquivo. Os nomes extraídos são comparados com ele (revocação,
precisão, correspondência exata e similaridade de Levenshtein). O modelo nunca vê o cabeçalho, e assim o
oráculo continua independente. Uma extração determinística que o cabeçalho confirma por inteiro é usada
na hora (`extracted`); tudo o que o LLM produz vai sempre para pessoas. As etapas dos PDFs rodam antes
da validação, para que um esquema extraído numa execução seja o usado pela validação da mesma execução.

## Deriva de esquema

Cada execução compara o que vê com os esquemas commitados. Um arquivo com colunas acrescentadas,
removidas ou reordenadas, ou com outro delimitador ou codificação, é uma **deriva observada**; um
dicionário ou esquema que o portal alterou é uma **deriva declarada**. As duas são acrescentadas a
`results/drift.json` e cada uma abre uma Issue no GitHub (rótulos `layer1`, `drift:observed` ou
`drift:declared`). A primeira observação é uma linha de base, nunca uma deriva. A Camada 4 detecta
mudanças nos *metadados* do portal; esta é a deriva dos *arquivos*, e as duas se complementam.

## Achados de documentação

Além da nota, cada execução mede os próprios dicionários, para que afirmações sobre a prática de
documentação venham de um commit e possam ser comparadas entre portais:

- dicionários por formato, quantos puderam ser lidos e por que os outros não (uma página HTML por trás
  do link, um arquivo malformado, nenhuma tabela de campos);
- como os arquivos são ligados aos dicionários, dicionários que não descrevem nenhum arquivo, ligações
  que o portal declara para arquivos que não existem;
- de quantos jeitos o portal escreve um tipo, a fração de campos com tipo reconhecido, campos de data
  com formato declarado;
- com que frequência dicionário e arquivo discordam (campos ausentes, colunas não declaradas, diferenças
  de grafia, formatos inferidos porque nenhum foi declarado), codificações, delimitadores, arquivos que
  não podem ser baixados;
- a qualidade da extração de PDF por etapa, contra o oráculo.

Eles ficam em `findings` de `results/layer1_summary.json` e no painel *Achados de documentação*.

## Nota da Camada 1 e saída para as Camadas 4-5

`results/layer1_summary.json` é público, e a Camada 5 o lê por HTTPS sem token:

| Campo | Significado |
|---|---|
| `l1_rate` | fração dos arquivos verificáveis (com esquema declarado, validados) que estão conformes |
| `l1_pass` | `l1_rate >= L1_PASS_THRESHOLD` (0,85) |
| `datasets.by_level`, `tables.by_level` | distribuição de maturidade |
| `per_dataset` | nível, arquivos verificáveis e conformes de cada conjunto |
| `drift` | eventos de deriva até agora, observados e declarados |
| `findings` | os achados de documentação |

`results/history.json` guarda uma linha compacta por execução semanal (totais, níveis, conformidade,
arquivos consertados ou quebrados, novos ou removidos, deriva), e cada arquivo lembra se estava conforme nas
suas últimas 12 validações: o *Progresso ao longo do tempo* do painel sai daí, e a Camada 5 pode lê-lo para
acompanhar todas as camadas no tempo.

Maturidade e conformidade ficam separadas de propósito: um portal pode documentar pouco e estar
conforme onde documenta, ou o contrário.

## Tratamento de dados e privacidade

Arquivos brutos nunca são guardados nem commitados: são lidos em stream e só contagens, números de
linha, nomes de colunas e tipos inferidos são mantidos. Nenhum valor de célula é escrito em lugar algum
(resultados, issues, painel), já que os arquivos podem ter nomes e números de CPF/CNPJ. Um teste falha se
um valor de célula aparecer na saída.

## Segurança

Os arquivos do portal, seus dicionários em PDF e o LLM não são confiáveis. O workflow tem três jobs: o
que baixa arquivos e executa o modelo tem um token só de leitura e entrega seus resultados como
artefato; o que escreve aceita só os caminhos esperados, para conjuntos e recursos do levantamento commitado (`results/census.json`),
com Table Schemas válidos e textos limitados ([`src/safety.py`](src/safety.py)), e nunca executa o
modelo. Esquemas vindos do modelo só chegam como `suggested`, por pull request. O XML é lido com
`defusedxml`; o texto que vai para uma issue é neutralizado; o painel escapa tudo o que mostra. O Ollama
é uma versão fixa, com checksum verificado.

## Rede e cortesia com o portal

Todo pedido passa por uma única sessão com conexões reaproveitadas (*keep-alive*) que identifica o projeto
(User-Agent com o endereço do repositório), com **15 s para abrir uma conexão e 120 s para ler a resposta**,
e até quatro tentativas com esperas crescentes. O levantamento lê quatro conjuntos ao mesmo tempo; os
arquivos são baixados um de cada vez.

## Princípios FAIR e replicabilidade

- **Localizável / Acessível:** código, esquemas, resultados e resumos são públicos, versionados no Git,
  com `CITATION.cff`.
- **Interoperável:** todo esquema é um Table Schema do Frictionless; os resultados são JSON simples; o
  resumo segue o contrato que as outras camadas do 5L-TEP usam.
- **Reutilizável:** um valor (`portal_url` em `portal.json`) aponta a ferramenta para outro portal CKAN.
  As instâncias da ANEEL e do Recife rodam exatamente este código, com só esse valor (e os READMEs) trocados.
- **Reprodutível:** cada resultado registra o SHA-256 do arquivo, a impressão digital do esquema, os
  parâmetros do método e, nas extrações, o modelo, seu digest e a versão do prompt.

## Rodando a sua própria instância (fork)

1. Faça um fork deste repositório.
2. Edite `portal.json`: ponha em `portal_url` a URL raiz do seu portal CKAN (e, se quiser, `name` e `title`).
3. **Comece com um histórico limpo:** apague `results/`, `schemas/` e `docs/data/`, e faça commit.
4. Em *Settings → Actions → General*, permita que o GitHub Actions crie pull requests (para os esquemas sugeridos).
5. Em *Settings → Pages*, publique a partir do branch `main`, pasta `/docs`.
6. Habilite os workflows na aba *Actions* e rode *5L-TEP Layer 1 Structural Contracts* uma vez à mão.
   A primeira execução valida todos os arquivos do portal e pode levar vários lotes encadeados.

## Início rápido (local)

```bash
pip install -r requirements.txt
python main.py run --minutes 10          # levantamento, 10 minutos de validação, etapa 1 do PDF, relatório
python -m http.server -d docs 8000       # painel em http://localhost:8000
pytest tests/ -v
```

No Windows, habilite caminhos longos antes de clonar (`git config --global core.longpaths true`): os
esquemas ficam em `schemas/<conjunto>/<id do recurso>.<tipo>.json`, e os nomes de conjuntos podem ser longos.

Para experimentar outro portal sem editar nada:

```bash
CKAN_PORTAL_URL=https://dados.recife.pe.gov.br python main.py census
```

## Execução no GitHub Actions

| Workflow | Quando | O quê |
|---|---|---|
| `layer1.yml` | segundas-feiras 03:30 UTC e à mão | levantamento do portal → lotes de validação → extração de PDF → issues, pull request, relatório |
| `tests.yml` | push e pull request | a suíte de testes em Python 3.10–3.12 |

Variáveis do repositório (opcionais): `LLM_MODEL` (padrão `auto`: a escolha do benchmark de modelos; uma tag fixa o modelo) e `LLM_THINK`.
Repositórios públicos rodam nos runners padrão do GitHub sem custo.

## Avaliação

[`evaluation/`](evaluation/) tem os scripts que produzem os números usados na dissertação a partir dos
resultados commitados (ver o README da pasta). `evaluation/pdf_extraction.py` relata a qualidade da
extração de PDF por etapa contra o oráculo (correspondência exata e Levenshtein, as métricas de Al Hilmi
et al., 2026).

## Estrutura do projeto

```
portal.json                 o portal que esta instância avalia (o único valor a trocar)
main.py                     linha de comando: census, validate, extract, report...
src/
  census.py                 o levantamento: escala de maturidade, dicionários, ligações, esquemas declarados
  dictionaries.py           leitores escolhidos pelo conteúdo (CSV, JSON, XML, XLSX, descrição)
  pdf_extract.py            PDF: leitor determinístico, oráculo, cliente do Ollama
  extract.py                as três etapas do PDF
  linker.py                 qual dicionário descreve qual arquivo
  types_map.py              tipos declarados para o Table Schema
  tabular.py                arquivos em stream (CSV, zip, planilhas, Parquet, JSON, XML)
  validate.py               esquema observado e conformidade (leitores de célula do Frictionless)
  work.py                   fila de trabalho e lotes
  drift.py                  eventos de deriva e issues
  report.py                 resumo, achados, dados do painel, selo de status
  safety.py                 verificações na entrega entre os jobs
schemas/                    Table Schemas por conjunto e recurso (declarado, extraído, observado...)
results/                    levantamento (census.json), validação, extração, deriva, layer1_summary.json
docs/                       painel (GitHub Pages)
evaluation/                 scripts dos números da dissertação
tests/                      testes (sem rede)
```

## Configuração

Todo parâmetro de [`src/config.py`](src/config.py) pode ser trocado por uma variável de ambiente de
mesmo nome; os valores usados ficam registrados no resumo. Os principais:

| Variável | Padrão | Significado |
|---|---|---|
| `L1_MAX_ERROR_RATE` | 0,01 | fração de células com erro que um arquivo conforme pode ter |
| `L1_PASS_THRESHOLD` | 0,85 | fração de arquivos conformes para a Camada 1 passar |
| `ROTATION_DAYS` | 28 | um arquivo inalterado é validado de novo depois desse número de dias |
| `VALIDATE_MAX_MINUTES` | 270 | orçamento de tempo de um lote de validação |
| `ORACLE_ACCEPT` / `ORACLE_LLM_BELOW` | 1,0 / 0,8 | concordância para aceitar uma extração de PDF / para tentar o LLM |
| `LLM_MODEL` | `auto` | modelo do Ollama para a extração de PDF (`auto` = a escolha do benchmark; reserva `qwen3:8b`) |

## Limitações

- O mapeamento de tipos e o leitor de descrições são heurísticas, mantidas simples e auditáveis; tipos
  não reconhecidos são relatados como tal. Formatos que o dicionário não declara são inferidos dos
  dados, então um arquivo consistentemente errado num formato não declarado não é sinalizado.
- Os tipos de um DataStore podem ter sido inferidos pelo carregador do portal, não declarados pelo
  publicador; contam para o nível de maturidade, mas são a última opção para a validação.
- Dicionários em PDF sem camada de texto (digitalizações) não são lidos (sem OCR).
- Arquivos XLS e ODS são lidos inteiros em memória (limite `MAX_SHEET_BYTES`, a memória do runner);
  arquivos maiores que isso não são lidos. Arquivos que precisam ir para o disco (zips, planilhas, Parquet)
  são lidos até `MAX_ZIP_BYTES`, o disco livre do runner. Esses são os únicos limites de tamanho: todo o
  resto é lido por inteiro, em quantos lotes forem precisos (um PDF longo vai ao modelo em partes, nunca
  cortado).
- Nomes que diferem só na grafia não reprovam a conformidade; são relatados.
- O oráculo mede *nomes* de campos; os tipos declarados extraídos de um PDF só são verificados por pessoas.

## Referências acadêmicas

- Boehm, B. W. (1984). Verifying and validating software requirements and design specifications. *IEEE Software*, 1(1), 75–88.
- IEEE (2016). *IEEE 1012-2016: Standard for System, Software, and Hardware Verification and Validation*.
- Batini, C., & Scannapieco, M. (2016). *Data and Information Quality*. Springer.
- Open Knowledge Foundation. *Frictionless Table Schema*. https://specs.frictionlessdata.io/table-schema/
- CKAN. *ckanext-validation*. https://github.com/ckan/ckanext-validation
- Wilkinson, M. D. et al. (2016). The FAIR Guiding Principles for scientific data management and stewardship. *Scientific Data*, 3, 160018.
- Al Hilmi, M. A. et al. (2026). Tabular PDF Information Extraction with Local LLMs and Layout-Aware Parsing: A Reliability Evaluation. arXiv:2604.00003.
- Pinheiro, L. S. et al. (2026). 5L-TEP: A Five-Layer Trust Engineering Pyramid for Open Government Data. SOFTENG 2026.

## Licença

MIT para o código ([LICENSE](LICENSE)). Os dados vêm dos portais, sob as licenças de cada um; este
repositório guarda só esquemas e resultados agregados derivados deles.
