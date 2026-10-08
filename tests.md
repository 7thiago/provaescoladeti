# Tests — casos de borda por regra

Convenção: `entrada = agora − N min` (usar o campo `entrada` no `POST /bilhetes`, depois encerrar).
Cada linha é uma placa nova. Valores com tarifa 600, fração 30 (= 300), teto 5000, tolerância 10.

## R1 — Tolerância (UC7): ≤ 10 min grátis; > 10 cobra desde o 1º minuto

| N (min) | `minutos` | `valor_centavos` | Por quê |
| --- | --- | --- | --- |
| 0 | 0 | 0 | instantâneo |
| 9 | 9 | 0 | dentro |
| 10 | 10 | 0 | **exatamente** a tolerância |
| 11 | 11 | **300** | +1 min: cobra 1 fração inteira (não descontar 10) |

## R2 — Fração de 30 min, arredondando p/ cima (exata = 1; +1 min = próxima)

| N (min) | frações | `valor_centavos` |
| --- | --- | --- |
| 30 | 1 | 300 |
| 31 | 2 | 600 |
| 60 | 2 | 600 |
| 61 | 3 | 900 |
| 95 | 4 | 1200 |
| 90 | 3 | 900 |
| 91 | 4 | 1200 |

## R3 — Teto diário (5000) 

> [!WARNING]
> Teto = 5000. 16 frações = 4800 (abaixo). 17 frações = 5100 → **5000**.

| N (min) | frações | calculado | `valor_centavos` |
| --- | --- | --- | --- |
| 480 | 16 | 4800 | 4800 |
| 481 | 17 | 5100 | **5000** |
| 510 | 17 | 5100 | 5000 |
| 1500 | 50 | 15000 | 5000 |
| 4320 | 144 | 43200 | 5000 |

## R4 — Dinheiro inteiro

- Toda `valor_centavos`, `faturamento_centavos` e `tempo_medio_minutos` é `int` no JSON (sem `.`, sem `1200.0`).
- Resposta de encerramento **não** contém a chave `valor`.

## R5 — Relatório diário (UC4), data `2026-10-05`

| Cenário | total | faturamento | tempo_medio |
| --- | --- | --- | --- |
| Dia sem bilhetes | 0 | 0 | 0 |
| 1 encerrado de 95 min | 1 | 1200 | 95 |
| encerrados de 30 e 31 min (média 30,5) | 2 | 900 | **31** (0,5 sobe) |
| encerrados de 30 e 30 e 31 (média 30,33) | 3 | 1200 | 30 |
| encerrados 10 e 11 (média 10,5) | 2 | 300 | **11** |
| 1 encerrado + 1 aberto + 1 cancelado no dia | 3 | só do encerrado | só do encerrado |
| Bilhete com entrada em outro dia | não conta | — | — |

Fórmula conferida: média `(2*soma + n) // (2*n)`: soma 61, n 2 → (122+2)//4 = 31.

## R6 — Conflitos e estados (UC5, UC8, erros)

| Passos | Esperado |
| --- | --- |
| Abrir ABC1D23 duas vezes seguidas | 1º 201; 2º **409** `bilhete_em_aberto` |
| Abrir → encerrar → abrir mesma placa | 201, 200, **201** (novo id) |
| Abrir → cancelar → abrir mesma placa | 201, 200, **201** |
| Encerrar duas vezes | 1º 200; 2º **409** `bilhete_ja_encerrado` |
| Cancelar bilhete encerrado | **409** `bilhete_nao_aberto` |
| Cancelar duas vezes | 2º **409** `bilhete_nao_aberto` |
| Cancelar → resposta | `status: "cancelado"`, sem `saida` nem `valor_centavos` |
| Encerrar id 9999 / cancelar id 9999 | **404** `bilhete_nao_encontrado` |
| Encerrar `abc` (id não numérico) | **404** `bilhete_nao_encontrado` |

## R7 — Validação de entrada

| Requisição | Esperado |
| --- | --- |
| `{"placa": "ABC1D23"}` | 201, id 1 no 1º bilhete |
| `{}` / body vazio / JSON quebrado | 422 `placa_invalida` |
| `"abc1d23"` (minúscula) | 422 `placa_invalida` |
| `"ABC1D2"` (6) / `"ABC1D234"` (8) / `"ABC-D23"` | 422 `placa_invalida` |
| `{"placa": 1234567}` | 422 `placa_invalida` |
| `entrada: "ontem"` | 422 `entrada_invalida` |
| `entrada: "2026-10-05T10:00:00"` (sem fuso) | 422 `entrada_invalida` |
| `entrada: "2026-10-05T10:00:00Z"` | 201, devolvida como `2026-10-05T07:00:00-03:00` |
| placa inválida **e** entrada inválida | 422 `placa_invalida` (placa vem primeiro) |
| placa ocupada **e** entrada inválida | 422 `entrada_invalida` |
| `/relatorios/diario?data=05/10/2026` ou ausente ou `2026-02-30` | 422 `data_invalida` |

## R8 — Listagens

| Cenário | Esperado |
| --- | --- |
| `GET /bilhetes/ativos` sem abertos | `[]` |
| 3 abertos | 3 itens, `entrada` decrescente; encerrados e cancelados ausentes |
| `GET /bilhetes?placa=ZZZ9Z99` nunca usada | `[]` |
| Placa com 1 encerrado + 1 cancelado + 1 aberto | 3 itens, mais recente primeiro; encerrado com `saida/minutos/valor_centavos` |
| `entrada` no futuro, encerrar | `minutos 0`, `valor_centavos 0` |
