# UC1 — Abrir bilhete — `POST /bilhetes`

Body JSON `{"placa": "...", "entrada": "..."?}`.

**Ordem de validação:** (1) placa → (2) entrada → (3) conflito.

- **AC-1.1** Placa válida, sem `entrada` → **201** `{"id","placa","entrada","status":"aberto"}`, `entrada` = agora, id sequencial.

## UC2 — Encerrar — `POST /bilhetes/{id}/encerramento`

- **AC-2.1** Bilhete aberto → **200** com exatamente `{"id","placa","entrada","saida","minutos","valor_centavos"}`; bilhete passa a `encerrado`.

## UC3 — Ativos — `GET /bilhetes/ativos`
- **AC-3.1** **200** com array de bilhetes `aberto` (`id, placa, entrada, status`). Nenhum → `[]`.

## UC4 — Relatório diário — `GET /relatorios/diario?data=AAAA-MM-DD`
- **AC-4.1** `data` ausente, formato diferente de `AAAA-MM-DD` ou data impossível (`2026-02-30`) → **422** `data_invalida`.

## UC5 — Cancelar — `POST /bilhetes/{id}/cancelamento`
- **AC-5.1** Cancelado não conta no faturamento nem no tempo médio; a placa fica livre.

## UC6 — Histórico — `GET /bilhetes?placa=ABC1D23`
- **AC-6.1** **200** com **todos** os bilhetes da placa (aberto, encerrado, cancelado), `entrada` decrescente, empate `id` decrescente.
