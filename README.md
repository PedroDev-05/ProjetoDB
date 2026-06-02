# Modelagem e Implementação de Banco de Dados no MariaDB

## Sobre o Projeto
Este projeto consiste na modelagem conceitual, lógica e física de um sistema de banco de dados para gerenciamento de uma **Oficina Mecânica**. O projeto foi desenvolvido para a disciplina de Banco de Dados ministrada pelo **Professor Gabriel M. de Carvalho**.

* **Grupo:** A
* **Tema:** Oficina mecânica
* **Tecnologia Principal:** MariaDB / MySQL

---

## Criação do DER (Modelagem Conceitual e Lógica)

<img width="1369" height="729" alt="Captura de tela 2026-06-01 154558" src="https://github.com/user-attachments/assets/1a3607c6-54de-4d97-82aa-4bebe0f9f3ee" />


### Explicação das Cardinalidades:
* **CLIENTE (1,1) possui (1,n) VEICULO:** Um veículo pertence a exatamente um cliente, mas um cliente pode possuir vários veículos cadastrados.
* **VEICULO (1,1) gera (0,n) ORDEM_SERVICO:** Uma ordem de serviço é exclusiva para um veículo, porém um veículo pode acumular várias ordens de serviço no histórico ao longo do tempo.
* **MECANICO (1,1) executa (0,n) ORDEM_SERVICO:** Cada ordem de serviço possui exatamente um mecânico responsável, e um mecânico pode executar várias ordens de serviço.
* **ORDEM_SERVICO (0,n) utiliza (1,1) PECA:** Uma ordem de serviço pode utilizar nenhuma ou várias peças e cada registro de peça utilizada está vinculado a uma ordem de serviço específica.

---

## Modelagem Física (Scripts SQL)

### 1. Criação do Banco de Dados e Tabelas

```sql
-- CRIAÇÃO E ACESSO AO DATABASE
CREATE DATABASE oficina_mecanica;
USE oficina_mecanica;

-- INSERINDO TABELA CLIENTES
CREATE TABLE clientes (
    id_cliente INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(14) UNIQUE NOT NULL,
    telefone VARCHAR(20) NOT NULL,
    data_cadastro DATE NOT NULL DEFAULT (CURRENT_DATE)
);

-- INSERINDO TABELAS VEICULOS
CREATE TABLE veiculos (
    id_veiculo INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    marca VARCHAR(50) NOT NULL,
    modelo VARCHAR(50) NOT NULL,
    ano INT NOT NULL,
    placa VARCHAR(10) UNIQUE NOT NULL,
    FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente)
);

-- INSERINDO TABELA MECANICOS
CREATE TABLE mecanicos (
    id_mecanico INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    especialidade VARCHAR(100) NOT NULL, 
    salario DECIMAL(10,2) NOT NULL,
    ativo BOOLEAN NOT NULL DEFAULT TRUE,
    CONSTRAINT chk_salari CHECK (salario > 0)
);

-- INSERINDO TABELA ORDEM_SERVICOS
CREATE TABLE ordem_servicos (
    id_os INT PRIMARY KEY AUTO_INCREMENT,
    id_veiculo INT NOT NULL,
    id_mecanico INT NOT NULL,
    data_entrada DATETIME NOT NULL,
    descricao_problema TEXT NOT NULL,
    valor_total DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    status_os VARCHAR(30) NOT NULL DEFAULT 'Pendente',
    FOREIGN KEY (id_veiculo) REFERENCES veiculos(id_veiculo),
    FOREIGN KEY (id_mecanico) REFERENCES mecanicos(id_mecanico),
    CONSTRAINT chk_status CHECK (status_os IN ('Pendente', 'Em Andamento','Concluído','Cancelado'))
);

-- INSERINDO TABELA PECAS
CREATE TABLE pecas (
    id_peca INT PRIMARY KEY AUTO_INCREMENT,
    id_os INT NOT NULL,
    nome_peca VARCHAR(100) NOT NULL,
    valor_peca DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (id_os) REFERENCES ordem_servicos(id_os),
    CONSTRAINT chk_valor_peca CHECK (valor_peca >= 0)
);

Povoamento das Tabelas

-- INSERINDO CLIENTES
INSERT INTO clientes (nome, cpf, telefone) VALUES
('Carlos Silva', '111.111.111-11', '(21)99999-1111'),
('Mariana Souza', '222.222.222-22', '(21)99999-2222'),
('João Pereira', '333.333.333-33', '(21)99999-3333'),
('Fernanda Lima', '444.444.444-44', '(21)99999-4444'),
('Ricardo Alves', '555.555.555-55', '(21)99999-5555'),
('Julia Costa', '666.666.666-66', '(21) 99999-6666'),
('Gabriel Souza', '777.777.777-77', '(21) 99999-7777'),
('Amanda Rocha', '888.888.888-88', '(21) 99999-8888'),
('Patricia Lima', '999.999.999-99', '(21) 99999-9999'),
('Bruno Santos', '000.000.000-00', '(21) 99999-0000');

-- INSERINDO VEICULOS
INSERT INTO veiculos (id_cliente, marca, modelo, ano, placa) VALUES
(1, 'Ford', 'Ka', 2018, 'ABC1A11'),
(2, 'Chevrolet', 'Onix', 2020, 'DEF2B22'),
(3, 'Volkswagen', 'Gol', 2017, 'GHI3C33'),
(4, 'Fiat', 'Uno', 2015, 'JKL4D44'),
(5, 'Honda', 'Civic', 2022, 'MNO5E55'),
(6, 'Hyundai', 'HB20', 2017, 'HYU7777'),
(7, 'Renault', 'Sandero', 2016, 'REN8888'),
(8, 'Jeep', 'Compass', 2023, 'JEE9999'),
(9, 'Nissan', 'March', 2014, 'NZQ0210'),
(10, 'Lamborghini', 'Temerario', 2026, 'UQM12O9');

-- INSERINDO MECANICOS
INSERT INTO mecanicos (nome, especialidade, salario) VALUES
('Pedro Martins', 'Motor', 3500.00),
('Lucas Ferreira', 'Suspensão', 3200.00),
('André Costa', 'Freios', 3000.00),
('Bruno Rocha', 'Elétrica', 4000.00),
('Rafael Gomes', 'Injeção Eletrônica', 4500.00);

-- INSERINDO ORDEM_SERVICOS
INSERT INTO ordem_servicos (id_veiculo, id_mecanico, data_entrada, descricao_problema, valor_total, status_os) VALUES
(1, 1, '2026-05-20 08:30:00', 'Motor fazendo barulho', 850.00, 'Em Andamento'),
(2, 2, '2026-05-20 09:15:00', 'Problema na suspensão traseira', 1200.00, 'Pendente'),
(3, 3, '2026-05-20 10:00:00', 'Troca de pastilhas de freio', 450.00, 'Concluído'),
(4, 4, '2026-05-20 11:20:00', 'Falha elétrica no painel', 700.00, 'Em Andamento'),
(5, 5, '2026-05-20 13:45:00', 'Problema na injeção eletrônica', 1500.00, 'Pendente');

-- INSERINDO PECAS
INSERT INTO pecas (id_os, nome_peca, valor_peca) VALUES
(1, 'Correia Dentada', 250.00),
(2, 'Amortecedor', 400.00),
(3, 'Pastilha de Freio', 150.00),
(4, 'Bateria', 350.00),
(5, 'Bico Injetor', 500.00);

Consultas SQL

-- SELECT: Lista todos os clientes
SELECT * FROM clientes;

-- WHERE: Busca veículos do ano 2017
SELECT * FROM veiculos WHERE ano = 2017;

-- ORDER BY DESC: Ordena mecânicos pelo maior salário
SELECT * FROM mecanicos ORDER BY salario DESC;

-- ORDER BY ASC: Ordena clientes em ordem alfabética
SELECT * FROM clientes ORDER BY nome ASC;

-- LIKE: Busca clientes que possuem "Souza" ou "Lima" no nome
SELECT * FROM clientes WHERE nome LIKE 'Souza%' OR nome LIKE '%Lima%';

-- BETWEEN: Busca mecânicos com salário entre 3000 e 4000
SELECT * FROM mecanicos WHERE salario BETWEEN 3000 AND 4000;

-- IN: Busca ordens de serviço com status específicos
SELECT * FROM ordem_servicos WHERE status_os IN ('Pendente', 'Concluído');

Operações com JOINs

-- INNER JOIN: Mostra clientes e seus respectivos veículos (apenas correspondências)
SELECT clientes.nome, veiculos.marca, veiculos.modelo, veiculos.placa
FROM clientes
INNER JOIN veiculos ON clientes.id_cliente = veiculos.id_cliente;

-- LEFT JOIN: Lista todos os clientes, mesmo aqueles sem veículos associados
SELECT clientes.nome, veiculos.marca, veiculos.modelo
FROM clientes
LEFT JOIN veiculos ON clientes.id_cliente = veiculos.id_cliente;

-- RIGHT JOIN: Lista todos os veículos cadastrados, mesmo sem clientes correspondentes
SELECT clientes.nome, veiculos.marca, veiculos.modelo
FROM clientes
RIGHT JOIN veiculos ON clientes.id_cliente = veiculos.id_cliente;

-- CROSS JOIN: Combinação cartesiana entre todos os clientes e mecânicos
SELECT clientes.nome AS cliente, mecanicos.nome AS mecanico 
FROM clientes CROSS JOIN mecanicos;

Alterações Estruturais, Remoção e Atualização

-- ALTER TABLE ADD: Adicionar coluna email
ALTER TABLE clientes ADD email VARCHAR(100);

-- ALTER TABLE MODIFY: Modificar tipo ou tamanho da coluna telefone
ALTER TABLE clientes MODIFY telefone VARCHAR(25);

-- ALTER TABLE CHANGE: Alterar o nome da coluna de 'telefone' para 'celular'
ALTER TABLE clientes CHANGE telefone celular VARCHAR(25);

-- ALTER TABLE DROP COLUMN: Excluir coluna email
ALTER TABLE clientes DROP COLUMN email;

-- COMANDO DELETE: Remover um veículo da marca 'Lamborghini'
DELETE FROM veiculos WHERE marca = 'Lamborghini';

-- COMANDO UPDATE: Atualizar o status de uma ordem de serviço específica
UPDATE ordem_servicos SET status_os = 'Concluído' WHERE id_os = 4;

Compartilhamento e Backup
Para facilitar o trabalho em equipe, o banco foi exportado e importado via linha de comando (CMD):

Exportar Banco de Dados (Backup):
mysqldump -u root -p oficina_mecanica > oficina_mecanica.sql

Importar Banco de Dados:
mysql -u root -p oficina_mecanica < oficina_mecanica.sql
