# Laboratório: Manipulação de Dados

## Objetivo
Praticar operações DML (Data Manipulation Language) básicas em um banco de dados relacional, aplicando boas práticas de manipulação de dados.

## Cenário
Você é um desenvolvedor de banco de dados em uma empresa de e-commerce que precisa implementar operações de manipulação de dados para o sistema de gerenciamento de pedidos.

## Setup Inicial

```sql
-- Carregar dados de exemplo
INSERT INTO lab_workspace.clientes (nome, email, segmento, limite_credito)
VALUES 
    ('Ana Silva', 'ana.silva@email.com', 'Varejo', 2000.00),
    ('Bruno Costa', 'bruno@email.com', 'Atacado', 5000.00),
    ('Carla Mendes', 'carla@email.com', 'Varejo', 1500.00),
    ('Daniel Oliveira', 'daniel@email.com', 'Corporativo', 10000.00),
    ('Elena Santos', 'elena@email.com', 'Varejo', 2500.00);

INSERT INTO lab_workspace.produtos (codigo, nome, descricao, preco, estoque, categoria)
VALUES 
    ('PROD001', 'Smartphone X', 'Smartphone avançado com câmera de alta resolução', 1299.99, 50, 'Eletrônicos'),
    ('PROD002', 'Notebook Pro', 'Notebook para uso profissional', 3499.99, 20, 'Informática'),
    ('PROD003', 'Fones Bluetooth', 'Fones de ouvido sem fio', 199.99, 100, 'Acessórios'),
    ('PROD004', 'Monitor 24"', 'Monitor LED Full HD', 799.99, 30, 'Informática'),
    ('PROD005', 'Teclado Mecânico', 'Teclado para gamers', 249.99, 45, 'Periféricos');

-- Registrar início do laboratório
INSERT INTO lab_workspace.lab_progress (lab_name)
VALUES ('Manipulação de Dados');
```

## Parte 1: Operações CRUD Básicas

### 1.1 Inserção de Dados

```sql
-- Inserir um novo cliente
INSERT INTO lab_workspace.clientes (nome, email, segmento, limite_credito)
VALUES ('Fernando Gomes', 'fernando@email.com', 'Varejo', 3000.00);

-- Inserir múltiplos produtos
INSERT INTO lab_workspace.produtos (codigo, nome, descricao, preco, estoque, categoria)
VALUES 
    ('PROD006', 'Mouse Sem Fio', 'Mouse ergonômico', 89.90, 60, 'Periféricos'),
    ('PROD007', 'Caixa de Som', 'Caixa de som bluetooth', 159.90, 25, 'Acessórios');

-- Inserir um pedido com seus itens
INSERT INTO lab_workspace.pedidos (cliente_id, observacoes)
VALUES (1, 'Entrega prioritária');

INSERT INTO lab_workspace.itens_pedido (pedido_id, produto_id, quantidade, preco_unitario)
VALUES 
    (1, 1, 1, 1299.99),
    (1, 3, 2, 199.99);

UPDATE lab_workspace.pedidos
SET valor_total = (
    SELECT SUM(quantidade * preco_unitario)
    FROM lab_workspace.itens_pedido
    WHERE pedido_id = 1
)
WHERE id = 1;
```

### 1.2 Consulta de Dados

```sql
-- Consulta simples
SELECT * FROM lab_workspace.clientes WHERE segmento = 'Varejo';

-- Consulta com JOIN
SELECT 
    p.id as pedido_id,
    c.nome as cliente,
    p.data_pedido,
    p.valor_total,
    p.status
FROM lab_workspace.pedidos p
JOIN lab_workspace.clientes c ON p.cliente_id = c.id;

-- Consulta detalhada de pedido
SELECT 
    p.id as pedido_id,
    c.nome as cliente,
    pr.nome as produto,
    ip.quantidade,
    ip.preco_unitario,
    (ip.quantidade * ip.preco_unitario) as subtotal
FROM lab_workspace.pedidos p
JOIN lab_workspace.clientes c ON p.cliente_id = c.id
JOIN lab_workspace.itens_pedido ip ON p.id = ip.pedido_id
JOIN lab_workspace.produtos pr ON ip.produto_id = pr.id
WHERE p.id = 1;
```

### 1.3 Atualização de Dados

```sql
-- Atualização simples
UPDATE lab_workspace.clientes
SET email = 'ana.silva.nova@email.com'
WHERE id = 1;

-- Atualização com cálculo
UPDATE lab_workspace.produtos
SET preco = preco * 1.05
WHERE categoria = 'Eletrônicos';

-- Atualização com subconsulta
UPDATE lab_workspace.pedidos
SET status = 'Aprovado'
WHERE cliente_id IN (
    SELECT id FROM lab_workspace.clientes
    WHERE limite_credito > 2000.00
);
```

### 1.4 Exclusão de Dados

```sql
-- Exclusão simples
DELETE FROM lab_workspace.itens_pedido
WHERE pedido_id = 1 AND produto_id = 3;

-- Exclusão com subconsulta
DELETE FROM lab_workspace.produtos
WHERE estoque = 0 AND id NOT IN (
    SELECT produto_id FROM lab_workspace.itens_pedido
);

-- Exclusão em cascata
DELETE FROM lab_workspace.itens_pedido
WHERE pedido_id = 1;

DELETE FROM lab_workspace.pedidos
WHERE id = 1;
```

## Parte 2: Manipulação em Lote

### 2.1 Inserção em Lote

```sql
-- Inserir múltiplos clientes
INSERT INTO lab_workspace.clientes (nome, email, segmento, limite_credito)
SELECT 
    'Cliente ' || i,
    'cliente' || i || '@email.com',
    CASE WHEN i % 3 = 0 THEN 'Varejo' 
         WHEN i % 3 = 1 THEN 'Atacado' 
         ELSE 'Corporativo' END,
    1000 * (i % 10 + 1)
FROM generate_series(1, 10) i;

-- Verificar inserção
SELECT COUNT(*) FROM lab_workspace.clientes;
```

### 2.2 Atualização em Lote

```sql
-- Atualizar status de múltiplos pedidos
UPDATE lab_workspace.pedidos
SET status = 'Processando'
WHERE status = 'Pendente';

-- Atualizar preços com base na categoria
UPDATE lab_workspace.produtos
SET preco = preco * 0.95  -- 5% desconto
WHERE categoria = 'Eletrônicos';
```

### 2.3 Exclusão em Lote

```sql
-- Excluir pedidos antigos
DELETE FROM lab_workspace.itens_pedido
WHERE pedido_id IN (
    SELECT id FROM lab_workspace.pedidos
    WHERE status = 'Cancelado'
);

DELETE FROM lab_workspace.pedidos
WHERE status = 'Cancelado';
```

## Conclusão

Neste laboratório, você praticou as operações DML básicas:

- Inserção de dados (INSERT)
- Consulta de dados (SELECT)
- Atualização de dados (UPDATE)
- Exclusão de dados (DELETE)

Estas operações são fundamentais para qualquer sistema de banco de dados e formam a base para manipulação de dados em aplicações.
