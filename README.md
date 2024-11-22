# View-SP-e-trigger

## TRÊS EXERCICIOS COM VIEW 

-- DQL <br>
CREATE VIEW pais_descrecente AS 
SELECT COUNT(C.CIDADE), P.PAIS
FROM PAIS P, CIDADE C
WHERE P.PAIS_ID = C.PAIS_ID
GROUP BY P.PAIS
ORDER BY 1 DESC;

SELECT * FROM pais_descrecente;

-- DQL <br>
CREATE VIEW filme_titulo AS 
SELECT titulo , descricao FROM filme;

SELECT * FROM filme_titulo;

-- DQL <br>
CREATE VIEW filme_categoria AS
SELECT COUNT(F.TITULO), C.NOME
FROM FILME F, FILME_CATEGORIA FA, CATEGORIA C
WHERE F.FILME_ID = FA.FILME_ID
AND C.CATEGORIA_ID = FA.CATEGORIA_ID
GROUP BY C.CATEGORIA_ID;

SELECT * FROM filme_categoria;


## TRÊS EXERCICIOS COM SP

-- 1 essa query ira mostrar o id do do flime, seu titulo e seu ano de lançamento 

DELIMITER //

CREATE PROCEDURE filme_info (in id int)
BEGIN
	SELECT filme_id, titulo, ano_de_lancamento FROM FILME
    WHERE filme_id=id; 
END //

DELIMITER ;

CALL filme_info (20);


-- 2 lista filme por atores 
DELIMITER //

CREATE PROCEDURE ListarFilmesPorAtor(IN nome_ator VARCHAR(50))
BEGIN
    SELECT 
        f.film_id AS id_filme,
        f.title AS titulo_filme,
        f.release_year AS ano_lancamento,
        c.name AS categoria
    FROM ator a
    INNER JOIN filme_ator fa ON a.actor_id = fa.actor_id
    INNER JOIN filme f ON fa.film_id = f.film_id
    INNER JOIN filme_categoria fc ON f.film_id = fc.film_id
    INNER JOIN categoria c ON fc.category_id = c.category_id
    WHERE CONCAT(a.first_name, ' ', a.last_name) LIKE CONCAT('%', nome_ator, '%')
    ORDER BY f.release_year DESC;
END//

DELIMITER ;

CALL ListarFilmesPorAtor('4');

-- 3 Total de locação por cliente

DELIMITER //

CREATE PROCEDURE CalcularTotalLocacoesPorCliente(IN id_cliente INT)
BEGIN
    SELECT 
        CONCAT(c.first_name, ' ', c.last_name) AS nome_cliente,
        COUNT(r.rental_id) AS total_locacoes,
        SUM(p.amount) AS valor_total_gasto
    FROM cliente c
    INNER JOIN locacao r ON c.customer_id = r.customer_id
    INNER JOIN pagamento p ON r.rental_id = p.rental_id
    WHERE c.customer_id = id_cliente
    GROUP BY c.customer_id;
END//

DELIMITER ;

CALL CalcularTotalLocacoesPorCliente(1);


## TRÊS EXERCICIOS COM TRIGGER

-- 1 Esse trigger sera usado para fazer algum update na sua loja

DELIMITER $$
CREATE TRIGGER after_update_loja
AFTER UPDATE ON LOJA
FOR EACH ROW
BEGIN
INSERT INTO log_updates (descricao, data_atualizacao)
VALUES ('Loja atualizada', NOW());
END $$
DELIMITER ;

-- 2 essa trigger ira notificar quando o estoque de algum produto ficar baixo
DELIMITER $$

CREATE TRIGGER AtualizarDataDeRetorno
BEFORE UPDATE ON rental
FOR EACH ROW
BEGIN
    IF NEW.return_date IS NOT NULL AND OLD.return_date IS NULL THEN
        SET NEW.return_date = NOW();
    END IF;
END$$

DELIMITER ;



-- 3 Esse trigger serve para atulização do estoque
 
DELIMITER $$

CREATE TRIGGER VerificarEstoqueBaixo
AFTER UPDATE ON produto
FOR EACH ROW
BEGIN
    IF NEW.estoque < 5 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = CONCAT('Atenção: O estoque do produto "', NEW.nome_produto, '" está baixo (', NEW.estoque, ' unidades).');
    END IF;
END$$

DELIMITER ;

UPDATE produto SET estoque = 3 WHERE produto_id = 4;


