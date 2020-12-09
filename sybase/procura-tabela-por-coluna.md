#### Busca tabela pelo nome da coluna

```
select mensagem_erro from bethadba.controle_migracao_registro_ocor 
where sistema != 333 and mensagem_erro is not null and tipo_registro = 'credores'
order by tipo_registro
```