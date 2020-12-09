#### Verifica total de registros/erros por cadastro enviado

```sql
SELECT tipo_registro, 
       count(tipo_registro) as total_registros,
	   sum(case coalesce(id_gerado, 0) when 0 then 1 else 0 end) as erros
FROM public.controle_migracao_registro
where sistema = '300'
GROUP BY tipo_registro
ORDER BY 3 DESC, 1 asc
```

#### Verifica qual o erro de cada registro

```sql
SELECT cmr.tipo_registro, 
	   cmr.hash_chave_dsk,
	   cmro.id_integracao,
	   cmr.i_chave_dsk1,
	   cmr.i_chave_dsk2,
	   cmr.i_chave_dsk3,
	   cmro.mensagem_erro,
	   cmr.json_enviado 
FROM public.controle_migracao_registro cmr
inner join public.controle_migracao_registro_ocor cmro on (cmro.hash_chave_dsk = cmr.hash_chave_dsk)
where cmr.tipo_registro = 'afastamento'
and cmr.id_gerado is null
order by 7, 1, 4, 5
```

#### Atualiza hashes de determinado cadastro

```sql
update public.controle_migracao_registro 
set hash_chave_dsk = md5(concat(sistema, tipo_registro, i_chave_dsk1, i_chave_dsk2)) 
where tipo_registro = 'motivo-alteracao-salarial';
```

#### Insere registro manualmente na tabela de controle

```sql
insert into public.controle_migracao_registro (sistema, tipo_registro, hash_chave_dsk, descricao_tipo_registro, id_gerado, i_chave_dsk1, i_chave_dsk2) values
('300', 'tipo-afastamento', md5(concat('300', 'tipo-afastamento', '56', '65')), 'Cadastro de Tipo de Afastamento', 7309, '56', '65')
```

#### Verifica ultimos lotes enviados por tipos de registro

```sql
select * FROM public.controle_migracao_lotes where tipo_registro = 'periodo-aquisitivo-decimo-terceiro' order by data_hora_env desc
```
