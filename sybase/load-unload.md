#### Unload

```
select * from bethadba.controle_migracao_registro order by 1 
to 'C:\kaio\registro.txt' format ascii delimited by '|'
```

#### Load

```
load into table bethadba.controle_migracao_registro
from 'C:\kaio\registro.txt' format 'ascii' delimited by '|';
commit;
```