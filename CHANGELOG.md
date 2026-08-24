# Changelog

This file records changes to the public dataset package. The workbook filename retains `v2` because it is the second internal revision; the first proposed public release is version `1.0.0`.

**Dataset creator:** Maria Karolina Silva de Souza

## [1.0.0] — pre-publication candidate

**Status:** prepared locally; not published and not released.

### Included

- Eight raw relational tables with 46,378 total rows.
- Temporal robot-state and environmental snapshots.
- 10,000 mission records and 1,042 maintenance orders.
- `BASE_ML_MISSOES` with 10,000 rows and 49 columns.
- `BASE_ML_MANUTENCAO` with 4,680 rows and 22 columns.
- A 173-row, 14-column data dictionary.
- Complete XLSX workbook and comma-delimited UTF-8 CSV distributions.
- English and Brazilian Portuguese README files.
- Dataset card, readable data dictionary, relational schema, generation methodology, validation report, limitations, and use-case documentation.
- Relational diagram, explanatory PDF, and introductory notebook included in the local package.
- Data and documentation license notice for CC BY-NC 4.0.
- Final CFF 1.2.0 citation metadata identifying Maria Karolina Silva de Souza as the creator.
- A pre-publication checklist documenting unresolved release gates.

### Validation recorded for this candidate

- All 11 validations embedded in the workbook are approved with zero occurrences.
- Twelve independent structural and temporal checks report zero violations.
- Distributed CSV tables are round-trip equivalent to their corresponding workbook sheets.
- The dictionary covers all 173 expected table-column entries, with no missing, duplicate, or extra entries.
- The final audit reports zero declared-type mismatches, no missing-declaration mismatches, and zero numeric-range violations.

### Corrections after the initial audit

- The initial audit found 57,923 declared-type mismatch occurrences across 12 columns containing decimal values.
- Those 12 declarations were changed from `inteiro` to `numérico` in the workbook's `DICIONARIO` sheet and in `data/dictionary.csv`.
- The initial audit found 11 missing-value declaration divergences; all 11 declarations were corrected with their applicable reasons.
- All 493 nonblank `MISSOES.colisao_detectada_em` values and all 216 nonblank `MISSOES.falha_detectada_em` values were normalized from Excel/OLE date serials to ISO 8601 timestamps in the final CSVs.
- The post-correction audit confirmed that both issue sets were resolved and that dictionary coverage remains `173/173`.

### Required before changing this entry to a release

- Remove the byte-identical root-level staging copy so the workbook under `full_dataset/` is the only distributed XLSX.
- Add the actual release date and canonical repository URL only after they exist.
- Add a DOI only after it has been issued.

Future published versions must receive a new version number. Published data files must not be silently replaced.
