Decomposição 
**Esqueleto**: `app.py` com constantes da variante, `agora()` (-03:00), helper `responder()`, servidor na porta **8005** | — | 
`python app.py` sobe; rota desconhecida → 404 `rota_nao_encontrada` em JSON |
*Validadores**: `validar_placa` (`^[A-Z0-9]{7}$`, só `str`), `parse_entrada` (exige fuso), `parse_data` (`AAAA-MM-DD` real) casos R7 retornam o código de erro correto |
**Verificação final**: script ou checklist manual com `curl` percorrendo `tests.md`; confirmar nenhum `float` e nenhuma chave `valor` todas as tabelas de `tests.md` conferem