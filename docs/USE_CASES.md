---
title: "Casos de uso - Frota de robôs móveis em centro logístico"
author: "Maria Karolina Silva de Souza"
version: "1.0.0"
lang: "pt-BR"
---

# Casos de uso

**Autoria:** Maria Karolina Silva de Souza

## Princípio de uso

Este dataset 100% sintético é indicado para aprendizagem, prototipação e comparação de métodos em um cenário relacional e temporal controlado. Ele não substitui dados reais na validação de decisões industriais.

Antes de analisar, preserve IDs como texto, leia datas sem presumir fuso horário, respeite os tipos finais do dicionário e escolha uma política explícita para ausências intencionais e estruturais.

## Guia de seleção das tabelas

| Objetivo | Tabelas recomendadas | Observação principal |
| --- | --- | --- |
| Conhecer equipamentos | `MODELOS_ROBO`, `ROBOS` | Capacidades do modelo e cadastro das 60 unidades |
| Conhecer espaço e trajetos | `AMBIENTES`, `ROTAS` | Rotas são direcionais |
| Reconstruir estado temporal | `HISTORICO_ROBOS`, `CONDICOES_AMBIENTAIS` | Use o último snapshot não posterior à referência |
| Estudar execução operacional | `MISSOES` | A linha mistura início, durante e após a missão |
| Estudar indisponibilidade e reparo | `MANUTENCOES` | Detecção, início e fim são instantes diferentes |
| Modelar resultados de missões | `BASE_ML_MISSOES` | Exclua todas as colunas `_alvo` das entradas |
| Modelar risco de manutenção | `BASE_ML_MANUTENCAO` | Respeite janelas retrospectivas e censura no fim da série |
| Auditar metadados e integridade | `DICIONARIO`, `VALIDACOES` | Confronte também o relatório independente |

## Ciência de Dados

### Limpeza e qualidade

- importar CSVs com tipos explícitos;
- preservar códigos com zeros à esquerda;
- distinguir nulo de sensor, não aplicabilidade e censura temporal;
- comparar estratégias de imputação para Wi-Fi e temperatura do robô;
- reproduzir um fluxo de auditoria e correção de metadados a partir do histórico de preparação do pacote;
- validar categorias e intervalos sem substituir silenciosamente valores.

### Junções e transformação

- reconstruir uma missão ligando robô, modelo, rota e condição ambiental;
- executar as-of joins para obter o último snapshot do robô até o início da missão;
- verificar que a condição ambiental pertence ao setor de destino da rota;
- agregar eventos por robô, modelo, setor, rota ou janela de tempo;
- comparar uma base relacional construída pelo estudante com `BASE_ML_MISSOES`.

### Análise exploratória e visualização

- contagens por modelo, setor, rota, turno e status;
- evolução temporal de missões, falhas e manutenções;
- distribuição de duração, consumo, carga e custo;
- matrizes origem-destino e mapas de fluxo abstratos;
- padrões de ausência por período, robô e tipo de evento.

Essas análises descrevem somente o cenário sintético e não devem ser apresentadas como estatísticas de uma empresa real.

## Estatística

- estatística descritiva e comparação de grupos;
- associação entre variáveis categóricas;
- análise de medidas contínuas e contagens;
- séries temporais agregadas por dia, semana ou mês;
- estudo de dados ausentes e análise de sensibilidade;
- avaliação de desbalanceamento de classes;
- desenho e discussão de validação temporal.

O dataset pode ser usado para ensinar hipóteses e estimadores, mas conclusões causais ou inferência para uma população industrial real não são suportadas.

## Machine Learning com `BASE_ML_MISSOES`

A unidade é uma missão. Preditores incluem cadastro do robô e modelo, estado no início, rota, carga, bateria inicial, condição ambiental e calendário. Os alvos possíveis incluem:

- regressão: `distancia_real_m_alvo`, `duracao_real_min_alvo`, `consumo_bateria_pp_alvo`, `atraso_min_alvo`;
- classificação binária: `colisao_alvo`, `dano_detectado_alvo`, `intervencao_necessaria_alvo`, `atraso_alvo`, `missao_concluida_alvo`;
- classificação multiclasse: `gravidade_colisao_alvo`, `status_missao_alvo`.

Fluxo recomendado:

1. defina um único alvo e o instante de predição;
2. exclua todas as colunas `_alvo` das entradas, inclusive alvos alternativos;
3. remova identificadores quando eles não tiverem significado preditivo legítimo;
4. use divisão temporal para avaliar generalização futura;
5. ajuste pré-processamento somente no conjunto de treino;
6. reporte métricas adequadas ao balanceamento da classe e intervalos de incerteza;
7. registre o tratamento das 216 missões para as quais atraso não se aplica.

## Machine Learning com `BASE_ML_MANUTENCAO`

A unidade é o par (`data_referencia`, `robo_id`). Há 78 semanas para cada um dos 60 robôs. Tarefas possíveis:

- prever `falha_nos_proximos_7_dias_alvo`;
- estimar `dias_ate_proxima_manutencao_alvo`;
- classificar `tipo_proxima_manutencao_alvo`;
- prever `custo_proxima_manutencao_brl_alvo`.

Fluxo recomendado:

1. ordene por `data_referencia` e preserve agrupamento por robô;
2. use somente atributos disponíveis na referência e janelas retrospectivas;
3. mantenha todos os campos `_alvo` fora das entradas;
4. trate separadamente as 192 linhas sem próxima manutenção observável;
5. avalie se a tarefa deve ser formulada como regressão censurada, classificação ou análise de sobrevivência;
6. evite que semanas adjacentes do mesmo robô atravessem aleatoriamente treino e teste.

## Engenharia de Dados

- modelagem de esquema relacional e cardinalidades;
- validação de chaves primárias e estrangeiras;
- implementação de as-of joins e janelas móveis;
- criação de pipelines XLSX-CSV com testes de round-trip;
- criação de testes para sobreposição de intervalos;
- verificação de contratos de dados e evolução de esquema;
- construção de camadas raw e analytical.

O relatório final oferece valores de referência: 11 validações declaradas e 12 verificações independentes passaram com zero violações; o dicionário cobre 173 de 173 colunas; `declaredTypeMismatchTotal = 0`, `missingDeclarationMismatches = []` e `numericRangeViolationTotal = 0`; e os 11 CSVs tiveram round-trip equivalente.

## Ensino de governança de dados

O pacote é apropriado para discutir:

- diferença entre metadado declarado e conteúdo observado;
- disponibilidade temporal de atributos;
- data leakage;
- rastreabilidade entre dado bruto e base analítica;
- versionamento de uma revisão interna `v2` como primeiro pacote público `v1.0.0`;
- limites de reprodutibilidade quando há seed, mas não há algoritmo;
- diferença entre varredura heurística de PII e certificação formal de privacidade.

## Detecção de anomalias e aprendizado não supervisionado

- procurar missões com consumo, duração ou temperatura incomuns dentro do cenário;
- agrupar rotas, missões ou robôs por perfis sintéticos;
- investigar eventos raros e classes minoritárias;
- comparar métodos robustos a valores ausentes.

Uma anomalia estatística no dataset não representa necessariamente uma falha industrial real; pode ser consequência direta da geração artificial.

## Exemplos de perguntas adequadas

- Como construir uma junção temporal sem usar snapshots futuros?
- Como diferentes políticas de imputação afetam um classificador de colisão?
- Uma divisão aleatória superestima o desempenho em relação a uma divisão temporal?
- Quais atributos de rota e ambiente são mais úteis dentro deste cenário sintético?
- Como a censura no final da série afeta a previsão da próxima manutenção?
- Quais verificações detectam sobreposição de missões e manutenções?

Formule conclusões como resultados “neste dataset sintético”, não como fatos sobre frotas reais.

## Usos inadequados

Não use este dataset para:

- controlar ou despachar robôs reais;
- definir manutenção, segurança ou orçamento operacional;
- estimar taxas reais de colisão, falha ou atraso;
- avaliar conformidade regulatória;
- identificar pessoas ou empresas;
- justificar conclusões causais;
- afirmar que a geração completa é reproduzível apenas porque a seed está registrada;
- uso comercial incompatível com a licença CC BY-NC 4.0.

## Checklist antes de publicar resultados

- [ ] Declare que os dados são 100% sintéticos.
- [ ] Registre a versão `v1.0.0` e o hash do artefato usado.
- [ ] Informe que `v2` é a revisão interna do XLSX.
- [ ] Documente a ausência do algoritmo de geração e o limite da seed.
- [ ] Descreva a divisão temporal e as medidas contra leakage.
- [ ] Explique como cada tipo de valor ausente foi tratado.
- [ ] Preserve IDs como texto e datas sem fuso presumido.
- [ ] Use o tipo `numérico` do dicionário para preservar casas decimais.
- [ ] Não extrapole resultados para operações industriais reais.
- [ ] Cite [VALIDATION.md](VALIDATION.md) e [LIMITATIONS.md](LIMITATIONS.md) para o escopo das verificações.
