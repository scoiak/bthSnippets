#### REPLICA CC DE UM ANO PARA OUTRO


A etapa "ConfiguracoesOrganogramasSQL" faz uma validação impeditiva que verifica a existência de centro de custos no exercício. Esse arqjob foi criado a fim de realizar um ajuste para sanar essa inconsistência. (O sistema possui a rotina 'Arquivos -> Orçamento -> Copiar mesmos dados para inicio de novo exercicio', porém a mesma só funciona para os exercícios seguintes.)

A regra consiste em replicar a configuração de CC de um ano de origem para ano destino, bastanto referenciar esses exercícios nos locais especificados. É importante verificar se a configuração de orgão/unidade de origem/destino são compatíveis. O sequencial do campo 'reduz_cus_ger' da tabela continua a contagem do último valor lido do cadastro mais atual do exercício vigente.

```sql
INSERT INTO compras.ccustos (
    SELECT
        i_ano           = 2002, // !! INSERIR AQUI O ANO DESTINO !! //
        i_ccusto        = origem.i_ccusto,
        i_orgaos        = origem.i_orgaos,
        i_unidades      = origem.i_unidades,
        cc_nome         = origem.cc_nome,
        cc_pess         = origem.cc_pess,
        reduz_cus_ano   = origem.reduz_cus_ano,
        reduz_cus_ger   = Number() + (SELECT MAX(cc.reduz_cus_ger) FROM compras.ccustos cc),
        endereco_cc     = origem.endereco_cc, 
        roteiro_cc      = origem.roteiro_cc, 
        cnpj_cc         = origem.cnpj_cc, 
        cep_cc          = origem.cep_cc, 
        sigla           = origem.sigla, 
        flag_compras    = origem.flag_compras, 
        data_alt        = origem.data_alt, 
        deletar         = origem.deletar, 
        i_entidades     = origem.i_entidades     
    FROM compras.ccustos origem
    WHERE origem.i_ano = 2003 // !! INSERIR AQUI O ANO ORIGEM !! //
)
```