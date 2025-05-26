# CREATE-DATABASE
CREATE DATABASE IF NOT EXISTS IndustriaDB;
USE IndustriaDB;

-- Tabela de funcionários
CREATE TABLE funcionarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cargo VARCHAR(50),
    data_admissao DATE
);

-- Dados de exemplo - funcionários
INSERT INTO funcionarios (nome, cargo, data_admissao)
VALUES
('Ana Souza', 'Operadora de Máquinas', '2023-01-10'),
('Carlos Lima', 'Supervisor de Produção', '2022-03-15');

-- Tabela de produtos
CREATE TABLE produtos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL
);

-- Dados de exemplo - produtos
INSERT INTO produtos (nome, preco_unitario)
VALUES
('Parafuso A', 0.50),
('Peça B', 2.75);

-- Tabela de produção
CREATE TABLE producao (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT NOT NULL,
    funcionario_id INT NOT NULL,
    quantidade INT NOT NULL,
    data_producao DATE NOT NULL,
    FOREIGN KEY (produto_id) REFERENCES produtos(id),
    FOREIGN KEY (funcionario_id) REFERENCES funcionarios(id)
);

-- Tabela de estoque
CREATE TABLE estoque (
    produto_id INT PRIMARY KEY,
    quantidade INT NOT NULL DEFAULT 0,
    FOREIGN KEY (produto_id) REFERENCES produtos(id)
);

-- Dados de exemplo - estoque inicial
INSERT INTO estoque (produto_id, quantidade)
VALUES (1, 100), (2, 50);

-- Tabela de pedidos com chave estrangeira correta
CREATE TABLE pedidos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT NOT NULL,
    quantidade INT NOT NULL,
    status VARCHAR(50) DEFAULT 'Pendente',
    FOREIGN KEY (produto_id) REFERENCES produtos(id)
);

-- View para mostrar o estoque detalhado
CREATE VIEW view_estoque_detalhado AS
SELECT
    p.nome AS nome_produto,
    e.quantidade
FROM estoque e
JOIN produtos p ON e.produto_id = p.id;

-- Função para calcular o valor total do estoque
DELIMITER //
CREATE FUNCTION calcular_valor_estoque()
RETURNS DECIMAL(15,2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(15,2);
    SELECT SUM(p.preco_unitario * e.quantidade)
    INTO total
    FROM produtos p
    JOIN estoque e ON p.id = e.produto_id;
    RETURN IFNULL(total, 0.00);
END //
DELIMITER ;

-- Procedure para registrar produção e atualizar estoque
DELIMITER //
CREATE PROCEDURE registrar_producao(
    IN p_produto_id INT,
    IN p_funcionario_id INT,
    IN p_quantidade INT
)
BEGIN
    -- Inserir novo registro de produção
    INSERT INTO producao (produto_id, funcionario_id, quantidade, data_producao)
    VALUES (p_produto_id, p_funcionario_id, p_quantidade, CURDATE());

    -- Atualizar estoque se já existir
    UPDATE estoque
    SET quantidade = quantidade + p_quantidade
    WHERE produto_id = p_produto_id;

    -- Inserir novo item no estoque se ainda não existir
    INSERT INTO estoque (produto_id, quantidade)
    SELECT p_produto_id, p_quantidade
    WHERE NOT EXISTS (
        SELECT 1 FROM estoque WHERE produto_id = p_produto_id
    );
END //
DELIMITER ;

-- Exemplo de uso da procedure
CALL registrar_producao(1, 1, 20);
CALL registrar_producao(2, 2, 30);

-- Consultas úteis
-- SELECT * FROM view_estoque_detalhado;
-- SELECT calcular_valor_estoque();
