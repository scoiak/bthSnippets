#### INSERE ITENS PENDENTES PROPOSTA PARTICIPANTE 

Cria procedure temporária que realiza a rotina de unificação de credores

```sql
CREATE TEMPORARY PROCEDURE unificarCredoresSapo (IN p_credorAtual INT, IN p_novoCredor INT, IN p_entidade INT)
BEGIN
    -- Declaração de variáveis utilizadas durante a lógica
    DECLARE sql_attTabelasGerais     LONG VARCHAR;
    DECLARE sql_attContasCredores    LONG VARCHAR;
    DECLARE sql_attCredoresAdic      LONG VARCHAR;
    DECLARE sql_attContasDadosAdic   LONG VARCHAR;
    DECLARE sql_attCredores          LONG VARCHAR;

    -- Cria SQL para tratamento da unificação da tabela 'sapo.contas_credores'
    SET sql_attContasCredores = 
        'IF NOT EXISTS(SELECT i_credores 
                       FROM sapo.contas_credores 
                       WHERE i_entidades = '|| p_entidade ||
                        ' AND i_credores = '|| p_novoCredor || ') 
         THEN
            UPDATE sapo.contas_credores 
            SET i_credores = ' || p_novoCredor || 
            ' WHERE i_entidades = '|| p_entidade ||
            '  AND i_credores = '|| p_credorAtual ||';
        ELSE
            DELETE FROM sapo.contas_credores 
            WHERE i_entidades = '|| p_entidade ||
            ' AND i_credores =' || p_credorAtual|| ';                                 
        END IF;';

    -- Cria SQL para tratamento da unificação da tabela 'sapo.credores_adic'
    SET sql_attCredoresAdic =
        'IF NOT EXISTS(SELECT i_credores 
                       FROM sapo.credores_adic 
                       WHERE i_entidades =' || p_entidade ||
                        ' AND i_credores =' || p_novoCredor || ') 
        THEN
             UPDATE sapo.credores_adic 
             SET i_credores = ' || p_novoCredor ||
             ' WHERE i_entidades = ' || p_entidade ||
                ' AND i_credores =' || p_credorAtual || ';
        ELSE
            DELETE FROM sapo.credores_adic 
            WHERE i_entidades =' || p_entidade ||
            ' AND i_credores =' || p_credorAtual || ';                                
        END IF;';

    -- Cria SQL para tratamento da unificação da tabela 'sapo.contas_dados_adic'
    set sql_attContasDadosAdic = 
        'IF NOT EXISTS(SELECT cod_relac 
                       FROM sapo.contas_dados_adic 
                       WHERE i_entidades = ' ||p_entidade||
                        ' AND tipo_relac = 5 
                          AND cod_relac = ' || p_novoCredor || ') 
        THEN
           UPDATE sapo.contas_dados_adic 
           SET cod_relac = ' || p_novoCredor ||
           ' WHERE i_entidades =' || p_entidade ||
           ' AND tipo_relac = 5 
             AND cod_relac =' || p_credorAtual || ';
        ELSE
           DELETE FROM sapo.contas_dados_adic 
           WHERE i_entidades =' || p_entidade ||
           ' AND tipo_relac = 5 
             AND cod_relac =' || p_credorAtual || ';                          
        END IF;'; 

    -- Cria SQL para tratamento da unificação da tabela 'sapo.credores'
    set sql_attCredores = 
        'IF NOT EXISTS(SELECT i_credores 
                       FROM sapo.credores 
                       WHERE i_entidades = ' || p_entidade ||
                       ' AND i_credores = ' || p_novoCredor || ') 
        THEN
            UPDATE sapo.credores 
            SET i_credores = ' || p_novoCredor ||
            ' WHERE i_entidades = ' || p_entidade ||
            ' AND i_credores = ' || p_credorAtual || ';
        ELSE
            DELETE FROM sapo.credores 
            WHERE i_entidades = ' || p_entidade ||
                ' AND i_credores =' || p_credorAtual || ';                                
        END IF;';

    -- Cria tabela temporária que armazena a listagem de tabelas 
    -- no sistemas que referenciam a coluna 'i_credores';      
    SELECT user_name || '.' || table_name AS nome 
    INTO #tempRefCredores
    FROM systable 
        KEY JOIN syscolumn   
        KEY JOIN sysuserperms
    WHERE column_name = 'i_credores' 
        AND LEFT(table_name,5) NOT IN ('audit') 
        AND user_name NOT IN ('desbth','tecbth_rgriodosul') 
        AND LEFT(table_name,2) NOT IN ('vw') 
        AND table_name NOT IN ('senhas_credores', 'contas_credores', 'credores_dctos',
                               'credores', 'credores_adic', 'credores_avaliacao',
                               'credores_consorcio', 'credores_situacao', 'credores_socios',
                               'credores_wbc');
    
    -- Desabilita chamada de Triggers para as mesmas não
    -- serem chamadas erroneamente durante a operação
    CALL bethadba.pg_setoption('fire_triggers','off');
    COMMIT;
    
    -- Habilita necessidade de commit para evitar que interrupções
    -- na execução não comprometam os dados no banco
    CALL bethadba.pg_setoption('wait_for_commit','on');  
    COMMIT;

    -- Inicializa o cursor
    FOR credores AS cur_credores CURSOR 
    FOR
        SELECT nome AS curs_tabelaAtual
        FROM #tempRefCredores 
        ORDER BY nome 
    DO
        -- Faz a atualização de todos os credores nas tabelas que possuem alguma referência '1:n'
        IF curs_tabelaAtual <> 'sapo.contas_dados_adic' 
        THEN
              SET sql_attTabelasGerais = 'UPDATE ' || curs_tabelaAtual || 
                                         ' SET i_credores = ' || p_novoCredor ||
                                         ' WHERE i_entidades = ' || p_entidade||
                                            ' AND i_credores = ' || p_credorAtual;
              MESSAGE 'Unificando tabela ' || curs_tabelaAtual || ' - ' || p_credorAtual || ' >--> ' || p_novoCredor TO CLIENT; 
              EXECUTE IMMEDIATE sql_attTabelasGerais;
        END IF;
       
        -- Faz a atualização da tabela 'sapo.contas_credores'
        MESSAGE 'Unificando tabela sapo.contas_credores - ' || p_credorAtual || ' >--> ' || p_novoCredor TO CLIENT;    
        EXECUTE IMMEDIATE sql_attContasCredores;
          
        -- Faz a atualização da tabela 'sapo.credores_adic'
        MESSAGE 'Unificando tabela sapo.credores_adic - ' || p_credorAtual || ' >--> ' || p_novoCredor TO CLIENT;    
        EXECUTE IMMEDIATE sql_attCredoresAdic;

        -- Faz a atualização da tabela 'sapo.contas_dados_adic'
        MESSAGE 'Unificando tabela sapo.contas_dados_adic - ' || p_credorAtual || ' >--> ' || p_novoCredor TO CLIENT;    
        EXECUTE IMMEDIATE sql_attContasDadosAdic;

        -- Faz a atualização da tabela 'sapo.credores'
        MESSAGE 'Unificando tabela sapo.credores - ' || p_credorAtual || ' >--> ' || p_novoCredor TO CLIENT;    
        EXECUTE IMMEDIATE sql_attCredores;
    END FOR;

    -- Habilita chamada de Triggers 
    CALL bethadba.pg_setoption('fire_triggers','on');
    COMMIT;
    
    -- Desabilita necessidade de commit para evitar que interrupções
    CALL bethadba.pg_setoption('wait_for_commit','off');  
    COMMIT;

    -- Finaliza a procedure com sucesso
    RETURN 0;
END;

BEGIN
    -- Seta o parâmetro de entidade utilizada para cada chamada
    DECLARE @entidade INT;
    SET @entidade = 1;

    -- Chamadas das procedures de unificação
    CALL unificarCredoresSapo(8734, 9370, @entidade);
    CALL unificarCredoresSapo(6477, 6806, @entidade);
    CALL unificarCredoresSapo(7914, 7968, @entidade);
    CALL unificarCredoresSapo(6956, 12012, @entidade);
END;
```