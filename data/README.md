---
title: "Data distribution"
author: "Maria Karolina Silva de Souza"
version: "1.0.0"
---

# Data distribution

**Author:** Maria Karolina Silva de Souza

All CSV files in this directory are UTF-8 without BOM, use a comma delimiter, use LF line endings, and end with a newline. Dates are exported as sortable ISO-style strings. Identifiers such as `RB001`, `S01`, and `MS000001` must be kept as text.

## Contents

- `raw/`: eight registration, temporal, event, and maintenance tables; 46,378 records in total.
- `analytical/`: `BASE_ML_MISSOES` with 10,000 mission rows and `BASE_ML_MANUTENCAO` with 4,680 robot-week rows.
- `dictionary.csv`: 173 metadata records covering every column in the 12 tabular workbook sheets.

The final dictionary uses `numérico` for fractional measurements and explicitly documents structural or temporal missingness. The post-correction audit found zero declared-type mismatches, zero missingness-declaration mismatches, and zero violations in 30 numeric range checks.

CSV files do not provide a portable embedded authorship field without modifying the data schema. Their authorship and SHA-256 hashes are recorded in [`../FILE_MANIFEST.json`](../FILE_MANIFEST.json), with citation metadata in [`../CITATION.cff`](../CITATION.cff) and license terms in [`../LICENSE-DATA.md`](../LICENSE-DATA.md).

Copyright 2026 Maria Karolina Silva de Souza.
