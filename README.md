# Synthetic Mobile Robot Fleet Dataset for Logistics Operations

[Português (Brasil)](README.pt-BR.md)

> **Publication status:** local pre-publication package for the proposed public version **v1.0.0**. It is derived from the second internal revision of the dataset and **has not been published or released yet**.

**Creator:** Maria Karolina Silva de Souza

This is a fully synthetic, relational, and temporal dataset that represents a mobile robot fleet operating in a fictional logistics center. It was designed for Data Science, Statistics, Machine Learning, and Data Engineering studies.

No row describes a real company, facility, robot, employee, customer, or operational event. The synthetic operational horizon runs from January 2025 through June 2026. Mission starts range from `2025-01-01 04:05:46` to `2026-06-28 23:49:06`; environmental snapshots extend through `2026-06-30 16:00:00`.

The generation seed recorded during development is `20260730`. A seed alone is not sufficient to reproduce a dataset, and no generation script is included in the current package. Treat the distributed files as frozen artifacts unless a reviewed generation implementation is added later.

## What is included

The eight raw relational tables contain **46,378 rows** in total. The analytical tables contain another **14,680 rows**, and the data dictionary contains **173 metadata rows**. Analytical rows are derived from the raw entities and events, so these counts must not be added together as if every row represented an independent observation.

### Raw relational tables

| Table | Rows | Columns | Purpose |
| --- | ---: | ---: | --- |
| [`MODELOS_ROBO`](data/raw/MODELOS_ROBO.csv) | 4 | 6 | Robot model specifications |
| [`ROBOS`](data/raw/ROBOS.csv) | 60 | 4 | Physical robot registry |
| [`AMBIENTES`](data/raw/AMBIENTES.csv) | 14 | 5 | Logistics-center sectors |
| [`ROTAS`](data/raw/ROTAS.csv) | 182 | 9 | Directed routes between sectors |
| [`CONDICOES_AMBIENTAIS`](data/raw/CONDICOES_AMBIENTAIS.csv) | 22,932 | 8 | Environmental snapshots |
| [`HISTORICO_ROBOS`](data/raw/HISTORICO_ROBOS.csv) | 12,144 | 10 | Time-varying robot-state snapshots |
| [`MISSOES`](data/raw/MISSOES.csv) | 10,000 | 28 | Mission execution records and outcomes |
| [`MANUTENCOES`](data/raw/MANUTENCOES.csv) | 1,042 | 14 | Maintenance orders and downtime |

### Analysis-ready tables and metadata

| File | Rows | Columns | Unit of analysis |
| --- | ---: | ---: | --- |
| [`BASE_ML_MISSOES.csv`](data/analytical/BASE_ML_MISSOES.csv) | 10,000 | 49 | One row per mission |
| [`BASE_ML_MANUTENCAO.csv`](data/analytical/BASE_ML_MANUTENCAO.csv) | 4,680 | 22 | One robot at one weekly reference date; 78 weeks × 60 robots |
| [`dictionary.csv`](data/dictionary.csv) | 173 | 14 | One metadata record per documented variable |

The complete [`dataset_frota_robos_centro_logistico_v2.xlsx`](full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx) workbook contains 13 sheets: `LEIA_ME`, the eight raw tables, the two analytical tables, `VALIDACOES`, and `DICIONARIO`.

## Relational and temporal structure

`MODELOS_ROBO` describes robot models; `ROBOS` instantiates those models and assigns each robot a base sector. `AMBIENTES` describes sectors, while `ROTAS` connects origin and destination sectors. `CONDICOES_AMBIENTAIS` records sector snapshots. `MISSOES` links a robot, route, and environmental condition. `HISTORICO_ROBOS` captures robot state over time, and `MANUTENCOES` records equipment unavailability and repair events.

The mission analytical table joins each mission to robot, route, environmental, and historical attributes. The weekly maintenance table aggregates only history available at each reference date and adds future prediction targets.

## Loading the files

All distributed CSVs use a comma delimiter, UTF-8 without a byte-order mark, and LF line endings. Column names are preserved in Portuguese.

### CSV with Python and pandas

```python
from pathlib import Path

import pandas as pd

root = Path(".")
missions = pd.read_csv(
    root / "data" / "raw" / "MISSOES.csv",
    encoding="utf-8",
    parse_dates=["data_hora_inicio", "data_hora_fim"],
)

mission_ml = pd.read_csv(
    root / "data" / "analytical" / "BASE_ML_MISSOES.csv",
    encoding="utf-8",
    parse_dates=["data_hora_inicio", "condicao_observada_em"],
)
```

### XLSX with Python and pandas

Reading `.xlsx` files with pandas normally requires an Excel engine such as `openpyxl`.

```python
from pathlib import Path

import pandas as pd

workbook = Path("full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx")
missions = pd.read_excel(
    workbook,
    sheet_name="MISSOES",
    parse_dates=["data_hora_inicio", "data_hora_fim"],
)

sheet_names = pd.ExcelFile(workbook).sheet_names
```

### Timestamp normalization

The final CSVs store all 493 nonblank values in `MISSOES.colisao_detectada_em` and all 216 nonblank values in `MISSOES.falha_detectada_em` as ISO 8601 timestamps, consistent with the documented date representation.

Use [`dictionary.csv`](data/dictionary.csv) before converting types or selecting features. It documents units, analytical roles, availability times, valid values, derivations, and whether each variable is suitable before, during, or after a mission.

## Avoiding data leakage

Prediction time matters. A field that is valid for post-mission analysis may leak the answer in a model intended to run before a mission begins.

- In `BASE_ML_MISSOES`, columns ending in `_alvo` are outcomes or targets and must not be used as predictors for those same outcomes.
- Fields observed during or after a mission are not valid for a pre-mission model. Check `momento_disponivel`, `pode_usar_pre_missao`, `pode_usar_online`, and `pode_usar_pos_missao` in the dictionary.
- In `BASE_ML_MANUTENCAO`, historical windows are anchored at `data_referencia`; columns describing the next maintenance or the next seven days are future targets.
- The local audit verified that the environmental and robot snapshots joined to a mission are not later than the mission start. This protects the supplied joins, but it does not make every column valid for every prediction scenario.
- Split temporal experiments chronologically when evaluating future performance. A random split can mix later operational periods into training data.

## Missing values

Missingness is part of the dataset design and must not be treated automatically as a data error.

- `MISSOES.qualidade_sinal_wifi_durante_pct` has 137 missing observations, and `MISSOES.temperatura_robo_durante_c` has 74. These represent designed sensor missingness.
- Conditional event fields are blank when the event does not apply: 9,507 missions have no collision timestamp, 9,784 have no failure timestamp, and the 216 interrupted missions have no delay measurement.
- All 759 preventive maintenance orders have no related mission, while the 283 corrective orders are linked to a mission.
- In `BASE_ML_MANUTENCAO`, missing historical aggregates can occur before enough history exists, and 192 rows have no observed next-maintenance target within the dataset horizon.

The initial pre-publication audit found **57,923 declared-type mismatch occurrences across 12 columns** whose decimal values had been described as integers. It also found **11 mismatches** between observed blanks and the `ausentes` declarations. These metadata issues were corrected in the workbook's `DICIONARIO` sheet and in [`data/dictionary.csv`](data/dictionary.csv): the 12 decimal-valued columns are now declared `numérico`, and all 11 missing-value declarations now include the applicable reason. They are no longer outstanding findings.

## Validation status

A local pre-publication audit found:

- all 11 validations embedded in the workbook marked `APROVADO`, with zero reported occurrences;
- zero violations across 12 independently evaluated structural and temporal rules;
- unique, nonblank primary keys in all eight raw tables;
- all declared foreign keys resolving, including nullable mission links in maintenance;
- no overlapping missions or maintenance orders for the same robot and no mission/maintenance overlap;
- no duplicate rows or blank/duplicate headers in the audited tables;
- every distributed CSV round-trip equivalent to its corresponding workbook sheet;
- complete dictionary coverage for all 173 expected table-column entries (`173/173`), with no missing or extra entries;
- zero declared-type mismatches after the 12 metadata corrections;
- no remaining mismatches between observed blanks and missing-value declarations;
- zero numeric-range violations across the ranges checked; and
- no suspicious headers or pattern matches in a heuristic scan for email addresses, URLs, formatted CPF/CNPJ values, and phone-like strings.

These checks cover defined rules only. They are not a proof of every possible statistical property, a formal privacy certification, or evidence that the data represent real operations. The remaining release gates are listed in the [`PRE_PUBLICATION_CHECKLIST`](PRE_PUBLICATION_CHECKLIST.md).

## Suitable uses

Examples include exploratory analysis, relational joins, visualization, missing-value treatment, temporal aggregation, classification, regression, anomaly detection, clustering, predictive-maintenance exercises, feature engineering, and demonstrations of leakage-aware modeling.

## Limitations and unsuitable uses

- The data were generated, not measured from industrial equipment.
- Variable relationships and rare events were established by the generation process and may not match any specific operation.
- Model performance here does not imply deployment performance in a real facility.
- The dataset must not be used as empirical evidence about a real company or as support for causal claims about real mobile robots.
- Safety-critical, operational, financial, or procurement decisions require appropriate real-world data and domain validation.
- Heuristic privacy and consistency scans do not replace expert review.
- The recorded seed does not make generation reproducible without the original algorithm and dependencies.

## Files and project status

- [`data/raw/`](data/raw/) — eight normalized source tables.
- [`data/analytical/`](data/analytical/) — the two analysis-ready tables.
- [`data/dictionary.csv`](data/dictionary.csv) — variable-level metadata.
- [`full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx`](full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx) — canonical complete internal-revision-v2 workbook used for the v1.0.0 public-package candidate.
- [`docs/DATASET_CARD.md`](docs/DATASET_CARD.md) — purpose, composition, uses, and limitations.
- [`docs/DATA_DICTIONARY.md`](docs/DATA_DICTIONARY.md) — readable data-dictionary guide; the complete records remain in CSV.
- [`docs/SCHEMA.md`](docs/SCHEMA.md) and [`docs/relational-schema.svg`](docs/relational-schema.svg) — relational structure and diagram.
- [`docs/GENERATION_METHODOLOGY.md`](docs/GENERATION_METHODOLOGY.md) — documented generation facts and reproducibility boundary.
- [`docs/VALIDATION.md`](docs/VALIDATION.md) — audit scope, results, and known issues.
- [`docs/LIMITATIONS.md`](docs/LIMITATIONS.md) — detailed interpretation and external-validity limits.
- [`docs/USE_CASES.md`](docs/USE_CASES.md) — recommended and discouraged uses.
- [`docs/guia_explicativo_dataset_frota_robos_v2.pdf`](docs/guia_explicativo_dataset_frota_robos_v2.pdf) — explanatory guide in PDF.
- [`examples/01_dataset_overview.ipynb`](examples/01_dataset_overview.ipynb) — local overview notebook; its executable code still requires a separate software license before publication.
- [`LICENSE-DATA.md`](LICENSE-DATA.md) — scope and terms for data and documentation.
- [`CHANGELOG.md`](CHANGELOG.md) — version history and current pre-release state.
- [`CITATION.cff`](CITATION.cff) — CFF 1.2.0 citation metadata with confirmed authorship.
- [`AUTHORS.md`](AUTHORS.md) — package-wide authorship statement.
- [`FILE_MANIFEST.json`](FILE_MANIFEST.json) — per-file authorship, byte lengths, media types, and SHA-256 hashes.
- [`PRE_PUBLICATION_CHECKLIST.md`](PRE_PUBLICATION_CHECKLIST.md) — unresolved release gates.

## License

The dataset files and project documentation are prepared for distribution under the **Creative Commons Attribution-NonCommercial 4.0 International license (CC BY-NC 4.0)**. See [`LICENSE-DATA.md`](LICENSE-DATA.md) for the exact local scope.

That license does **not** cover source code, executable scripts, or notebook files/code. The included overview notebook therefore requires a separate software license before publication. No commercial permission for the data is granted by the CC BY-NC 4.0 terms.

## Citation

Citation metadata with the confirmed creator is available in [`CITATION.cff`](CITATION.cff):

> Silva de Souza, Maria Karolina. *Synthetic Mobile Robot Fleet Dataset for Logistics Operations*. Version 1.0.0. Dataset prepared for publication.

The release date, repository URL, DOI, ORCID, and affiliation are omitted because they have not been established or confirmed. Add them to the CFF metadata only when they are real; do not interpret the presence of `CITATION.cff` as evidence that v1.0.0 has already been published.
