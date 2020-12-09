#### Verifica total de registros/erros por cadastro enviado

```
SELECT tipo_registro, 
       count(tipo_registro) as total_registros,
	   sum(case coalesce(id_gerado, 0) when 0 then 1 else 0 end) as erros
FROM public.controle_migracao_registro
where sistema = '300'
GROUP BY tipo_registro
ORDER BY 3 DESC, 1 asc
```

#### Verifica qual o erro de cada registro

```
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

```
update public.controle_migracao_registro 
set hash_chave_dsk = md5(concat(sistema, tipo_registro, i_chave_dsk1, i_chave_dsk2)) 
where tipo_registro = 'motivo-alteracao-salarial';
```