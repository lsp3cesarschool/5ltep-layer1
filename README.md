# 5LTEP-L1: 5L-TEP Layer 1 Structural Contracts Toolkit

[![Tests](https://github.com/lsp3cesarschool/5ltep-layer1/actions/workflows/tests.yml/badge.svg)](https://github.com/lsp3cesarschool/5ltep-layer1/actions/workflows/tests.yml) [![Layer 1](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2Flsp3cesarschool%2F5ltep-layer1%2Fmain%2Fdocs%2Fdata%2Fstatus.json)](https://github.com/lsp3cesarschool/5ltep-layer1/actions/workflows/layer1.yml) [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**English** · [Português](LEIAME.md)

**Checks whether an open data portal says what its files should look like, and whether the files
look like that.** A survey of every dataset of a CKAN portal on a schema maturity scale, readers for
the data dictionaries the portal publishes (including PDF, with a local AI model and human review),
and validation of every published file against its declared schema with the Frictionless Table Schema.

| Resource | What you find there |
|---|---|
| 📊 **Dashboard** | [lsp3cesarschool.github.io/5ltep-layer1](https://lsp3cesarschool.github.io/5ltep-layer1/?lang=en): maturity, documentation findings, every dataset and file |
| 🔀 **Schema drift** | [![drift issues](https://img.shields.io/github/issues/lsp3cesarschool/5ltep-layer1/layer1?label=drift%20issues&color=0366d6)](https://github.com/lsp3cesarschool/5ltep-layer1/issues?q=is%3Aissue+label%3Alayer1): one issue each time a file, or the schema declared for it, changes structure |
| 🧑‍⚖️ **Suggested schemas** | [pull requests](https://github.com/lsp3cesarschool/5ltep-layer1/pulls?q=is%3Apr+schemas+suggested) with schemas extracted from PDF dictionaries, waiting for a person |
| 🧪 **Model choice** | [5ltep-layer1-modeltest](https://github.com/lsp3cesarschool/5ltep-layer1-modeltest): the benchmark that picks the LLM for PDF dictionaries |
| 🔁 **Control experiments** | [5ltep-layer1-aneel](https://github.com/lsp3cesarschool/5ltep-layer1-aneel) and [5ltep-layer1-recife](https://github.com/lsp3cesarschool/5ltep-layer1-recife): the same code on two other portals |

> **Status: research demonstration.** This toolkit is part of a master's research project and is
> maintained by its author. It is not an official IBAMA (or ANEEL, or Recife) service, and it does not
> assume that any agency will review its results or adopt it. The full flow, human review included, is
> working and ready to be adopted; the author does not act as a reviewer of the tool's own output.

## Use case in one paragraph

Before anyone can say that a value in an open dataset is wrong, someone has to say what the value
should look like: which columns the file has, which type each one holds, how long a code can be. That
is the publisher's own contract, defined before the data exists, and it is the first layer of trust.
This toolkit asks two questions of a whole portal. **Does the portal publish a verifiable contract?**
Some publish none, some publish a PDF only people can read, some a spreadsheet a program can read.
**Do the files honour it?** A dictionary that lists a column the file does not have, a date written
in a format nobody declared, or a code longer than the declared size are all found automatically,
every week, file by file. The answer helps a data team decide where documentation work pays off.

## Key terms

| Term | Meaning here |
|---|---|
| **Data dictionary** | What the publisher declares about a file: its fields, types, sizes and descriptions, in any format (CSV, JSON, XLSX, PDF, or a list in the resource's description). |
| **Table Schema** | The [Frictionless Data](https://specs.frictionlessdata.io/table-schema/) standard for describing a table (field names, types, formats, constraints) from the same community as CKAN. Every schema here is stored in it. |
| **Declared schema** | The Table Schema built from what the portal declares. **Observed schema**: the one the file itself shows (header, inferred types, delimiter, encoding). |
| **Conformance** | A file conforms when every declared field is in it, it has no undeclared column, and at most 1% of the checked cells break the declared type or constraints. |
| **Maturity level** | 0 to 4: how verifiable the structure the portal declares is (see below). |
| **Oracle** | The published file's own header, used to measure how well a PDF dictionary was extracted. |
| **Schema drift** | A change in the structure of a file, or in the schema declared for it, between two runs. |

## Overview

Layer 1 of the Five-Layer Trust Engineering Pyramid (5L-TEP) is *verification*, in the sense of
Boehm (1984) and IEEE 1012: does the product conform to its own specification? It sits apart from
Layer 2 (domain rules, written *after* the data exists by experts), because the owner and the
moment of each rule differ: here the specification is the publisher's, and it predates the data.

### Architecture

```
portal.json (one value: the portal URL)
        │
        ▼
 survey ──── CKAN API: every dataset and resource ──── dictionaries (CSV, JSON, XML, XLSX, PDF, description)
   │              DataStore types, attached schema            │
   │                                                          ▼
   │                                          schemas/<dataset>/<resource>.declared.json
   ▼
 work queue (new, changed, schema changed, read by an older reader, monthly rotation)
   │
   ▼
 validate ── stream every file (no raw data stored) ── observed schema + conformance per field
   │                                                          │
   │                                                          ▼
   │                                     drift (file or declared schema changed) → GitHub Issue
   ▼
 PDF dictionaries: deterministic → local LLM → pull request (the file's header is the oracle)
   │
   ▼
 report ── results/layer1_summary.json (Layer 5) · documentation findings · dashboard
```

## The maturity scale

Every tabular file (CSV, XLSX, XLS, ODS, Parquet, JSON, XML, or a zip holding any of them) gets a level
from what the portal declares about it:

| Level | Criterion (read from the CKAN API) |
|---|---|
| 0 | no dictionary or schema is associated with the file |
| 1 | a dictionary readable only by people (PDF, HTML, or a field list written in the description), or a machine-readable one that cannot be processed (an HTML page behind the link, malformed JSON) |
| 2 | a machine-readable dictionary (CSV, JSON, XML, XLSX) that the toolkit reads |
| 3 | types exposed through the API in a standard form: a Table Schema attached to the resource, or DataStore fields with real types (a DataStore whose fields are all `text` declares nothing) |
| 4 | level 3, and the file conforms to its declared schema |

A dataset is as verifiable as its least documented file. The level describes the **portal**: it
does not change when this toolkit extracts a schema from a PDF. That extraction makes conformance
checkable, which is reported separately.

## Where the declared schema comes from

Dictionary readers are chosen by the file's *content*, not by the portal, so another portal needs no
code change when its dictionaries look like any of these:

- a table with one row per field and a header naming the columns (`nome_atributo;datatype;descricao`,
  `Nome da variável | Tipo`...), in CSV or XLSX (one part per sheet);
- JSON with a list of field objects (`metadados.campos[{codigo, tipo, tamanho}]`), which may also name
  the resources it describes;
- XML with one repeated element per field;
- a list of fields in the resource's description (`* SEQ_TAD – Chave...`, `- **UF**: Sigla. Formato: texto`);
- a PDF (next section).

Each dictionary is then linked to the file it describes, by the strongest evidence available, and the
method is recorded: the dictionary names the resource id; its fields overlap the file's header; its
name matches the resource's name; or it is the only dictionary of the dataset, with a generic name.
The header of a file never validated is read in the survey (its first line; nothing else is kept), so
that the file is linked by its header, and validated against its dictionary, already in its first run.

Declared types are written in many ways (`TEXTO (STRING)`, `Cadeia de caracteres`, `VARCHAR`,
`char`...). They are mapped to Table Schema types by keywords, in a fixed and auditable order
([`src/types_map.py`](src/types_map.py)); a type not recognised becomes `any` and is reported, not guessed.
A format written in the type (`DATA (DD/MM/AAAA)`) is kept.

When several sources exist for a file, validation uses the first of: attached Table Schema, machine-readable
dictionary, schema extracted from a PDF (once confirmed), list in the description, DataStore types.

## Validation

Every file is read as a stream: CSV, JSON (a list of records) and XML (repeated elements) straight from
the HTTP response; a zip, Parquet or XLS from a temporary file, member by member, sheet by sheet or
batch by batch. The format is recognised from the first bytes, not from the label on the portal. Zips
inside zips are opened too, and a zip larger than the runner's disk is read as it streams. Nothing is kept but counts. Encoding and delimiter are detected
for text formats; bytes that do not decode are counted. For each file:

- the **observed schema** (header, delimiter, encoding, and the narrowest type the first 5,000 rows of
  each column fit) is written to `schemas/<dataset>/<resource>.observed.json`;
- against the declared schema: declared fields missing from the file, columns not declared, names
  that differ only in spelling (`Nome/Razão Social` and `NOME_RAZAO_SOCIAL`), and every cell of the
  matched columns checked with the Frictionless cell readers (type, `maxLength`...).

Formats a dictionary does not declare (a date written `03/05/2022`, a decimal comma) are taken from
what most sampled values follow and recorded as *inferred*: the check is whether the values are
consistent with the declared type in one format, not whether they follow ISO 8601.

**One table, several formats.** When a dataset publishes the same table in several formats (the
same name with `CSV`, `XLSX`, `JSON`...), it counts as one table: the first in the order CSV, ZIP,
Parquet, XLSX, ODS, XLS, JSON, XML is validated in full, and the others are only checked for their
columns (same columns, or which are missing or extra). **Not a table:** a zip holding only documents
or maps is listed apart and left out of the tabular universe; a link that returns a web page or a PDF
instead of the table is a link failure. Geographic formats (shapefile, KML, GeoJSON, WMS/WFS) and HTML
tables are out of scope.

The first run of a portal validates every file, in batches that fit the runner's time limit. Later
runs validate only what is new, changed (URL, CKAN metadata, or the server's ETag, Last-Modified or
size), validated against a schema that changed, failed, or was last checked more than 28 days ago.

## PDF dictionaries: deterministic, LLM, people

Many portals publish dictionaries only as PDF. They are turned into schemas in three stages, the order
Al Hilmi et al. (2026) found most reliable for tabular PDFs with local models:

1. **Deterministic:** pdfplumber finds the tables; each row is read by *content* (an identifier is the
   name, a type word is the type, a number is the size), since cells drift between columns across pages.
2. **Local LLM:** only where stage 1 found nothing or disagrees with the data, a model run with Ollama
   on the runner reads the PDF text and returns the field list as JSON (temperature 0, fixed seed).
   The model is the one the [Layer 1 model benchmark](https://github.com/lsp3cesarschool/5ltep-layer1-modeltest)
   approves for this task (`LLM_MODEL=auto`, read from its public `recommendation.json` at every run;
   `qwen3:8b` until the benchmark has published); every extraction records the model and its digest.
3. **People:** whatever is not confirmed goes to a pull request, with status `suggested`. It is used only
   after a person reviews it and sets `"status": "verified"`.

The **oracle** is the file's own header. The extracted names are compared with it (recall, precision,
exact match and Levenshtein similarity). The model never sees the header, so the oracle stays
independent. A deterministic extraction that the header confirms in full is used right away
(`extracted`); anything the LLM produced always goes to people. The PDF stages run before the
validation, so that a schema extracted in a run is the one the same run validates against.

## Schema drift

Each run compares what it sees with the committed schemas. A file whose columns were added, removed or
reordered, or whose delimiter or encoding changed, is an **observed drift**; a dictionary or schema the
portal changed is a **declared drift**. Both are appended to `results/drift.json` and each opens one
GitHub Issue (labels `layer1`, `drift:observed` or `drift:declared`). The first observation is a
baseline, never a drift. Layer 4 detects changes in the portal's *metadata*; this is the drift of the
*files*, and the two complement each other.

## Documentation findings

Besides the score, each run measures the dictionaries themselves, so that statements about
documentation practice come from a commit and can be compared between portals:

- dictionaries by format, how many could be read, and why the others could not (an HTML page behind the
  link, a malformed file, no field table);
- how files are linked to dictionaries, dictionaries that describe no file, links the portal declares to
  files that do not exist;
- how many spellings of a type the portal uses, the share of fields with a recognised type, date fields
  with a declared format;
- how often dictionary and file disagree (missing fields, undeclared columns, spelling differences,
  formats inferred because none was declared), encodings, delimiters, files that cannot be downloaded;
- the quality of PDF extraction by stage, against the oracle.

They are in `findings` of `results/layer1_summary.json` and in the *Documentation findings* panel of the dashboard.

## Layer 1 score and output for Layers 4-5

`results/layer1_summary.json` is public, so Layer 5 reads it over HTTPS with no token:

| Field | Meaning |
|---|---|
| `l1_rate` | share of the verifiable files (those with a declared schema, validated) that conform |
| `l1_pass` | `l1_rate >= L1_PASS_THRESHOLD` (0.85) |
| `datasets.by_level`, `tables.by_level` | maturity distribution |
| `per_dataset` | level, verifiable and conformant files of each dataset |
| `drift` | drift events so far, observed and declared |
| `findings` | the documentation findings |

`results/history.json` keeps one compact row per weekly run (totals, levels, conformance, files fixed or
broken, new or removed, drift), and each file remembers whether it conformed in its last 12 validations:
the dashboard's *Progress over time* is drawn from it, and Layer 5 can read it to follow every layer over time.

Maturity and conformance are kept apart on purpose: a portal can document little and conform well
where it documents, or the reverse.

## Data handling and privacy

Raw files are never stored or committed: they are read as streams and only counts, row numbers,
column names and inferred types are kept. No cell value is written anywhere (results, issues,
dashboard), since files may carry names and CPF/CNPJ numbers. A test fails if a cell value appears in
the output.

## Security

The portal's files, its PDF dictionaries and the LLM are not trusted. The workflow has three jobs: the
one that downloads files and runs the model has a read-only token and hands its results over as an
artifact; the one that writes accepts only the expected paths, for datasets and resources in the
committed survey (`results/census.json`), with valid Table Schemas and bounded texts ([`src/safety.py`](src/safety.py)), and
never runs the model. Schemas from the model can only arrive as `suggested`, through a pull request.
XML is parsed with `defusedxml`; text that reaches an issue is neutralised; the dashboard escapes
everything it shows. Ollama is a pinned, checksum-verified release.

## Network and politeness

Every request goes through one keep-alive session that identifies the project (User-Agent with the
repository's address), with **15 s to open a connection and 120 s to read the answer**, and up to four
attempts with growing waits. The survey reads four datasets at a time; files are downloaded one at a time.

A server that does not answer is a fact about that moment, not about the portal's documentation:

- After 8 failures in a row (`UNREACHABLE_STREAK`) the survey stops asking the file server, and asks
  again what was left after 5, 10 and 20 minutes (`DICTIONARY_RETRY_ROUNDS`, `DICTIONARY_RETRY_WAIT_S`).
- A dataset whose dictionary still does not answer keeps its last reading (marked `kept_from`), and its
  committed schemas stay: no drift issue, no lower level.
- In a first run there is no last reading: its files wait, pending, for the next run, instead of being
  checked against a weaker schema (the DataStore) as if that were what the portal documents.
- The dashboard shows how many dictionaries did not answer, apart from the ones the portal publishes
  in a form that cannot be read.

## FAIR principles and replicability

- **Findable / Accessible:** code, schemas, results and summaries are public, versioned in Git, with a
  `CITATION.cff`.
- **Interoperable:** every schema is a Frictionless Table Schema; results are plain JSON; the summary
  follows the contract the other 5L-TEP layers use.
- **Reusable:** one value (`portal_url` in `portal.json`) points the toolkit at another CKAN portal. The
  ANEEL and Recife instances run this exact code with only that value (and their READMEs) changed.
- **Reproducible:** each result records the file's SHA-256, the schema fingerprint, the method parameters
  and, for extractions, the model, its digest and the prompt version.

## Running your own instance (fork)

1. Fork this repository.
2. Edit `portal.json`: set `portal_url` to your CKAN portal's root URL (and, optionally, `name` and `title`).
3. **Start with a clean history:** delete `results/`, `schemas/` and `docs/data/`, and commit.
4. In *Settings → Actions → General*, allow GitHub Actions to create pull requests (for suggested schemas).
5. In *Settings → Pages*, publish from the `main` branch, folder `/docs`.
6. Enable the workflows in the *Actions* tab and run *5L-TEP Layer 1 Structural Contracts* once by hand.
   The first run validates every file of the portal and may take several chained batches.

## Quick start (local)

```bash
pip install -r requirements.txt
python main.py run --minutes 10          # survey, 10 minutes of validation, PDF stage 1, report
python -m http.server -d docs 8000       # dashboard at http://localhost:8000
pytest tests/ -v
```

On Windows, enable long paths before cloning (`git config --global core.longpaths true`): schema
files are stored as `schemas/<dataset>/<resource id>.<kind>.json`, and dataset names can be long.

To try another portal without editing anything:

```bash
CKAN_PORTAL_URL=https://dados.recife.pe.gov.br python main.py census
```

## GitHub Actions deployment

| Workflow | When | What |
|---|---|---|
| `layer1.yml` | Mondays 03:30 UTC and by hand | survey of the portal → validation batches → PDF extraction → issues, pull request, report |
| `tests.yml` | push and pull request | the test suite on Python 3.10–3.12 |

Repository variables (optional): `LLM_MODEL` (default `auto`: the model benchmark's choice; a tag pins the model) and `LLM_THINK`.
Public repositories run on GitHub's standard runners at no cost.

## Evaluation

[`evaluation/`](evaluation/) holds the scripts that produce the numbers used in the thesis from the
committed results (see its README). `evaluation/pdf_extraction.py` reports the PDF extraction quality
per stage against the oracle (exact match and Levenshtein, the metrics of Al Hilmi et al., 2026).

## Project structure

```
portal.json                 the portal this instance evaluates (the only value to change)
main.py                     command line: census, validate, extract, report...
src/
  census.py                 the survey: maturity scale, dictionaries, links, declared schemas
  dictionaries.py           readers chosen by content (CSV, JSON, XML, XLSX, description)
  pdf_extract.py            PDF: deterministic reader, oracle, Ollama client
  extract.py                the three PDF stages
  linker.py                 which dictionary describes which file
  types_map.py              declared types to Table Schema
  tabular.py                files as streams (CSV, zip, spreadsheets, Parquet, JSON, XML)
  validate.py               observed schema and conformance (Frictionless cell readers)
  work.py                   work queue and batches
  drift.py                  drift events and issues
  report.py                 summary, findings, dashboard data, status badge
  safety.py                 hand-over checks between the jobs
schemas/                    Table Schemas per dataset and resource (declared, extracted, observed...)
results/                    survey (census.json), validation, extraction, drift, layer1_summary.json
docs/                       dashboard (GitHub Pages)
evaluation/                 scripts for the thesis numbers
tests/                      tests (no network)
```

## Configuration

Every parameter in [`src/config.py`](src/config.py) can be overridden by an environment variable of
the same name; the values used are recorded in the summary. The main ones:

| Variable | Default | Meaning |
|---|---|---|
| `L1_MAX_ERROR_RATE` | 0.01 | share of cells with errors a conformant file may have |
| `L1_PASS_THRESHOLD` | 0.85 | share of conformant files for Layer 1 to pass |
| `ROTATION_DAYS` | 28 | an unchanged file is validated again after this many days |
| `VALIDATE_MAX_MINUTES` | 270 | time budget of one validation batch |
| `ORACLE_ACCEPT` / `ORACLE_LLM_BELOW` | 1.0 / 0.8 | agreement to accept a PDF extraction / to try the LLM |
| `LLM_MODEL` | `auto` | Ollama model for PDF extraction (`auto` = the benchmark's choice; fallback `qwen3:8b`) |

## Size and time limits

Nothing is left out to save time: running costs nothing on a public repository, so the work is split
into as many batches as it takes. The only limits are what GitHub's machines can hold.

**What GitHub accepts** (public repository, standard runner `ubuntu-latest`):

| Resource | GitHub limit | How this repository fits in it |
|---|---|---|
| Minutes of Actions | free and unlimited | full validation of every file, weekly |
| Machine | 4 CPUs, 16 GB of memory, 14 GB of disk | everything is read in pieces (rows, members, batches); only spreadsheets and Parquet need the whole file on disk |
| One job | 6 hours | one batch: up to 240 minutes of validation (`VALIDATE_MAX_MINUTES`) in a job of at most 350 |
| A chain of batches | (no limit; a run lasts up to 35 days) | up to 40 batches in a row (`MAX_BATCHES`, about 160 hours); if that is not enough, the next run continues where it stopped |
| One file in the repository | above 50 MB a warning, above 100 MB refused | each result file at most 80 MB (checked before it is committed) |
| GitHub Pages | site up to 1 GB | the dashboard reads only the summary data |

**What the system accepts:**

| What | Limit | Why |
|---|---|---|
| CSV, TXT, JSON, XML | none | read as they stream, record by record, never held whole |
| Zip | none | read from disk up to 12 GB (`MAX_ZIP_BYTES`); a larger zip is read as it streams, member by member; zips inside zips up to 5 levels (`ZIP_MAX_DEPTH`, a guard against a zip that nests itself) |
| XLSX, ODS | 12 GB per file (the disk) | the file must be on disk (its parts are spread through it); rows are then read one at a time. XLSX holds at most 1,048,576 rows per sheet |
| XLS | none in practice | the format holds at most 65,536 rows per sheet; read a sheet at a time |
| Parquet | 12 GB per file (the disk) | its index is at the end, so the file must be on disk; rows are then read in batches |
| Dictionaries (CSV, JSON, XLSX...) | 2 GB (`MAX_DICTIONARY_BYTES`) | read whole in memory |
| PDF dictionary to the local model | none: pieces of 20,000 characters (`LLM_MAX_PDF_CHARS`) | each piece fits in the model's context; an answer cut by its length limit is asked again with the piece in halves |
| Local model per batch | 60 minutes (`EXTRACT_MAX_MINUTES`) | the PDFs left wait for the next batch |

A file above a limit is not dropped silently: it is listed with the reason (`too-large`) in the
summary and on the dashboard. What is kept *about* each file is bounded on purpose, never the reading:
every row is validated, but only the first 5 row numbers of each kind of error per field are stored
(`ERROR_ROWS_KEPT`; the counts are complete), the observed types are inferred from the first 5,000
rows (`SAMPLE_ROWS`), and long lists in the summary keep their first items next to the full count.

## Limitations

- Type mapping and the description parser are heuristics, kept simple and auditable; unrecognised
  types are reported as such. Formats a dictionary does not declare are inferred from the data, so a
  file that is consistently wrong in an undeclared format is not flagged.
- A DataStore's types may have been inferred by the portal's loader, not declared by the publisher;
  they count for the maturity level but are the last choice for validation.
- PDF dictionaries without a text layer (scans) are not read (no OCR).
- A single XLSX, ODS or Parquet larger than the runner's disk cannot be read on GitHub's machines; see
  [Size and time limits](#size-and-time-limits).
- Names that differ only in spelling do not fail conformance; they are reported.
- The oracle measures field *names*; declared types extracted from a PDF are checked only by people.

## Academic references

- Boehm, B. W. (1984). Verifying and validating software requirements and design specifications. *IEEE Software*, 1(1), 75–88.
- IEEE (2016). *IEEE 1012-2016: Standard for System, Software, and Hardware Verification and Validation*.
- Batini, C., & Scannapieco, M. (2016). *Data and Information Quality*. Springer.
- Open Knowledge Foundation. *Frictionless Table Schema*. https://specs.frictionlessdata.io/table-schema/
- CKAN. *ckanext-validation*. https://github.com/ckan/ckanext-validation
- Wilkinson, M. D. et al. (2016). The FAIR Guiding Principles for scientific data management and stewardship. *Scientific Data*, 3, 160018.
- Al Hilmi, M. A. et al. (2026). Tabular PDF Information Extraction with Local LLMs and Layout-Aware Parsing: A Reliability Evaluation. arXiv:2604.00003.
- Pinheiro, L. S. et al. (2026). 5L-TEP: A Five-Layer Trust Engineering Pyramid for Open Government Data. SOFTENG 2026.

## License

MIT for the code ([LICENSE](LICENSE)). Data comes from the portals under their own licenses; this
repository stores only schemas and aggregate results derived from them.
