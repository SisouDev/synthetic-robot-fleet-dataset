# Dataset sintético de frota de robôs móveis para operações logísticas

[English](README.md)

> **Situação de publicação:** pacote local em preparação para a versão pública proposta **v1.0.0**. Ele deriva da segunda revisão interna do dataset e **ainda não foi publicado nem lançado**.

**Criadora:** Maria Karolina Silva de Souza

Este é um dataset totalmente sintético, relacional e temporal que representa uma frota de robôs móveis em operação em um centro logístico fictício. Foi elaborado para estudos de Ciência de Dados, Estatística, Machine Learning e Engenharia de Dados.

Nenhuma linha descreve uma empresa, instalação, robô, funcionário, cliente ou evento operacional real. O horizonte operacional sintético vai de janeiro de 2025 a junho de 2026. Os inícios das missões vão de `2025-01-01 04:05:46` a `2026-06-28 23:49:06`; os snapshots ambientais chegam a `2026-06-30 16:00:00`.

A semente registrada durante o desenvolvimento é `20260730`. Uma semente, sozinha, não é suficiente para reproduzir um dataset, e nenhum script de geração integra o pacote atual. Os arquivos distribuídos devem ser tratados como artefatos congelados, salvo se uma implementação revisada do gerador for adicionada posteriormente.

## Conteúdo do pacote

As oito tabelas relacionais brutas possuem **46.378 registros** ao todo. As bases analíticas possuem mais **14.680 registros**, e o dicionário contém **173 registros de metadados**. As linhas analíticas derivam das entidades e eventos brutos; portanto, essas quantidades não devem ser somadas como se todas representassem observações independentes.

### Tabelas relacionais brutas

| Tabela | Registros | Colunas | Finalidade |
| --- | ---: | ---: | --- |
| [`MODELOS_ROBO`](data/raw/MODELOS_ROBO.csv) | 4 | 6 | Especificações dos modelos de robô |
| [`ROBOS`](data/raw/ROBOS.csv) | 60 | 4 | Cadastro dos robôs físicos |
| [`AMBIENTES`](data/raw/AMBIENTES.csv) | 14 | 5 | Setores do centro logístico |
| [`ROTAS`](data/raw/ROTAS.csv) | 182 | 9 | Rotas direcionais entre setores |
| [`CONDICOES_AMBIENTAIS`](data/raw/CONDICOES_AMBIENTAIS.csv) | 22.932 | 8 | Snapshots das condições ambientais |
| [`HISTORICO_ROBOS`](data/raw/HISTORICO_ROBOS.csv) | 12.144 | 10 | Snapshots temporais do estado dos robôs |
| [`MISSOES`](data/raw/MISSOES.csv) | 10.000 | 28 | Execuções e resultados de missões |
| [`MANUTENCOES`](data/raw/MANUTENCOES.csv) | 1.042 | 14 | Ordens de manutenção e indisponibilidade |

### Bases prontas para análise e metadados

| Arquivo | Registros | Colunas | Unidade de análise |
| --- | ---: | ---: | --- |
| [`BASE_ML_MISSOES.csv`](data/analytical/BASE_ML_MISSOES.csv) | 10.000 | 49 | Uma linha por missão |
| [`BASE_ML_MANUTENCAO.csv`](data/analytical/BASE_ML_MANUTENCAO.csv) | 4.680 | 22 | Um robô em uma data semanal de referência; 78 semanas × 60 robôs |
| [`dictionary.csv`](data/dictionary.csv) | 173 | 14 | Um registro de metadados por variável documentada |

O arquivo completo [`dataset_frota_robos_centro_logistico_v2.xlsx`](full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx) possui 13 planilhas: `LEIA_ME`, as oito tabelas brutas, as duas bases analíticas, `VALIDACOES` e `DICIONARIO`.

## Estrutura relacional e temporal

`MODELOS_ROBO` descreve os modelos; `ROBOS` instancia esses modelos e atribui cada robô a um setor-base. `AMBIENTES` descreve os setores, enquanto `ROTAS` conecta setores de origem e destino. `CONDICOES_AMBIENTAIS` registra snapshots dos setores. `MISSOES` relaciona um robô, uma rota e uma condição ambiental. `HISTORICO_ROBOS` registra o estado dos robôs ao longo do tempo, e `MANUTENCOES` registra indisponibilidade e intervenções.

A base analítica de missões associa cada missão a atributos do robô, da rota, do ambiente e do histórico. A base semanal de manutenção agrega somente o histórico disponível em cada data de referência e adiciona alvos futuros de predição.

## Como carregar os arquivos

Todos os CSVs distribuídos usam vírgula como separador, UTF-8 sem marca de ordem de bytes e terminações de linha LF. Os nomes das colunas estão em português.

### CSV com Python e pandas

```python
from pathlib import Path

import pandas as pd

raiz = Path(".")
missoes = pd.read_csv(
    raiz / "data" / "raw" / "MISSOES.csv",
    encoding="utf-8",
    parse_dates=["data_hora_inicio", "data_hora_fim"],
)

base_ml_missoes = pd.read_csv(
    raiz / "data" / "analytical" / "BASE_ML_MISSOES.csv",
    encoding="utf-8",
    parse_dates=["data_hora_inicio", "condicao_observada_em"],
)
```

### XLSX com Python e pandas

A leitura de arquivos `.xlsx` pelo pandas normalmente exige um mecanismo para Excel, como `openpyxl`.

```python
from pathlib import Path

import pandas as pd

arquivo = Path("full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx")
missoes = pd.read_excel(
    arquivo,
    sheet_name="MISSOES",
    parse_dates=["data_hora_inicio", "data_hora_fim"],
)

nomes_das_planilhas = pd.ExcelFile(arquivo).sheet_names
```

### Normalização de timestamps

Os CSVs finais armazenam todos os 493 valores não vazios de `MISSOES.colisao_detectada_em` e todos os 216 valores não vazios de `MISSOES.falha_detectada_em` como timestamps ISO 8601, de acordo com a representação de datas documentada.

Consulte [`dictionary.csv`](data/dictionary.csv) antes de converter tipos ou selecionar variáveis. Ele documenta unidades, papéis analíticos, momento de disponibilidade, valores válidos, derivações e possibilidade de uso antes, durante ou depois de uma missão.

## Como evitar data leakage

O instante da predição importa. Uma variável válida para análise posterior pode revelar a resposta em um modelo que deveria executar antes do início da missão.

- Em `BASE_ML_MISSOES`, as colunas terminadas em `_alvo` são resultados ou alvos e não devem ser usadas como preditores desses mesmos resultados.
- Variáveis observadas durante ou depois da missão não são válidas para um modelo pré-missão. Consulte `momento_disponivel`, `pode_usar_pre_missao`, `pode_usar_online` e `pode_usar_pos_missao` no dicionário.
- Em `BASE_ML_MANUTENCAO`, as janelas históricas são ancoradas em `data_referencia`; colunas sobre a próxima manutenção ou os próximos sete dias são alvos futuros.
- A auditoria local verificou que os snapshots ambientais e dos robôs associados às missões não são posteriores ao início das missões. Isso protege as junções fornecidas, mas não torna toda coluna válida para todo cenário preditivo.
- Em avaliações de desempenho futuro, faça divisões cronológicas. Uma divisão aleatória pode misturar períodos operacionais posteriores no treinamento.

## Valores ausentes

A ausência de valores faz parte do projeto do dataset e não deve ser tratada automaticamente como erro.

- `MISSOES.qualidade_sinal_wifi_durante_pct` possui 137 observações ausentes, e `MISSOES.temperatura_robo_durante_c` possui 74. Elas representam ausência de sensor introduzida no projeto.
- Campos de eventos condicionais ficam vazios quando o evento não se aplica: 9.507 missões não possuem instante de colisão, 9.784 não possuem instante de falha, e as 216 missões interrompidas não possuem medição de atraso.
- As 759 manutenções preventivas não têm missão relacionada, enquanto as 283 manutenções corretivas estão associadas a uma missão.
- Em `BASE_ML_MANUTENCAO`, agregações históricas podem estar ausentes antes de existir histórico suficiente, e 192 linhas não possuem alvo de próxima manutenção observado dentro do horizonte do dataset.

A auditoria inicial de pré-publicação encontrou **57.923 ocorrências de incompatibilidade de tipo em 12 colunas** cujos valores decimais estavam descritos como inteiros. Também encontrou **11 divergências** entre os valores vazios observados e as declarações de `ausentes`. Esses problemas de metadados foram corrigidos na planilha `DICIONARIO` do XLSX e em [`data/dictionary.csv`](data/dictionary.csv): as 12 colunas com decimais agora estão declaradas como `numérico`, e as 11 declarações de ausentes agora registram o motivo aplicável. Esses achados não estão mais pendentes.

## Situação da validação

Uma auditoria local pré-publicação identificou:

- as 11 validações internas do XLSX com resultado `APROVADO` e zero ocorrências informadas;
- zero violações em 12 regras estruturais e temporais verificadas de forma independente;
- chaves primárias únicas e não vazias nas oito tabelas brutas;
- todas as chaves estrangeiras declaradas resolvidas, inclusive a relação opcional entre manutenção e missão;
- ausência de sobreposição de missões ou manutenções do mesmo robô e de sobreposição entre missão e manutenção;
- ausência de linhas duplicadas e de cabeçalhos vazios ou duplicados nas tabelas auditadas;
- equivalência de ida e volta entre cada CSV distribuído e sua planilha correspondente no XLSX;
- cobertura completa do dicionário para as 173 combinações esperadas de tabela e coluna (`173/173`), sem entradas ausentes ou extras;
- zero incompatibilidades entre os tipos declarados e os valores após as 12 correções de metadados;
- nenhuma divergência restante entre valores vazios observados e declarações de ausentes;
- zero violações nos intervalos numéricos verificados; e
- nenhum cabeçalho suspeito ou correspondência em uma busca heurística por e-mails, URLs, CPF/CNPJ formatados e sequências semelhantes a telefones.

Essas verificações cobrem apenas as regras definidas. Elas não comprovam toda propriedade estatística possível, não são uma certificação formal de privacidade e não demonstram que os dados representam operações reais. Os requisitos ainda pendentes estão na [`PRE_PUBLICATION_CHECKLIST`](PRE_PUBLICATION_CHECKLIST.md).

## Usos adequados

Exemplos incluem análise exploratória, junções relacionais, visualização, tratamento de valores ausentes, agregação temporal, classificação, regressão, detecção de anomalias, clusterização, exercícios de manutenção preditiva, engenharia de atributos e demonstrações de modelagem com controle de data leakage.

## Limitações e usos inadequados

- Os dados foram gerados, não medidos em equipamentos industriais.
- Relações entre variáveis e eventos raros foram estabelecidos pelo processo de geração e podem não corresponder a nenhuma operação específica.
- O desempenho de um modelo nesta base não implica desempenho em uma instalação real.
- O dataset não deve ser utilizado como evidência empírica sobre uma empresa real nem para sustentar conclusões causais sobre robôs móveis reais.
- Decisões críticas de segurança, operação, finanças ou aquisição exigem dados reais adequados e validação de domínio.
- Buscas heurísticas de privacidade e consistência não substituem revisão especializada.
- A semente registrada não torna a geração reproduzível sem o algoritmo e suas dependências.

## Arquivos e situação do projeto

- [`data/raw/`](data/raw/) — oito tabelas normalizadas de origem.
- [`data/analytical/`](data/analytical/) — duas bases prontas para análise.
- [`data/dictionary.csv`](data/dictionary.csv) — metadados por variável.
- [`full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx`](full_dataset/dataset_frota_robos_centro_logistico_v2.xlsx) — arquivo completo canônico da revisão interna v2 utilizado no pacote público candidato v1.0.0.
- [`docs/DATASET_CARD.md`](docs/DATASET_CARD.md) — finalidade, composição, usos e limitações.
- [`docs/DATA_DICTIONARY.md`](docs/DATA_DICTIONARY.md) — guia legível do dicionário; os registros completos permanecem no CSV.
- [`docs/SCHEMA.md`](docs/SCHEMA.md) e [`docs/relational-schema.svg`](docs/relational-schema.svg) — estrutura e diagrama relacionais.
- [`docs/GENERATION_METHODOLOGY.md`](docs/GENERATION_METHODOLOGY.md) — fatos documentados da geração e limite de reprodutibilidade.
- [`docs/VALIDATION.md`](docs/VALIDATION.md) — escopo da auditoria, resultados e problemas conhecidos.
- [`docs/LIMITATIONS.md`](docs/LIMITATIONS.md) — limites detalhados de interpretação e validade externa.
- [`docs/USE_CASES.md`](docs/USE_CASES.md) — usos recomendados e desaconselhados.
- [`docs/guia_explicativo_dataset_frota_robos_v2.pdf`](docs/guia_explicativo_dataset_frota_robos_v2.pdf) — guia explicativo em PDF.
- [`examples/01_dataset_overview.ipynb`](examples/01_dataset_overview.ipynb) — notebook local de visão geral; o código executável ainda exige uma licença de software separada antes da publicação.
- [`LICENSE-DATA.md`](LICENSE-DATA.md) — escopo e termos dos dados e da documentação.
- [`CHANGELOG.md`](CHANGELOG.md) — histórico de versões e situação atual de pré-lançamento.
- [`CITATION.cff`](CITATION.cff) — metadados de citação CFF 1.2.0 com autoria confirmada.
- [`AUTHORS.md`](AUTHORS.md) — declaração de autoria de todo o pacote.
- [`FILE_MANIFEST.json`](FILE_MANIFEST.json) — autoria por arquivo, tamanhos em bytes, tipos de mídia e hashes SHA-256.
- [`PRE_PUBLICATION_CHECKLIST.md`](PRE_PUBLICATION_CHECKLIST.md) — requisitos pendentes para o lançamento.

## Licença

Os arquivos de dados e a documentação do projeto estão preparados para distribuição sob a licença **Creative Commons Atribuição-NãoComercial 4.0 Internacional (CC BY-NC 4.0)**. Consulte [`LICENSE-DATA.md`](LICENSE-DATA.md) para conhecer o escopo local exato.

Essa licença **não** abrange código-fonte, scripts executáveis nem arquivos/código de notebooks. Portanto, o notebook de visão geral incluído exige uma licença de software separada antes da publicação. A CC BY-NC 4.0 não concede permissão comercial para os dados.

## Citação

Os metadados de citação com a criadora confirmada estão em [`CITATION.cff`](CITATION.cff):

> Silva de Souza, Maria Karolina. *Synthetic Mobile Robot Fleet Dataset for Logistics Operations*. Versão 1.0.0. Dataset preparado para publicação.

A data de lançamento, a URL do repositório, o DOI, o ORCID e a afiliação foram omitidos porque ainda não foram estabelecidos ou confirmados. Adicione-os ao CFF somente quando forem reais; a presença do `CITATION.cff` não significa que a v1.0.0 já foi publicada.
