# Laboratório: Integração de Dados

## Objetivo
Desenvolver habilidades de integração de dados utilizando SQL, incluindo importação, exportação, sincronização entre tabelas, migração e validação de integridade.

## Cenário
Você é um engenheiro de dados responsável por integrar dados de diferentes fontes em um sistema unificado, garantindo consistência e qualidade durante todo o processo.

## Setup Inicial

```sql
-- Criar tabelas para o laboratório
CREATE TABLE lab_workspace.sistema_legado_clientes (
    cliente_id INTEGER PRIMARY KEY,
    nome VARCHAR(100),
    email VARCHAR(100),
    telefone VARCHAR(20),
    data_cadastro DATE,
    ultima_compra DATE,
    status VARCHAR(20)
);

CREATE TABLE lab_workspace.sistema_legado_produtos (
    produto_id INTEGER PRIMARY KEY,
    nome VARCHAR(100),
    categoria VARCHAR(50),
    preco DECIMAL(10,2),
    estoque INTEGER,
    fornecedor VARCHAR(100)
);

CREATE TABLE lab_workspace.sistema_legado_vendas (
    venda_id INTEGER PRIMARY KEY,
    cliente_id INTEGER,
    data_venda DATE,
    valor_total DECIMAL(10,2),
    status VARCHAR(20)
);

CREATE TABLE lab_workspace.sistema_legado_itens_venda (
    venda_id INTEGER,
    produto_id INTEGER,
    quantidade INTEGER,
    preco_unitario DECIMAL(10,2),
    PRIMARY KEY (venda_id, produto_id)
);

-- Criar tabelas do novo sistema
CREATE TABLE lab_workspace.clientes_novo (
    id SERIAL PRIMARY KEY,
    codigo_legado INTEGER UNIQUE,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    telefone VARCHAR(20),
    data_cadastro DATE,
    ultima_atualizacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'Ativo',
    segmento VARCHAR(50),
    limite_credito DECIMAL(10,2) DEFAULT 1000.00
);

CREATE TABLE lab_workspace.produtos_novo (
    id SERIAL PRIMARY KEY,
    codigo_legado INTEGER UNIQUE,
    nome VARCHAR(100) NOT NULL,
    descricao TEXT,
    categoria VARCHAR(50),
    subcategoria VARCHAR(50),
    preco DECIMAL(10,2) NOT NULL,
    custo DECIMAL(10,2),
    margem_lucro DECIMAL(5,2),
    estoque INTEGER DEFAULT 0,
    estoque_minimo INTEGER DEFAULT 5,
    fornecedor VARCHAR(100),
    data_cadastro DATE,
    ultima_atualizacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'Ativo'
);

CREATE TABLE lab_workspace.vendas_novo (
    id SERIAL PRIMARY KEY,
    codigo_legado INTEGER UNIQUE,
    cliente_id INTEGER REFERENCES lab_workspace.clientes_novo(id),
    data_venda TIMESTAMP NOT NULL,
    valor_subtotal DECIMAL(10,2) NOT NULL,
    valor_desconto DECIMAL(10,2) DEFAULT 0,
    valor_frete DECIMAL(10,2) DEFAULT 0,
    valor_total DECIMAL(10,2) NOT NULL,
    forma_pagamento VARCHAR(50),
    status VARCHAR(20) DEFAULT 'Processando',
    data_atualizacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE lab_workspace.itens_venda_novo (
    venda_id INTEGER REFERENCES lab_workspace.vendas_novo(id),
    produto_id INTEGER REFERENCES lab_workspace.produtos_novo(id),
    quantidade INTEGER NOT NULL CHECK (quantidade > 0),
    preco_unitario DECIMAL(10,2) NOT NULL,
    valor_desconto DECIMAL(10,2) DEFAULT 0,
    valor_total DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (venda_id, produto_id)
);

-- Inserir dados de exemplo no sistema legado
INSERT INTO lab_workspace.sistema_legado_clientes
VALUES 
    (1001, 'João Silva', 'joao.silva@email.com', '11987654321', '2020-01-15', '2022-05-10', 'Ativo'),
    (1002, 'Maria Santos', 'maria.santos@email.com', '21987654321', '2020-02-20', '2022-06-15', 'Ativo'),
    (1003, 'Pedro Oliveira', 'pedro.oliveira@email.com', '31987654321', '2020-03-25', '2022-04-05', 'Inativo'),
    (1004, 'Ana Costa', 'ana.costa@email.com', '41987654321', '2020-04-30', '2022-06-20', 'Ativo'),
    (1005, 'Carlos Souza', 'carlos.souza@email.com', '51987654321', '2020-05-05', '2022-03-10', 'Inativo');

INSERT INTO lab_workspace.sistema_legado_produtos
VALUES 
    (2001, 'Notebook Basic', 'Informática', 2499.90, 15, 'Tech Distribuidora'),
    (2002, 'Smartphone X', 'Eletrônicos', 1899.90, 25, 'Mobile Imports'),
    (2003, 'Mouse Sem Fio', 'Informática', 89.90, 50, 'Tech Distribuidora'),
    (2004, 'Teclado Mecânico', 'Informática', 299.90, 30, 'Tech Distribuidora'),
    (2005, 'Monitor 24"', 'Informática', 899.90, 10, 'Visual Displays');

INSERT INTO lab_workspace.sistema_legado_vendas
VALUES 
    (3001, 1001, '2022-05-10', 2589.80, 'Concluída'),
    (3002, 1002, '2022-06-15', 1899.90, 'Concluída'),
    (3003, 1003, '2022-04-05', 389.80, 'Concluída'),
    (3004, 1004, '2022-06-20', 3399.80, 'Concluída'),
    (3005, 1005, '2022-03-10', 89.90, 'Cancelada');

INSERT INTO lab_workspace.sistema_legado_itens_venda
VALUES 
    (3001, 2001, 1, 2499.90),
    (3001, 2003, 1, 89.90),
    (3002, 2002, 1, 1899.90),
    (3003, 2003, 1, 89.90),
    (3003, 2004, 1, 299.90),
    (3004, 2001, 1, 2499.90),
    (3004, 2005, 1, 899.90),
    (3005, 2003, 1, 89.90);

-- Registrar início do laboratório
INSERT INTO lab_workspace.lab_progress (lab_name)
VALUES ('Integração de Dados');
```

## Parte 1: Importação de Dados

### 1.1 Migração de Clientes

```sql
-- Migrar clientes do sistema legado para o novo sistema
INSERT INTO lab_workspace.clientes_novo (
    codigo_legado,
    nome,
    email,
    telefone,
    data_cadastro,
    ultima_atualizacao,
    status,
    segmento,
    limite_credito
)
SELECT
    cliente_id,
    nome,
    email,
    telefone,
    data_cadastro,
    CURRENT_TIMESTAMP,
    status,
    CASE
        WHEN ultima_compra >= CURRENT_DATE - INTERVAL '3 months' THEN 'Frequente'
        WHEN ultima_compra >= CURRENT_DATE - INTERVAL '6 months' THEN 'Regular'
        ELSE 'Ocasional'
    END AS segmento,
    CASE
        WHEN ultima_compra >= CURRENT_DATE - INTERVAL '3 months' THEN 2000.00
        WHEN ultima_compra >= CURRENT_DATE - INTERVAL '6 months' THEN 1500.00
        ELSE 1000.00
    END AS limite_credito
FROM lab_workspace.sistema_legado_clientes;

-- Verificar resultados
SELECT * FROM lab_workspace.clientes_novo;
```

### 1.2 Migração de Produtos

```sql
-- Migrar produtos do sistema legado para o novo sistema
INSERT INTO lab_workspace.produtos_novo (
    codigo_legado,
    nome,
    categoria,
    preco,
    custo,
    margem_lucro,
    estoque,
    estoque_minimo,
    fornecedor,
    data_cadastro,
    ultima_atualizacao,
    status
)
SELECT
    produto_id,
    nome,
    categoria,
    preco,
    preco * 0.7 AS custo, -- Estimativa de custo (70% do preço)
    0.3 AS margem_lucro, -- Margem de lucro estimada (30%)
    estoque,
    CASE
        WHEN categoria = 'Informática' THEN 10
        WHEN categoria = 'Eletrônicos' THEN 5
        ELSE 3
    END AS estoque_minimo,
    fornecedor,
    CURRENT_DATE - INTERVAL '1 year' AS data_cadastro, -- Data estimada
    CURRENT_TIMESTAMP,
    'Ativo' AS status
FROM lab_workspace.sistema_legado_produtos;

-- Adicionar descrições aos produtos
UPDATE lab_workspace.produtos_novo
SET descricao = 
    CASE 
        WHEN nome LIKE '%Notebook%' THEN 'Notebook para uso pessoal e profissional'
        WHEN nome LIKE '%Smartphone%' THEN 'Smartphone com câmera de alta resolução'
        WHEN nome LIKE '%Mouse%' THEN 'Mouse ergonômico com conexão sem fio'
        WHEN nome LIKE '%Teclado%' THEN 'Teclado mecânico com iluminação RGB'
        WHEN nome LIKE '%Monitor%' THEN 'Monitor LED com alta definição'
        ELSE 'Produto eletrônico de alta qualidade'
    END;

-- Adicionar subcategorias
UPDATE lab_workspace.produtos_novo
SET subcategoria = 
    CASE 
        WHEN nome LIKE '%Notebook%' THEN 'Laptops'
        WHEN nome LIKE '%Smartphone%' THEN 'Celulares'
        WHEN nome LIKE '%Mouse%' THEN 'Periféricos'
        WHEN nome LIKE '%Teclado%' THEN 'Periféricos'
        WHEN nome LIKE '%Monitor%' THEN 'Monitores'
        ELSE 'Outros'
    END;

-- Verificar resultados
SELECT * FROM lab_workspace.produtos_novo;
```

### 1.3 Migração de Vendas

```sql
-- Migrar vendas do sistema legado para o novo sistema
INSERT INTO lab_workspace.vendas_novo (
    codigo_legado,
    cliente_id,
    data_venda,
    valor_subtotal,
    valor_desconto,
    valor_frete,
    valor_total,
    forma_pagamento,
    status,
    data_atualizacao
)
SELECT
    v.venda_id,
    c.id AS cliente_id,
    v.data_venda::TIMESTAMP,
    v.valor_total AS valor_subtotal,
    0 AS valor_desconto,
    0 AS valor_frete,
    v.valor_total,
    'Cartão de Crédito' AS forma_pagamento, -- Valor padrão
    CASE
        WHEN v.status = 'Concluída' THEN 'Finalizada'
        WHEN v.status = 'Cancelada' THEN 'Cancelada'
        ELSE 'Processando'
    END AS status,
    CURRENT_TIMESTAMP
FROM lab_workspace.sistema_legado_vendas v
JOIN lab_workspace.clientes_novo c ON v.cliente_id = c.codigo_legado;

-- Migrar itens de venda
INSERT INTO lab_workspace.itens_venda_novo (
    venda_id,
    produto_id,
    quantidade,
    preco_unitario,
    valor_desconto,
    valor_total
)
SELECT
    v.id AS venda_id,
    p.id AS produto_id,
    i.quantidade,
    i.preco_unitario,
    0 AS valor_desconto,
    i.quantidade * i.preco_unitario AS valor_total
FROM lab_workspace.sistema_legado_itens_venda i
JOIN lab_workspace.vendas_novo v ON i.venda_id = v.codigo_legado
JOIN lab_workspace.produtos_novo p ON i.produto_id = p.codigo_legado;

-- Verificar resultados
SELECT * FROM lab_workspace.vendas_novo;
SELECT * FROM lab_workspace.itens_venda_novo;
```

## Parte 2: Sincronização de Dados

### 2.1 Criação de Tabelas de Staging

```sql
-- Criar tabelas de staging para novos dados
CREATE TABLE lab_workspace.staging_clientes (
    cliente_id INTEGER PRIMARY KEY,
    nome VARCHAR(100),
    email VARCHAR(100),
    telefone VARCHAR(20),
    data_cadastro DATE,
    ultima_compra DATE,
    status VARCHAR(20),
    observacoes TEXT
);

CREATE TABLE lab_workspace.staging_produtos (
    produto_id INTEGER PRIMARY KEY,
    nome VARCHAR(100),
    categoria VARCHAR(50),
    subcategoria VARCHAR(50),
    preco DECIMAL(10,2),
    custo DECIMAL(10,2),
    estoque INTEGER,
    fornecedor VARCHAR(100),
    status VARCHAR(20)
);

-- Inserir dados de exemplo nas tabelas de staging
INSERT INTO lab_workspace.staging_clientes
VALUES 
    (1001, 'João Silva', 'joao.silva.novo@email.com', '11987654321', '2020-01-15', '2022-07-05', 'Ativo', 'Cliente atualizado'),
    (1006, 'Fernanda Lima', 'fernanda.lima@email.com', '61987654321', '2022-07-01', '2022-07-02', 'Ativo', 'Novo cliente'),
    (1007, 'Roberto Alves', 'roberto.alves@email.com', '71987654321', '2022-07-02', '2022-07-03', 'Ativo', 'Novo cliente');

INSERT INTO lab_workspace.staging_produtos
VALUES 
    (2001, 'Notebook Basic Plus', 'Informática', 'Laptops', 2699.90, 1889.93, 10, 'Tech Distribuidora', 'Ativo'),
    (2006, 'Headphone Bluetooth', 'Eletrônicos', 'Áudio', 199.90, 139.93, 40, 'Sound Solutions', 'Ativo'),
    (2007, 'Webcam HD', 'Informática', 'Periféricos', 149.90, 104.93, 30, 'Visual Tech', 'Ativo');
```

### 2.2 Sincronização de Clientes

```sql
-- Sincronizar clientes (inserir novos e atualizar existentes)
WITH clientes_atualizados AS (
    -- Atualizar clientes existentes
    UPDATE lab_workspace.clientes_novo c
    SET 
        nome = s.nome,
        email = s.email,
        telefone = s.telefone,
        status = s.status,
        ultima_atualizacao = CURRENT_TIMESTAMP,
        segmento = CASE
            WHEN s.ultima_compra >= CURRENT_DATE - INTERVAL '3 months' THEN 'Frequente'
            WHEN s.ultima_compra >= CURRENT_DATE - INTERVAL '6 months' THEN 'Regular'
            ELSE 'Ocasional'
        END
    FROM lab_workspace.staging_clientes s
    WHERE c.codigo_legado = s.cliente_id
    RETURNING c.codigo_legado
)
-- Inserir novos clientes
INSERT INTO lab_workspace.clientes_novo (
    codigo_legado,
    nome,
    email,
    telefone,
    data_cadastro,
    ultima_atualizacao,
    status,
    segmento,
    limite_credito
)
SELECT
    cliente_id,
    nome,
    email,
    telefone,
    data_cadastro,
    CURRENT_TIMESTAMP,
    status,
    CASE
        WHEN ultima_compra >= CURRENT_DATE - INTERVAL '3 months' THEN 'Frequente'
        WHEN ultima_compra >= CURRENT_DATE - INTERVAL '6 months' THEN 'Regular'
        ELSE 'Ocasional'
    END AS segmento,
    CASE
        WHEN ultima_compra >= CURRENT_DATE - INTERVAL '3 months' THEN 2000.00
        WHEN ultima_compra >= CURRENT_DATE - INTERVAL '6 months' THEN 1500.00
        ELSE 1000.00
    END AS limite_credito
FROM lab_workspace.staging_clientes s
WHERE NOT EXISTS (
    SELECT 1 FROM clientes_atualizados ca
    WHERE ca.codigo_legado = s.cliente_id
);

-- Verificar resultados
SELECT * FROM lab_workspace.clientes_novo ORDER BY id;
```

### 2.3 Sincronização de Produtos

```sql
-- Sincronizar produtos (inserir novos e atualizar existentes)
WITH produtos_atualizados AS (
    -- Atualizar produtos existentes
    UPDATE lab_workspace.produtos_novo p
    SET 
        nome = s.nome,
        categoria = s.categoria,
        subcategoria = s.subcategoria,
        preco = s.preco,
        custo = s.custo,
        estoque = s.estoque,
        fornecedor = s.fornecedor,
        status = s.status,
        ultima_atualizacao = CURRENT_TIMESTAMP
    FROM lab_workspace.staging_produtos s
    WHERE p.codigo_legado = s.produto_id
    RETURNING p.codigo_legado
)
-- Inserir novos produtos
INSERT INTO lab_workspace.produtos_novo (
    codigo_legado,
    nome,
    categoria,
    subcategoria,
    preco,
    custo,
    margem_lucro,
    estoque,
    estoque_minimo,
    fornecedor,
    data_cadastro,
    ultima_atualizacao,
    status
)
SELECT
    produto_id,
    nome,
    categoria,
    subcategoria,
    preco,
    custo,
    (preco - custo) / preco AS margem_lucro,
    estoque,
    CASE
        WHEN categoria = 'Informática' THEN 10
        WHEN categoria = 'Eletrônicos' THEN 5
        ELSE 3
    END AS estoque_minimo,
    fornecedor,
    CURRENT_DATE AS data_cadastro,
    CURRENT_TIMESTAMP,
    status
FROM lab_workspace.staging_produtos s
WHERE NOT EXISTS (
    SELECT 1 FROM produtos_atualizados pa
    WHERE pa.codigo_legado = s.produto_id
);

-- Verificar resultados
SELECT * FROM lab_workspace.produtos_novo ORDER BY id;
```

## Parte 3: Validação e Correção de Dados

### 3.1 Verificação de Integridade Referencial

```sql
-- Verificar integridade referencial entre vendas e clientes
SELECT v.id, v.codigo_legado, v.cliente_id
FROM lab_workspace.vendas_novo v
LEFT JOIN lab_workspace.clientes_novo c ON v.cliente_id = c.id
WHERE c.id IS NULL;

-- Verificar integridade referencial entre itens de venda e produtos
SELECT i.venda_id, i.produto_id
FROM lab_workspace.itens_venda_novo i
LEFT JOIN lab_workspace.produtos_novo p ON i.produto_id = p.id
WHERE p.id IS NULL;

-- Verificar integridade referencial entre itens de venda e vendas
SELECT i.venda_id, i.produto_id
FROM lab_workspace.itens_venda_novo i
LEFT JOIN lab_workspace.vendas_novo v ON i.venda_id = v.id
WHERE v.id IS NULL;
```

### 3.2 Correção de Inconsistências

```sql
-- Atualizar valores totais das vendas com base nos itens
UPDATE lab_workspace.vendas_novo v
SET 
    valor_subtotal = subquery.subtotal,
    valor_total = subquery.subtotal
FROM (
    SELECT 
        venda_id,
        SUM(quantidade * preco_unitario) AS subtotal
    FROM lab_workspace.itens_venda_novo
    GROUP BY venda_id
) AS subquery
WHERE v.id = subquery.venda_id
AND (v.valor_subtotal <> subquery.subtotal OR v.valor_total <> subquery.subtotal);

-- Verificar resultados
SELECT 
    v.id, 
    v.valor_subtotal, 
    v.valor_total,
    (SELECT SUM(quantidade * preco_unitario) FROM lab_workspace.itens_venda_novo WHERE venda_id = v.id) AS calculado
FROM lab_workspace.vendas_novo v;
```

### 3.3 Enriquecimento de Dados

```sql
-- Adicionar informações de forma de pagamento com base em padrões
UPDATE lab_workspace.vendas_novo
SET forma_pagamento = 
    CASE 
        WHEN id % 3 = 0 THEN 'Cartão de Crédito'
        WHEN id % 3 = 1 THEN 'Boleto Bancário'
        WHEN id % 3 = 2 THEN 'PIX'
    END
WHERE forma_pagamento = 'Cartão de Crédito';

-- Adicionar valor de frete com base no valor total
UPDATE lab_workspace.vendas_novo
SET 
    valor_frete = 
        CASE
            WHEN valor_total < 100 THEN 15.00
            WHEN valor_total < 500 THEN 25.00
            WHEN valor_total < 1000 THEN 35.00
            ELSE 0.00 -- Frete grátis para compras acima de 1000
        END,
    valor_total = valor_total + 
        CASE
            WHEN valor_total < 100 THEN 15.00
            WHEN valor_total < 500 THEN 25.00
            WHEN valor_total < 1000 THEN 35.00
            ELSE 0.00
        END;

-- Verificar resultados
SELECT id, valor_subtotal, valor_frete, valor_total, forma_pagamento
FROM lab_workspace.vendas_novo;
```

## Parte 4: Exportação e Relatórios

### 4.1 Criação de Visões para Relatórios

```sql
-- Visão de resumo de vendas por cliente
CREATE OR REPLACE VIEW lab_workspace.vw_vendas_por_cliente AS
SELECT 
    c.id AS cliente_id,
    c.nome,
    c.segmento,
    COUNT(v.id) AS total_pedidos,
    SUM(v.valor_total) AS valor_total_compras,
    MAX(v.data_venda) AS ultima_compra,
    MIN(v.data_venda) AS primeira_compra
FROM lab_workspace.clientes_novo c
LEFT JOIN lab_workspace.vendas_novo v ON c.id = v.cliente_id
GROUP BY c.id, c.nome, c.segmento
ORDER BY valor_total_compras DESC NULLS LAST;

-- Visão de produtos mais vendidos
CREATE OR REPLACE VIEW lab_workspace.vw_produtos_mais_vendidos AS
SELECT 
    p.id AS produto_id,
    p.nome,
    p.categoria,
    p.subcategoria,
    SUM(i.quantidade) AS quantidade_vendida,
    SUM(i.quantidade * i.preco_unitario) AS valor_total_vendas,
    COUNT(DISTINCT i.venda_id) AS numero_vendas,
    p.estoque AS estoque_atual,
    CASE 
        WHEN p.estoque = 0 THEN 'Sem estoque'
        WHEN p.estoque < p.estoque_minimo THEN 'Estoque baixo'
        ELSE 'Estoque normal'
    END AS status_estoque
FROM lab_workspace.produtos_novo p
LEFT JOIN lab_workspace.itens_venda_novo i ON p.id = i.produto_id
GROUP BY p.id, p.nome, p.categoria, p.subcategoria, p.estoque, p.estoque_minimo
ORDER BY quantidade_vendida DESC NULLS LAST;

-- Consultar visões
SELECT * FROM lab_workspace.vw_vendas_por_cliente;
SELECT * FROM lab_workspace.vw_produtos_mais_vendidos;
```

### 4.2 Exportação para Tabelas de Relatórios

```sql
-- Criar tabela de relatório de vendas mensais
CREATE TABLE lab_workspace.relatorio_vendas_mensais (
    ano INTEGER,
    mes INTEGER,
    categoria VARCHAR(50),
    subcategoria VARCHAR(50),
    total_vendas INTEGER,
    valor_total DECIMAL(12,2),
    ticket_medio DECIMAL(12,2),
    data_geracao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Preencher tabela de relatório
INSERT INTO lab_workspace.relatorio_vendas_mensais (
    ano,
    mes,
    categoria,
    subcategoria,
    total_vendas,
    valor_total,
    ticket_medio
)
SELECT 
    EXTRACT(YEAR FROM v.data_venda) AS ano,
    EXTRACT(MONTH FROM v.data_venda) AS mes,
    p.categoria,
    p.subcategoria,
    COUNT(DISTINCT v.id) AS total_vendas,
    SUM(i.quantidade * i.preco_unitario) AS valor_total,
    SUM(i.quantidade * i.preco_unitario) / COUNT(DISTINCT v.id) AS ticket_medio
FROM lab_workspace.vendas_novo v
JOIN lab_workspace.itens_venda_novo i ON v.id = i.venda_id
JOIN lab_workspace.produtos_novo p ON i.produto_id = p.id
GROUP BY 
    EXTRACT(YEAR FROM v.data_venda),
    EXTRACT(MONTH FROM v.data_venda),
    p.categoria,
    p.subcategoria
ORDER BY 
    ano, mes, categoria, subcategoria;

-- Verificar resultados
SELECT * FROM lab_workspace.relatorio_vendas_mensais;
```

### 4.3 Exportação para Formatos Externos

```sql
-- Preparar dados para exportação em CSV (simulado com uma consulta)
SELECT 
    c.id,
    c.nome,
    c.email,
    c.telefone,
    c.segmento,
    COUNT(v.id) AS total_pedidos,
    SUM(v.valor_total) AS valor_total,
    MAX(v.data_venda) AS ultima_compra
FROM lab_workspace.clientes_novo c
LEFT JOIN lab_workspace.vendas_novo v ON c.id = v.cliente_id
GROUP BY c.id, c.nome, c.email, c.telefone, c.segmento
ORDER BY c.id;

-- Preparar dados para exportação em JSON (simulado com uma consulta)
SELECT 
    json_build_object(
        'id', p.id,
        'nome', p.nome,
        'categoria', p.categoria,
        'subcategoria', p.subcategoria,
        'preco', p.preco,
        'estoque', p.estoque,
        'vendas', (
            SELECT json_agg(
                json_build_object(
                    'venda_id', i.venda_id,
                    'quantidade', i.quantidade,
                    'valor', i.preco_unitario
                )
            )
            FROM lab_workspace.itens_venda_novo i
            WHERE i.produto_id = p.id
        )
    ) AS produto_json
FROM lab_workspace.produtos_novo p
ORDER BY p.id;
```

## Parte 5: Manutenção e Limpeza

### 5.1 Arquivamento de Dados

```sql
-- Criar tabela de arquivamento para vendas antigas
CREATE TABLE lab_workspace.vendas_arquivo (
    id INTEGER,
    codigo_legado INTEGER,
    cliente_id INTEGER,
    data_venda TIMESTAMP,
    valor_subtotal DECIMAL(10,2),
    valor_desconto DECIMAL(10,2),
    valor_frete DECIMAL(10,2),
    valor_total DECIMAL(10,2),
    forma_pagamento VARCHAR(50),
    status VARCHAR(20),
    data_arquivamento TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Mover vendas antigas para arquivamento
WITH vendas_arquivadas AS (
    DELETE FROM lab_workspace.vendas_novo
    WHERE data_venda < CURRENT_DATE - INTERVAL '1 year'
    RETURNING *
)
INSERT INTO lab_workspace.vendas_arquivo (
    id,
    codigo_legado,
    cliente_id,
    data_venda,
    valor_subtotal,
    valor_desconto,
    valor_frete,
    valor_total,
    forma_pagamento,
    status
)
SELECT
    id,
    codigo_legado,
    cliente_id,
    data_venda,
    valor_subtotal,
    valor_desconto,
    valor_frete,
    valor_total,
    forma_pagamento,
    status
FROM vendas_arquivadas;

-- Verificar resultados
SELECT * FROM lab_workspace.vendas_arquivo;
```

### 5.2 Limpeza de Dados Temporários

```sql
-- Limpar tabelas de staging após sincronização
TRUNCATE TABLE lab_workspace.staging_clientes;
TRUNCATE TABLE lab_workspace.staging_produtos;

-- Verificar se as tabelas estão vazias
SELECT COUNT(*) FROM lab_workspace.staging_clientes;
SELECT COUNT(*) FROM lab_workspace.staging_produtos;
```

### 5.3 Atualização de Estatísticas

```sql
-- Atualizar estatísticas para melhorar performance de consultas
ANALYZE lab_workspace.clientes_novo;
ANALYZE lab_workspace.produtos_novo;
ANALYZE lab_workspace.vendas_novo;
ANALYZE lab_workspace.itens_venda_novo;

-- Registrar conclusão do laboratório
UPDATE lab_workspace.lab_progress
SET 
    completion_time = CURRENT_TIMESTAMP,
    status = 'COMPLETED',
    notes = 'Laboratório de integração de dados concluído com sucesso. Foram realizadas operações de migração, sincronização, validação, exportação e manutenção de dados.'
WHERE lab_name = 'Integração de Dados'
AND completion_time IS NULL;
```

## Desafios Adicionais

1. **Implementar um processo de ETL completo**
   - Extrair dados de múltiplas fontes
   - Transformar e normalizar os dados
   - Carregar em um data warehouse

2. **Criar um sistema de detecção de anomalias**
   - Identificar valores atípicos
   - Detectar padrões suspeitos
   - Gerar alertas automáticos

3. **Desenvolver um processo de reconciliação de dados**
   - Comparar dados entre sistemas
   - Identificar discrepâncias
   - Resolver conflitos automaticamente

## Conclusão

Neste laboratório, você praticou técnicas essenciais de integração de dados, incluindo:

- Migração de dados entre sistemas
- Sincronização e atualização de registros
- Validação e correção de inconsistências
- Enriquecimento de dados
- Criação de relatórios e exportação
- Manutenção e arquivamento

Estas habilidades são fundamentais para profissionais que trabalham com engenharia de dados, ETL, data warehousing e análise de dados.