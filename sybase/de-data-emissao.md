#### Despesas extras com data de emissão maior que data de vencimento

```
update sapo.despexs
set data_vencimento = data_emissao--dateadd(dd, 1, data_emissao)
where data_emissao > data_vencimento
and data_pagamento is null
and ano_exerc = 2019
```