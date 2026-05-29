# Medidas DAX — Dashboard Comercial

36 medidas organizadas em 4 pastas: `_Principais`, `_Temporais`, `Tooltips` e `HTML`.

---

## Pasta: `_Principais`

### Métricas Base

```dax
// Qtde Vendas
// Conta transações únicas — evita dupla contagem
DISTINCTCOUNT(fVendas[Num Venda])
```

```dax
// Positivação Clientes
// Qtde de clientes que efetuaram compras
DISTINCTCOUNT(fVendas[Id Cliente])
```

```dax
// Positivação Produtos
DISTINCTCOUNT(fVendas[Id Produto])
```

---

### Financeiras (com filtro de status aprovado)

Todas as métricas financeiras usam `dStatus[Id Status] = 1` para considerar apenas vendas aprovadas.

```dax
// Faturamento
// SUMX itera linha a linha multiplicando Qtde × Valor Unit
CALCULATE(
    SUMX(
        fVendas,
        fVendas[Qtde] * fVendas[Valor Unit]
    ),
    dStatus[Id Status] = 1
)
```

```dax
// Custo
CALCULATE(
    SUMX(
        fVendas,
        fVendas[Qtde] * fVendas[Custo Unit]
    ),
    dStatus[Id Status] = 1
)
```

```dax
// Despesa
CALCULATE(
    SUMX(
        fVendas,
        fVendas[Qtde] * fVendas[Despesa Unit]
    ),
    dStatus[Id Status] = 1
)
```

```dax
// Impostos
CALCULATE(
    SUMX(
        fVendas,
        fVendas[Qtde] * fVendas[Impostos Unit]
    ),
    dStatus[Id Status] = 1
)
```

```dax
// Comissao
CALCULATE(
    SUMX(
        fVendas,
        fVendas[Qtde] * fVendas[Comissão Unit]
    ),
    dStatus[Id Status] = 1
)
```

```dax
// Abatimento — soma de todos os custos/deduções
[Custo] + [Despesa] + [Impostos] + [Comissao]
```

```dax
// Resultado — lucro líquido
[Faturamento] - [Abatimento]
```

```dax
// Margem Percentual
// DIVIDE evita divisão por zero
DIVIDE([Resultado], [Faturamento])
```

---

### Análise de Participação e Ranking

```dax
// Margem Contribuição
// Participação do contexto atual sobre o total selecionado
// ALLSELECTED remove filtros da fVendas mas mantém filtros externos (slicers)
VAR vTotalFixo =
    CALCULATE(
        [Faturamento],
        ALLSELECTED(fVendas)
    )
RETURN
    DIVIDE(
        [Faturamento],
        vTotalFixo
    )
```

```dax
// Rank Vendedor Faturamento
// HASONEVALUE garante que o rank só aparece quando há um único vendedor no contexto
IF(
    HASONEVALUE(dVendedores[Vendedor]),
    RANKX(
        ALL(dVendedores),
        [Faturamento]
    )
)
```

```dax
// Rank Cidade Faturamento
RANKX(
    ALL(dClientes[Cidade]),
    [Faturamento]
)
```

---

### Cards com Formatação Abreviada

Padrão reutilizável para exibir valores grandes de forma legível (k, Mi, Bi):

```dax
// Faturamento Card
VAR vValor   = [Faturamento]
VAR vBilhao  = 1000000000
VAR vMilhao  = 1000000
VAR vMil     = 1000
RETURN
    SWITCH(
        TRUE(),
        vValor >= vBilhao, ROUND(DIVIDE(vValor, vBilhao), 2) & " Bi",
        vValor >= vMilhao, ROUND(DIVIDE(vValor, vMilhao), 2) & " Mi",
        vValor >= vMil,    ROUND(DIVIDE(vValor, vMil), 2) & " k",
        vValor + 0
    )
```

```dax
// Resultado Card
VAR vValor   = [Resultado]
VAR vBilhao  = 1000000000
VAR vMilhao  = 1000000
VAR vMil     = 1000
RETURN
    SWITCH(
        TRUE(),
        vValor >= vBilhao, ROUND(DIVIDE(vValor, vBilhao), 2) & " Bi",
        vValor >= vMilhao, ROUND(DIVIDE(vValor, vMilhao), 2) & " Mi",
        vValor >= vMil,    ROUND(DIVIDE(vValor, vMil), 2) & " k",
        vValor + 0
    )
```

```dax
// Comissão Card
VAR vValor   = [Comissao]
VAR vBilhao  = 1000000000
VAR vMilhao  = 1000000
VAR vMil     = 1000
RETURN
    SWITCH(
        TRUE(),
        vValor >= vBilhao, ROUND(DIVIDE(vValor, vBilhao), 2) & " Bi",
        vValor >= vMilhao, ROUND(DIVIDE(vValor, vMilhao), 2) & " Mi",
        vValor >= vMil,    ROUND(DIVIDE(vValor, vMil), 2) & " k",
        vValor + 0
    )
```

```dax
// Faturamento Card com Format — variante usando FORMAT()
VAR vFaturamento = [Faturamento]
VAR vBilhao      = 1000000000
VAR vMilhao      = 1000000
VAR vMil         = 1000
RETURN
    SWITCH(
        TRUE(),
        vFaturamento >= vBilhao, FORMAT(vFaturamento, "0,0,, Bi"),
        vFaturamento >= vMilhao, FORMAT(vFaturamento, "0,0, mi"),
        vFaturamento > vMil,     FORMAT(vFaturamento, "0,0, k"),
        vFaturamento
    )
```

---

### Metas

```dax
// Meta
SUM(fMetas[Valor Meta])
```

```dax
// Porcentagem Meta
DIVIDE([Faturamento], [Meta], 0)
```

```dax
// Diferença Faturamento x Meta (R$)
[Faturamento] - [Meta]
```

```dax
// Meta Cor — retorna hex color para formatação condicional dinâmica
// Azul = atingiu meta, Amarelo = acima de 80%, Vermelho = abaixo de 80%
VAR vPorcentagem = [Porcentagem Meta]
RETURN
    SWITCH(
        TRUE(),
        vPorcentagem >= 1,   "#59B7C8",   -- azul: meta atingida
        vPorcentagem > 0.8,  "#897D2E",   -- amarelo: quase lá
        "#C54339"                          -- vermelho: abaixo de 80%
    )
```

---

## Pasta: `_Temporais` (Time Intelligence)

Padrão utilizado nas medidas LY: `VAR vUltimaDataComVenda` + `FILTER(VALUES(...))` para cortar o período na última data com dados reais, evitando comparações com o futuro.

```dax
// Faturamento Acumulado (YTD)
// CALCULATETABLE com DATESYTD filtrado pela coluna calculada Datas com Venda
CALCULATE(
    [Faturamento],
    CALCULATETABLE(
        DATESYTD(dCalendario[Id Data]),
        dCalendario[Datas com Venda] = TRUE()
    )
)
```

```dax
// Faturamento Acumulado MTD
CALCULATE(
    [Faturamento],
    DATESMTD(dCalendario[Id Data])
)
```

```dax
// Faturamento LY (Last Year)
// Padrão com VAR para limitar o período ao último dado real antes de SAMEPERIODLASTYEAR
VAR vUltimaDataComVenda =
    CALCULATE(
        MAX(fVendas[Data Pedido]),
        ALLSELECTED(dCalendario)
    )
VAR vTabela =
    FILTER(
        VALUES(dCalendario[Id Data]),
        dCalendario[Id Data] <= vUltimaDataComVenda
    )
RETURN
    CALCULATE(
        [Faturamento],
        SAMEPERIODLASTYEAR(vTabela)
    )
```

```dax
// Faturamento YoY %
DIVIDE(
    [Faturamento] - [Faturamento LY],
    [Faturamento LY]
) + 0
```

```dax
// Margem LY
VAR vUltimaDataComVenda =
    CALCULATE(
        MAX(fVendas[Data Pedido]),
        ALLSELECTED(dCalendario)
    )
VAR vTabela =
    FILTER(
        VALUES(dCalendario[Id Data]),
        dCalendario[Id Data] <= vUltimaDataComVenda
    )
RETURN
    CALCULATE(
        [Margem Percentual],
        DATEADD(vTabela, -1, YEAR)
    )
```

```dax
// Margem YoY %
DIVIDE(
    [Margem Percentual] - [Margem LY],
    [Margem LY]
)
```

```dax
// Resultado LY
VAR vUltimaDataComVenda =
    CALCULATE(
        MAX(fVendas[Data Pedido]),
        ALLSELECTED(dCalendario)
    )
VAR vTabela =
    FILTER(
        VALUES(dCalendario[Id Data]),
        dCalendario[Id Data] <= vUltimaDataComVenda
    )
RETURN
    CALCULATE(
        [Resultado],
        DATEADD(vTabela, -1, YEAR)
    )
```

```dax
// Resultado YoY %
DIVIDE(
    [Resultado] - [Resultado LY],
    [Resultado LY]
)
```

---

## Pasta: `Tooltips`

```dax
// Circulo Farol — caractere Unicode para semáforo visual
UNICHAR(11044)
```

```dax
// Texto Tooltip Categoria — texto dinâmico baseado no filtro ativo
"Faturamento por: " & SELECTEDVALUE(dPagamento[Forma de Pagamento])
```

---

## Pasta: `HTML` (Visuais HTML renderizados no Power BI)

Medidas que retornam HTML+CSS renderizados via visual HTML/SVG do Power BI.

```dax
// Vendedor HTML — foto com animação CSS de entrada em espiral
VAR vVendedorSelecionado = SELECTEDVALUE(dVendedores[URL Foto])
RETURN
"
<style>
    img {
        position: relative;
        width: 100vw;
        height: 100vh;
        animation: swirl-in-fwd 1.5s ease-out both;
    }
    @keyframes swirl-in-fwd {
        0%   { transform: rotate(-540deg) scale(0); opacity: 0; }
        100% { transform: rotate(0) scale(0.8);    opacity: 1; }
    }
</style>
<img src='" & vVendedorSelecionado & "'>
"
```

```dax
// HTML Rank Cidade — layout vertical
VAR vFaturamento = [Faturamento Card]
VAR vMargem      = ROUND([Margem Percentual] * 100, 2)
VAR vCidade      = SELECTEDVALUE(dClientes[Cidade], "Selecione")
RETURN
"
<style>
    .cidade, .faturamento, .margem {
        position: relative; width: 100vw; font-size: 14vw;
    }
    .cidade      { color: #ddd; }
    .faturamento { color: #59B7C8; }
    .margem      { color: #FD8D42; }
</style>
<div class='cidade'>      " & vCidade      & " </div>
<div class='faturamento'> R$ " & vFaturamento & " </div>
<div class='margem'>      " & vMargem      & "% </div>
"
```

```dax
// Html Rank Cidade Horizontal — flexbox
VAR vFaturamento = [Faturamento Card]
VAR vMargem      = ROUND([Margem Percentual] * 100, 2) + 0
VAR vCidade      = SELECTEDVALUE(dClientes[Cidade], "Selecione")
RETURN
"
<style>
    div { display: flex }
    .cidade, .faturamento, .margem { position: relative; color: #ddd; font-size: 5vw; }
    .cidade      { width: 40vw; }
    .faturamento { width: 40vw; color: #59B7C8; }
    .margem      { color: #FD8D42; font-weight: bold; width: 10vw; }
</style>
<div class='cidade'>      " & vCidade      & " </div>
<div class='faturamento'> R$ " & vFaturamento & " </div>
<div class='margem'>      " & vMargem      & "% </div>
"
```

```dax
// Html Rank Cidade v2 — cartão interativo com hover animation (mais sofisticado)
// Ao passar o mouse: cidade desliza para direita, valores deslizam para esquerda (transition: 1s)
VAR vFaturamento = [Faturamento Card]
VAR vMargem      = ROUND([Margem Percentual] * 100, 2) + 0
VAR vCidade      = SELECTEDVALUE(dClientes[Cidade], "Selecione")
RETURN
"
<style>
    .cartao {
        position: relative; width: 100vw; height: 100vh; background: #411C5200;
    }
    .cartao .cidade {
        position: absolute; width: 30%; left: 8%; top: 50%;
        transform: translateY(-50%);
        font-family: Segoe UI; font-size: 7vw; color: #FFFFFF;
        transition: all 1s;
    }
    .cartao:hover .cidade { left: 60%; }
    .cartao .faturamento,
    .cartao .margem {
        position: absolute; width: 45%; right: 3%;
        font-family: Segoe UI; font-size: 7vw;
        display: flex; align-items: center; justify-content: center;
        transition: all 1s;
    }
    .cartao .faturamento { top: 10%; color: #59B7C8; }
    .cartao .margem      { bottom: 10%; color: #D1783C; }
    .cartao:hover .faturamento,
    .cartao:hover .margem { right: 55%; }
    .cartao .linha {
        position: absolute; width: 20%; top: 50%; left: 38%;
        border: 1px solid rgba(255,255,255,0.1);
        transform: rotate(-90deg);
    }
</style>
<div class='cartao'>
    <div class='cidade'>      " & vCidade      & " </div>
    <div class='faturamento'> R$ " & vFaturamento & " </div>
    <div class='margem'>      " & vMargem      & "% </div>
    <div class='linha'></div>
</div>
"
```
