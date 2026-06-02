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
```

<img width="494" height="56" alt="Captura de tela 2026-06-01 223156" src="https://github.com/user-attachments/assets/c1330aa6-f07a-48f5-b3eb-ec19c398829a" />

<img width="383" height="65" alt="Captura de tela 2026-06-01 223230" src="https://github.com/user-attachments/assets/636302de-640d-441a-8031-d6c7485410ee" />

```sql
-- INSERINDO TABELA CLIENTES
CREATE TABLE clientes (
    id_cliente INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(14) UNIQUE NOT NULL,
    telefone VARCHAR(20) NOT NULL,
    data_cadastro DATE NOT NULL DEFAULT (CURRENT_DATE)
);
```

<img width="565" height="302" alt="Captura de tela 2026-06-01 223248" src="https://github.com/user-attachments/assets/912c3c05-0388-4971-80d3-dc50c1553d79" />

```sql
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
```

<img width="481" height="459" alt="Captura de tela 2026-06-01 223259" src="https://github.com/user-attachments/assets/50c734a9-35bf-4569-a851-4fdeab3a4edd" />

```sql
-- INSERINDO TABELA MECANICOS
CREATE TABLE mecanicos (
    id_mecanico INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    especialidade VARCHAR(100) NOT NULL, 
    salario DECIMAL(10,2) NOT NULL,
    ativo BOOLEAN NOT NULL DEFAULT TRUE,
    CONSTRAINT chk_salari CHECK (salario > 0)
);
```

<img width="494" height="381" alt="Captura de tela 2026-06-01 223314" src="https://github.com/user-attachments/assets/180e38e2-6e25-44f3-b441-46df5150ce83" />

```sql
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
```

<img width="567" height="424" alt="Captura de tela 2026-06-01 223347" src="https://github.com/user-attachments/assets/7414e653-1a0e-426d-9c92-230a4d1a4d1e" />

```sql
-- INSERINDO TABELA PECAS
CREATE TABLE pecas (
    id_peca INT PRIMARY KEY AUTO_INCREMENT,
    id_os INT NOT NULL,
    nome_peca VARCHAR(100) NOT NULL,
    valor_peca DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (id_os) REFERENCES ordem_servicos(id_os),
    CONSTRAINT chk_valor_peca CHECK (valor_peca >= 0)
);
```

<img width="466" height="419" alt="Captura de tela 2026-06-01 223403" src="https://github.com/user-attachments/assets/3031452a-548c-4d25-af99-06cbcbdb5ef6" />


Povoamento das Tabelas

```sql
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
```

<img width="569" height="457" alt="Captura de tela 2026-06-01 223420" src="https://github.com/user-attachments/assets/e96e864a-a573-4de8-a305-eb48dfb171e0" />

```sql
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
```

<img width="568" height="349" alt="Captura de tela 2026-06-01 223433" src="https://github.com/user-attachments/assets/facca7d9-9cf6-48b2-8329-c7d1223842f4" />

```sql
-- INSERINDO MECANICOS
INSERT INTO mecanicos (nome, especialidade, salario) VALUES
('Pedro Martins', 'Motor', 3500.00),
('Lucas Ferreira', 'Suspensão', 3200.00),
('André Costa', 'Freios', 3000.00),
('Bruno Rocha', 'Elétrica', 4000.00),
('Rafael Gomes', 'Injeção Eletrônica', 4500.00);
```

<img width="566" height="254" alt="Captura de tela 2026-06-01 223445" src="https://github.com/user-attachments/assets/8a6d9416-f46a-4484-8786-b2bdc5539c9b" />

```sql
-- INSERINDO ORDEM_SERVICOS
INSERT INTO ordem_servicos (id_veiculo, id_mecanico, data_entrada, descricao_problema, valor_total, status_os) VALUES
(1, 1, '2026-05-20 08:30:00', 'Motor fazendo barulho', 850.00, 'Em Andamento'),
(2, 2, '2026-05-20 09:15:00', 'Problema na suspensão traseira', 1200.00, 'Pendente'),
(3, 3, '2026-05-20 10:00:00', 'Troca de pastilhas de freio', 450.00, 'Concluído'),
(4, 4, '2026-05-20 11:20:00', 'Falha elétrica no painel', 700.00, 'Em Andamento'),
(5, 5, '2026-05-20 13:45:00', 'Problema na injeção eletrônica', 1500.00, 'Pendente');
```

<img width="505" height="221" alt="Captura de tela 2026-06-01 223456" src="https://github.com/user-attachments/assets/a143236c-e89f-4949-aa64-0491e4bf8403" />

```sql
-- INSERINDO PECAS
INSERT INTO pecas (id_os, nome_peca, valor_peca) VALUES
(1, 'Correia Dentada', 250.00),
(2, 'Amortecedor', 400.00),
(3, 'Pastilha de Freio', 150.00),
(4, 'Bateria', 350.00),
(5, 'Bico Injetor', 500.00);
```

<img width="514" height="215" alt="Captura de tela 2026-06-01 223506" src="https://github.com/user-attachments/assets/7231732a-56ba-4e9f-96cc-8cace7565ad9" />

Consultas SQL

```sql
-- SELECT: Lista todos os clientes
SELECT * FROM clientes;
```

<img width="568" height="264" alt="Captura de tela 2026-06-01 223516" src="https://github.com/user-attachments/assets/28847677-6b51-41b2-9b28-4e92b235e960" />

```sql
-- WHERE: Busca veículos do ano 2017
SELECT * FROM veiculos WHERE ano = 2017;
```

<img width="568" height="176" alt="Captura de tela 2026-06-01 223527" src="https://github.com/user-attachments/assets/283faef5-f39f-4c3f-b07a-c126b5b8a743" />

```sql
-- ORDER BY DESC: Ordena mecânicos pelo maior salário
SELECT * FROM mecanicos ORDER BY salario DESC;
```

<img width="568" height="214" alt="Captura de tela 2026-06-01 223537" src="https://github.com/user-attachments/assets/44181975-dd71-4512-aab5-518fb40cef43" />

```sql
-- ORDER BY ASC: Ordena clientes em ordem alfabética
SELECT * FROM clientes ORDER BY nome ASC;
```

<img width="565" height="258" alt="Captura de tela 2026-06-01 223548" src="https://github.com/user-attachments/assets/e5843fe3-5865-4c99-8951-db11a95548b6" />

```sql
-- LIKE: Busca clientes que possuem "Souza" ou "Lima" no nome
SELECT * FROM clientes WHERE nome LIKE 'Souza%' OR nome LIKE '%Lima%';
```

<img width="566" height="150" alt="Captura de tela 2026-06-01 223616" src="https://github.com/user-attachments/assets/563d442d-62bf-4f0f-95e2-61b4b5715607" />

```sql
-- BETWEEN: Busca mecânicos com salário entre 3000 e 4000
SELECT * FROM mecanicos WHERE salario BETWEEN 3000 AND 4000;
```

<img width="567" height="164" alt="Captura de tela 2026-06-01 223623" src="https://github.com/user-attachments/assets/23304ed0-8085-4b71-a4df-61ef471d18f0" />

```sql
-- IN: Busca ordens de serviço com status específicos
SELECT * FROM ordem_servicos WHERE status_os IN ('Pendente', 'Concluído');
```

<img width="566" height="118" alt="Captura de tela 2026-06-01 223631" src="https://github.com/user-attachments/assets/719bf22a-8c18-4991-b415-9a3abff79199" />

Operações com JOINs

```sql
-- INNER JOIN: Mostra clientes e seus respectivos veículos (apenas correspondências)
SELECT clientes.nome, veiculos.marca, veiculos.modelo, veiculos.placa
FROM clientes
INNER JOIN veiculos ON clientes.id_cliente = veiculos.id_cliente;
```

<img width="450" height="273" alt="Captura de tela 2026-06-01 223644" src="https://github.com/user-attachments/assets/84eaa921-94a8-48fe-9428-0ed6d7909233" />

```sql
-- LEFT JOIN: Lista todos os clientes, mesmo aqueles sem veículos associados
SELECT clientes.nome, veiculos.marca, veiculos.modelo
FROM clientes
LEFT JOIN veiculos ON clientes.id_cliente = veiculos.id_cliente;
```

<img width="442" height="310" alt="Captura de tela 2026-06-01 223654" src="https://github.com/user-attachments/assets/1919d6bf-40bb-429d-ad5c-ccaaa8b6137b" />

```sql
-- RIGHT JOIN: Lista todos os veículos cadastrados, mesmo sem clientes correspondentes
SELECT clientes.nome, veiculos.marca, veiculos.modelo
FROM clientes
RIGHT JOIN veiculos ON clientes.id_cliente = veiculos.id_cliente;
```

<img width="466" height="298" alt="Captura de tela 2026-06-01 223703" src="https://github.com/user-attachments/assets/efd8bb49-3214-4fa6-94c3-121e062a0a26" />

```sql
-- CROSS JOIN: Combinação cartesiana entre todos os clientes e mecânicos
SELECT clientes.nome AS cliente, mecanicos.nome AS mecanico 
FROM clientes CROSS JOIN mecanicos;
```

<img width="478" height="549" alt="Captura de tela 2026-06-01 223718" src="https://github.com/user-attachments/assets/d826ff3b-0527-4c50-b3d8-262b2cd6cf65" />

Alterações Estruturais, Remoção e Atualização

```sql
-- ALTER TABLE ADD: Adicionar coluna email
ALTER TABLE clientes ADD email VARCHAR(100);
```

<img width="567" height="356" alt="Captura de tela 2026-06-01 223726" src="https://github.com/user-attachments/assets/d8dbf999-ae49-45e4-9767-f78a26529f7c" />

```sql
-- ALTER TABLE MODIFY: Modificar tipo ou tamanho da coluna telefone
ALTER TABLE clientes MODIFY telefone VARCHAR(25);
```

<img width="409" height="117" alt="Captura de tela 2026-06-01 223740" src="https://github.com/user-attachments/assets/b8826b68-8842-4c78-ab96-d2add4bfcd71" />

```sql
-- ALTER TABLE CHANGE: Alterar o nome da coluna de 'telefone' para 'celular'
ALTER TABLE clientes CHANGE telefone celular VARCHAR(25);
```

<img width="538" height="297" alt="Captura de tela 2026-06-01 223751" src="https://github.com/user-attachments/assets/8a9c2eb8-5073-44c4-9acc-be76b892ae79" />

```sql
-- ALTER TABLE DROP COLUMN: Excluir coluna email
ALTER TABLE clientes DROP COLUMN email;
```

<img width="566" height="334" alt="Captura de tela 2026-06-01 223759" src="https://github.com/user-attachments/assets/56d206a1-ca17-410a-895f-f62516b7d6dd" />

```sql
-- COMANDO DELETE: Remover um veículo da marca 'Lamborghini'
DELETE FROM veiculos WHERE marca = 'Lamborghini';
```

<img width="567" height="276" alt="Captura de tela 2026-06-01 223819" src="https://github.com/user-attachments/assets/bc52cc80-ca55-4d29-9d10-855df5dd9b2e" />

<img width="568" height="280" alt="Captura de tela 2026-06-01 223834" src="https://github.com/user-attachments/assets/a9b13d21-c6bd-4759-823c-655ffc1b056f" />

```sql
-- COMANDO UPDATE: Atualizar o status de uma ordem de serviço específica
UPDATE ordem_servicos SET status_os = 'Concluído' WHERE id_os = 4;
```

<img width="568" height="286" alt="Captura de tela 2026-06-01 223846" src="https://github.com/user-attachments/assets/02d56458-29c4-4b00-b6bb-0399c376996e" />

```
Compartilhamento e Backup
Para facilitar o trabalho em equipe, o banco foi exportado e importado via linha de comando (CMD):

Exportar Banco de Dados (Backup):
mysqldump -u root -p oficina_mecanica > oficina_mecanica.sql

Importar Banco de Dados:
mysql -u root -p oficina_mecanica < oficina_mecanica.sql
```
