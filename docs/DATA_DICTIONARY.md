---
title: "Dicionário de dados - Frota de robôs móveis em centro logístico"
author: "Maria Karolina Silva de Souza"
version: "1.0.0"
lang: "pt-BR"
---

# Dicionário de dados

**Autoria:** Maria Karolina Silva de Souza

## Fonte canônica

Este documento é uma referência navegável das 12 tabelas tabulares. A fonte completa, com uma linha por coluna e 14 atributos de metadados, é [`data/dictionary.csv`](../data/dictionary.csv). Ela contém 173 linhas e deve ser consultada para tipo lógico, unidade, papel analítico, descrição, domínio, política declarada de ausentes, momento de disponibilidade, origem, fórmula e permissões de uso temporal.

A auditoria final confirmou cobertura de **173 de 173 colunas**, sem entradas ausentes, extras ou duplicadas. Os tipos declarados coincidem com o conteúdo auditado (`declaredTypeMismatchTotal = 0`), e não restaram divergências na declaração de ausentes (`missingDeclarationMismatches = []`).

O resumo abaixo não substitui o CSV. Ele organiza granularidade, chave e nomes exatos de todas as colunas.

## Índice

- [MODELOS_ROBO](#modelos_robo)
- [ROBOS](#robos)
- [AMBIENTES](#ambientes)
- [ROTAS](#rotas)
- [HISTORICO_ROBOS](#historico_robos)
- [CONDICOES_AMBIENTAIS](#condicoes_ambientais)
- [MISSOES](#missoes)
- [MANUTENCOES](#manutencoes)
- [BASE_ML_MISSOES](#base_ml_missoes)
- [BASE_ML_MANUTENCAO](#base_ml_manutencao)
- [VALIDACOES](#validacoes)
- [DICIONARIO](#dicionario)
- [Histórico de correção dos tipos](#historico-de-correcao-dos-tipos)
- [Política de valores ausentes](#politica-de-valores-ausentes)

## Convenções globais

- Identificadores terminados em `_id`, versões e categorias são texto. Preserve zeros à esquerda; `RB008`, `RT044` e `S06` não são números.
- Datas são datetime sem fuso horário especificado. Não presuma UTC.
- Indicadores binários usam 0/1, com suporte a nulo quando a variável não se aplica.
- `_c` indica graus Celsius; `_pct`, percentual; `_pp`, pontos percentuais; `_m`, metros; `_min`, minutos; `_h`, horas; `_brl`, reais.
- Colunas `_alvo` são resultados posteriores ao instante de referência e não devem ser usadas como preditores do mesmo problema.
- O tipo lógico `numérico` identifica medidas que podem conter casas decimais. A auditoria final encontrou zero divergências entre tipos declarados e valores observados.
- `valores_validos` pode descrever um domínio permitido mais amplo do que as categorias observadas. Por exemplo, `tipo_manutencao` admite “preditiva” no domínio, mas a versão atual contém somente “preventiva” e “corretiva”.

## Resumo estrutural

| Tabela | Registros | Colunas | Granularidade | Chave |
| --- | ---: | ---: | --- | --- |
| `MODELOS_ROBO` | 4 | 6 | Um modelo de robô | `modelo_id` |
| `ROBOS` | 60 | 4 | Um robô físico sintético | `robo_id` |
| `AMBIENTES` | 14 | 5 | Um setor físico | `setor_id` |
| `ROTAS` | 182 | 9 | Um par direcional origem-destino | `rota_id` |
| `HISTORICO_ROBOS` | 12.144 | 10 | Um snapshot de um robô em um instante | `historico_id` |
| `CONDICOES_AMBIENTAIS` | 22.932 | 8 | Um snapshot ambiental de um setor em um instante | `condicao_id` |
| `MISSOES` | 10.000 | 28 | Uma execução de missão | `missao_id` |
| `MANUTENCOES` | 1.042 | 14 | Uma ordem de manutenção | `manutencao_id` |
| `BASE_ML_MISSOES` | 10.000 | 49 | Uma linha analítica por missão | `missao_id` |
| `BASE_ML_MANUTENCAO` | 4.680 | 22 | Um robô em uma segunda-feira | (`data_referencia`, `robo_id`) |
| `VALIDACOES` | 11 | 4 | Uma regra de integridade | Sem chave formal; `validacao` identifica logicamente a regra |
| `DICIONARIO` | 173 | 14 | Uma coluna documentada | (`tabela`, `coluna`) |

## MODELOS_ROBO

- Arquivo: [`data/raw/MODELOS_ROBO.csv`](../data/raw/MODELOS_ROBO.csv)
- Granularidade: uma linha por modelo de robô.
- Chave primária: `modelo_id`.
- Relação: um modelo pode identificar vários registros em `ROBOS`.
- Colunas: `modelo_id`, `nome_modelo`, `capacidade_carga_kg`, `tipo_bateria`, `velocidade_nominal_m_min`, `autonomia_nominal_h`.

Esta tabela descreve capacidades nominais de uma família, não o estado temporal de uma unidade.

## ROBOS

- Arquivo: [`data/raw/ROBOS.csv`](../data/raw/ROBOS.csv)
- Granularidade: uma linha por robô físico sintético.
- Chave primária: `robo_id`.
- Chaves estrangeiras: `modelo_id` -> `MODELOS_ROBO.modelo_id`; `setor_base` -> `AMBIENTES.setor_id`.
- Colunas: `robo_id`, `modelo_id`, `ano_fabricacao`, `setor_base`.

`setor_base` é o local habitual de estacionamento ou recarga; não é uma posição instantânea.

## AMBIENTES

- Arquivo: [`data/raw/AMBIENTES.csv`](../data/raw/AMBIENTES.csv)
- Granularidade: uma linha por setor físico do centro logístico.
- Chave primária: `setor_id`.
- Relações: é referenciada por `ROBOS.setor_base`, pelos dois extremos de `ROTAS` e por `CONDICOES_AMBIENTAIS.setor_id`.
- Colunas: `setor_id`, `nome_setor`, `tipo_piso`, `iluminacao_instalada`, `largura_corredor_m`.

## ROTAS

- Arquivo: [`data/raw/ROTAS.csv`](../data/raw/ROTAS.csv)
- Granularidade: uma linha por par direcional origem-destino.
- Chave primária: `rota_id`.
- Chaves estrangeiras: `setor_origem_id` e `setor_destino_id` -> `AMBIENTES.setor_id`.
- Colunas: `rota_id`, `setor_origem_id`, `setor_destino_id`, `distancia_nominal_m`, `dificuldade`, `quantidade_curvas`, `inclinacao_media_pct`, `largura_minima_m`, `tempo_nominal_min`.

Com 14 setores e sem autorrotas, os 182 registros cobrem `14 x 13` pares ordenados. A direção importa.

## HISTORICO_ROBOS

- Arquivo: [`data/raw/HISTORICO_ROBOS.csv`](../data/raw/HISTORICO_ROBOS.csv)
- Granularidade: um snapshot do estado de um robô em um instante.
- Chave primária: `historico_id`.
- Chave estrangeira: `robo_id` -> `ROBOS.robo_id`.
- Referência de evento: `evento_referencia_id` aponta para missão ou manutenção conforme `evento_origem`; é vazio nos snapshots de `inicio_periodo`.
- Colunas: `historico_id`, `robo_id`, `data_hora`, `horas_operacao_acumuladas`, `ciclos_bateria`, `versao_firmware`, `indice_confiabilidade`, `status_operacional`, `evento_origem`, `evento_referencia_id`.

Para obter o estado conhecido em uma data, selecione o snapshot do mesmo robô com a maior `data_hora` menor ou igual à referência.

## CONDICOES_AMBIENTAIS

- Arquivo: [`data/raw/CONDICOES_AMBIENTAIS.csv`](../data/raw/CONDICOES_AMBIENTAIS.csv)
- Granularidade: um snapshot ambiental por setor às 00:00, 08:00 ou 16:00.
- Chave primária: `condicao_id`.
- Chave estrangeira: `setor_id` -> `AMBIENTES.setor_id`.
- Colunas: `condicao_id`, `setor_id`, `data_hora`, `temperatura_c`, `umidade_pct`, `movimento_pessoas`, `qualidade_wifi_pct`, `obstaculos_temporarios`.

Em uma missão, o snapshot selecionado é o mais recente até o início e pertence ao setor de destino da rota.

## MISSOES

- Arquivo: [`data/raw/MISSOES.csv`](../data/raw/MISSOES.csv)
- Granularidade: uma linha por execução de missão.
- Chave primária: `missao_id`.
- Chaves estrangeiras: `robo_id` -> `ROBOS.robo_id`; `rota_id` -> `ROTAS.rota_id`; `condicao_inicio_id` -> `CONDICOES_AMBIENTAIS.condicao_id`.
- Colunas: `missao_id`, `robo_id`, `rota_id`, `condicao_inicio_id`, `data_hora_inicio`, `data_hora_fim`, `distancia_real_m`, `carga_kg`, `tempo_previsto_min`, `duracao_real_min`, `nivel_bateria_inicial_pct`, `nivel_bateria_final_pct`, `consumo_bateria_pp`, `quantidade_obstaculos_encontrados`, `qualidade_sinal_wifi_durante_pct`, `temperatura_robo_durante_c`, `evento_atipico`, `colisao`, `gravidade_colisao`, `colisao_detectada_em`, `dano_detectado`, `intervencao_necessaria`, `atraso_min`, `atraso`, `missao_concluida`, `status_missao`, `falha_detectada_em`, `motivo_interrupcao`.

A linha combina informação disponível no início, telemetria durante a execução e resultados finais. Para previsão, não trate todos os campos como simultaneamente disponíveis. `atraso` é o rótulo oficial; não invente um limiar para recriá-lo a partir de `atraso_min`.

## MANUTENCOES

- Arquivo: [`data/raw/MANUTENCOES.csv`](../data/raw/MANUTENCOES.csv)
- Granularidade: uma linha por ordem de manutenção.
- Chave primária: `manutencao_id`.
- Chaves estrangeiras: `robo_id` -> `ROBOS.robo_id`; `missao_relacionada_id` -> `MISSOES.missao_id` quando preenchida.
- Colunas: `manutencao_id`, `robo_id`, `missao_relacionada_id`, `falha_detectada_em`, `data_hora_inicio_manutencao`, `data_hora_fim_manutencao`, `tempo_parado_h`, `componente`, `tipo_manutencao`, `custo_brl`, `falha_encontrada`, `severidade_falha`, `equipe_responsavel`, `peca_substituida`.

Detecção da falha, início da manutenção e fim da manutenção são instantes distintos. O intervalo entre início e fim representa indisponibilidade. Ordens preventivas não possuem missão relacionada nem falha detectada.

## BASE_ML_MISSOES

- Arquivo: [`data/analytical/BASE_ML_MISSOES.csv`](../data/analytical/BASE_ML_MISSOES.csv)
- Granularidade: uma linha analítica por missão.
- Chave primária: `missao_id`.
- Relações verificadas: `missao_id` -> `MISSOES.missao_id`; `historico_robo_id` -> `HISTORICO_ROBOS.historico_id`; `condicao_inicio_id` -> `CONDICOES_AMBIENTAIS.condicao_id`.
- Identidade e tempo: `missao_id`, `data_hora_inicio`, `robo_id`, `modelo_id`, `nome_modelo`, `ano_fabricacao`, `idade_robo_anos`.
- Cadastro e estado: `capacidade_carga_kg`, `tipo_bateria`, `setor_base`, `historico_robo_id`, `horas_operacao_inicio_missao`, `ciclos_bateria_inicio_missao`, `firmware_no_momento`, `confiabilidade_no_momento`.
- Rota e planejamento: `rota_id`, `setor_origem_id`, `setor_destino_id`, `distancia_nominal_m`, `dificuldade_rota`, `quantidade_curvas`, `inclinacao_media_pct`, `largura_minima_m`, `tempo_nominal_min`, `carga_kg`, `carga_pct_capacidade`, `nivel_bateria_inicial_pct`.
- Ambiente: `condicao_inicio_id`, `condicao_observada_em`, `temperatura_ambiente_inicio_c`, `umidade_inicio_pct`, `movimento_pessoas_inicio`, `wifi_ambiente_inicio_pct`, `obstaculos_temporarios_inicio`.
- Calendário: `turno`, `mes`, `dia_semana`, `hora_inicio`.
- Alvos: `distancia_real_m_alvo`, `duracao_real_min_alvo`, `consumo_bateria_pp_alvo`, `colisao_alvo`, `gravidade_colisao_alvo`, `dano_detectado_alvo`, `intervencao_necessaria_alvo`, `atraso_alvo`, `atraso_min_alvo`, `missao_concluida_alvo`, `status_missao_alvo`.

Escolha um alvo por experimento e exclua todas as colunas `_alvo` das entradas. O snapshot de robô e o ambiental foram validados como não futuros.

## BASE_ML_MANUTENCAO

- Arquivo: [`data/analytical/BASE_ML_MANUTENCAO.csv`](../data/analytical/BASE_ML_MANUTENCAO.csv)
- Granularidade: uma linha por robô por segunda-feira.
- Chave composta: (`data_referencia`, `robo_id`).
- Chave estrangeira: `robo_id` -> `ROBOS.robo_id`.
- Estado na referência: `data_referencia`, `robo_id`, `modelo_id`, `nome_modelo`, `idade_robo_anos`, `horas_operacao_na_data`, `ciclos_bateria_na_data`, `confiabilidade_na_data`, `status_na_data`.
- Histórico retrospectivo: `dias_desde_ultima_manutencao`, `manutencoes_ultimos_30_dias`, `falhas_ultimos_90_dias`, `colisoes_ultimos_30_dias`, `missoes_ultimos_30_dias`, `consumo_medio_ultimos_30_dias_pp`, `temperatura_media_robo_ultimos_30_dias_c`, `duracao_media_ultimos_30_dias_min`, `distancia_total_ultimos_30_dias_m`.
- Alvos futuros: `falha_nos_proximos_7_dias_alvo`, `dias_ate_proxima_manutencao_alvo`, `tipo_proxima_manutencao_alvo`, `custo_proxima_manutencao_brl_alvo`.

As janelas históricas encerram antes de `data_referencia`. Alvos futuros podem ser censurados no fim do período por não existir próxima manutenção observável.

## VALIDACOES

- Localização: aba `VALIDACOES` do XLSX completo.
- Granularidade: uma linha por regra de integridade declarada.
- Chave: não há chave formal; `validacao` identifica logicamente a regra.
- Colunas: `validacao`, `resultado`, `ocorrencias`, `criterio`.

As 11 linhas estão marcadas como `APROVADO` e totalizam zero ocorrências. Essa tabela registra o resultado declarado no próprio artefato; a auditoria independente está em [VALIDATION.md](VALIDATION.md).

## DICIONARIO

- Arquivo: [`data/dictionary.csv`](../data/dictionary.csv)
- Granularidade: uma linha por coluna documentada.
- Chave lógica composta: (`tabela`, `coluna`).
- Colunas: `tabela`, `coluna`, `tipo`, `unidade`, `papel`, `descricao`, `valores_validos`, `ausentes`, `momento_disponivel`, `origem`, `formula_derivacao`, `pode_usar_pre_missao`, `pode_usar_online`, `pode_usar_pos_missao`.

Os três campos `pode_usar_*` descrevem disponibilidade analítica por momento. Eles são auxiliares de prevenção de leakage, mas não substituem a definição explícita do instante de predição de cada experimento.

<a id="historico-de-correcao-dos-tipos"></a>

## Histórico de correção dos tipos

A auditoria inicial encontrou 57.923 ocorrências com parte decimal em 12 colunas então declaradas como `inteiro`. Esse achado motivou a alteração dos 12 metadados para `numérico` no XLSX e em `data/dictionary.csv`; os valores dos dados não precisaram ser arredondados ou truncados.

| Tabela | Coluna | Ocorrências históricas | Tipo final |
| --- | --- | ---: | --- |
| `ROTAS` | `inclinacao_media_pct` | 150 | `numérico` |
| `ROTAS` | `tempo_nominal_min` | 173 | `numérico` |
| `MISSOES` | `carga_kg` | 8.987 | `numérico` |
| `MISSOES` | `tempo_previsto_min` | 8.976 | `numérico` |
| `MISSOES` | `nivel_bateria_inicial_pct` | 135 | `numérico` |
| `BASE_ML_MISSOES` | `horas_operacao_inicio_missao` | 8.977 | `numérico` |
| `BASE_ML_MISSOES` | `carga_kg` | 8.987 | `numérico` |
| `BASE_ML_MISSOES` | `carga_pct_capacidade` | 8.764 | `numérico` |
| `BASE_ML_MISSOES` | `nivel_bateria_inicial_pct` | 135 | `numérico` |
| `BASE_ML_MANUTENCAO` | `horas_operacao_na_data` | 4.207 | `numérico` |
| `BASE_ML_MANUTENCAO` | `temperatura_media_robo_ultimos_30_dias_c` | 4.219 | `numérico` |
| `BASE_ML_MANUTENCAO` | `duracao_media_ultimos_30_dias_min` | 4.213 | `numérico` |

O total histórico conta ocorrências por coluna, inclusive valores derivados ou copiados entre tabelas; não representa 57.923 observações independentes. A auditoria final retornou `declaredTypeMismatchTotal = 0` e `numericRangeViolationTotal = 0`.

<a id="politica-de-valores-ausentes"></a>

## Política de valores ausentes

A auditoria inicial encontrou 11 colunas cuja marcação de ausentes não descrevia vazios estruturalmente esperados. As marcações foram corrigidas com a razão específica:

| Tabela | Coluna | Ausentes observados | Razão registrada no dicionário final |
| --- | --- | ---: | --- |
| `HISTORICO_ROBOS` | `evento_referencia_id` | 60 | `sim, no snapshot inicial` |
| `BASE_ML_MISSOES` | `atraso_alvo` | 216 | `sim, quando a missão não é concluída` |
| `BASE_ML_MISSOES` | `atraso_min_alvo` | 216 | `sim, quando a missão não é concluída` |
| `BASE_ML_MANUTENCAO` | `dias_desde_ultima_manutencao` | 230 | `sim, sem manutenção anterior` |
| `BASE_ML_MANUTENCAO` | `consumo_medio_ultimos_30_dias_pp` | 13 | `sim, sem missão válida na janela` |
| `BASE_ML_MANUTENCAO` | `temperatura_media_robo_ultimos_30_dias_c` | 13 | `sim, sem missão válida na janela` |
| `BASE_ML_MANUTENCAO` | `duracao_media_ultimos_30_dias_min` | 13 | `sim, sem missão válida na janela` |
| `BASE_ML_MANUTENCAO` | `dias_ate_proxima_manutencao_alvo` | 192 | `sim, sem próxima manutenção observada` |
| `BASE_ML_MANUTENCAO` | `tipo_proxima_manutencao_alvo` | 192 | `sim, sem próxima manutenção observada` |
| `BASE_ML_MANUTENCAO` | `custo_proxima_manutencao_brl_alvo` | 192 | `sim, sem próxima manutenção observada` |
| `DICIONARIO` | `unidade` | 119 | `sim, quando a unidade não se aplica` |

Também há ausências documentadas em outras colunas: 137 em `MISSOES.qualidade_sinal_wifi_durante_pct`, 74 em `MISSOES.temperatura_robo_durante_c`, 9.507 em `MISSOES.colisao_detectada_em`, 9.784 em `MISSOES.falha_detectada_em`, 216 em `MISSOES.atraso` e `MISSOES.atraso_min`, e 759 em `MANUTENCOES.missao_relacionada_id` e `MANUTENCOES.falha_detectada_em`.

A auditoria final retornou `missingDeclarationMismatches = []`. Para tratamento analítico, use a razão registrada em cada coluna para distinguir sensor, não aplicabilidade, ausência de evento anterior e censura pelo fim da janela.
