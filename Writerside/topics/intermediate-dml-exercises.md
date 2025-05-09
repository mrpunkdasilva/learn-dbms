# Exercícios Intermediários de DML

## Exercício 1: Operações em Lote

### Descrição
Pratique operações DML em lote para manipular múltiplos registros de forma eficiente.

### Requisitos
- Inserção em lote com SELECT
- Atualização em lote com JOIN
- Exclusão em lote com subconsulta
- Operações condicionais em lote

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
```

## Exercício 2: Consultas Avançadas com JOIN

### Descrição
Pratique consultas mais complexas utilizando diferentes tipos de JOIN.

### Requisitos
- INNER JOIN com múltiplas tabelas
- LEFT/RIGHT JOIN com condições
- FULL OUTER JOIN
- CROSS JOIN

### Solução
```sql
-- INNER JOIN com múltiplas tabelas
SELECT 
    c.nome as cliente,
    p.id as pedido_id,
    p.data_pedido,
    SUM(ip.quantidade * ip.preco_unitario) as valor_total
FROM pratica.clientes c
JOIN pratica.pedidos p ON c.id = p.cliente_id
JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
GROUP BY c.nome, p.id, p.data_pedido
ORDER BY p.data_pedido DESC;

-- LEFT/RIGHT JOIN com condições
SELECT 
    c.nome as cliente,
    COUNT(p.id) as total_pedidos,
    COALESCE(SUM(ip.quantidade * ip.preco_unitario), 0) as valor_total_compras
FROM pratica.clientes c
LEFT JOIN pratica.pedidos p ON c.id = p.cliente_id AND p.status != 'Cancelado'
LEFT JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
GROUP BY c.nome
ORDER BY valor_total_compras DESC;

-- FULL OUTER JOIN
SELECT 
    c.nome as cliente,
    p.id as pedido_id,
    p.status
FROM pratica.clientes c
FULL OUTER JOIN pratica.pedidos p ON c.id = p.cliente_id
WHERE c.id IS NULL OR p.id IS NULL;

-- CROSS JOIN
SELECT 
    c.nome as cliente,
    pr.nome as produto,
    'Potencial interesse' as status
FROM pratica.clientes c
CROSS JOIN pratica.produtos pr
WHERE pr.categoria = 'Eletrônicos'
AND NOT EXISTS (
    SELECT 1 FROM pratica.pedidos p
    JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
    WHERE p.cliente_id = c.id
    AND ip.produto_id = pr.id
);
```

## Exercício 3: Subconsultas Correlacionadas

### Descrição
Pratique o uso de subconsultas correlacionadas para resolver problemas complexos.

### Requisitos
- Subconsulta correlacionada no WHERE
- Subconsulta correlacionada no SELECT
- Subconsulta com EXISTS/NOT EXISTS
- Subconsulta com operadores de comparação

### Solução
```sql
-- Subconsulta correlacionada no WHERE
SELECT c.nome, c.email
FROM pratica.clientes c
WHERE (
    SELECT COUNT(*)
    FROM pratica.pedidos p
    WHERE p.cliente_id = c.id
) > 3;

-- Subconsulta correlacionada no SELECT
SELECT 
    p.nome as produto,
    p.preco,
    (
        SELECT AVG(ip.preco_unitario)
        FROM pratica.itens_pedido ip
        WHERE ip.produto_id = p.id
    ) as preco_medio_vendido
FROM pratica.produtos p;

-- Subconsulta com EXISTS/NOT EXISTS
SELECT c.nome
FROM pratica.clientes c
WHERE EXISTS (
    SELECT 1
    FROM pratica.pedidos p
    JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
    JOIN pratica.produtos pr ON ip.produto_id = pr.id
    WHERE p.cliente_id = c.id
    AND pr.preco > 1000
);

-- Subconsulta com operadores de comparação
SELECT p.nome, p.preco
FROM pratica.produtos p
WHERE p.preco > ALL (
    SELECT AVG(preco)
    FROM pratica.produtos
    GROUP BY categoria
);
```

## Exercício 4: Expressões de Tabela Comuns (CTE)

### Descrição
Pratique o uso de CTEs para simplificar consultas complexas.

### Requisitos
- CTE básica
- CTE com múltiplas referências
- CTE recursiva
- CTE com operações DML

### Solução
```sql
-- CTE básica
WITH pedidos_recentes AS (
    SELECT *
    FROM pratica.pedidos
    WHERE data_pedido >= CURRENT_DATE - INTERVAL '30 days'
)
SELECT 
    c.nome as cliente,
    COUNT(pr.id) as total_pedidos_recentes
FROM pratica.clientes c
LEFT JOIN pedidos_recentes pr ON c.id = pr.cliente_id
GROUP BY c.nome;

-- CTE com múltiplas referências
WITH 
    pedidos_por_cliente AS (
        SELECT 
            cliente_id,
            COUNT(*) as total_pedidos,
            SUM(valor_total) as valor_total
        FROM pratica.pedidos
        GROUP BY cliente_id
    ),
    clientes_vip AS (
        SELECT cliente_id
        FROM pedidos_por_cliente
        WHERE total_pedidos >= 5 OR valor_total >= 5000
    )
SELECT 
    c.nome,
    c.email,
    'VIP' as status
FROM pratica.clientes c
JOIN clientes_vip vip ON c.id = vip.cliente_id;

-- CTE recursiva
WITH RECURSIVE hierarquia_categorias AS (
    -- Caso base: categorias raiz (sem pai)
    SELECT id, nome, 0 as nivel
    FROM pratica.categorias
    WHERE categoria_pai_id IS NULL
    
    UNION ALL
    
    -- Caso recursivo: categorias filhas
    SELECT c.id, c.nome, h.nivel + 1
    FROM pratica.categorias c
    JOIN hierarquia_categorias h ON c.categoria_pai_id = h.id
)
SELECT 
    REPEAT('  ', nivel) || nome as categoria_hierarquia,
    nivel
FROM hierarquia_categorias
ORDER BY nivel, nome;

-- CTE com operações DML
WITH produtos_sem_movimento AS (
    SELECT p.id
    FROM pratica.produtos p
    LEFT JOIN pratica.itens_pedido ip ON p.id = ip.produto_id
    WHERE ip.produto_id IS NULL
    AND p.data_cadastro < CURRENT_DATE - INTERVAL '180 days'
)
UPDATE pratica.produtos
SET estoque = 0
WHERE id IN (SELECT id FROM produtos_sem_movimento);
```

## Exercício 5: Operações com MERGE (UPSERT)

### Descrição
Pratique operações de inserção/atualização condicional.

### Requisitos
- INSERT ... ON CONFLICT (PostgreSQL)
- MERGE (SQL Server/Oracle)
- Atualização condicional
- Inserção condicional

### Solução
```sql
-- INSERT ... ON CONFLICT (PostgreSQL)
INSERT INTO pratica.produtos (codigo, nome, preco, estoque)
VALUES 
    ('PROD001', 'Smartphone X', 1299.99, 50),
    ('PROD008', 'Câmera Digital', 899.99, 25)
ON CONFLICT (codigo) 
DO UPDATE SET
    nome = EXCLUDED.nome,
    preco = EXCLUDED.preco,
    estoque = pratica.produtos.estoque + EXCLUDED.estoque;

-- Simulação de MERGE para bancos que não suportam nativamente
-- Atualização condicional
WITH dados_novos (codigo, nome, preco, estoque) AS (
    VALUES 
        ('PROD001', 'Smartphone X Pro', 1499.99, 10),
        ('PROD009', 'Smartwatch', 399.99, 30)
)
UPDATE pratica.produtos p
SET 
    nome = d.nome,
    preco = d.preco,
    estoque = p.estoque + d.estoque
FROM dados_novos d
WHERE p.codigo = d.codigo;

-- Inserção condicional
WITH dados_novos (codigo, nome, preco, estoque) AS (
    VALUES 
        ('PROD001', 'Smartphone X Pro', 1499.99, 10),
        ('PROD009', 'Smartwatch', 399.99, 30)
),
atualizados AS (
    UPDATE pratica.produtos p
    SET 
        nome = d.nome,
        preco = d.preco,
        estoque = p.estoque + d.estoque
    FROM dados_novos d
    WHERE p.codigo = d.codigo
    RETURNING p.codigo
)
INSERT INTO pratica.produtos (codigo, nome, preco, estoque)
SELECT d.codigo, d.nome, d.preco, d.estoque
FROM dados_novos d
WHERE NOT EXISTS (
    SELECT 1 FROM atualizados a
    WHERE a.codigo = d.codigo
);
```

## Exercício 6: Manipulação de Dados Temporais

### Descrição
Pratique operações DML com dados temporais.

### Requisitos
- Filtros por data/hora
- Cálculos com intervalos de tempo
- Agrupamento por períodos
- Comparações temporais

### Solução
```sql
-- Filtros por data/hora
SELECT *
FROM pratica.pedidos
WHERE 
    data_pedido >= CURRENT_DATE - INTERVAL '30 days'
    AND EXTRACT(HOUR FROM data_pedido) BETWEEN 9 AND 18;

-- Cálculos com intervalos de tempo
SELECT 
    id,
    data_pedido,
    data_entrega,
    data_entrega - data_pedido AS tempo_processamento,
    EXTRACT(DAY FROM (data_entrega - data_pedido)) AS dias_processamento
FROM pratica.pedidos
WHERE status = 'Entregue';

-- Agrupamento por períodos
SELECT 
    DATE_TRUNC('month', data_pedido) AS mes,
    COUNT(*) AS total_pedidos,
    SUM(valor_total) AS valor_total,
    AVG(valor_total) AS ticket_medio
FROM pratica.pedidos
WHERE data_pedido >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY mes
ORDER BY mes;

-- Comparações temporais
UPDATE pratica.pedidos
SET status = 'Atrasado'
WHERE 
    status = 'Em Processamento'
    AND data_pedido < CURRENT_DATE - INTERVAL '3 days';
```

## Exercício 7: Operações com JSON

### Descrição
Pratique operações DML com dados em formato JSON.

### Requisitos
- Consulta de dados JSON
- Atualização de dados JSON
- Filtragem por valores JSON
- Transformação JSON para tabela

### Solução
```sql
-- Consulta de dados JSON
SELECT 
    id,
    nome,
    dados_adicionais->>'endereco' AS endereco,
    dados_adicionais->'contatos'->>'telefone' AS telefone
FROM pratica.clientes
WHERE dados_adicionais->>'tipo' = 'Pessoa Física';

-- Atualização de dados JSON
UPDATE pratica.clientes
SET dados_adicionais = jsonb_set(
    dados_adicionais,
    '{endereco}',
    '"Rua Nova, 123"'::jsonb
)
WHERE id = 1;

-- Filtragem por valores JSON
SELECT *
FROM pratica.produtos
WHERE 
    dados_tecnicos->'dimensoes'->>'altura' > '10'
    AND dados_tecnicos->'compatibilidade' ? 'Android';

-- Transformação JSON para tabela
SELECT 
    p.id,
    p.nome,
    a.valor->>'nome' AS atributo_nome,
    a.valor->>'valor' AS atributo_valor
FROM 
    pratica.produtos p,
    jsonb_array_elements(p.atributos) WITH ORDINALITY AS a(valor, idx)
ORDER BY p.id, a.idx;
```

## Exercício 8: Operações com Dados Geoespaciais

### Descrição
Pratique operações DML com dados geoespaciais.

### Requisitos
- Consulta por proximidade
- Cálculo de distâncias
- Filtragem por região
- Agrupamento por área

### Solução
```sql
-- Consulta por proximidade
SELECT 
    id,
    nome,
    ST_Distance(
        localizacao,
        ST_SetSRID(ST_MakePoint(-23.5505, -46.6333), 4326)
    ) AS distancia_km
FROM pratica.lojas
ORDER BY distancia_km
LIMIT 5;

-- Cálculo de distâncias
UPDATE pratica.entregas
SET 
    distancia_km = ST_Distance(
        origem::geography,
        destino::geography
    ) / 1000,
    valor_frete = (ST_Distance(
        origem::geography,
        destino::geography
    ) / 1000) * 2.50  -- R$ 2,50 por km
WHERE status = 'Pendente';

-- Filtragem por região
SELECT 
    c.nome,
    c.email
FROM pratica.clientes c
WHERE ST_Contains(
    (SELECT area FROM pratica.regioes WHERE nome = 'Zona Sul'),
    c.endereco_geo
);

-- Agrupamento por área
SELECT 
    r.nome AS regiao,
    COUNT(c.id) AS total_clientes,
    AVG(c.limite_credito) AS limite_medio
FROM pratica.regioes r
JOIN pratica.clientes c ON ST_Contains(r.area, c.endereco_geo)
GROUP BY r.nome
ORDER BY total_clientes DESC;
```

## Critérios de Avaliação

- Sintaxe correta e otimizada
- Uso adequado de recursos intermediários
- Eficiência das consultas
- Integridade dos dados após manipulação
- Aplicação de boas práticas

## Dicas de Estudo

- Analise o plano de execução das consultas
- Compare diferentes abordagens para o mesmo problema
- Teste com conjuntos de dados maiores
- Verifique o impacto de índices nas consultas
- Pratique a escrita de consultas sem consultar exemplos
