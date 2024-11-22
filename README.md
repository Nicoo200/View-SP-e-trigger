# View-SP-e-trigger

## TRÊS EXERCICIOS COM VIEW 

DQL <br>
CREATE VIEW pais_descrecente AS <br>
SELECT COUNT(C.CIDADE), P.PAIS <br>
FROM PAIS P, CIDADE C <br>
WHERE P.PAIS_ID = C.PAIS_ID <br>
GROUP BY P.PAIS <br>
ORDER BY 1 DESC;
<br>
SELECT * FROM pais_descrecente;

DQL <br>
CREATE VIEW filme_titulo AS <br>
SELECT titulo , descricao FROM filme;
<br>
SELECT * FROM filme_titulo;

DQL <br>
CREATE VIEW filme_categoria AS <br>
SELECT COUNT(F.TITULO), C.NOME <br>
FROM FILME F, FILME_CATEGORIA FA, CATEGORIA C <br>
WHERE F.FILME_ID = FA.FILME_ID <br>
AND C.CATEGORIA_ID = FA.CATEGORIA_ID <br>
GROUP BY C.CATEGORIA_ID; <br>
 <br>
SELECT * FROM filme_categoria;


## TRÊS EXERCICIOS COM SP

### 1 essa query ira mostrar o id do do flime, seu titulo e seu ano de lançamento 

DELIMITER //

CREATE PROCEDURE filme_info (in id int)  <br>
BEGIN
	SELECT filme_id, titulo, ano_de_lancamento FROM FILME
    WHERE filme_id=id; 
END //  <br>

DELIMITER ;

CALL filme_info (20);
 <br>
### 2 lista filme por atores 
DELIMITER //

CREATE PROCEDURE ListarFilmesPorAtor(IN nome_ator VARCHAR(50))  <br>
BEGIN
    SELECT 
        f.film_id AS id_filme,  <br>
        f.title AS titulo_filme,  <br>
        f.release_year AS ano_lancamento,  <br>
        c.name AS categoria <br>
    FROM ator a
    INNER JOIN filme_ator fa ON a.actor_id = fa.actor_id  <br>
    INNER JOIN filme f ON fa.film_id = f.film_id  <br>
    INNER JOIN filme_categoria fc ON f.film_id = fc.film_id  <br>
    INNER JOIN categoria c ON fc.category_id = c.category_id  <br>
    WHERE CONCAT(a.first_name, ' ', a.last_name) LIKE CONCAT('%', nome_ator, '%')  <br>
    ORDER BY f.release_year DESC;
END//

DELIMITER ;
 <br>
CALL ListarFilmesPorAtor('4');
 <br>
### 3 Total de locação por cliente

DELIMITER //

CREATE PROCEDURE CalcularTotalLocacoesPorCliente(IN id_cliente INT)  <br>
BEGIN
    SELECT 
        CONCAT(c.first_name, ' ', c.last_name) AS nome_cliente,  <br>
        COUNT(r.rental_id) AS total_locacoes,  <br>
        SUM(p.amount) AS valor_total_gasto  <br>
    FROM cliente c
    INNER JOIN locacao r ON c.customer_id = r.customer_id  <br>
    INNER JOIN pagamento p ON r.rental_id = p.rental_id <br>
    WHERE c.customer_id = id_cliente <br>
    GROUP BY c.customer_id;
END//

DELIMITER ;
 <br>
CALL CalcularTotalLocacoesPorCliente(1);
 <br>
## TRÊS EXERCICIOS COM TRIGGER

### 1 Esse trigger sera usado para fazer algum update na sua loja

DELIMITER $$
CREATE TRIGGER after_update_loja  <br>
AFTER UPDATE ON LOJA <br>
FOR EACH ROW <br>
BEGIN <br>
INSERT INTO log_updates (descricao, data_atualizacao)  <br>
VALUES ('Loja atualizada', NOW()); <br>
END $$
DELIMITER ;
 <br>
### 2 essa trigger ira notificar quando o estoque de algum produto ficar baixo
DELIMITER $$

CREATE TRIGGER AtualizarDataDeRetorno  <br>
BEFORE UPDATE ON rental  <br>
FOR EACH ROW  <br>
BEGIN <br>
    IF NEW.return_date IS NOT NULL AND OLD.return_date IS NULL THEN  <br>
        SET NEW.return_date = NOW();  <br>
    END IF;  <br>
END$$  <br>

DELIMITER ; 
 <br>
### 3 Esse trigger serve para atulização do estoque
 
DELIMITER $$

CREATE TRIGGER VerificarEstoqueBaixo  <br>
AFTER UPDATE ON produto  <br>
FOR EACH ROW  <br>
BEGIN  <br>
    IF NEW.estoque < 5 THEN  <br>
        SIGNAL SQLSTATE '45000'  <br>
        SET MESSAGE_TEXT = CONCAT('Atenção: O estoque do produto "', NEW.nome_produto, '" está baixo (', NEW.estoque, ' unidades).'); <br>
    END IF;  <br>
END$$  <br>

DELIMITER ;
 <br>
UPDATE produto SET estoque = 3 WHERE produto_id = 4;


