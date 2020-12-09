#### LIMPA INSCRIÇÃO ESTADUAL DUPLICADA

Esse arqjob seta o campo 'inscricao_estadual' para NULL nos casos que existem duplicidade, a fim de sanar a inconsistência do envio de fornecedores. Ambos cadastros ficaram com o campo nulo.

```sql
UPDATE compras.credores
SET inscricao_estadual = NULL
WHERE i_credores in (
    SELECT a.i_credores
    FROM compras.credores a
    WHERE a.inscricao_estadual IS NOT NULL
    AND EXISTS(SELECT 1 
                FROM compras.credores b 
                WHERE a.inscricao_estadual = b.inscricao_estadual 
                AND a.i_credores <> b.i_credores)
)
```