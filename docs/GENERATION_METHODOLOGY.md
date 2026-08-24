---
title: "Metodologia de geração e proveniência - Frota de robôs móveis em centro logístico"
author: "Maria Karolina Silva de Souza"
version: "1.0.0"
lang: "pt-BR"
---

# Metodologia de geração e proveniência

**Autoria:** Maria Karolina Silva de Souza

## Escopo deste documento

Este documento registra o que pode ser afirmado sobre a origem do dataset com base no arquivo Excel, no guia explicativo e na auditoria local. Ele não reconstrói nem presume detalhes do algoritmo que não estão disponíveis.

## Proveniência

- Natureza: dados **100% sintéticos**.
- Domínio representado: operação logística com frota de robôs móveis.
- Período operacional sintético: janeiro de 2025 a junho de 2026.
- Semente registrada durante a geração: `20260730`.
- Artefato de origem: `dataset_frota_robos_centro_logistico_v2.xlsx`.
- Tamanho auditado do XLSX final: 6.136.028 bytes.
- SHA-256 auditado do XLSX final: `aa31190ea25bbde9313d00b7e76f4c58d7e19e0a6e58a3d60f9bd50c11ffe79b`.
- Fontes externas: nenhuma fonte externa foi declarada ou utilizada na documentação estrutural fornecida.

O dataset não representa registros anonimizados ou pseudonimizados de uma operação real. Modelos de robô, setores, rotas, eventos, falhas, custos e medições pertencem ao cenário artificial.

## Linhagem de versão

`v2` é a identificação da segunda revisão interna da planilha durante o desenvolvimento. O pacote foi preparado como **v1.0.0**, a primeira versão pública. Essa mudança de numeração não afirma uma nova geração dos dados: ela separa a revisão interna do artefato da versão de distribuição.

## Estrutura conhecida da geração

O resultado congelado organiza a simulação em cinco camadas:

1. cadastros relativamente estáveis: modelos, robôs, ambientes e rotas;
2. snapshots temporais: estado dos robôs e condições ambientais;
3. eventos operacionais: missões e manutenções;
4. bases analíticas: junções temporais por missão e por robô-semana;
5. governança: validações e dicionário de dados.

A documentação permite verificar relações no resultado, mas não descreve todos os mecanismos probabilísticos ou todas as distribuições usadas para produzi-lo. Não se deve inferir do arquivo, por exemplo, a forma exata das distribuições, pesos causais, ordem das chamadas aleatórias ou regras completas de amostragem.

## Regras temporais documentadas

As bases analíticas foram construídas para preservar a ordem temporal:

- para um robô, usa-se o snapshot do mesmo `robo_id` com a maior `data_hora` menor ou igual ao instante de referência;
- para o ambiente de uma missão, usa-se o snapshot mais recente do setor de destino da rota até o início da missão;
- na base semanal, variáveis retrospectivas encerram antes da `data_referencia`;
- campos terminados em `_alvo` representam resultados posteriores ao instante de predição.

Essas regras foram verificadas no artefato distribuído. Elas não equivalem ao código de geração original.

## Cadência e cobertura observadas

- `CONDICOES_AMBIENTAIS` possui um snapshot por setor às 00:00, 08:00 e 16:00, entre 1º de janeiro de 2025 e 30 de junho de 2026.
- `HISTORICO_ROBOS` reúne 60 snapshots de início do período, 10.000 snapshots de início de missão e dois snapshots por ordem de manutenção, totalizando 12.144.
- `ROTAS` contém os 182 pares direcionais possíveis entre 14 setores distintos: `14 x 13`.
- `BASE_ML_MANUTENCAO` cobre 78 semanas por 60 robôs, totalizando 4.680 linhas.

Essas igualdades descrevem o resultado observado e validado; não revelam o algoritmo que gerou os valores de cada variável.

## Valores ausentes produzidos por projeto

O artefato contém mais de um mecanismo de ausência:

- ausência intencional de sensor em missões, especialmente Wi-Fi e temperatura do robô;
- ausência por não aplicabilidade, como horário de colisão quando não houve colisão;
- ausência por estrutura temporal, como falta de manutenção anterior no começo da série ou de próxima manutenção observável no seu final;
- ausência de ligação para manutenções preventivas que não nasceram de uma missão.

Esses mecanismos não devem ser combinados automaticamente em uma única categoria de “falha de coleta”. A interpretação depende da coluna e do momento do processo.

## O que é reproduzível

O pacote permite:

- verificar a integridade do XLSX por tamanho e SHA-256;
- repetir as verificações estruturais sobre o artefato congelado;
- reexportar as planilhas e comparar o conteúdo tabular;
- reproduzir análises que fixem explicitamente software, versões, filtros, tratamento de ausentes e particionamento.

Os CSVs distribuídos são UTF-8 sem BOM, separados por vírgula e com final de linha LF. Todos passaram por round-trip: exportação, nova leitura e comparação com as células normalizadas da planilha produziram equivalência para linhas, colunas e valores.

O round-trip preservou os valores fracionários sem arredondamento ou truncamento. Na auditoria inicial, 57.923 ocorrências em 12 colunas revelaram tipos `inteiro` inadequados, e 11 declarações de ausentes precisavam de razões específicas. Esses metadados foram corrigidos no XLSX e em `data/dictionary.csv`. Na auditoria final, a cobertura permaneceu completa, com 173 de 173 colunas documentadas, `declaredTypeMismatchTotal = 0` e `missingDeclarationMismatches = []`.

## O que não é reproduzível com o pacote atual

O algoritmo ou script original de geração não está presente. A semente `20260730`, isoladamente, não informa:

- o gerador pseudoaleatório e sua versão;
- a ordem das operações e chamadas aleatórias;
- as distribuições e parâmetros;
- as dependências e versões de software;
- as regras completas de coerência e pós-processamento;
- a forma exata de introdução de eventos raros e ausências.

Portanto, **não é possível regenerar integralmente o mesmo dataset somente com a seed**. O artefato distribuído deve ser citado e preservado como a versão congelada. Não se deve afirmar reprodutibilidade da geração até que o algoritmo original, as dependências e um procedimento verificável estejam disponíveis.

## Datas e fuso horário

As datas foram exportadas em forma ISO 8601 sem deslocamento, como `2025-01-01T04:05:46`. Nenhum fuso horário está especificado. Consumidores não devem acrescentar `Z`, converter automaticamente para UTC nem aplicar horário de verão sem uma decisão externa documentada.

## Recomendações para análises reprodutíveis

1. Registre a versão pública `v1.0.0` e o hash do arquivo usado.
2. Importe IDs como texto, datas como datetime sem fuso e colunas declaradas como `numérico` com suporte a casas decimais.
3. Documente a política para ausências estruturais e de sensor.
4. Separe dados por tempo quando o experimento simular previsão futura.
5. Não use colunas `_alvo` ou outras informações posteriores como preditores.
6. Registre bibliotecas, versões, semente do experimento e métricas.
7. Compare contagens e chaves com [VALIDATION.md](VALIDATION.md) antes de modelar.
