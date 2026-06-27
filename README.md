# Previsão de Demanda por Categoria — Bagaggio
### MVP Sprint 3 | Machine Learning | PUC-Rio

**Aluna:** Diana Serrano  
**Entrega:** 05/07/2025

---

## O que é esse projeto?

Trabalho em, uma rede de lojas especializada em malas, mochilas e acessórios de viagem. Todo ano o time de produtos precisa decidir quanto comprar de cada categoria — e hoje isso é feito com base no histórico do ano anterior mais um percentual de crescimento que varia por categoria, definido manualmente.

Funciona? Mais ou menos. O problema é que esse processo ignora que uma mochila escolar e uma mala de bordo têm comportamentos completamente diferentes ao longo do ano. Uma explode em janeiro, a outra na Black Friday.

A ideia desse MVP foi testar se um modelo de Machine Learning consegue prever melhor a quantidade de peças vendidas por categoria do que esse método atual — e a resposta foi sim, com 32.7% de redução no erro médio.

---

## O negócio tem uma sazonalidade bem definida

Antes de qualquer código, é importante entender o calendário:

| Época | Período | O que vende |
|-------|---------|-------------|
| Volta às Aulas | Jan – Fev | Mochilas, Lancheiras, Estojos |
| Dia das Mães | Maio | Bolsas — mas menos do que se imagina |
| Dia dos Pais | Agosto | Mochilas Executivas, Malas |
| Black Friday | Out – Nov | Malas de Bordo e Despacho |
| Natal | Dezembro | Mix mais acessível, presentes |

Uma descoberta interessante durante a análise: **julho também é forte**, puxado pelas férias escolares — não estava no radar inicial e acabou virando uma feature do modelo.

Outra surpresa: o Dia das Mães não mostrou sazonalidade própria nos dados. Diferente do que o senso comum diria, MALAS | BORDO lidera em maio assim como em todos os outros meses.

---

## Os dados

Exportei do ERP — vendas diárias de junho/2024 a junho/2026, com três níveis de categoria e o canal de venda (loja própria, franquia ou e-commerce). Dados que foram disponibilizados para estudo.

Um detalhe importante que apareceu na análise: NIVEL2 sozinho é ambíguo. TERMICOS, por exemplo, existe tanto em ESCOLARES quanto em COTIDIANO — e têm padrões de demanda completamente diferentes. Por isso criei uma chave combinada NIVEL1 + NIVEL2 (ex: "ESCOLARES | TERMICOS") que resolve isso.

| Coluna | Descrição |
|--------|-----------|
| `DATA_EMISSAO` | Data da venda |
| `CATEGORIA` | Chave combinada NIVEL1 \| NIVEL2 — 25 categorias únicas |
| `CANAL` | P/A = Loja Própria, F = Franquia, E = E-commerce |
| `QTD` | Quantidade vendida — variável alvo |
| `VALOR` | Receita — removida do modelo (data leakage) |

Após limpeza: **59.422 registros**, sem nulos, devoluções removidas.

---

## Como o modelo foi construído

**Divisão temporal:** treino com jun/2024–mai/2025, teste com jun/2025–jun/2026. Cada período tem um ciclo sazonal completo — mais realista do que uma divisão aleatória.

**Features principais:**
- Categoria e canal (encodados)
- Mês, dia, dia da semana, trimestre
- Flags sazonais: Volta às Aulas, Férias de Julho, Dia dos Pais, Black Friday, Natal

**Modelos testados:** Regressão Linear, Ridge, Random Forest e Gradient Boosting — todos comparados contra o baseline (método atual da empresa).

---

## Resultados

| Modelo | MAE | RMSE | R² |
|--------|-----|------|----|
| 📌 Baseline (método atual) | 38.78 | 100.19 | 0.5519 |
| Regressão Linear | 78.19 | 124.52 | 0.0106 |
| Ridge Regression | 78.20 | 124.52 | 0.0106 |
| Gradient Boosting | 41.50 | 76.25 | 0.6291 |
| **Random Forest** | **26.09** | **57.92** | **0.7859** |

**Random Forest venceu** — erra em média 26 peças por categoria/canal/dia, contra 39 do método atual. Redução de 32.7% no erro.

Os modelos lineares ficaram piores que o baseline, o que faz sentido: demanda sazonal no varejo não é linear.

Outro ponto importante: o baseline só consegue prever onde existe histórico do ano anterior — cobre apenas 44% dos registros. O modelo ML prevê tudo.

---

## Limitações e próximos passos

O modelo ainda erra bastante nos picos extremos (Black Friday, Volta às Aulas) — exatamente quando o planejamento é mais crítico. Com mais dados históricos e talvez uma abordagem de série temporal por categoria, isso poderia melhorar.

Com alguns ajustes, dá para usar o modelo como apoio ao time de compras — não para substituir o processo, mas para dar uma referência mais objetiva além do histórico + percentual fixo.

---

## Como rodar

```python
URL = 'https://raw.githubusercontent.com/DianaSerrano1/MVP-MachineLearning/main/base_ML.csv'
```

Abre o notebook e roda tudo em sequência. Requer pandas, scikit-learn, matplotlib e seaborn.

---

*PUC-Rio — Pós-Graduação em Ciência de Dados*
