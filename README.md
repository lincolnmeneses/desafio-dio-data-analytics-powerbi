# 📈 Projeto de Data Analytics com Power BI

Desafio de projeto da trilha **Primeiros Passos em Power BI** (DIO): evoluir o relatório criativo (Financial Sample) com **páginas de detalhes analíticos**: TOP N, dispersão, agrupamentos e compartimentação de dados.

Este projeto parte do [relatório com foco em experiência do usuário](https://github.com/lincolnmeneses/desafio-dio-relatorio-ux-powerbi). Ele mantém o mesmo visual e o mesmo menu de navegação e acrescenta duas páginas de detalhes.

---

## 🗂️ Páginas

| Página | O que mostra |
| --- | --- |
| Início | Capa com resumo e botões de entrada (Vendas, Lucro e Detalhes) |
| Visão de Vendas | KPIs, evolução mensal, matriz de vendas por país e produto, vendas por segmento e por produto |
| Análise de Lucro | Cascata por trimestre, árvore de decomposição, lucro por segmento e por produto |
| **Detalhes: TOP N e Dispersão** | TOP 3 produtos, principais países (vendas e lucro), TOP 5 meses e dispersão de unidades × vendas por mês |
| **Detalhes: Agrupamentos e Faixas** | Vendas por semestre, histograma por faixa de unidades, vendas por grupo de segmento e por região |

As duas páginas de detalhes respondem ao que o desafio pede ("até duas páginas podem compor o que é pedido"). Os visuais foram organizados pensando em como o cliente consome a informação:
1. **Cartões no topo** dão a resposta direta: quais são os TOP 3, qual é o país líder, qual é o melhor mês.
2. **Os gráficos abaixo** explicam essa resposta.
3. A grade na **proporção áurea** é a mesma das outras páginas.

---

## 🔎 Visuais pedidos no desafio

| Pedido | Visual | Como foi feito |
| --- | --- | --- |
| Visuais sobre os TOP 3 produtos | Barras "TOP 3 produtos em vendas" e cartões com os nomes e a participação | Medidas com `TOPN` + `KEEPFILTERS` e `CONCATENATEX` |
| Principais países em vendas e/ou lucro | Barras com **Vendas** e **Lucro** por país, ordenadas, e o cartão "País líder em vendas" | Duas medidas no mesmo visual e `TOPN(1, ...)` no cartão |
| Dispersão de unidades vendidas e vendas por mês | **Gráfico de dispersão**: eixo X = unidades, eixo Y = vendas, um ponto por mês e tamanho = lucro | Mês Ano em Detalhes, medidas nos eixos e em Tamanho |
| Visuais de agrupamentos | Vendas por segmento coloridas pelo **grupo** (Grandes Contas × PMEs e Parceiros), por **região** (América do Norte × Europa) e por **semestre** | Colunas de agrupamento |
| Visuais de compartimentação | **Histograma**: quantidade de vendas por faixa de unidades (0 a 499, 500 a 999, …) | Coluna de compartimentos de 500 unidades |

---

## 🧮 Agrupamentos e compartimentos

Os agrupamentos e compartimentos foram feitos como **colunas calculadas em DAX**. É o mesmo resultado dos recursos *Novo grupo* e *Nova compartimentação* do Power BI, mas com a regra visível e versionada:

```DAX
Região =
SWITCH ( financials[Country],
    "Canada", "América do Norte", "Mexico", "América do Norte", "United States of America", "América do Norte",
    "France", "Europa", "Germany", "Europa",
    "Outros" )

Segmento (grupo) =
SWITCH ( financials[Segment],
    "Government", "Grandes Contas",
    "Enterprise", "Grandes Contas",
    "PMEs e Parceiros" )

-- Compartimentos de 500 unidades
Faixa de Unidades          = INT ( financials[Units Sold] / 500 ) * 500
Faixa de Unidades (rótulo) = FORMAT ( financials[Faixa de Unidades], "#,0" ) & " a "
                             & FORMAT ( financials[Faixa de Unidades] + 499, "#,0" )   -- ordenada pela faixa numérica
```

Na tabela de datas, o **semestre** foi criado no Power Query (`1º Semestre` / `2º Semestre` e `Ano Semestre`).

## 📐 Medidas analíticas

```DAX
Qtd Vendas                  = COUNTROWS ( financials )
Ticket Médio                = DIVIDE ( [Total Vendas], [Qtd Vendas] )
Média de Unidades por Venda = AVERAGE ( financials[Units Sold] )

Vendas TOP 3 Produtos =
VAR _top = TOPN ( 3, ALLSELECTED ( financials[Product] ), [Total Vendas] )
RETURN CALCULATE ( [Total Vendas], KEEPFILTERS ( _top ) )

TOP 3 Produtos =
CONCATENATEX ( TOPN ( 3, ALLSELECTED ( financials[Product] ), [Total Vendas] ),
               financials[Product], ", ", [Total Vendas], DESC )

% Vendas TOP 3 Produtos =
DIVIDE (
    CALCULATE ( [Total Vendas], TOPN ( 3, ALLSELECTED ( financials[Product] ), [Total Vendas] ) ),
    CALCULATE ( [Total Vendas], ALLSELECTED ( financials[Product] ) ) )

País com Mais Vendas = CONCATENATEX ( TOPN ( 1, ALLSELECTED ( financials[Country] ), [Total Vendas] ), financials[Country], ", " )
Mês com Mais Vendas  = CONCATENATEX ( TOPN ( 1, ALLSELECTED ( dCalendario[Mês Ano], dCalendario[Ano Mês Num] ), [Total Vendas] ), dCalendario[Mês Ano], ", " )

Vendas TOP 5 Meses =
VAR _top = TOPN ( 5, ALLSELECTED ( dCalendario[Mês Ano], dCalendario[Ano Mês Num] ), [Total Vendas] )
RETURN CALCULATE ( [Total Vendas], KEEPFILTERS ( _top ) )

Vendas América do Norte = CALCULATE ( [Total Vendas], financials[Região] = "América do Norte" )
Vendas Europa           = CALCULATE ( [Total Vendas], financials[Região] = "Europa" )
```

Nos meses, o `ALLSELECTED` inclui também a coluna de ordenação (`Ano Mês Num`). Sem ela, o filtro dessa coluna continua ativo em cada linha do gráfico e todos os meses aparecem como TOP 5.

Com `ALLSELECTED`, os TOP N **respeitam os filtros** de ano, segmento e país. Por exemplo, ao filtrar só a Alemanha, o visual mostra os 3 produtos que mais vendem lá.

Medidas base: Total Vendas, Lucro Total, Margem de Lucro, Unidades Vendidas, Custo (COGS), Total Descontos e % Desconto.

---

## 📁 Estrutura

```
├── dados/Financial Sample.xlsx
├── RelatorioDataAnalytics.pbip             # abre o projeto no Power BI Desktop
├── RelatorioDataAnalytics.Report/          # páginas, visuais, botões e indicadores
└── RelatorioDataAnalytics.SemanticModel/   # modelo em TMDL (colunas de grupo e compartimento, medidas)
```

## ▶️ Como abrir
1. Abra `RelatorioDataAnalytics.pbip` no Power BI Desktop.
2. Clique em **Página Inicial → Atualizar**.
3. Se o repositório estiver em outra pasta, ajuste o parâmetro **CaminhoArquivo** em **Transformar dados → Editar parâmetros**.
4. No Desktop, use **Ctrl + clique** nos botões do menu.
