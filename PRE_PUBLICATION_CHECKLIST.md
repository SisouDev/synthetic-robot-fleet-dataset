# Pre-publication checklist

This checklist applies to the proposed public dataset version **v1.0.0**, derived from internal revision v2. Checked items were verified locally; unchecked items remain release gates. Nothing in this file states that the dataset has already been published.

**Dataset creator:** Maria Karolina Silva de Souza

## Identity, authorship, and citation — blocking

- [x] Adopt the editorial title `Synthetic Mobile Robot Fleet Dataset for Logistics Operations`.
- [x] Prepare `v1.0.0` as the first proposed public version while retaining the internal-revision-v2 XLSX filename.
- [x] Confirm the creator as Maria Karolina Silva de Souza, with `given-names = Maria Karolina` and `family-names = Silva de Souza`.
- [x] Create `CITATION.cff` using CFF 1.2.0 and the confirmed authorship.
- [x] Embed Maria Karolina Silva de Souza as author in XLSX, PDF, SVG, and notebook metadata and visible authorship fields.
- [x] Attribute every distributed file, including schema-preserving CSVs, in `FILE_MANIFEST.json` with SHA-256 hashes.
- [x] Remove the prohibited generator/service markers from text files, PDF content and metadata, and every OOXML part in the XLSX; the final local scan found zero occurrences.
- [x] Omit unconfirmed ORCID and affiliation rather than inventing values.
- [ ] Add the actual release date only when the release occurs.
- [ ] Add a real canonical repository URL only after the repository exists.
- [ ] Add a DOI only after it has been issued; use the bare `10.xxxx/...` value in CFF.
- [x] Confirm that `CITATION.cff` contains no placeholder strings.
- [ ] Run an independent CFF 1.2.0 schema validator before publication.

The final CFF intentionally omits release date, URL, DOI, ORCID, and affiliation. Add optional fields only after they have been verified; never use placeholders in the release metadata.

## Data package verified locally

- [x] Eight raw CSV tables are present under `data/raw/`.
- [x] The raw tables contain 46,378 total records.
- [x] `BASE_ML_MISSOES.csv` contains 10,000 records and 49 columns.
- [x] `BASE_ML_MANUTENCAO.csv` contains 4,680 records and 22 columns.
- [x] `data/dictionary.csv` contains 173 records and 14 columns.
- [x] The canonical XLSX under `full_dataset/` contains 13 expected sheets.
- [x] CSVs use comma delimiters, UTF-8 without BOM, and LF line endings.
- [x] Each distributed CSV was round-trip equivalent to its corresponding XLSX sheet in the local audit.
- [x] The canonical XLSX has the same byte length and SHA-256 recorded by the audit.

## Structural and temporal validation verified locally

- [x] All eight raw-table primary keys are nonblank and unique.
- [x] All 15 declared foreign-key checks pass; the maintenance-to-mission link is correctly nullable.
- [x] All 11 validations embedded in the workbook are marked `APROVADO` with zero occurrences.
- [x] All 12 independent structural and temporal checks report zero violations.
- [x] No audited table has duplicate rows, blank headers, or duplicate headers.
- [x] Missions do not overlap for the same robot.
- [x] Maintenance orders do not overlap for the same robot.
- [x] Missions do not overlap maintenance for the same robot.
- [x] Mission loads remain within model capacity.
- [x] Joined robot and environmental snapshots are not later than mission start.
- [x] `BASE_ML_MISSOES` has exactly one analytical row per mission.
- [x] `BASE_ML_MANUTENCAO` covers 78 weeks × 60 robots.

## Metadata and representation review

- [x] Record the observed sensor missingness: 137 missing Wi-Fi values and 74 missing robot-temperature values in `MISSOES`.
- [x] Confirm conditional mission nulls: 9,507 collision timestamps absent when no collision, 9,784 failure timestamps absent when no failure, and 216 delay fields absent for interrupted missions.
- [x] Confirm that 759 preventive maintenance rows have no related mission and all 283 corrective rows have one.
- [x] Preserve the initial-audit finding of 57,923 declared-type mismatch occurrences across 12 decimal-valued columns.
- [x] Change all 12 affected declarations from `inteiro` to `numérico` in both the workbook dictionary and `data/dictionary.csv`.
- [x] Correct the missing-value declaration and reason for `HISTORICO_ROBOS.evento_referencia_id` (60 blanks).
- [x] Correct the missing-value declarations and reasons for `BASE_ML_MISSOES.atraso_alvo` and `atraso_min_alvo` (216 blanks each).
- [x] Correct the missing-value declaration and reason for `BASE_ML_MANUTENCAO.dias_desde_ultima_manutencao` (230 blanks).
- [x] Correct the missing-value declarations and reasons for the three 30-day mean fields in `BASE_ML_MANUTENCAO` (13 blanks each).
- [x] Correct the missing-value declarations and reasons for the three next-maintenance target fields in `BASE_ML_MANUTENCAO` (192 blanks each).
- [x] Correct the declaration for blank `DICIONARIO.unidade` values (119 blanks) with the applicable not-applicable reason.
- [x] Confirm final dictionary coverage of all 173 expected table-column entries (`173/173`), with no missing, duplicate, or extra entries.
- [x] Confirm the final audit reports zero declared-type mismatches.
- [x] Confirm the final audit reports an empty `missingDeclarationMismatches` collection.
- [x] Confirm the final audit reports zero numeric-range violations.
- [x] Normalize all 493 nonblank `MISSOES.colisao_detectada_em` and 216 nonblank `MISSOES.falha_detectada_em` values to ISO 8601 timestamps in the final CSVs.
- [x] Re-run the audit after the metadata corrections and confirm that the README counts still match.

## Privacy, synthetic status, and limitations

- [x] State prominently that the dataset is fully synthetic and represents no real organization or fleet.
- [x] Record the synthetic period as January 2025 through June 2026.
- [x] State that model performance on synthetic data does not imply real-world performance.
- [x] State that the dataset is not empirical evidence and is unsuitable for causal claims about real robots.
- [x] State that seed `20260730` alone does not make the dataset reproducible.
- [x] Run heuristic header and pattern scans for emails, URLs, formatted CPF/CNPJ values, and phone-like strings; the local audit found no matches.
- [ ] Perform a final independent human review for personal data, real company names, confidential content, or text that could imply the records are real.
- [x] Confirm that all public descriptions distinguish heuristic screening from formal privacy certification.

## Documentation and usability

- [x] Provide English and Brazilian Portuguese README files.
- [x] Document the raw, analytical, dictionary, and XLSX artifacts with real relative links.
- [x] Document CSV and XLSX loading examples.
- [x] Explain target columns, prediction time, temporal splitting, and leakage risks.
- [x] Document designed and conditional missingness.
- [x] Document suitable uses and material limitations.
- [x] Resolve every final relative link after the complete package layout is frozen.
- [x] Import and render the XLSX, then inspect previews of all 13 sheets.
- [x] Load every final CSV with an independent parser after the final copy.
- [x] Add all local documentation required by the project plan and verify its internal links.

## Licensing — blocking

- [x] Apply CC BY-NC 4.0 explicitly to dataset files and project documentation through `LICENSE-DATA.md`.
- [x] Exclude source code, executable scripts, and executable notebook code from `LICENSE-DATA.md`.
- [x] Identify Maria Karolina Silva de Souza consistently as the dataset creator and record the selected CC BY-NC 4.0 terms.
- [ ] Choose and add a separate software license before distributing any generator, executable script, or notebook code.
- [ ] Add the actual canonical source to the recommended attribution once it exists.
- [ ] Check that no included third-party material is incorrectly presented as covered by the dataset license.

## Final local package audit

- [x] Keep `.codex-work/` excluded from version control.
- [x] Confirm that the audit's `source.file` path matches the canonical copy under `full_dataset/` and that its byte length and SHA-256 match.
- [x] Remove the byte-identical root-level staging copy of the XLSX before freezing the package; retain only `full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx`.
- [x] Recompute file hashes in the final post-correction audit.
- [x] Confirm that no temporary, credential, cache, model, or generated-output files are included in the final package.
- [x] Confirm that the CSV and XLSX data artifacts are not excluded by `.gitignore`.
- [x] Review the complete local package and freeze its exact contents in `FILE_MANIFEST.json`.
- [ ] Replace the pre-publication wording in `CHANGELOG.md` with an actual release date only when publication occurs.

## Online publication — deliberately not performed

- [ ] Create the public repository only after all blocking items above are complete and the user separately authorizes the online action.
- [ ] Verify rendered Markdown, links, downloads, license, and attribution on the hosting platform.
- [ ] Create release `v1.0.0` from the frozen local package.
- [ ] Archive the release in an appropriate data repository if desired.
- [ ] Add the issued DOI to `CITATION.cff`, both READMEs, and other citation metadata.
- [ ] Never silently replace files belonging to a published version; issue a new patch, minor, or major version as appropriate.
