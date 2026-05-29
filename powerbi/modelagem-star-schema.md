# Modelagem — Star Schema

## Diagrama do Modelo

```
                        ┌─────────────────┐
                        │   dCalendario   │  ← dimensão de tempo compartilhada
                        │   (isKey: Id Data)│
                        └────────┬────────┘
                      ativo/      └──── ativo (data)
                  (Data Pedido)         │
                        │               │
              ┌─────────┴────────┐  ┌───┴──────┐
              │    fVendas       │  │  fMetas  │
              │  (fato central)  │  │(fato sec.)│
              └─────────┬────────┘  └───┬──────┘
                        │               │
          ┌─────────────┼───────────────┘
          │             │
    ┌─────┴──┐   ┌──────┴──────┐   ┌───────────┐
    │dProdutos│  │ dVendedores │   │ dClientes │
    └─────────┘  └─────────────┘   └───────────┘
    ┌─────────┐  ┌─────────────┐   ┌───────────┐
    │dUnidades│  │   dStatus   │   │ dPagamento│
    └─────────┘  └─────────────┘   └───────────┘
```

## Tabelas

### Tabela Fato: `fVendas` (fato central)

Fonte: `Vendas.xlsx` → aba `fVendas`

| Coluna | Tipo | Descrição |
|---|---|---|
| Data Pedido | dateTime | FK ativa para dCalendario |
| Data Envio | dateTime | FK inativa para dCalendario |
| Num Venda | string | Identificador único da venda |
| Id Produto | int64 | FK para dProdutos |
| Id Vendedor | int64 | FK para dVendedores |
| Id Cliente | int64 | FK para dClientes |
| Id Unidade | int64 | FK para dUnidades |
| Id Status | int64 | FK para dStatus (= 1 = aprovado) |
| Id Pgto | int64 | FK para dPagamento |
| Qtde | int64 | Quantidade de itens |
| Valor Unit | double | Valor unitário de venda |
| Custo Unit | double | Custo unitário |
| Despesa Unit | int64 | Despesa unitária |
| Impostos Unit | double | Impostos por unidade |
| Comissão Unit | double | Comissão por unidade |

---

### Tabela Fato: `fMetas` (fato secundário)

Fonte: Pasta de arquivos `.xlsx` com unpivot de colunas mensais

| Coluna | Tipo | Descrição |
|---|---|---|
| data | dateTime | FK para dCalendario (mês/ano) |
| Id Vendedor | int64 | FK para dVendedores |
| Valor Meta | double | Meta do vendedor no período |

---

### Dimensão: `dCalendario`

Gerada via Power Query com `List.Dates` — 01/01/2018 por 1461 dias (4 anos). `dataCategory: Time`.

| Coluna | Tipo | Observação |
|---|---|---|
| Id Data | dateTime | PK (isKey = true) |
| Ano | int64 | |
| Nome do Mês | string | Ordenado por coluna Mês |
| Mês | int64 | 1–12 |
| Dia | int64 | |
| Semestre | string | "Sem1" / "Sem2" |
| Trimestre | string | "Tri1"–"Tri4" |
| Mes Abreviado | string | Ordenado por Mês |
| Mes-Ano | string | "Jan-2018" |
| Mes Ano Classificação | int64 | Ano * 100 + Mês (ordenação) |
| **Datas com Venda** | boolean | **Coluna calculada** (ver abaixo) |

**Hierarquia:** Ano → Mes Abreviado → Dia

**Coluna Calculada:**
```dax
// dCalendario[Datas com Venda]
dCalendario[Id Data] <= MAX(fVendas[Data Pedido])
```
Marca datas até a última com venda — usada para limitar YTD.

---

### Dimensão: `dProdutos`

| Coluna | Tipo | Descrição |
|---|---|---|
| Id Produto | int64 | PK |
| Produto | string | Nome do produto |
| Categoria | string | |
| Subcategoria | string | |
| Marca | string | |

---

### Dimensão: `dVendedores`

| Coluna | Tipo | Descrição |
|---|---|---|
| Id Vendedor | int64 | PK |
| Vendedor | string | Nome |
| URL Foto | string | URL da foto para visual HTML |
| Gerente | string | Nome do gerente |

---

### Dimensão: `dClientes`

| Coluna | Tipo | Descrição |
|---|---|---|
| Id Cliente | int64 | PK |
| Cliente | string | Nome |
| Cidade | string | Expandida via merge com dCidades |
| UF | string | Sigla do estado |
| Estado | string | Nome completo |

---

### Dimensão: `dStatus`

| Coluna | Tipo | Descrição |
|---|---|---|
| Id Status | int64 | PK |
| Status | string | "Aprovado" = 1 |

---

### Dimensão: `dUnidades`

| Coluna | Tipo | Descrição |
|---|---|---|
| Id Unidade | int64 | PK |
| Unidade | string | Nome da filial/unidade |

---

### Dimensão: `dPagamento`

| Coluna | Tipo | Descrição |
|---|---|---|
| Id Pagamento | int64 | PK |
| Forma de Pagamento | string | Descrição |

---

## Relacionamentos

| # | Origem (Fato) | Coluna | Destino (Dim) | Coluna | Cardinalidade | Ativo | Cross-filter |
|---|---|---|---|---|---|---|---|
| 1 | fVendas | Id Cliente | dClientes | Id Cliente | N:1 | ✅ | Único |
| 2 | fVendas | Id Unidade | dUnidades | Id Unidade | N:1 | ✅ | Único |
| 3 | fVendas | Id Status | dStatus | Id Status | N:1 | ✅ | Único |
| 4 | fVendas | Id Produto | dProdutos | Id Produto | N:1 | ✅ | Único |
| 5 | fVendas | Id Vendedor | dVendedores | Id Vendedor | N:1 | ✅ | Único |
| 6 | fVendas | **Data Pedido** | dCalendario | Id Data | N:1 | ✅ **ativo** | Único |
| 7 | fVendas | **Data Envio** | dCalendario | Id Data | N:1 | ❌ **inativo** | Único |
| 8 | fVendas | Id Pgto | dPagamento | Id Pagamento | N:1 | ✅ | Único |
| 9 | fMetas | Id Vendedor | dVendedores | Id Vendedor | N:1 | ✅ | Único |
| 10 | fMetas | data | dCalendario | Id Data | N:1 | ✅ | Único |

> **Nota:** O relacionamento inativo (Data Envio ↔ dCalendario) pode ser ativado pontualmente com `USERELATIONSHIP()` para análises por data de envio.
