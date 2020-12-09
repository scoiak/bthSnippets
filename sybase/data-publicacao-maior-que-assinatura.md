#### AJUSTA DATA PUBLICAÇÃO MENOR QUE DATA DE ASSINATURA

Essa rotina verifica e ajusta contratos que estão com a data de publicação do , contrato com uma data anterior a data de assinatura do mesmo, setando ambas, para ficar igual a data de assinatura.

```
CALL bethadba.pg_habilitartriggers('off');

UPDATE compras.publicacoes_contratos
SET data_publ = data_ass
FROM compras.publicacoes_contratos
KEY JOIN compras.contratos
WHERE publicacoes_contratos.data_publ < contratos.data_ass;

CALL bethadba.pg_habilitartriggers('on');
```