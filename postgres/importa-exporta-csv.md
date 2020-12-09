#### Exportar CSV

```sql
COPY public.controle_migracao_registro 
TO 'C:\meu_arquivo.csv'
DELIMITER '|'
CSV HEADER;
```

#### Importar CSV

```sql
COPY public.controle_migracao_registro 
FROM 'C:\meu_arquivo.csv'
DELIMITER '|'
CSV HEADER;
```