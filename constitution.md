Constitution.md 

Constante | Valor 
`TARIFA_HORA_CENTAVOS` 600 
`FRACAO_MINUTOS` 30
`TETO_DIARIO_CENTAVOS` 5000 
`TOLERANCIA_MINUTOS` 10 
`PORTA_SERVICO` 8005 
`VALOR_FRACAO_CENTAVOS` (derivado) | 600 ÷ (60 ÷ 30) = 300


 1. **Validação antes de efeito.** Nenhum bilhete é criado/alterado se qualquer validação falhar.
2. **O contrato vence qualquer exemplo.** O enunciado traz um exemplo contraditório (`{"id": 7, "valor": 12.50}`):
é ignorado Chaves de resposta são exatamente as do contrato (`valor_centavos`, nunca `valor`).
