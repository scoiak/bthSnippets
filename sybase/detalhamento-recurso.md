#### Detalhamento recurso

Resolver situação onde o detalhamento dos recursos não foi gerado corretamente

```sql
INSERT INTO sapo.cfg_detalhe_recursos (i_codigo,i_cfg_recursos,i_entidades,descricao,i_tipo,i_digitos,separador)
VALUES (1,1,1,'IDuso',1, 1, null);

INSERT INTO sapo.cfg_detalhe_recursos (i_codigo,i_cfg_recursos,i_entidades,descricao,i_tipo,i_digitos,separador)
VALUES (2,1,1,'Grupo',2,1, null);  
    
INSERT INTO sapo.cfg_detalhe_recursos (i_codigo,i_cfg_recursos,i_entidades,descricao,i_tipo,i_digitos,separador)
VALUES (3,1,1,'Especificação TCE',5,2, null);  
   
INSERT INTO sapo.cfg_detalhe_recursos (i_codigo,i_cfg_recursos,i_entidades,descricao,i_tipo,i_digitos,separador)
VALUES (4,1,1,'Especificação',3,4, null);
```