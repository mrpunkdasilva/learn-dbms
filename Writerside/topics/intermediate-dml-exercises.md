# Exercícios Intermediários de DML

## Exercício 1: Consultas Avançadas com JOIN

### Descrição
Pratique consultas mais complexas utilizando diferentes tipos de JOIN.

### Requisitos
- INNER JOIN com múltiplas tabelas
- LEFT JOIN com condições complexas
- RIGHT JOIN para análise de dados
- FULL OUTER JOIN para identificar discrepâncias
- CROSS JOIN para análise combinatória

### Solução
```sql
-- INNER JOIN com múltiplas tabelas
SELECT 
    c.nome as cliente,
    p.id as pedido_id,
    p.data_pedido,
    pr.nome as produto,
    ip.quantidade,
    ip.preco_unitario,
    (ip.quantidade * ip.preco_unitario) as subtotal
FROM pratica.clientes c
JOIN pratica.pedidos p ON c.id = p.cliente_id
JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
JOIN pratica.produtos pr ON ip.produto_id = pr.id
WHERE p.status = 'Aprovado'
ORDER BY p.data_pedido DESC;

-- LEFT JOIN com condições complexas
SELECT 
    c.nome,
    c.email,
    COUNT(p.id) as total_pedidos,
    COALESCE(SUM(ip.quantidade * ip.preco_unitario), 0) as valor_total
FROM pratica.clientes c
LEFT JOIN pratica.pedidos p ON c.id = p.cliente_id AND p.status != 'Cancelado'
LEFT JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
GROUP BY c.id, c.nome, c.email
ORDER BY valor_total DESC;

-- RIGHT JOIN para análise de dados
SELECT 
    pr.nome as produto,
    COALESCE(SUM(ip.quantidade), 0) as quantidade_vendida
FROM pratica.itens_pedido ip
RIGHT JOIN pratica.produtos pr ON ip.produto_id = pr.id
GROUP BY pr.id, pr.nome
ORDER BY quantidade_vendida DESC;

-- FULL OUTER JOIN para identificar discrepâncias
SELECT 
    p.id as pedido_id,
    p.status,
    COUNT(ip.produto_id) as itens_no_pedido
FROM pratica.pedidos p
FULL OUTER JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
GROUP BY p.id, p.status
HAVING COUNT(ip.produto_id) = 0 OR p.id IS NULL;

-- CROSS JOIN para análise combinatória
SELECT 
    pr.nome as produto,
    c.nome as cliente,
    'Potencial venda' as oportunidade
FROM pratica.produtos pr
CROSS JOIN pratica.clientes c
WHERE NOT EXISTS (
    SELECT 1 FROM pratica.pedidos p
    JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
    WHERE p.cliente_id = c.id AND ip.produto_id = pr.id
);
```

## Exercício 2: Subconsultas Avançadas

### Descrição
Explore o uso de subconsultas mais complexas para análise de dados.

### Requisitos
- Subconsultas correlacionadas
- Subconsultas com múltiplos níveis
- Subconsultas com operadores de conjunto
- Subconsultas na cláusula HAVING
- Common Table Expressions (CTEs)

### Solução
```sql
-- Subconsultas correlacionadas
SELECT 
    c.nome,
    c.email,
    (SELECT COUNT(*) FROM pratica.pedidos p WHERE p.cliente_id = c.id) as total_pedidos,
    (SELECT MAX(p.data_pedido) FROM pratica.pedidos p WHERE p.cliente_id = c.id) as ultimo_pedido
FROM pratica.clientes c
WHERE (SELECT SUM(ip.quantidade * ip.preco_unitario) 
       FROM pratica.pedidos p 
       JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id 
       WHERE p.cliente_id = c.id) > 1000;

-- Subconsultas com múltiplos níveis
SELECT nome, preco
FROM pratica.produtos
WHERE id IN (
    SELECT produto_id
    FROM pratica.itens_pedido
    WHERE pedido_id IN (
        SELECT id
        FROM pratica.pedidos
        WHERE cliente_id IN (
            SELECT id
            FROM pratica.clientes
            WHERE nome LIKE 'A%'
        )
    )
);

-- Subconsultas com operadores de conjunto
SELECT id, nome
FROM pratica.produtos
WHERE id IN (
    SELECT produto_id
    FROM pratica.itens_pedido
    WHERE quantidade > 5
    
    INTERSECT
    
    SELECT produto_id
    FROM pratica.itens_pedido
    WHERE preco_unitario > 100
);

-- Subconsultas na cláusula HAVING
SELECT 
    p.cliente_id,
    COUNT(*) as total_pedidos
FROM pratica.pedidos p
GROUP BY p.cliente_id
HAVING COUNT(*) > (
    SELECT AVG(pedidos_por_cliente)
    FROM (
        SELECT COUNT(*) as pedidos_por_cliente
        FROM pratica.pedidos
        GROUP BY cliente_id
    ) as stats
);

-- Common Table Expressions (CTEs)
WITH pedidos_por_cliente AS (
    SELECT 
        c.id,
        c.nome,
        COUNT(p.id) as total_pedidos,
        SUM(ip.quantidade * ip.preco_unitario) as valor_total
    FROM pratica.clientes c
    LEFT JOIN pratica.pedidos p ON c.id = p.cliente_id
    LEFT JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
    GROUP BY c.id, c.nome
),
estatisticas AS (
    SELECT 
        AVG(total_pedidos) as media_pedidos,
        AVG(valor_total) as media_valor
    FROM pedidos_por_cliente
)
SELECT 
    ppc.nome,
    ppc.total_pedidos,
    ppc.valor_total,
    CASE 
        WHEN ppc.total_pedidos > e.media_pedidos THEN 'Acima da média'
        ELSE 'Abaixo da média'
    END as status_pedidos,
    CASE 
        WHEN ppc.valor_total > e.media_valor THEN 'Alto valor'
        ELSE 'Baixo valor'
    END as status_valor
FROM pedidos_por_cliente ppc, estatisticas e
ORDER BY ppc.valor_total DESC;
```

## Exercício 3: Operações em Lote

### Descrição
Pratique operações DML que afetam múltiplos registros de forma eficiente.

### Requisitos
- Inserção em lote com SELECT
- Atualização em lote com JOIN
- Exclusão em lote com subconsulta
- Operações condicionais em lote
- Merge/Upsert de dados

### Solução
```sql
-- Inserção em lote com SELECT
INSERT INTO pratica.controle_exercicios (exercicio, completado)
SELECT 'Exercício ' || id, false
FROM generate_series(1, 10) as id;

-- Atualização em lote com JOIN
UPDATE pratica.produtos p
SET estoque = p.estoque - ip.quantidade
FROM pratica.itens_pedido ip
JOIN pratica.pedidos pe ON ip.pedido_id = pe.id
WHERE ip.produto_id = p.id
AND pe.status = 'Aprovado'
AND pe.data_pedido >= CURRENT_DATE - INTERVAL '1 day';

-- Exclusão em lote com subconsulta
DELETE FROM pratica.itens_pedido
WHERE pedido_id IN (
    SELECT id
    FROM pratica.pedidos
    WHERE status = 'Cancelado'
    AND data_pedido < CURRENT_DATE - INTERVAL '90 days'
);

-- Operações condicionais em lote
UPDATE pratica.produtos
SET 
    preco = CASE
        WHEN estoque > 100 THEN preco * 0.9  -- Desconto para produtos com muito estoque
        WHEN estoque < 10 THEN preco * 1.1   -- Aumento para produtos com pouco estoque
        ELSE preco                           -- Mantém o preço para estoque normal
    END,
    nome = CASE
        WHEN nome NOT LIKE '%Novo%' AND data_cadastro > CURRENT_DATE - INTERVAL '30 days'
        THEN nome || ' (Novo)'
        ELSE nome
    END;

-- Merge/Upsert de dados
INSERT INTO pratica.produtos (nome, preco, estoque)
VALUES 
    ('Smartphone Premium', 1999.99, 15),
    ('Notebook Ultra', 4999.99, 8),
    ('Mouse Sem Fio', 99.99, 50)
ON CONFLICT (nome)  -- Assumindo que existe uma constraint UNIQUE em nome
DO UPDATE SET
    preco = EXCLUDED.preco,
    estoque = pratica.produtos.estoque + EXCLUDED.estoque;
```

## Exercício 4: Funções de Janela (Window Functions)

### Descrição
Explore o uso de funções de janela para análises avançadas.

### Requisitos
- Ranking de registros
- Particionamento de dados
- Funções de agregação com janela
- Cálculos cumulativos
- Comparações com registros anteriores/posteriores

### Solução
```sql
-- Ranking de registros
SELECT 
    c.nome,
    COUNT(p.id) as total_pedidos,
    RANK() OVER (ORDER BY COUNT(p.id) DESC) as ranking,
    DENSE_RANK() OVER (ORDER BY COUNT(p.id) DESC) as dense_ranking,
    ROW_NUMBER() OVER (ORDER BY COUNT(p.id) DESC) as row_num
FROM pratica.clientes c
LEFT JOIN pratica.pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nome;

-- Particionamento de dados
SELECT 
    pr.nome as produto,
    EXTRACT(MONTH FROM p.data_pedido) as mes,
    SUM(ip.quantidade) as quantidade_vendida,
    RANK() OVER (
        PARTITION BY EXTRACT(MONTH FROM p.data_pedido)
        ORDER BY SUM(ip.quantidade) DESC
    ) as ranking_mensal
FROM pratica.produtos pr
JOIN pratica.itens_pedido ip ON pr.id = ip.produto_id
JOIN pratica.pedidos p ON ip.pedido_id = p.id
WHERE p.status = 'Aprovado'
GROUP BY pr.nome, mes;

-- Funções de agregação com janela
SELECT 
    c.nome,
    p.id as pedido_id,
    p.data_pedido,
    SUM(ip.quantidade * ip.preco_unitario) as valor_pedido,
    SUM(SUM(ip.quantidade * ip.preco_unitario)) OVER (
        PARTITION BY c.id
    ) as valor_total_cliente,
    SUM(ip.quantidade * ip.preco_unitario) / 
    SUM(SUM(ip.quantidade * ip.preco_unitario)) OVER (
        PARTITION BY c.id
    ) * 100 as percentual_do_total
FROM pratica.clientes c
JOIN pratica.pedidos p ON c.id = p.cliente_id
JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
WHERE p.status = 'Aprovado'
GROUP BY c.id, c.nome, p.id, p.data_pedido
ORDER BY c.nome, p.data_pedido;

-- Cálculos cumulativos
SELECT 
    p.data_pedido::date,
    SUM(ip.quantidade * ip.preco_unitario) as valor_diario,
    SUM(SUM(ip.quantidade * ip.preco_unitario)) OVER (
        ORDER BY p.data_pedido::date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as valor_acumulado
FROM pratica.pedidos p
JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
WHERE p.status = 'Aprovado'
GROUP BY p.data_pedido::date
ORDER BY p.data_pedido::date;

-- Comparações com registros anteriores/posteriores
SELECT 
    p.data_pedido::date,
    SUM(ip.quantidade * ip.preco_unitario) as valor_diario,
    LAG(SUM(ip.quantidade * ip.preco_unitario), 1, 0) OVER (
        ORDER BY p.data_pedido::date
    ) as valor_dia_anterior,
    SUM(ip.quantidade * ip.preco_unitario) - 
    LAG(SUM(ip.quantidade * ip.preco_unitario), 1, 0) OVER (
        ORDER BY p.data_pedido::date
    ) as diferenca,
    CASE 
        WHEN SUM(ip.quantidade * ip.preco_unitario) > 
             LAG(SUM(ip.quantidade * ip.preco_unitario), 1, 0) OVER (
                 ORDER BY p.data_pedido::date
             )
        THEN 'Aumento'
        WHEN SUM(ip.quantidade * ip.preco_unitario) < 
             LAG(SUM(ip.quantidade * ip.preco_unitario), 1, 0) OVER (
                 ORDER BY p.data_pedido::date
             )
        THEN 'Redução'
        ELSE 'Estável'
    END as tendencia
FROM pratica.pedidos p
JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
WHERE p.status = 'Aprovado'
GROUP BY p.data_pedido::date
ORDER BY p.data_pedido::date;
```

## Exercício 5: Manipulação de JSON

### Descrição
Pratique operações com dados em formato JSON.

### Requisitos
- Criação e armazenamento de JSON
- Consulta de propriedades JSON
- Atualização de dados JSON
- Transformação entre JSON e tabelas
- Agregação de dados em JSON

### Solução
```sql
-- Assumindo que temos uma coluna JSON em algumas tabelas
-- Criação e armazenamento de JSON
UPDATE pratica.clientes
SET propriedades = '{"preferencias": {"cor": "azul", "tamanho": "M"}, "contatos": [{"tipo": "telefone", "valor": "123456789"}, {"tipo": "whatsapp", "valor": "987654321"}]}'::jsonb
WHERE id = 1;

-- Consulta de propriedades JSON
SELECT 
    id,
    nome,
    propriedades->'preferencias'->>'cor' as cor_preferida,
    propriedades->'contatos'->0->>'valor' as contato_principal
FROM pratica.clientes
WHERE propriedades->'preferencias'->>'tamanho' = 'M';

-- Atualização de dados JSON
UPDATE pratica.clientes
SET propriedades = jsonb_set(
    propriedades,
    '{preferencias,cor}',
    '"vermelho"'::jsonb
)
WHERE id = 1;

-- Transformação entre JSON e tabelas
SELECT 
    c.id,
    c.nome,
    contato->>'tipo' as tipo_contato,
    contato->>'valor' as valor_contato
FROM pratica.clientes c,
jsonb_array_elements(c.propriedades->'contatos') as contato;

-- Agregação de dados em JSON
SELECT 
    c.id,
    c.nome,
    jsonb_agg(
        jsonb_build_object(
            'pedido_id', p.id,
            'data', p.data_pedido,
            'status', p.status,
            'valor', (
                SELECT SUM(ip.quantidade * ip.preco_unitario)
                FROM pratica.itens_pedido ip
                WHERE ip.pedido_id = p.id
            )
        )
    ) as historico_pedidos
FROM pratica.clientes c
LEFT JOIN pratica.pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nome;
```

## Exercício 6: Expressões de Tabela Comuns (CTE) Recursivas

### Descrição
Explore o uso de CTEs recursivas para análise de dados hierárquicos.

### Requisitos
- Consulta de estruturas hierárquicas
- Cálculo de caminhos
- Agregação em hierarquias
- Detecção de ciclos
- Geração de séries

### Solução
```sql
-- Assumindo uma tabela de categorias com estrutura hierárquica
CREATE TABLE pratica.categorias (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    categoria_pai_id INTEGER REFERENCES pratica.categorias(id)
);

-- Consulta de estruturas hierárquicas
WITH RECURSIVE hierarquia_categorias AS (
    -- Caso base: categorias raiz (sem pai)
    SELECT 
        id, 
        nome, 
        categoria_pai_id, 
        1 as nivel,
        nome as caminho,
        ARRAY[id] as id_path
    FROM pratica.categorias
    WHERE categoria_pai_id IS NULL
    
    UNION ALL
    
    -- Caso recursivo: categorias filhas
    SELECT 
        c.id, 
        c.nome, 
        c.categoria_pai_id, 
        h.nivel + 1,
        h.caminho || ' > ' || c.nome,
        h.id_path || c.id
    FROM pratica.categorias c
    JOIN hierarquia_categorias h ON c.categoria_pai_id = h.id
)
SELECT 
    id,
    nome,
    nivel,
    caminho
FROM hierarquia_categorias
ORDER BY caminho;

-- Geração de séries temporais
WITH RECURSIVE serie_datas AS (
    -- Caso base: data inicial
    SELECT 
        CURRENT_DATE - INTERVAL '30 days' as data
    
    UNION ALL
    
    -- Caso recursivo: próxima data
    SELECT 
        data + INTERVAL '1 day'
    FROM serie_datas
    WHERE data < CURRENT_DATE
)
SELECT 
    s.data,
    COALESCE(COUNT(p.id), 0) as total_pedidos,
    COALESCE(SUM(ip.quantidade * ip.preco_unitario), 0) as valor_total
FROM serie_datas s
LEFT JOIN pratica.pedidos p ON s.data = p.data_pedido::date
LEFT JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
GROUP BY s.data
ORDER BY s.data;
```

## Exercício 7: Pivot e Unpivot

### Descrição
Pratique transformações de dados entre formatos de linha e coluna.

### Requisitos
- Transformação de linhas para colunas (pivot)
- Transformação de colunas para linhas (unpivot)
- Agregações em pivot
- Pivot dinâmico
- Análise cruzada de dados
