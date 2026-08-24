---
title: "Dataset Card - Frota de robôs móveis em centro logístico"
author: "Maria Karolina Silva de Souza"
version: "1.0.0"
lang: "pt-BR"
---

# Dataset Card - Frota de robôs móveis em centro logístico

**Autoria:** Maria Karolina Silva de Souza

## Visão geral

Este pacote contém um dataset relacional e temporal sobre a operação de uma frota de robôs móveis em um centro logístico. Os dados são **100% sintéticos**: não foram coletados de uma empresa, instalação, pessoa ou equipamento real.

O artefato de origem é a segunda revisão interna do arquivo Excel (`dataset_frota_robos_centro_logistico_v2.xlsx`). Para distribuição, o pacote foi preparado como a primeira versão pública, **v1.0.0**. A diferença de numeração registra a linhagem do desenvolvimento: `v2` identifica a revisão interna da planilha; `v1.0.0` identifica a primeira versão pública preparada do conjunto.

## Finalidade

O dataset foi estruturado para ensino, estudo, experimentação e desenvolvimento de fluxos de Ciência de Dados, Estatística, Machine Learning e Engenharia de Dados. Ele permite explorar cadastros relacionais, snapshots temporais, missões, ordens de manutenção e duas bases analíticas prontas para tarefas supervisionadas.

O dataset não deve ser tratado como evidência empírica sobre uma operação industrial real. Resultados obtidos nele não demonstram desempenho, segurança, confiabilidade ou causalidade em robôs reais.

## Escopo temporal

- Período operacional sintético: janeiro de 2025 a junho de 2026.
- Missões: de `2025-01-01T04:05:46` a `2026-06-28T23:49:06` no campo de início.
- Condições ambientais: de `2025-01-01T00:00:00` a `2026-06-30T16:00:00`.
- Base semanal de manutenção: 78 segundas-feiras, de `2025-01-06T00:00:00` a `2026-06-29T00:00:00`, para 60 robôs.
- Os valores de data e hora não possuem fuso horário especificado. Devem ser interpretados como horários locais sintéticos e não como instantes UTC.

## Composição

| Camada | Tabela | Registros | Unidade de uma linha |
| --- | --- | ---: | --- |
| Cadastro | `MODELOS_ROBO` | 4 | Um modelo de robô |
| Cadastro | `ROBOS` | 60 | Um robô físico sintético |
| Cadastro | `AMBIENTES` | 14 | Um setor do centro logístico |
| Cadastro | `ROTAS` | 182 | Um par direcional origem-destino |
| Temporal | `HISTORICO_ROBOS` | 12.144 | Um snapshot de um robô em um instante |
| Temporal | `CONDICOES_AMBIENTAIS` | 22.932 | Um snapshot ambiental de um setor em um instante |
| Operacional | `MISSOES` | 10.000 | Uma execução de missão |
| Operacional | `MANUTENCOES` | 1.042 | Uma ordem de manutenção |
| Analítica | `BASE_ML_MISSOES` | 10.000 | Uma linha analítica por missão |
| Analítica | `BASE_ML_MANUTENCAO` | 4.680 | Um robô em uma segunda-feira |
| Governança | `VALIDACOES` | 11 | Uma regra de integridade declarada |
| Governança | `DICIONARIO` | 173 | Uma coluna documentada |

As oito tabelas cadastrais, temporais e operacionais totalizam 46.378 registros. As bases analíticas são derivadas e, por isso, não entram nesse total.

## Modelo de dados

Os principais vínculos são:

- `MODELOS_ROBO.modelo_id` -> `ROBOS.modelo_id`;
- `AMBIENTES.setor_id` -> `ROBOS.setor_base`;
- `AMBIENTES.setor_id` -> `ROTAS.setor_origem_id` e `ROTAS.setor_destino_id`;
- `AMBIENTES.setor_id` -> `CONDICOES_AMBIENTAIS.setor_id`;
- `ROBOS.robo_id` -> histórico, missões, manutenções e base semanal;
- `MISSOES` referencia um robô, uma rota e um snapshot ambiental;
- `MANUTENCOES.missao_relacionada_id` é opcional e referencia uma missão quando a ordem corretiva nasce de um evento operacional.

Rotas são direcionais: origem-destino e destino-origem são registros diferentes. `setor_base` é um atributo cadastral habitual, não a posição instantânea do robô.

## Construção temporal e prevenção de leakage

Um snapshot representa o estado conhecido em um instante. A regra lógica é selecionar, para a mesma entidade, o registro de maior data e hora que não ultrapasse o instante de referência.

Na `BASE_ML_MISSOES`, o estado do robô e a condição ambiental não são posteriores ao início da missão. A condição ambiental pertence ao setor de destino da rota; a expressão “início” em `condicao_inicio_id` se refere ao momento da seleção do snapshot, não ao setor de origem.

Na `BASE_ML_MANUTENCAO`, os preditores de histórico usam somente eventos anteriores à `data_referencia`. Campos terminados em `_alvo` representam informação futura e não devem ser usados como entradas do mesmo problema preditivo.

## Valores ausentes

Há ausências intencionais e ausências estruturais:

- falhas simuladas de sensores durante missões: 137 valores ausentes em `qualidade_sinal_wifi_durante_pct` e 74 em `temperatura_robo_durante_c`;
- campos aplicáveis somente quando há um evento, como instante de colisão ou falha;
- atraso e atraso em minutos não se aplicam às 216 missões interrompidas;
- manutenções preventivas não possuem missão relacionada nem instante de detecção de falha;
- linhas anteriores à primeira manutenção não possuem `dias_desde_ultima_manutencao`;
- médias retrospectivas podem ser ausentes quando a janela não contém observação utilizável;
- alvos sobre a próxima manutenção são ausentes no fim da série quando não existe evento futuro observável.

O dicionário final registra, por coluna, se a ausência é permitida e a razão correspondente. A auditoria final não encontrou divergências entre essas declarações e os valores observados.

## Convenções de importação

- IDs, versões de firmware e categorias devem ser importados como texto. Não remova zeros à esquerda de códigos como `RB008`, `RT044` ou `S06`.
- Datas devem ser importadas como datetime sem fuso presumido.
- Indicadores 0/1 podem ser tratados como inteiros ou booleanos com suporte a nulo.
- Medidas físicas, temporais, percentuais e monetárias declaradas como `numérico` devem ser importadas com suporte a casas decimais. O dicionário final classifica corretamente as 12 colunas fracionárias identificadas durante a preparação.
- Sufixo `_pct` indica percentual; `_pp` indica pontos percentuais; `_alvo` indica resultado a prever.

## Usos recomendados

- ensino de limpeza, junções, agregações e modelagem relacional;
- análise exploratória e visualização, com a ressalva de que os padrões são artificiais;
- classificação, regressão, detecção de anomalias e agrupamento;
- manutenção preditiva e avaliação de classes desbalanceadas;
- estudos de valores ausentes e imputação;
- exercícios de validação estrutural, temporal e de integridade referencial.

Consulte [USE_CASES.md](USE_CASES.md) para exemplos e restrições de uso.

## Usos não recomendados

- inferir desempenho de uma empresa ou frota real;
- justificar decisões operacionais, de segurança ou manutenção em equipamentos reais;
- fazer afirmações causais sobre robôs industriais;
- usar os resultados como benchmark representativo de um setor industrial;
- treinar um sistema destinado à produção sem validação em dados reais adequados;
- usar variáveis posteriores ao evento como preditores de uma decisão anterior.

## Geração e reprodutibilidade

A semente registrada é `20260730`, mas **uma semente não reproduz um dataset sem o algoritmo, as dependências e as regras de geração correspondentes**. O algoritmo original não está presente no pacote. Assim, esta versão deve ser tratada como um artefato congelado e verificável, não como uma geração integralmente reproduzível.

Os CSVs distribuídos passaram por verificação de ida e volta contra as planilhas de origem. Detalhes e limites estão em [GENERATION_METHODOLOGY.md](GENERATION_METHODOLOGY.md) e [VALIDATION.md](VALIDATION.md).

## Validação

A planilha declara 11 regras aprovadas, todas com zero ocorrências. Uma auditoria independente executou 12 verificações estruturais e temporais, também com zero violações. O dicionário cobre 173 de 173 colunas, sem entradas ausentes, extras ou duplicadas. A auditoria final registrou `declaredTypeMismatchTotal = 0`, `missingDeclarationMismatches = []` e `numericRangeViolationTotal = 0`.

Na auditoria inicial, 57.923 ocorrências fracionárias em 12 colunas e 11 declarações incorretas de ausentes motivaram correções no XLSX e no CSV do dicionário. Esses achados estão resolvidos na versão final. As aprovações se limitam às propriedades testadas; não constituem certificação de qualidade estatística, privacidade ou adequação a um uso real.

A varredura de possíveis dados pessoais foi heurística, baseada em nomes de cabeçalhos e expressões regulares para e-mail, URL, CPF/CNPJ formatado e padrões semelhantes a telefone. Não houve correspondências, mas o procedimento não é uma certificação formal de privacidade.

## Documentação relacionada

- [Dicionário resumido e convenções](DATA_DICTIONARY.md)
- [Dicionário completo em CSV](../data/dictionary.csv)
- [Metodologia de geração](GENERATION_METHODOLOGY.md)
- [Relatório de validação](VALIDATION.md)
- [Limitações](LIMITATIONS.md)
- [Casos de uso](USE_CASES.md)

## Licença

Os dados são disponibilizados sob Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0). Consulte o arquivo de licença do pacote para condições e escopo exatos.
