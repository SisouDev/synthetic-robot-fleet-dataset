---
title: "Relatório de validação - Frota de robôs móveis em centro logístico"
author: "Maria Karolina Silva de Souza"
version: "1.0.0"
lang: "pt-BR"
---

# Relatório de validação

**Autoria:** Maria Karolina Silva de Souza

## Resultado executivo

O artefato auditado abriu corretamente e apresentou as 13 abas esperadas, sendo uma aba explicativa (`LEIA_ME`) e 12 tabelas. As **11 validações declaradas** na planilha estão aprovadas, com zero ocorrências. Uma auditoria separada executou **12 verificações independentes**, todas aprovadas, também com zero violações.

Todos os 11 CSVs distribuídos passaram por round-trip contra suas planilhas de origem. Não foram encontrados cabeçalhos vazios, cabeçalhos duplicados ou linhas totalmente duplicadas nas 12 tabelas auditadas.

O dicionário documenta 173 de 173 colunas, sem lacunas, extras ou duplicatas. Depois das correções de metadados, a auditoria final registrou `declaredTypeMismatchTotal = 0`, `missingDeclarationMismatches = []` e `numericRangeViolationTotal = 0`. Os CSVs preservam integralmente os valores fracionários.

Esses resultados demonstram as propriedades estruturais e temporais descritas neste relatório. Não são uma certificação de qualidade estatística, privacidade, representatividade industrial ou adequação de modelos.

## Artefato e execução

- Auditoria final gerada em: `2026-08-24T13:27:23.837Z` (UTC).
- Arquivo de origem: `full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx`.
- Tamanho: 6.136.028 bytes.
- SHA-256: `aa31190ea25bbde9313d00b7e76f4c58d7e19e0a6e58a3d60f9bd50c11ffe79b`.
- Afirmação de dados sintéticos: presente no artefato.
- Busca por erros de fórmula: zero correspondências.

Abas encontradas: `LEIA_ME`, `MODELOS_ROBO`, `ROBOS`, `HISTORICO_ROBOS`, `AMBIENTES`, `CONDICOES_AMBIENTAIS`, `ROTAS`, `MISSOES`, `MANUTENCOES`, `BASE_ML_MISSOES`, `BASE_ML_MANUTENCAO`, `VALIDACOES` e `DICIONARIO`.

## Perfil estrutural

| Tabela | Registros | Colunas | Cabeçalhos vazios | Cabeçalhos duplicados | Linhas duplicadas |
| --- | ---: | ---: | ---: | ---: | ---: |
| `MODELOS_ROBO` | 4 | 6 | 0 | 0 | 0 |
| `ROBOS` | 60 | 4 | 0 | 0 | 0 |
| `HISTORICO_ROBOS` | 12.144 | 10 | 0 | 0 | 0 |
| `AMBIENTES` | 14 | 5 | 0 | 0 | 0 |
| `CONDICOES_AMBIENTAIS` | 22.932 | 8 | 0 | 0 | 0 |
| `ROTAS` | 182 | 9 | 0 | 0 | 0 |
| `MISSOES` | 10.000 | 28 | 0 | 0 | 0 |
| `MANUTENCOES` | 1.042 | 14 | 0 | 0 | 0 |
| `BASE_ML_MISSOES` | 10.000 | 49 | 0 | 0 | 0 |
| `BASE_ML_MANUTENCAO` | 4.680 | 22 | 0 | 0 | 0 |
| `VALIDACOES` | 11 | 4 | 0 | 0 | 0 |
| `DICIONARIO` | 173 | 14 | 0 | 0 | 0 |

## Cobertura do dicionário, tipos e faixas numéricas

### Cobertura

- Colunas esperadas: 173.
- Linhas documentadas: 173.
- Entradas duplicadas: 0.
- Entradas ausentes: 0.
- Entradas extras: 0.
- Resultado de cobertura: aprovado.

### Histórico de correção dos tipos

A auditoria inicial encontrou 57.923 valores decimais em 12 colunas então declaradas como `inteiro`. O XLSX e `data/dictionary.csv` foram corrigidos para declarar essas colunas como `numérico`:

| Tabela | Coluna | Valores decimais |
| --- | --- | ---: |
| `ROTAS` | `inclinacao_media_pct` | 150 |
| `ROTAS` | `tempo_nominal_min` | 173 |
| `MISSOES` | `carga_kg` | 8.987 |
| `MISSOES` | `tempo_previsto_min` | 8.976 |
| `MISSOES` | `nivel_bateria_inicial_pct` | 135 |
| `BASE_ML_MISSOES` | `horas_operacao_inicio_missao` | 8.977 |
| `BASE_ML_MISSOES` | `carga_kg` | 8.987 |
| `BASE_ML_MISSOES` | `carga_pct_capacidade` | 8.764 |
| `BASE_ML_MISSOES` | `nivel_bateria_inicial_pct` | 135 |
| `BASE_ML_MANUTENCAO` | `horas_operacao_na_data` | 4.207 |
| `BASE_ML_MANUTENCAO` | `temperatura_media_robo_ultimos_30_dias_c` | 4.219 |
| `BASE_ML_MANUTENCAO` | `duracao_media_ultimos_30_dias_min` | 4.213 |
| **Total** | **12 colunas** | **57.923** |

O total histórico inclui valores derivados ou copiados em mais de uma tabela e não representa observações independentes. Os dados já continham e continuam contendo os decimais; a correção foi feita nos metadados. A auditoria final retornou `declaredTypeMismatchTotal = 0`.

### Faixas numéricas

Foram aplicadas 30 regras de mínimo e/ou máximo a medidas, percentuais, contagens, anos e valores monetários. Todas passaram, com `numericRangeViolationTotal = 0`.

## Validações declaradas no XLSX

A aba `VALIDACOES` contém 11 linhas. Todas têm `resultado = APROVADO`, e a soma de `ocorrencias` é zero.

| # | Regra declarada | Critério resumido | Ocorrências |
| ---: | --- | --- | ---: |
| 1 | Chaves primárias únicas | Nenhum identificador duplicado | 0 |
| 2 | Chaves estrangeiras das missões | Robô, rota e condição referenciados devem existir | 0 |
| 3 | Missões simultâneas do mesmo robô | A próxima missão começa no ou após o fim da anterior | 0 |
| 4 | Missão durante manutenção | Missão e manutenção do mesmo robô não se cruzam | 0 |
| 5 | Manutenções simultâneas do mesmo robô | Ordens do mesmo robô não se cruzam | 0 |
| 6 | Carga dentro da capacidade | `carga_kg` não excede a capacidade do modelo | 0 |
| 7 | Condição ambiental sem futuro | `condicao_observada_em <= data_hora_inicio` | 0 |
| 8 | Estado do robô sem futuro | Snapshot histórico não ocorre após o início da missão | 0 |
| 9 | Coerência da gravidade da colisão | Sem colisão corresponde a “sem colisão”; evento grave interrompe | 0 |
| 10 | Uma linha de ML por missão | `BASE_ML_MISSOES` e `MISSOES` têm correspondência 1:1 | 0 |
| 11 | Cobertura da base semanal | 4.680 linhas, uma por robô e semana | 0 |

Essas linhas são declarações armazenadas no próprio dataset. Por isso, foram conferidas novamente de forma independente.

## Verificações independentes

| # | Verificação executada | Violações | Resultado |
| ---: | --- | ---: | --- |
| 1 | Chaves primárias únicas e não vazias | 0 | Aprovado |
| 2 | Chaves estrangeiras declaradas resolvem | 0 | Aprovado |
| 3 | Não há missões sobrepostas para o mesmo robô | 0 | Aprovado |
| 4 | Não há missão sobreposta a manutenção do mesmo robô | 0 | Aprovado |
| 5 | Não há ordens de manutenção sobrepostas para o mesmo robô | 0 | Aprovado |
| 6 | Carga da missão não excede a capacidade do modelo | 0 | Aprovado |
| 7 | Snapshot ambiental não é posterior ao início da missão | 0 | Aprovado |
| 8 | Snapshot ambiental pertence ao setor de destino da rota | 0 | Aprovado |
| 9 | Snapshot do robô não é posterior ao início da missão | 0 | Aprovado |
| 10 | Campos de colisão e interrupção grave são coerentes | 0 | Aprovado |
| 11 | Existe exatamente uma linha analítica por missão | 0 | Aprovado |
| 12 | Base semanal cobre 78 semanas x 60 robôs | 0 | Aprovado |

A verificação 8 amplia as 11 regras declaradas ao confirmar também a associação do snapshot ambiental ao setor de destino da rota.

## Chaves primárias e estrangeiras

As chaves primárias de `MODELOS_ROBO`, `ROBOS`, `HISTORICO_ROBOS`, `AMBIENTES`, `CONDICOES_AMBIENTAIS`, `ROTAS`, `MISSOES` e `MANUTENCOES` foram testadas quanto a vazio e duplicação; todas tiveram zero em ambos.

Foram verificadas 15 relações de chave estrangeira, todas sem referências órfãs:

- `ROBOS.modelo_id` -> `MODELOS_ROBO.modelo_id`;
- `ROBOS.setor_base` -> `AMBIENTES.setor_id`;
- `HISTORICO_ROBOS.robo_id` -> `ROBOS.robo_id`;
- `CONDICOES_AMBIENTAIS.setor_id` -> `AMBIENTES.setor_id`;
- `ROTAS.setor_origem_id` e `ROTAS.setor_destino_id` -> `AMBIENTES.setor_id`;
- `MISSOES.robo_id` -> `ROBOS.robo_id`;
- `MISSOES.rota_id` -> `ROTAS.rota_id`;
- `MISSOES.condicao_inicio_id` -> `CONDICOES_AMBIENTAIS.condicao_id`;
- `MANUTENCOES.robo_id` -> `ROBOS.robo_id`;
- `MANUTENCOES.missao_relacionada_id` -> `MISSOES.missao_id`, permitindo nulo;
- `BASE_ML_MISSOES.missao_id` -> `MISSOES.missao_id`;
- `BASE_ML_MISSOES.historico_robo_id` -> `HISTORICO_ROBOS.historico_id`;
- `BASE_ML_MISSOES.condicao_inicio_id` -> `CONDICOES_AMBIENTAIS.condicao_id`;
- `BASE_ML_MANUTENCAO.robo_id` -> `ROBOS.robo_id`.

Os 759 vazios em `MANUTENCOES.missao_relacionada_id` são aceitos pela relação opcional e correspondem a ordens sem missão associada.

## CSV round-trip

Os CSVs foram exportados como UTF-8 sem BOM, delimitador vírgula e fim de linha LF. Cada arquivo foi lido novamente e comparado à respectiva planilha após normalização tabular de tipos e valores. Todos apresentaram equivalência de conteúdo.

| Arquivo | Registros | Colunas | Round-trip |
| --- | ---: | ---: | --- |
| `data/raw/MODELOS_ROBO.csv` | 4 | 6 | Equivalente |
| `data/raw/ROBOS.csv` | 60 | 4 | Equivalente |
| `data/raw/AMBIENTES.csv` | 14 | 5 | Equivalente |
| `data/raw/ROTAS.csv` | 182 | 9 | Equivalente |
| `data/raw/CONDICOES_AMBIENTAIS.csv` | 22.932 | 8 | Equivalente |
| `data/raw/HISTORICO_ROBOS.csv` | 12.144 | 10 | Equivalente |
| `data/raw/MISSOES.csv` | 10.000 | 28 | Equivalente |
| `data/raw/MANUTENCOES.csv` | 1.042 | 14 | Equivalente |
| `data/analytical/BASE_ML_MISSOES.csv` | 10.000 | 49 | Equivalente |
| `data/analytical/BASE_ML_MANUTENCAO.csv` | 4.680 | 22 | Equivalente |
| `data/dictionary.csv` | 173 | 14 | Equivalente |

Round-trip valida o conteúdo tabular exportado; não pretende preservar formatação visual, largura de colunas, fórmulas ou demais recursos específicos do XLSX.

## Faixas temporais auditadas

| Tabela e coluna | Mínimo | Máximo |
| --- | --- | --- |
| `MISSOES.data_hora_inicio` | `2025-01-01T04:05:46` | `2026-06-28T23:49:06` |
| `MISSOES.data_hora_fim` | `2025-01-01T04:14:04` | `2026-06-28T23:56:18` |
| `CONDICOES_AMBIENTAIS.data_hora` | `2025-01-01T00:00:00` | `2026-06-30T16:00:00` |
| `HISTORICO_ROBOS.data_hora` | `2025-01-01T00:00:00` | `2026-06-28T23:49:06` |
| `MANUTENCOES.data_hora_inicio_manutencao` | `2025-01-01T15:47:07` | `2026-06-28T17:48:33` |
| `MANUTENCOES.data_hora_fim_manutencao` | `2025-01-01T19:32:40` | `2026-06-28T21:07:42` |
| `BASE_ML_MANUTENCAO.data_referencia` | `2025-01-06T00:00:00` | `2026-06-29T00:00:00` |

Nenhuma dessas datas contém offset ou identificador de fuso. A auditoria não atribuiu UTC aos valores; o `Z` aparece somente no horário de execução do relatório.

## Valores ausentes e metadados

Os vazios observados se dividem em sensor, não aplicabilidade e censura temporal. As contagens por coluna e a razão final registrada estão em [DATA_DICTIONARY.md](DATA_DICTIONARY.md#politica-de-valores-ausentes).

A auditoria inicial encontrou 11 declarações que não descreviam vazios estruturalmente esperados. As marcações foram atualizadas no XLSX e em `data/dictionary.csv` com razões específicas, incluindo snapshot inicial, missão não concluída, ausência de manutenção anterior, janela sem missão válida, ausência de próxima manutenção e unidade não aplicável.

A auditoria final retornou `missingDeclarationMismatches = []`; não há divergência pendente na declaração de ausentes.

## Varredura heurística de PII

A auditoria examinou nomes de cabeçalhos e aplicou expressões regulares aos dados distribuídos para procurar:

- endereços de e-mail;
- URLs;
- CPF/CNPJ formatado;
- sequências semelhantes a números de telefone.

Não houve cabeçalhos suspeitos nem correspondências de padrões. O resultado é compatível com a afirmação de que os dados são 100% sintéticos. Entretanto, a varredura é heurística e **não constitui certificação formal de privacidade**.

## Limites da auditoria

Não foram validados por este procedimento:

- representatividade de distribuições reais;
- plausibilidade de toda correlação ou dependência estatística;
- desempenho mínimo de qualquer modelo;
- ausência de todo viés possível;
- validade causal das relações;
- segurança operacional para uso em equipamentos reais;
- regeneração integral a partir da seed;
- certificação jurídica ou formal de privacidade.

Consulte [LIMITATIONS.md](LIMITATIONS.md) antes de interpretar resultados analíticos.
