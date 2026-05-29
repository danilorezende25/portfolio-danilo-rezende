# Dashboard Comercial — Power BI

Projeto de Business Intelligence desenvolvido no Power BI Desktop com modelagem Star Schema, medidas DAX avançadas e visuais HTML customizados.

## Índice

- [Modelagem — Star Schema](./modelagem-star-schema.md)
- [Medidas DAX](./medidas-dax.md)
- [Screenshots do Report](./screenshots/)

## Visão Geral do Projeto

| Item | Detalhe |
|---|---|
| Ferramenta | Power BI Desktop |
| Arquivo | Dashboard Comercial.pbix |
| Período dos dados | 2018 – 2021 |
| Tabelas | 10 (2 fatos + 7 dimensões + 1 medidas) |
| Relacionamentos | 10 (9 ativos + 1 inativo) |
| Medidas DAX | 36 |
| Compatibility Level | 1600 (Power BI V3) |

## Destaques Técnicos

- **Star Schema** com dois fatos (`fVendas` e `fMetas`) compartilhando dimensões — padrão Constellation Schema
- **Time Intelligence** com padrão `VAR + FILTER(VALUES(...))` para limitar YTD/LY à última data com venda
- **Medidas HTML** que renderizam visuais interativos com CSS animations e hover effects diretamente no Power BI
- **Formatação condicional dinâmica** via medida `Meta Cor` retornando hex colors por KPI
- **Coluna calculada** `Datas com Venda` em `dCalendario` para evitar acúmulo futuro no YTD
- **Dual relacionamento** em `dCalendario` ↔ `fVendas` (ativo por `Data Pedido`, inativo por `Data Envio`)
