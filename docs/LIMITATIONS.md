---
title: "Limitações e cuidados de interpretação - Frota de robôs móveis em centro logístico"
author: "Maria Karolina Silva de Souza"
version: "1.0.0"
lang: "pt-BR"
---

# Limitações e cuidados de interpretação

**Autoria:** Maria Karolina Silva de Souza

## Natureza sintética

O dataset é **100% sintético**. Nenhuma linha representa uma empresa, instalação, pessoa, robô, missão, falha, manutenção ou custo real. Os dados servem para estudo, engenharia e experimentação; não são evidência observacional de uma operação industrial.

As relações entre variáveis foram estabelecidas durante a geração. Correlações, eventos raros, falhas, colisões e ausências refletem o cenário artificial e não devem ser generalizados para uma população real.

## Representatividade e generalização

- Os 60 robôs, 14 setores, 182 rotas, 10.000 missões e 1.042 manutenções formam uma amostra didática ampliada, não uma descrição completa de todas as operações possíveis.
- O período sintético cobre janeiro de 2025 a junho de 2026. Sazonalidades ou tendências fora dessa janela não estão representadas.
- Categorias e frequências não foram calibradas contra uma empresa ou indústria específica.
- Bom desempenho preditivo neste dataset não implica bom desempenho em ambiente real.
- Comparações entre modelos avaliam comportamento dentro deste cenário, não estabelecem um benchmark industrial.

## Sem inferência causal

As associações observadas não autorizam conclusões causais. Não use o dataset para afirmar que uma característica de rota, condição ambiental, versão de firmware ou indicador do robô causa colisão, falha, atraso, custo ou indisponibilidade em equipamentos reais.

## Reprodutibilidade limitada da geração

A seed `20260730` está registrada, mas o algoritmo original de geração não está presente. Uma seed não basta sem gerador pseudoaleatório, ordem das operações, parâmetros, dependências e regras completas. O pacote é reproduzível como artefato congelado e auditável; não é integralmente regenerável a partir da seed.

Consulte [GENERATION_METHODOLOGY.md](GENERATION_METHODOLOGY.md).

## Versão e linhagem

O sufixo `v2` identifica a segunda revisão interna do XLSX. O pacote foi preparado como `v1.0.0`, a primeira versão pública. Comparações futuras devem usar versão e hash, pois não se deve presumir equivalência entre arquivos apenas pelo título.

## Valores ausentes

Há mecanismos diferentes de ausência:

- falhas intencionais de sensores durante missões;
- não aplicabilidade, como instante de colisão quando não houve colisão;
- eventos não ocorridos, como ausência de manutenção anterior;
- censura à direita, quando não há próxima manutenção observável antes do fim da série;
- ligações opcionais, como missão relacionada em manutenção preventiva.

Uma imputação única para todos esses casos pode distorcer a semântica. Indicadores de ausência e análises de sensibilidade podem ser necessários.

## Metadados de valores ausentes

O dicionário final registra razões específicas para ausências estruturais, de sensor, de não aplicabilidade e de censura temporal. Na auditoria inicial, 11 marcações incorretas motivaram a atualização desses metadados; a auditoria final retornou `missingDeclarationMismatches = []`.

A ausência continua exigindo interpretação semântica, mas não há divergência pendente entre o campo `DICIONARIO.ausentes` e o conteúdo auditado. Consulte [DATA_DICTIONARY.md](DATA_DICTIONARY.md#politica-de-valores-ausentes) para as razões documentadas.

## Datas sem fuso horário

Datas e horas não possuem offset nem fuso especificado. Trate-as como horários locais sintéticos. Acrescentar `Z`, converter para UTC ou aplicar horário de verão pode deslocar eventos e invalidar junções temporais.

## Identificadores e tipos

- IDs são texto e podem conter prefixos e zeros à esquerda. Converter `RB008`, `RT044` ou `S06` para número corrompe a chave.
- As 12 colunas fracionárias identificadas na auditoria inicial agora estão declaradas como `numérico`. A auditoria final retornou `declaredTypeMismatchTotal = 0`, e os CSVs preservam as casas decimais.
- Indicadores 0/1 podem conter nulos quando o conceito não se aplica; um booleano sem suporte a nulo pode introduzir valores falsos.
- `_pct` e `_pp` não são equivalentes: percentual mede nível; pontos percentuais medem diferença entre níveis.

## Restrições temporais e risco de leakage

Uma linha de `MISSOES` contém informação do início, da execução e do encerramento. Usar indiscriminadamente todos os campos para prever o resultado cria vazamento de informação.

Em `BASE_ML_MISSOES`, campos `_alvo` são resultados posteriores e devem ser excluídos dos preditores. Em `BASE_ML_MANUTENCAO`, a próxima manutenção é informação futura; alvos podem ficar vazios no fim da série porque o evento seguinte não é observável.

Mesmo nas bases analíticas, o pesquisador deve definir o instante de predição, a divisão temporal e o tratamento de linhas censuradas. Uma divisão aleatória pode permitir que informações de períodos próximos do mesmo robô apareçam nos conjuntos de treino e teste.

## Semântica relacional que pode causar erro

- `setor_base` é o local habitual do robô, não sua posição na missão.
- `condicao_inicio_id` é escolhido no início, mas descreve o setor de destino da rota.
- Rotas são direcionais; A->B e B->A não são intercambiáveis.
- Um snapshot informa o estado conhecido naquele instante, não uma propriedade permanente.
- `falha_detectada_em`, início da manutenção e fim da manutenção são instantes distintos.
- Manutenções preventivas não têm missão relacionada nem falha detectada.

## Rótulos e domínios

- `atraso` é o rótulo oficial. O arquivo não documenta um limiar determinístico único para recriá-lo a partir de `atraso_min`; não invente um corte.
- O domínio de `tipo_manutencao` inclui “preditiva”, mas essa categoria não ocorre nos registros desta versão.
- Valores válidos no dicionário podem representar o domínio permitido, não necessariamente o conjunto observado.
- Eventos raros foram simulados deliberadamente; métricas globais podem ocultar desempenho ruim nas classes minoritárias.

## Bases analíticas não são neutras

`BASE_ML_MISSOES` e `BASE_ML_MANUTENCAO` incorporam escolhas de junção, janela temporal e feature engineering. Elas são úteis para ensino, mas não substituem a compreensão das tabelas relacionais. Resultados podem depender dessas escolhas e devem ser comparados com pipelines construídos a partir das tabelas de origem quando isso fizer parte do objetivo do estudo.

## Privacidade e segurança

Uma varredura heurística não encontrou e-mails, URLs, CPF/CNPJ formatado, padrões semelhantes a telefone ou cabeçalhos suspeitos. Isso é coerente com a natureza sintética, mas não é certificação formal de privacidade.

O dataset não foi validado para controle de robôs, planejamento de segurança, manutenção de equipamentos reais ou tomada de decisão operacional. Não o use diretamente nesses contextos.

## Alcance das validações

As 11 regras declaradas e as 12 verificações independentes passaram com zero violações. O dicionário cobre 173 de 173 colunas, e a auditoria final registrou `declaredTypeMismatchTotal = 0`, `missingDeclarationMismatches = []` e `numericRangeViolationTotal = 0`. Elas confirmam somente as propriedades testadas: chaves, referências, sobreposições, capacidade, temporalidade de snapshots, coerência de colisão, cobertura das bases analíticas, metadados auditados e limites numéricos definidos.

Elas não demonstram que toda propriedade estatística foi validada, que não há todo tipo possível de erro ou viés, nem que qualquer análise será correta. Consulte [VALIDATION.md](VALIDATION.md) para o escopo exato.
