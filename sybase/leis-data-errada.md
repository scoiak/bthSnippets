#### Leis com data incorreta

Corrige leis com data sanção maior publicação ou divulgação

```sql
SELECT  atos.num_ato as numeroAtoCompleto,
        dateformat(isnull(atos.dt_publicacao, atos.dt_vigorar, atos.dt_inicial), 'yyyy-mm-dd') as dataSancao,
        dateformat(isnull(atos.dt_publicacao, '1900-01-01'), 'yyyy-mm-dd') as dataPublicacao,
        dateformat(isnull(atos.dt_vigorar, atos.dt_publicacao, atos.dt_inicial), 'yyyy-mm-dd') as dataVigorar
FROM bethadba.atos
WHERE isnull(year(atos.dt_publicacao),year(atos.dt_inicial)) >= 2013
AND (dataSancao > dataVigorar OR dataSancao > dataPublicacao)
```