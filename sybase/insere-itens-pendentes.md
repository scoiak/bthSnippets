#### INSERE ITENS PENDENTES PROPOSTA PARTICIPANTE 

Saneamento de base para correção do erro 'Existe um ou mais participantes que não possuem , todas as propostas cadastradas para todos os itens do processo. Favor verificar e ajustar , antes de homologar o processo. Essa procedure busca para o processo apontado, quais participantes, não possuem algum dos itens cotados, e quando encontra, insere tal item com o valor zerado., Após executar para tal processo, deve-se enviar novamente as propostas, para em seguinte reenviar, o registro de atos finais.

```
CREATE TEMPORARY PROCEDURE insereItensPendentesCotacao(IN p_entidade INT, IN p_anoProc INT, IN p_nrProc INT)
BEGIN
    -- Variáveis utilizadas na procedure
    DECLARE modalidade              INT;
    DECLARE tipoObjeto              INT;
    DECLARE situacao                INT;
    DECLARE dtHomologProcesso       DATE;
    DECLARE dtDescredenciamento     VARCHAR(12);
    DECLARE sql_attPropostas        LONG VARCHAR;
    
    -- Desabilita triggers (usar com cautela somente se necessário)
    --CALL bethadba.pg_habilitartriggers('off'); COMMIT;

    -- Obtem dados do processo selecionado
    SET modalidade          = (SELECT modalidade FROM compras.processos WHERE i_entidades = p_entidade AND i_ano_proc = p_anoProc AND i_processo = p_nrProc);
    SET dtHomologProcesso   = (SELECT data_homolog FROM compras.processos WHERE i_entidades = p_entidade AND i_ano_proc = p_anoProc AND i_processo = p_nrProc);
    SET tipoObjeto          = (SELECT tipo_objeto FROM compras.processos WHERE i_entidades = p_entidade AND i_ano_proc = p_anoProc AND i_processo = p_nrProc);
    SET dtHomologProcesso   = (SELECT data_homolog FROM compras.processos WHERE i_entidades = p_entidade AND i_ano_proc = p_anoProc AND i_processo = p_nrProc);

    -- Configura variaveis específicas com base do tipo de objeto
    IF (tipoObjeto = 5 AND modalidade <> 99)
    THEN
        SET situacao = 11;

        IF dtHomologProcesso IS NOT NULL
        THEN
            SET dtDescredenciamento = '''' || dtHomologProcesso || '''';
        ELSE
            SET dtDescredenciamento = '''' || (SELECT dateformat(getdate(),'yyyy-mm-dd hh:mm:ss') FROM dummy) || '''';
        END IF;
    ELSE
        SET situacao = 4;
        SET dtDescredenciamento = 'null';
    END IF;

    MESSAGE 'tipoObjeto: ' || tipoObjeto TO CLIENT;
    MESSAGE 'dtDescredenciamento: ' || dtDescredenciamento TO CLIENT;
    MESSAGE 'dtHomologProcesso: ' || dtHomologProcesso TO CLIENT;

   -- Faz a busca dos credores do processo e verifica as cotações de cado um deles
    FOR credores AS cur_credores CURSOR
    FOR
        SELECT DISTINCT i_credores AS credorProcesso
        FROM compras.participantes_processos
        WHERE i_entidades = p_entidade 
        AND i_ano_proc = p_anoProc 
        AND i_processo = p_nrProc
    DO
        MESSAGE 'Ajustando os dados para o credor: ' || credorProcesso TO CLIENT;
        --SQL que insere os dados das propostas faltantes para cada credor com valor zerado
        SET sql_attPropostas = '
            INSERT INTO compras.participantes
            SELECT i_ano_proc, i_processo, i_credores, i_item, preco_unit_part, preco_total, qtde_cotada, situacao,
                   atual_objeto, nome_marca, ordem_clas, credenciado, vlr_descto, i_entidades, i_lotes, art_43_lcf_123_06,
                   art_44_lcf_123_06, dt_descredencia, observacao, percent_bdi_tce, percent_encargo_tce
            FROM (
                SELECT 
                    itens_processo.i_ano_proc as i_ano_proc, itens_processo.i_processo as i_processo, '|| credorProcesso ||' as i_credores, 
                    itens_processo.i_item as i_item, 0 as preco_unit_part, 0 as preco_total, 0 as qtde_cotada, '|| situacao ||' as situacao,
                    null as atual_objeto, null as nome_marca, 0 as ordem_clas, 0 as credenciado, null as vlr_descto,
                    itens_processo.i_entidades as i_entidades, null as i_lotes, ''A'' as art_43_lcf_123_06, ''A'' as art_44_lcf_123_06,
                    '|| dtDescredenciamento ||' as dt_descredencia, null as observacao, null as percent_bdi_tce, null as percent_encargo_tce
                FROM compras.itens_processo
                WHERE i_entidades = '|| p_entidade ||'
                AND i_ano_proc = '|| p_anoProc ||'
                AND i_processo = '|| p_nrProc ||'          
                AND NOT EXISTS(
                    SELECT *
                    FROM compras.participantes
                    WHERE i_entidades = itens_processo.i_entidades
                    AND i_ano_proc = itens_processo.i_ano_proc
                    AND i_processo = itens_processo.i_processo 
                    AND i_item = itens_processo.i_item 
                    AND i_credores = '|| credorProcesso ||'
                )
            ) AS tab';
        --MESSAGE sql_attPropostas TO CLIENT;
        EXECUTE IMMEDIATE sql_attPropostas;
    END FOR;

    -- Habilita triggers
    --CALL bethadba.pg_habilitartriggers('on'); COMMIT;     
END;

-- Exemplo chamada:
CALL insereItensPendentesCotacao(1, 2020, 26);
```