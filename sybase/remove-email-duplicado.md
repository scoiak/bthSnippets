#### REMOVE EMAILS DUPLICADOS DE FORNECEDORES

Essa procedure faz um tratamento no campo 'email' de fornecedores, que no desktop é um campo texto, e ocorre normalmente do usuário cadastrar múltiplos e-mails. A procedure realiza um tratamento para esses casos visando manter somente o primeiro email do campo. 

```sql
CREATE TEMPORARY PROCEDURE removeMultiplosEmailsFornecedores()
BEGIN
    -- Declaração de variáveis
    DECLARE sqlUpdate LONG VARCHAR;
    DECLARE emailAtualizado VARCHAR(30);

    -- Cria temporária com os emails a serem corrigidos
    SELECT i_credores, nome, email_fornecedor
    INTO #tabTempCredoresMultiplosEmails
    FROM compras.credores
    WHERE CHAR_LENGTH(email_fornecedor) - CHAR_LENGTH(REPLACE(email_fornecedor, '@', '')) > 1;

    -- Loop corrignido os emails necessários
    FOR credores AS cur_credores CURSOR FOR
        SELECT i_credores, email_fornecedor FROM #tabTempCredoresMultiplosEmails
    DO
        SET emailAtualizado = '';        
        -- Verifica possibilidades de separador de email, e caso encontrar, gera uma variável com o 
        -- primeiro email encontrado
        IF CHARINDEX('/', email_fornecedor) > 0 THEN
            SET emailAtualizado = SUBSTR(email_fornecedor, 0, CHARINDEX('/', email_fornecedor) - 1);
        END IF;

        IF CHARINDEX('|', email_fornecedor) > 0 THEN
            SET emailAtualizado = SUBSTR(email_fornecedor, 0, CHARINDEX('|', email_fornecedor) - 1);
        END IF;

        IF CHARINDEX(';', email_fornecedor) > 0 THEN
            SET emailAtualizado = SUBSTR(email_fornecedor, 0, CHARINDEX(';', email_fornecedor) - 1);
        END IF;

        IF  emailAtualizado = '' AND CHARINDEX(' ', email_fornecedor) > 0 THEN
            SET emailAtualizado = SUBSTR(email_fornecedor, 0, CHARINDEX(' ', email_fornecedor) - 1);
        END IF;
        
        -- Caso tenha encontrado, atualiza a tabela com o email formatado
        IF  emailAtualizado = '' THEN
            MESSAGE 'Não foi possível atualizar o email do credor ' || i_credores TO CLIENT;
        ELSE
            SET sqlUpdate = 'UPDATE compras.credores
                             SET email_fornecedor = ''' || REPLACE(emailAtualizado, ' ', '') || '''
                             WHERE CHAR_LENGTH(email_fornecedor) - CHAR_LENGTH(REPLACE(email_fornecedor, ''@'', '''')) > 1
                               AND i_credores = ' || i_credores;
            --MESSAGE sqlUpdate TO CLIENT;                                 
            EXECUTE IMMEDIATE sqlUpdate;
        END IF;         
    END FOR;
    COMMIT; 
END;

CALL removeMultiplosEmailsFornecedores;
```