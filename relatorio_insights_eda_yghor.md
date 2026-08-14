# Relatório de Insights do Dataset de Credit Risk

## Origem

Este relatório foi consolidado a partir do notebook de EDA disponível em `notebooks/eda.ipynb` e da base de dados `data/raw/cs-training.csv`.

Observação: não foi localizado um arquivo com o nome `eda_yghor.ipynb` no repositório; o notebook ativo e relevante no projeto é `notebooks/eda.ipynb`, e os insights abaixo refletem a análise já executada sobre esse dataset.

## 1) Visão geral da base

- Total de registros: 150.000
- Total de variáveis: 12
- Coluna alvo: `inadimplente`
- Distribuição do alvo:
  - `0` (não inadimplente): 93,316% 
  - `1` (inadimplente): 6,684%
- A base é desbalanceada, com uma proporção de inadimplentes bem menor do que os clientes regulares.

## 2) Estrutura e qualidade dos dados

### 2.1. Missing values

Os principais campos com ausência são:

- `renda_mensal`: 29.731 ausentes (19,82%)
- `num_dependentes`: 3.924 ausentes (2,62%)

As demais colunas não apresentam valores nulos.

Isso indica que a limpeza de dados precisa tratar especificamente renda mensal e dependentes antes do treinamento de modelos.

### 2.2. Valores extremos e outliers

Algumas variáveis apresentam distribuições fortemente assimétricas e máximos muito altos, o que sugere presença de outliers ou codificações especiais:

- `utilizacao_credito_rotativo`: máximo = 50.708
- `indice_endividamento`: máximo = 329.664
- `renda_mensal`: máximo = 3.008.750
- `num_linhas_credito`: máximo = 58
- `num_emprestimos_imobiliarios`: máximo = 54
- `num_dependentes`: máximo = 20

Esses valores parecem indicar outliers reais e também possivelmente flags/encoding de "muitos atrasos" ou cenários extremos.

## 3) Principais sinais de risco no dataset

### 3.1. Utilização de crédito rotativo

A variável `utilizacao_credito_rotativo` é uma das mais fortes associadas ao risco.

- 3.321 registros (2,21%) possuem `utilizacao_credito_rotativo > 1`
- Entre esses clientes, a taxa de inadimplência sobe para 37%
- Sem essa anomalia, a taxa de inadimplência cai para 6%
- Mediana por grupo:
  - não inadimplente: 0,13
  - inadimplente: 0,84

Conclusão: o uso de crédito rotativo acima de 100% do limite parece ser um forte indicativo de risco, e é provavelmente uma variável altamente preditiva.

### 3.2. Índice de endividamento

A variável `indice_endividamento` é a razão de obrigações sobre renda/recursos disponíveis e tem uma distribuição muito elevada em alguns registros.

- 35.137 registros (23,42%) têm `indice_endividamento > 1`
- A proporção de inadimplentes não muda substancialmente entre clientes com e sem anomalia de endividamento
- Mediana por grupo:
  - não inadimplente: 0,36
  - inadimplente: 0,43

Conclusão: embora a variável tenha relação com risco, ela não funciona como um ponto de corte simples como `> 1`, ou seja, o efeito parece ser mais suave e distribuído ao longo da escala, em vez de um gatilho binário claro.

### 3.3. Idade

- Mediana da idade:
  - não inadimplente: 52 anos
  - inadimplente: 45 anos
- Existe 1 registro com idade = 0, claramente inválido

Conclusão: clientes mais jovens tendem a apresentar mais inadimplência. A idade é relevante para se entender o perfil de risco, mas precisa de tratamento de valores inconsistentes.

### 3.4. Renda mensal

- 29.731 registros faltando em `renda_mensal`
- Mediana da renda por grupo:
  - não inadimplente: R$ 5.466
  - inadimplente: R$ 4.500

Conclusão: clientes com renda menor possuem mais risco de inadimplência. A renda mensal também deve ser tratada com cuidado por causa dos valores ausentes e da cauda longa da distribuição.

### 3.5. Atrasos em pagamentos

As variáveis de atraso (`num_atrasos_30_59_dias`, `num_atrasos_60_89_dias`, `num_atrasos_acima_90_dias`) possuem muitos valores em 0, mas também incluem valores altos e sinais anômalos.

- Máximo observado: 98 em todas as classes de atraso
- Há 269 registros com valores iguais a 98 em pelo menos um desses campos
- Isso sugere que 98 funciona como codificação especial/flag para "muito atrasado" ou cenário extremo

Conclusão: esses campos carregam forte informação sobre risco, mas precisam ser tratados como indicadores que podem conter valores de outlier/flag, não como contagens normais em toda a extensão da escala.

### 3.6. Quantidade de linhas de crédito e imóveis financiados

- `num_linhas_credito`: mediana de 8 para não inadimplentes e 7 para inadimplentes
- `num_emprestimos_imobiliarios`: valores altos como 54 indicam outliers
- Há 354 registros com `num_linhas_credito > 30`
- Há 10 registros com `num_emprestimos_imobiliarios > 20`

Conclusão: essas variáveis têm algumas observações extremas, mas não parecem ser, por si só, as principais drivers do risco. Elas podem agir como fatores de contexto, mas o sinal mais forte vem do uso de crédito e dos atrasos.

### 3.7. Dependentes

- 3.924 registros faltando em `num_dependentes`
- Mediana: 0 para ambos os grupos
- Máximo observado: 20
- Apenas 2 registros com `num_dependentes > 10`

Conclusão: a maioria dos clientes tem poucos ou nenhum dependente. A variável pode ter valor, mas a informação não parece ser tão forte quanto a renda ou os atrasos.

## 4) Fatores com maior poder explicativo

Com base na análise de EDA, os fatores que mais parecem carregar informação sobre inadimplência são:

1. `utilizacao_credito_rotativo`
2. `num_atrasos_30_59_dias`
3. `num_atrasos_60_89_dias`
4. `num_atrasos_acima_90_dias`
5. `renda_mensal`
6. `idade`
7. `indice_endividamento`

Entre esses, a utilização de crédito rotativo se destaca como o sinal mais forte e visualmente claro na EDA.

## 5) Observações importantes para modelagem

- O dataset tem baixa taxa de inadimplência (aprox. 6,7%), então o problema é classificado como desbalanceado.
- Há outliers extremamente grandes em diversas colunas, principalmente financeiras.
- Há valores ausentes relevantes em renda e dependentes.
- Algumas variáveis parecem usar codificação de flag em valores elevados (como 96/98), o que exige cuidado na preparação do modelo.
- A segmentação por risco parece ser clara para algumas variáveis: uso de crédito rotativo, atrasos, renda e idade.

## 6) Conclusão executiva

O conjunto de dados mostra um padrão consistente de risco de crédito: clientes com alta utilização do crédito rotativo, baixa renda, mais jovens e com atrasos recorrentes são muito mais propensos a inadimplir. No entanto, o dataset também possui outliers, dados ausentes e codificações especiais que exigem tratamento antes da modelagem preditiva. O próximo passo natural, com base nessa EDA, é construir um pipeline de limpeza e engenharia de features, seguir com modelagem de baseline (ex.: regressão logística, árvore, XGBoost) e validar o desempenho em amostras balanceadas/estratificadas.

## 7) Próximo passo sugerido

- Limpeza de outliers e valores inconsistentes
- Imputação de renda e dependentes
- Categorização ou clipping para colunas de atraso e crédito
- Treino de baseline com métricas de recall/precision/F1/ROC-AUC
- Comparação de modelos diante do desbalanceamento
