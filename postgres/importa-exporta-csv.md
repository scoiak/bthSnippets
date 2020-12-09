#### Exportar CSV

```
COPY public.controle_migracao_registro 
TO 'C:\meu_arquivo.csv'
DELIMITER '|'
CSV HEADER;
```

#### Importar CSV

```
COPY public.controle_migracao_registro 
FROM 'C:\meu_arquivo.csv'
DELIMITER '|'
CSV HEADER;
```