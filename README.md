# Desafio MySQL - Transações, Procedures e Backup

Este repositório contém os scripts para o desafio de Banco de Dados:

- **Parte 1:** Transações com manipulação de dados.
- **Parte 2:** Transação com Procedure e controle de erros.
- **Parte 3:** Backup e Recovery usando `mysqldump`.

## Como rodar

1. Execute os scripts do MySQL na ordem:
   ```bash
   mysql -u root -p < parte1_transacoes.sql
   mysql -u root -p < parte2_procedure.sql

#### Parte1_transacoes.sql
Exemplo de **transação manual sem autocommit**, incluindo UPDATE e INSERT.

```sql
-- Desabilitar autocommit para iniciar transações manuais
SET autocommit = 0;

START TRANSACTION;

-- Exemplo: Atualizar o estoque de um produto
UPDATE produtos SET estoque = estoque - 5 WHERE id = 10;

-- Inserir registro de venda
INSERT INTO vendas (produto_id, quantidade, data_venda)
VALUES (10, 5, NOW());

-- Verificar se tudo está certo antes de confirmar
SELECT * FROM produtos WHERE id = 10;
SELECT * FROM vendas ORDER BY data_venda DESC LIMIT 5;

Parte2_procedure.sql
Transação dentro de uma procedure com tratamento de erro e SAVEPOINT.

sql
Copiar
Editar
DELIMITER $$

CREATE PROCEDURE realizar_venda(IN p_produto INT, IN p_qtd INT)
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    -- Em caso de erro, desfaz tudo
    ROLLBACK;
  END;

  START TRANSACTION;

  -- Criar um ponto de salvamento
  SAVEPOINT antes_estoque;

  -- Atualizar estoque
  UPDATE produtos SET estoque = estoque - p_qtd WHERE id = p_produto;

  -- Caso o estoque fique negativo, desfazer só a atualização de estoque
  IF (SELECT estoque FROM produtos WHERE id = p_produto) < 0 THEN
    ROLLBACK TO antes_estoque;
  ELSE
    -- Inserir venda
    INSERT INTO vendas (produto_id, quantidade, data_venda)
    VALUES (p_produto, p_qtd, NOW());
  END IF;

  COMMIT;
END $$

DELIMITER ;

Parte3_backup_recovery.md
Instruções para gerar e restaurar backup.

markdown
Copiar
Editar
# Backup e Recovery - MySQL

## Gerar Backup do banco ecommerce

Execute:
```bash
mysqldump -u root -p --databases ecommerce --routines --events > backups/ecommerce_backup.sql
--routines: inclui procedures e funções.
