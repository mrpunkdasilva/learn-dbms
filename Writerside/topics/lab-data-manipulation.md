# Laboratório: Manipulação de Dados

## Objetivo
Praticar operações CRUD (Create, Read, Update, Delete) em um banco de dados relacional, aplicando boas práticas de manipulação de dados e garantindo a integridade dos dados.

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

-- Inserir um pedido com seus itens (transação)
BEGIN;

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

COMMIT;
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

-- Exclusão em cascata (com transação)
BEGIN;

DELETE FROM lab_workspace.itens_pedido
WHERE pedido_id = 1;

DELETE FROM lab_workspace.pedidos
WHERE id = 1;

COMMIT;
```

## Parte 2: Transações e Consistência

### 2.1 Transações Básicas

```sql
-- Transação com rollback
BEGIN;

UPDATE lab_workspace.produtos
SET estoque = estoque - 5
WHERE id = 1;

-- Verificar estoque
SELECT nome, estoque FROM lab_workspace.produtos WHERE id = 1;

-- Desfazer alterações
ROLLBACK;

-- Verificar estoque novamente
SELECT nome, estoque FROM lab_workspace.produtos WHERE id = 1;
```

### 2.2 Savepoints

```sql
-- Transação com savepoints
BEGIN;

INSERT INTO lab_workspace.clientes (nome, email, segmento)
VALUES ('Gabriel Lima', 'gabriel@email.com', 'Varejo');

SAVEPOINT novo_cliente;

UPDATE lab_workspace.produtos
SET estoque = -10
WHERE id = 1;

-- Ops, valor inválido!
ROLLBACK TO novo_cliente;

INSERT INTO lab_workspace.pedidos (cliente_id, status)
VALUES ((SELECT id FROM lab_workspace.clientes WHERE nome = 'Gabriel Lima'), 'Novo');

COMMIT;
```

### 2.3 Controle de Concorrência

```sql
-- Sessão 1
BEGIN;
SELECT * FROM lab_workspace.produtos WHERE id = 1 FOR UPDATE;
-- Simular processamento
-- (Aguardar 10 segundos)
UPDATE lab_workspace.produtos SET estoque = estoque - 1 WHERE id = 1;
COMMIT;

-- Sessão 2 (executar simultaneamente)
BEGIN;
SELECT * FROM lab_workspace.produtos WHERE id = 1 FOR UPDATE;
-- Esta operação aguardará até que a Sessão 1 faça COMMIT
UPDATE lab_workspace.produtos SET estoque = estoque - 2 WHERE id = 1;
COMMIT;
```

## Parte 3: Manipulação em Lote

### 3.1 Inserção em Lote

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
FROM generate_series(1, 100) i;

-- Verificar inserção
SELECT COUNT(*) FROM lab_workspace.clientes;
```

### 3.2 Atualização em Lote

```sql
-- Atualizar status de múltiplos pedidos
UPDATE lab_workspace.pedidos
SET status = 'Processando'
WHERE status = 'Pendente'
AND data_pedido < CURRENT_TIMESTAMP - INTERVAL '1 day';

-- Atualizar preços com base na categoria
UPDATE lab_workspace.produtos p
SET preco = preco * 
    CASE 
        WHEN p.categoria = 'Eletrônicos' THEN 0.95  -- 5% desconto
        WHEN p.categoria = 'Informática' THEN 0.90  -- 10% desconto
        ELSE 0.97  -- 3% desconto
    END
WHERE p.id IN (
    SELECT id FROM lab_workspace.produtos
    WHERE estoque > 50
);
```

### 3.3 Exclusão em Lote

```sql
-- Excluir pedidos antigos
DELETE FROM lab_workspace.itens_pedido
WHERE pedido_id IN (
    SELECT id FROM lab_workspace.pedidos
    WHERE data_pedido < CURRENT_DATE - INTERVAL '1 year'
);

DELETE FROM lab_workspace.pedidos
WHERE data_pedido < CURRENT_DATE - INTERVAL '1 year';

-- Excluir produtos sem movimento
DELETE FROM lab_workspace.produtos
WHERE id NOT IN (
    SELECT DISTINCT produto_id FROM lab_workspace.itens_pedido
)
AND data_cadastro < CURRENT_DATE - INTERVAL '6 months';
```

## Parte 4: Manipulação Avançada

### 4.1 Inserção Condicional

```sql
-- Inserir cliente apenas se não existir
INSERT INTO lab_workspace.clientes (nome, email, segmento, limite_credito)
SELECT 'Henrique Dias', 'henrique@email.com', 'Varejo', 2500.00
WHERE NOT EXISTS (
    SELECT 1 FROM lab_workspace.clientes WHERE email = 'henrique@email.com'
);

-- Inserir ou atualizar produto (UPSERT)
INSERT INTO lab_workspace.produtos (codigo, nome, descricao, preco, estoque, categoria)
VALUES ('PROD008', 'Tablet Pro', 'Tablet de alta performance', 1899.90, 15, 'Eletrônicos')
ON CONFLICT (codigo) DO UPDATE
SET 
    nome = EXCLUDED.nome,
    descricao = EXCLUDED.descricao,
    preco = EXCLUDED.preco,
    estoque = lab_workspace.produtos.estoque + EXCLUDED.estoque;
```

### 4.2 Atualização com JOIN

```sql
-- Atualizar preços de produtos com base em vendas
UPDATE lab_workspace.produtos p
SET preco = p.preco * 1.10  -- Aumento de 10%
FROM (
    SELECT produto_id, SUM(quantidade) as total_vendido
    FROM lab_workspace.itens_pedido
    GROUP BY produto_id
) as vendas
WHERE p.id = vendas.produto_id
AND vendas.total_vendido > 10;

-- Atualizar limite de crédito com base em histórico de pedidos
UPDATE lab_workspace.clientes c
SET limite_credito = limite_credito * 1.20  -- Aumento de 20%
FROM (
    SELECT cliente_id, COUNT(*) as total_pedidos, SUM(valor_total) as valor_total
    FROM lab_workspace.pedidos
    WHERE status = 'Finalizado'
    GROUP BY cliente_id
) as historico
WHERE c.id = historico.cliente_id
AND historico.total_pedidos >= 3
AND historico.valor_total > 5000;
```

### 4.3 Manipulação com CTE

```sql
-- Atualizar estoque e registrar movimentação usando CTE
WITH produtos_atualizados AS (
    UPDATE lab_workspace.produtos
    SET estoque = estoque - 5
    WHERE categoria = 'Eletrônicos'
    RETURNING id, nome, estoque
)
INSERT INTO lab_workspace.movimentacao_estoque (produto_id, quantidade, tipo_movimento, observacao)
SELECT 
    id, 
    5, 
    'Saída', 
    'Ajuste de inventário para produtos eletrônicos'
FROM produtos_atualizados;

-- Identificar e marcar produtos sem movimento
WITH produtos_sem_movimento AS (
    SELECT p.id
    FROM lab_workspace.produtos p
    LEFT JOIN lab_workspace.itens_pedido ip ON p.id = ip.produto_id
    WHERE ip.produto_id IS NULL
)
UPDATE lab_workspace.produtos p
SET status = 'Inativo'
FROM produtos_sem_movimento psm
WHERE p.id = psm.id;
```

## Parte 5: Técnicas de Otimização

### 5.1 Inserção em Massa Otimizada

```sql
-- Inserção em massa com valores múltiplos
INSERT INTO lab_workspace.produtos_temp (codigo, nome, preco, estoque, categoria)
VALUES 
    ('TEMP001', 'Produto Temp 1', 99.90, 10, 'Diversos'),
    ('TEMP002', 'Produto Temp 2', 199.90, 20, 'Diversos'),
    ('TEMP003', 'Produto Temp 3', 299.90, 30, 'Diversos'),
    ('TEMP004', 'Produto Temp 4', 399.90, 40, 'Diversos'),
    ('TEMP005', 'Produto Temp 5', 499.90, 50, 'Diversos');

-- Mover dados da tabela temporária para a tabela principal
INSERT INTO lab_workspace.produtos (codigo, nome, preco, estoque, categoria)
SELECT codigo, nome, preco, estoque, categoria
FROM lab_workspace.produtos_temp
ON CONFLICT (codigo) DO NOTHING;
```

### 5.2 Atualização em Lotes

```sql
-- Atualizar em lotes pequenos para evitar bloqueios longos
DO $$
DECLARE
    batch_size INTEGER := 100;
    total_rows INTEGER;
    processed_rows INTEGER := 0;
BEGIN
    SELECT COUNT(*) INTO total_rows FROM lab_workspace.produtos WHERE categoria = 'Diversos';
    
    WHILE processed_rows < total_rows LOOP
        UPDATE lab_workspace.produtos
        SET preco = preco * 0.9
        WHERE id IN (
            SELECT id FROM lab_workspace.produtos
            WHERE categoria = 'Diversos'
            ORDER BY id
            LIMIT batch_size
            OFFSET processed_rows
        );
        
        processed_rows := processed_rows + batch_size;
        COMMIT;
    END LOOP;
END $$;
```

### 5.3 Consultas Otimizadas

```sql
-- Consulta com filtros eficientes
SELECT 
    p.id,
    p.nome,
    p.preco,
    p.estoque
FROM lab_workspace.produtos p
WHERE p.categoria = 'Eletrônicos'
AND p.preco > 1000
AND p.estoque > 0;

-- Consulta com agregações otimizadas
SELECT 
    categoria,
    COUNT(*) as total_produtos,
    SUM(estoque) as estoque_total,
    AVG(preco) as preco_medio
FROM lab_workspace.produtos
WHERE estoque > 0
GROUP BY categoria
HAVING COUNT(*) > 1
ORDER BY estoque_total DESC;
```

## Conclusão

Neste laboratório, você praticou diversas técnicas de manipulação de dados em SQL, incluindo:

- Operações CRUD básicas (INSERT, SELECT, UPDATE, DELETE)
- Transações e controle de concorrência (BEGIN, COMMIT, ROLLBACK, SAVEPOINT)
- Manipulação em lote
- Técnicas avançadas como UPSERT, CTEs e manipulação com JOIN
- Otimização de operações de manipulação de dados

Estas habilidades são essenciais para desenvolvedores de banco de dados que precisam implementar operações eficientes e seguras em sistemas de produção.