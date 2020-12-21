#### Instalando Extensões

A consulta a seguir permite visualizar quais extensões estão disponíveis para serem instaladas:

```sql
select * from pg_available_extensions();
```

#### Exemplo

A consulta a seguir faz a instalação da extensão 'unaccent'

```sql
create extension unaccent;
```

Após a instalação, é possível utilizar a extensão em qualquer sessão iniciada naquele banco, conforme documentação da extensão instalada.

```sql
select unaccent('café');
-- Retorno: 'cafe'
```