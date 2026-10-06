# Porsche · Painel de Análise de Vendas (Power BI)

Painel interativo em Power BI para analisar vendas de veículos Porsche: receita, volume, ticket médio, quilometragem, formas de pagamento, status de entrega e distribuição por modelo e estado.

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black)
![Status](https://img.shields.io/badge/status-em%20evolu%C3%A7%C3%A3o-orange)

## Sumário
- [Visão geral](#visão-geral)
- [Páginas e visuais](#páginas-e-visuais)
- [Base de dados](#base-de-dados)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Como usar](#como-usar)
- [Tratamento de dados (Power Query)](#tratamento-de-dados-power-query)
- [Medidas DAX](#medidas-dax)
- [Qualidade dos dados e premissas](#qualidade-dos-dados-e-premissas)
- [Valores de conciliação](#valores-de-conciliação)
- [Roadmap](#roadmap)

## Visão geral
O painel responde a perguntas como:
- Quanto foi vendido e qual o ticket médio?
- Quais modelos e estados concentram a receita?
- Como os pedidos se distribuem por forma de pagamento e status de entrega?
- Qual a relação entre preço e quilometragem por modelo?

Todos os visuais são filtráveis por segmentações (ano do modelo, status, pagamento e estado) com filtro cruzado entre gráficos.

## Páginas e visuais

### 1. Visão Geral
| Visual | Campos |
|---|---|
| Cartões | Receita total · Pedidos · Ticket médio · Km médio |
| Colunas | Receita por ano do modelo |
| Rosca | Receita por status de entrega |
| Barras | Pedidos por forma de pagamento |
| Barras | Receita por modelo (ordenado) |
| Dispersão | Preço médio × quilometragem média por modelo |
| Colunas | Receita por estado |
| Segmentações | Ano do modelo · Status · Pagamento · Estado |

### 2. Detalhe
Tabela com todos os pedidos (data, modelo, ano, preço, km, pagamento, cidade, estado e status), filtrada pelas segmentações.

## Base de dados
Arquivo Excel com 100 pedidos, aba `Sanitized`.

| Coluna | Tipo | Descrição |
|---|---|---|
| `SaleDateSanitized` | texto | Data da venda (`yyyy-MM-dd`) ou `INVALID` |
| `PorscheModelSanitized` | texto | Modelo (40 valores distintos) |
| `ModelYearSanitized` | inteiro | Ano do modelo (2020–2026) |
| `SalesPriceSanitized` | número | Preço de venda |
| `VehicleMileageSanitized` | inteiro | Quilometragem |
| `PayMethodSanitized` | texto | Forma de pagamento (9 valores) |
| `CitySanitized` | texto | Cidade |
| `StateSanitized` | texto | Estado (sigla dos EUA) |
| `DeliveryStatusSanitized` | texto | Status de entrega (10 valores) |

## Estrutura do repositório
```
.
├── Porsche_Painel.pbix          # Painel
├── dados/
│   └── base_porsche.xlsx        # Base de origem
├── powerquery/
│   └── 01_PowerQuery_Vendas.m   # Limpeza e colunas derivadas
├── dax/
│   └── 02_Medidas_DAX.dax       # Calendário, medidas e parâmetros
├── docs/
│   └── 03_Layout_Painel.md      # Especificação de visuais e conciliação
├── tema/
│   └── Tema_Porsche.json        # Tema de cores
└── README.md
```

## Como usar
1. Abra `Porsche_Painel.pbix` no **Power BI Desktop** (versão recente, com formato de relatório aprimorado ativado).
2. O caminho do Excel salvo no arquivo aponta para a máquina do autor. Em **Transformar dados > Configurações da fonte de dados > Alterar origem**, aponte para `dados/base_porsche.xlsx` e clique em **Atualizar**.
3. Opcional: importe o tema em **Exibir > Temas > Procurar temas** e selecione `tema/Tema_Porsche.json`.

## Tratamento de dados (Power Query)
O script `powerquery/01_PowerQuery_Vendas.m` substitui a consulta bruta e:
- renomeia as colunas para português;
- converte `INVALID` em data nula, sem remover a linha;
- cria `Familia` (718, 911, Cayenne, Macan, Panamera, Taycan) e `Motorizacao` (combustão, híbrido, elétrico);
- agrupa `StatusOrig` em `StatusGrupo` (Entregue, Em logística, Em aprovação, Cancelado);
- agrupa e traduz as formas de pagamento (`PagamentoGrupo`, `Pagamento`);
- cria `Condicao` (Novo se Km ≤ 100), `FaixaPreco` e `FaixaKm`.

Para usá-lo, crie o parâmetro de texto `pCaminhoArquivo` com o caminho do Excel.

## Medidas DAX
O arquivo `dax/02_Medidas_DAX.dax` traz:
- **Tabela Calendário** e relação com `Vendas[DataVenda]`;
- **Base:** Receita Bruta, Receita Líquida (sem cancelados), Veículos Vendidos, Ticket Médio, Receita Entregue, Receita em Andamento, Taxas de Entrega e de Cancelamento, Km Médio;
- **Tempo:** MoM, YoY, YTD e acumulado;
- **Qualidade:** pedidos e receita sem data válida;
- **Dinâmico:** seletor de métrica (receita, veículos, ticket, km) e parâmetro de campo "Analisar por" (família, modelo, estado, pagamento, status, condição, faixa de preço).

A versão atual do `.pbix` usa as colunas brutas da base. Para ativar as medidas e os parâmetros, aplique o `.m` e o `.dax` e troque os campos dos visuais conforme `docs/03_Layout_Painel.md`.

## Qualidade dos dados e premissas
- **Datas inválidas:** 24 das 100 linhas têm `INVALID` (US$ 3,34 mi). Elas contam nos totais, mas não aparecem em análises por mês.
- **Datas futuras:** as vendas válidas vão de 2024-03-14 a 2027-10-01. Valide a origem.
- **Localização:** cidades e estados são dos EUA. A moeda não está informada e assume-se US$.
- **Receita líquida:** exclui pedidos cancelados (7 de 100).
- **Veículo novo:** definido como Km ≤ 100 (premissa ajustável no `.m`).

## Valores de conciliação
Sem filtros aplicados:

| Medida | Valor |
|---|---|
| Pedidos | 100 |
| Receita bruta | 12.827.800,50 |
| Receita líquida (sem cancelados) | 12.202.550,50 |
| Veículos vendidos (sem cancelados) | 93 |
| Ticket médio (líquido) | ≈ 131.210 |
| Receita entregue | 5.078.800,50 |
| Taxa de cancelamento | 7% |
| Pedidos por família | 911: 23 · Cayenne: 18 · Macan: 17 · Taycan: 16 · Panamera: 14 · 718: 12 |

> No `.pbix` atual, que usa as colunas brutas, os cartões mostram a receita **bruta** (12.827.800,50) e o ticket médio sobre os 100 pedidos (≈ 128.278).

## Roadmap
- [ ] Aplicar o Power Query e o modelo com Calendário
- [ ] Trocar os visuais para as medidas DAX (receita líquida, YoY, MoM)
- [ ] Adicionar seletor de métrica e "Analisar por"
- [ ] Página de qualidade de dados e drill-through por modelo
- [ ] Mapa preenchido por estado
