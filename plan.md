Plano de teste: 

Populando o banco direto no teste: 
O teste vai inserir o bilhete com a entrada desejada e depois só ira chamar o endpoint de saída/cálculo, evitando a poluição da API. 

Entrada opcional no body: 
Os testes funcionam por HTTP puro, sem mock nem acesso ao banco, e isso serve bem para testes de contrato e de aceitação.
