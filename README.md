# Criando_Transacoes_Executando_Backup_Recovery_BD
Lidar com transações para executar modificações na base de dados
## TRANSAÇÕES

SET autocommit = 0;<br/>

START TRANSACTION;<br/>


-- Suponhamos que o cliente quer comprar o produto com ID 1<br/>

SET @cliente_id := 1;<br/>

SET @produto_id := 1;<br/>



SELECT quantidade FROM estoque WHERE Produto_idProdutoe = @produto_id AND elocation = 'LocalPadrao' FOR UPDATE;<br/>


UPDATE estoque SET quantidade = quantidade - 1 WHERE Produto_idProdutoe = @produto_id AND elocation = 'LocalPadrao';<br/>


INSERT INTO pedido (idpedidocliente, pedidostatus, descricao, Frete, idpagamento)<br/>
VALUES (@cliente_id, 'em processamento', 'Novo Pedido', 10, NULL);<br/>


SET @id_pedido := LAST_INSERT_ID();<br/>


INSERT INTO produto_pedido (Pedido_idPedido, Produto_idProduto, quantidade, ppstatus)<br/>
VALUES (@id_pedido, @produto_id, 1, 'disponivel');<br/>


UPDATE pedido SET pedidostatus = 'confirmado' WHERE idpedido = @id_pedido;<br/>


COMMIT;<br/>

-- Caso ocorra algum erro, desfaz a transação<br/>
-- ROLLBACK;  <br/>
## TRANSAÇÃO COM PROCEDURE
DELIMITER //<br/>

CREATE PROCEDURE RealizarCompra(<br/>
    IN clienteID INT,<br/>
    IN produtoID INT<br/>
)<br/>
BEGIN<br/>
    DECLARE quantidadeDisponivel INT;<br/>
    SET autocommit = 0;<br/>
    START TRANSACTION;<br/>

    -- Verificação de erro<br/>
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION<br/>
    BEGIN<br/>
        -- Caso ocorra algum erro, desfaz a transação<br/>
        ROLLBACK;<br/>
        SELECT 'Erro na transação. Rollback realizado.' AS Resultado;<br/>
    END;<br/>

    SELECT quantidade INTO quantidadeDisponivel FROM estoque WHERE Produto_idProdutoe = produtoID AND elocation = 'LocalPadrao' FOR UPDATE;<br/>

    IF quantidadeDisponivel > 0 THEN<br/>
        UPDATE estoque SET quantidade = quantidade - 1 WHERE Produto_idProdutoe = produtoID AND elocation = 'LocalPadrao';<br/>
        INSERT INTO pedido (idpedidocliente, pedidostatus, descricao, Frete, idpagamento)<br/>
        VALUES (clienteID, 'em processamento', 'Novo Pedido', 10, NULL);<br/>

        SET @id_pedido := LAST_INSERT_ID();<br/>


        INSERT INTO produto_pedido (Pedido_idPedido, Produto_idProduto, quantidade, ppstatus)<br/>
        VALUES (@id_pedido, produtoID, 1, 'disponivel');<br/>


        UPDATE pedido SET pedidostatus = 'confirmado' WHERE idpedido = @id_pedido;<br/>

        COMMIT;<br/>
        SELECT 'Compra realizada com sucesso.' AS Resultado;<br/>
    ELSE<br/>
        -- Caso a quantidade disponível seja zero, retorna uma mensagem indicando a falta de estoque<br/>
        SELECT 'Produto fora de estoque. Compra não realizada.' AS Resultado;<br/>
    END IF;<br/>
END //<br/>

DELIMITER ;<br/>
## BACKUP E RECOVERY
- BACKUP<br/>
mysqldump -u seu_usuario -p sua_senha ecommerce > backup_ecommerce.sql<br/>
- RECOVERY<br/>
mysql -u seu_usuario -p sua_senha ecommerce < backup_ecommerce.sql<br/>

