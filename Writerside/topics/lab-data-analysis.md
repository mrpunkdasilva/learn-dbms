# Laboratório: Análise de Dados

## Objetivo
Desenvolver habilidades avançadas de análise de dados utilizando SQL, incluindo agregações, funções de janela, agrupamentos e técnicas de relatórios dinâmicos.

## Cenário
Você é um analista de dados em uma empresa de varejo que precisa extrair insights valiosos dos dados de vendas, clientes e produtos para apoiar decisões estratégicas.

## Setup Inicial

```sql
-- Criar tabelas adicionais para análise
CREATE TABLE lab_workspace.vendas (
    id SERIAL PRIMARY KEY,
    data_venda DATE NOT NULL,
    cliente_id INTEGER REFERENCES lab_workspace.clientes(id),
    vendedor_id INTEGER,
    loja_id INTEGER,
    valor_total DECIMAL(10,2) NOT NULL,
    desconto DECIMAL(10,2) DEFAULT 0,
    metodo_pagamento VARCHAR(50),
    parcelas INTEGER DEFAULT 1
);

CREATE TABLE lab_workspace.itens_venda (
    id SERIAL PRIMARY KEY,
    venda_id INTEGER REFERENCES lab_workspace.vendas(id),
    produto_id INTEGER REFERENCES lab_workspace.produtos(id),
    quantidade INTEGER NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL
);

CREATE TABLE lab_workspace.lojas (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    regiao VARCHAR(50),
    tipo VARCHAR(50),
    data_inauguracao DATE,
    tamanho_m2 INTEGER
);

CREATE TABLE lab_workspace.vendedores (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    loja_id INTEGER REFERENCES lab_workspace.lojas(id),
    data_contratacao DATE,
    meta_mensal DECIMAL(10,2),
    status VARCHAR(20) DEFAULT 'Ativo'
);

-- Carregar dados de exemplo
INSERT INTO lab_workspace.lojas (nome, regiao, tipo, data_inauguracao, tamanho_m2)
VALUES 
    ('Loja Centro', 'Sul', 'Shopping', '2020-01-15', 500),
    ('Loja Norte', 'Norte', 'Rua', '2019-05-20', 350),
    ('Loja Shopping Plaza', 'Leste', 'Shopping', '2021-03-10', 600),
    ('Loja Beira Mar', 'Nordeste', 'Shopping', '2018-11-05', 450),
    ('Loja Parque', 'Centro-Oeste', 'Rua', '2022-02-28', 300);

INSERT INTO lab_workspace.vendedores (nome, email, loja_id, data_contratacao, meta_mensal)
VALUES 
    ('Carlos Souza', 'carlos@email.com', 1, '2020-02-01', 20000.00),
    ('Mariana Lima', 'mariana@email.com', 1, '2020-03-15', 18000.00),
    ('Paulo Mendes', 'paulo@email.com', 2, '2019-06-10', 15000.00),
    ('Juliana Costa', 'juliana@email.com', 3, '2021-04-05', 22000.00),
    ('Roberto Silva', 'roberto@email.com', 4, '2019-01-10', 19000.00),
    ('Amanda Oliveira', 'amanda@email.com', 5, '2022-03-01', 16000.00);

-- Inserir dados de vendas (últimos 6 meses)
INSERT INTO lab_workspace.vendas (data_venda, cliente_id, vendedor_id, loja_id, valor_total, desconto, metodo_pagamento, parcelas)
SELECT
    CURRENT_DATE - (random() * 180)::integer AS data_venda,
    (random() * 50 + 1)::integer AS cliente_id,
    (random() * 6 + 1)::integer AS vendedor_id,
    (random() * 5 + 1)::integer AS loja_id,
    (random() * 1000 + 100)::numeric(10,2) AS valor_total,
    (random() * 100)::numeric(10,2) AS desconto,
    (ARRAY['Crédito', 'Débito', 'Dinheiro', 'Pix', 'Boleto'])[floor(random() * 5 + 1)] AS metodo_pagamento,
    (ARRAY[1, 2, 3, 6, 10, 12])[floor(random() * 6 + 1)] AS parcelas
FROM generate_series(1, 1000);

-- Inserir itens de venda
INSERT INTO lab_workspace.itens_venda (venda_id, produto_id, quantidade, preco_unitario)
SELECT
    v.id AS venda_id,
    (random() * 100 + 1)::integer AS produto_id,
    (random() * 5 + 1)::integer AS quantidade,
    (random() * 200 + 50)::numeric(10,2) AS preco_unitario
FROM lab_workspace.vendas v
CROSS JOIN generate_series(1, 3) AS num_items
ORDER BY v.id;

-- Registrar início do laboratório
INSERT INTO lab_workspace.lab_progress (lab_name)
VALUES ('Análise de Dados');
```

## Parte 1: Agregações Básicas

### 1.1 Contagem e Somatórios

```sql
-- Total de vendas por loja
SELECT 
    l.nome as loja,
    COUNT(v.id) as total_vendas,
    SUM(v.valor_total) as valor_total
FROM lab_workspace.vendas v
JOIN lab_workspace.lojas l ON v.loja_id = l.id
GROUP BY l.nome
ORDER BY valor_total DESC;

-- Média de vendas por dia da semana
SELECT 
    TO_CHAR(data_venda, 'Day') as dia_semana,
    COUNT(id) as total_vendas,
    ROUND(AVG(valor_total), 2) as ticket_medio
FROM lab_workspace.vendas
GROUP BY dia_semana
ORDER BY ticket_medio DESC;

-- Produtos mais vendidos
SELECT 
    p.nome as produto,
    SUM(iv.quantidade) as quantidade_total,
    SUM(iv.quantidade * iv.preco_unitario) as valor_total
FROM lab_workspace.itens_venda iv
JOIN lab_workspace.produtos p ON iv.produto_id = p.id
GROUP BY p.nome
ORDER BY quantidade_total DESC
LIMIT 10;
```

### 1.2 Funções de Agregação Avançadas

```sql
-- Estatísticas de vendas
SELECT 
    COUNT(*) as total_vendas,
    MIN(valor_total) as menor_venda,
    MAX(valor_total) as maior_venda,
    ROUND(AVG(valor_total), 2) as media_vendas,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY valor_total) as mediana_vendas,
    ROUND(STDDEV(valor_total), 2) as desvio_padrao
FROM lab_workspace.vendas;

-- Distribuição de vendas por faixa de valor
SELECT 
    CASE 
        WHEN valor_total < 200 THEN 'Até R$ 200'
        WHEN valor_total < 500 THEN 'R$ 200 a R$ 500'
        WHEN valor_total < 800 THEN 'R$ 500 a R$ 800'
        ELSE 'Acima de R$ 800'
    END as faixa_valor,
    COUNT(*) as total_vendas,
    ROUND(AVG(valor_total), 2) as valor_medio,
    SUM(valor_total) as valor_total
FROM lab_workspace.vendas
GROUP BY faixa_valor
ORDER BY MIN(valor_total);

-- Análise de métodos de pagamento
SELECT 
    metodo_pagamento,
    COUNT(*) as total_vendas,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM lab_workspace.vendas), 2) as percentual,
    SUM(valor_total) as valor_total
FROM lab_workspace.vendas
GROUP BY metodo_pagamento
ORDER BY total_vendas DESC;
```

## Parte 2: Análise Temporal

### 2.1 Tendências por Período

```sql
-- Vendas por mês
SELECT 
    TO_CHAR(data_venda, 'YYYY-MM') as mes,
    COUNT(*) as total_vendas,
    SUM(valor_total) as valor_total,
    ROUND(AVG(valor_total), 2) as ticket_medio
FROM lab_workspace.vendas
GROUP BY mes
ORDER BY mes;

-- Vendas por semana
SELECT 
    TO_CHAR(data_venda, 'YYYY-IW') as semana,
    COUNT(*) as total_vendas,
    SUM(valor_total) as valor_total
FROM lab_workspace.vendas
GROUP BY semana
ORDER BY semana;

-- Comparação dia a dia (últimos 14 dias)
SELECT 
    data_venda,
    COUNT(*) as total_vendas,
    SUM(valor_total) as valor_total,
    LAG(SUM(valor_total), 1) OVER (ORDER BY data_venda) as valor_dia_anterior,
    ROUND(
        (SUM(valor_total) - LAG(SUM(valor_total), 1) OVER (ORDER BY data_venda)) / 
        NULLIF(LAG(SUM(valor_total), 1) OVER (ORDER BY data_venda), 0) * 100, 
    2) as variacao_percentual
FROM lab_workspace.vendas
WHERE data_venda >= CURRENT_DATE - INTERVAL '14 days'
GROUP BY data_venda
ORDER BY data_venda;
```

### 2.2 Análise de Sazonalidade

```sql
-- Vendas por hora do dia
SELECT 
    EXTRACT(HOUR FROM data_venda::timestamp) as hora,
    COUNT(*) as total_vendas,
    SUM(valor_total) as valor_total
FROM lab_workspace.vendas
GROUP BY hora
ORDER BY hora;

-- Vendas por dia da semana
SELECT 
    EXTRACT(DOW FROM data_venda) as dia_semana_num,
    TO_CHAR(data_venda, 'Day') as dia_semana,
    COUNT(*) as total_vendas,
    SUM(valor_total) as valor_total
FROM lab_workspace.vendas
GROUP BY dia_semana_num, dia_semana
ORDER BY dia_semana_num;

-- Análise de fim de semana vs. dia de semana
SELECT 
    CASE 
        WHEN EXTRACT(DOW FROM data_venda) IN (0, 6) THEN 'Fim de Semana'
        ELSE 'Dia de Semana'
    END as tipo_dia,
    COUNT(*) as total_vendas,
    ROUND(AVG(valor_total), 2) as ticket_medio,
    SUM(valor_total) as valor_total
FROM lab_workspace.vendas
GROUP BY tipo_dia;
```

## Parte 3: Funções de Janela (Window Functions)

### 3.1 Rankings e Partições

```sql
-- Ranking de vendedores por valor total
SELECT 
    v.nome as vendedor,
    COUNT(vd.id) as total_vendas,
    SUM(vd.valor_total) as valor_total,
    RANK() OVER (ORDER BY SUM(vd.valor_total) DESC) as ranking
FROM lab_workspace.vendas vd
JOIN lab_workspace.vendedores v ON vd.vendedor_id = v.id
GROUP BY v.nome
ORDER BY ranking;

-- Top 3 produtos por loja
WITH produtos_por_loja AS (
    SELECT 
        l.nome as loja,
        p.nome as produto,
        SUM(iv.quantidade) as quantidade_total,
        RANK() OVER (PARTITION BY l.id ORDER BY SUM(iv.quantidade) DESC) as ranking
    FROM lab_workspace.itens_venda iv
    JOIN lab_workspace.vendas v ON iv.venda_id = v.id
    JOIN lab_workspace.lojas l ON v.loja_id = l.id
    JOIN lab_workspace.produtos p ON iv.produto_id = p.id
    GROUP BY l.id, l.nome, p.nome
)
SELECT loja, produto, quantidade_total, ranking
FROM produtos_por_loja
WHERE ranking <= 3
ORDER BY loja, ranking;

-- Percentual de vendas por vendedor dentro de cada loja
SELECT 
    l.nome as loja,
    v.nome as vendedor,
    COUNT(vd.id) as total_vendas,
    SUM(vd.valor_total) as valor_total,
    ROUND(
        SUM(vd.valor_total) * 100.0 / 
        SUM(SUM(vd.valor_total)) OVER (PARTITION BY l.id),
    2) as percentual_loja
FROM lab_workspace.vendas vd
JOIN lab_workspace.vendedores v ON vd.vendedor_id = v.id
JOIN lab_workspace.lojas l ON vd.loja_id = l.id
GROUP BY l.id, l.nome, v.nome
ORDER BY l.nome, percentual_loja DESC;
```

### 3.2 Análises Cumulativas

```sql
-- Vendas cumulativas por mês
SELECT 
    TO_CHAR(data_venda, 'YYYY-MM') as mes,
    SUM(valor_total) as valor_mensal,
    SUM(SUM(valor_total)) OVER (ORDER BY TO_CHAR(data_venda, 'YYYY-MM')) as valor_acumulado
FROM lab_workspace.vendas
GROUP BY mes
ORDER BY mes;

-- Média móvel de 7 dias
SELECT 
    data_venda,
    SUM(valor_total) as valor_diario,
    ROUND(
        AVG(SUM(valor_total)) OVER (
            ORDER BY data_venda 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ), 
    2) as media_movel_7dias
FROM lab_workspace.vendas
GROUP BY data_venda
ORDER BY data_venda;

-- Análise de crescimento percentual
SELECT 
    TO_CHAR(data_venda, 'YYYY-MM') as mes,
    SUM(valor_total) as valor_total,
    LAG(SUM(valor_total), 1) OVER (ORDER BY TO_CHAR(data_venda, 'YYYY-MM')) as valor_mes_anterior,
    ROUND(
        (SUM(valor_total) - LAG(SUM(valor_total), 1) OVER (ORDER BY TO_CHAR(data_venda, 'YYYY-MM'))) / 
        NULLIF(LAG(SUM(valor_total), 1) OVER (ORDER BY TO_CHAR(data_venda, 'YYYY-MM')), 0) * 100, 
    2) as crescimento_percentual
FROM lab_workspace.vendas
GROUP BY mes
ORDER BY mes;
```

## Parte 4: Relatórios Dinâmicos

### 4.1 Pivotamento de Dados

```sql
-- Vendas por loja e método de pagamento (pivot)
SELECT 
    l.nome as loja,
    SUM(CASE WHEN v.metodo_pagamento = 'Crédito' THEN v.valor_total ELSE 0 END) as credito,
    SUM(CASE WHEN v.metodo_pagamento = 'Débito' THEN v.valor_total ELSE 0 END) as debito,
    SUM(CASE WHEN v.metodo_pagamento = 'Dinheiro' THEN v.valor_total ELSE 0 END) as dinheiro,
    SUM(CASE WHEN v.metodo_pagamento = 'Pix' THEN v.valor_total ELSE 0 END) as pix,
    SUM(CASE WHEN v.metodo_pagamento = 'Boleto' THEN v.valor_total ELSE 0 END) as boleto,
    SUM(v.valor_total) as total
FROM lab_workspace.vendas v
JOIN lab_workspace.lojas l ON v.loja_id = l.id
GROUP BY l.nome
ORDER BY total DESC;

-- Vendas por categoria de produto e mês
SELECT 
    p.categoria,
    SUM(CASE WHEN TO_CHAR(v.data_venda, 'MM') = '01' THEN iv.quantidade ELSE 0 END) as jan,
    SUM(CASE WHEN TO_CHAR(v.data_venda, 'MM') = '02' THEN iv.quantidade ELSE 0 END) as fev,
    SUM(CASE WHEN TO_CHAR(v.data_venda, 'MM') = '03' THEN iv.quantidade ELSE 0 END) as mar,
    SUM(CASE WHEN TO_CHAR(v.data_venda, 'MM') = '04' THEN iv.quantidade ELSE 0 END) as abr,
    SUM(CASE WHEN TO_CHAR(v.data_venda, 'MM') = '05' THEN iv.quantidade ELSE 0 END) as mai,
    SUM(CASE WHEN TO_CHAR(v.data_venda, 'MM') = '06' THEN iv.quantidade ELSE 0 END) as jun,
    SUM(iv.quantidade) as total
FROM lab_workspace.itens_venda iv
JOIN lab_workspace.vendas v ON iv.venda_id = v.id
JOIN lab_workspace.produtos p ON iv.produto_id = p.id
GROUP BY p.categoria
ORDER BY total DESC;
```

### 4.2 Análise de Correlação

```sql
-- Correlação entre tamanho da loja e vendas
SELECT 
    l.nome as loja,
    l.tamanho_m2,
    COUNT(v.id) as total_vendas,
    SUM(v.valor_total) as valor_total,
    ROUND(SUM(v.valor_total) / l.tamanho_m2, 2) as venda_por_m2
FROM lab_workspace.vendas v
JOIN lab_workspace.lojas l ON v.loja_id = l.id
GROUP BY l.nome, l.tamanho_m2
ORDER BY venda_por_m2 DESC;

-- Análise de vendas por tempo de experiência do vendedor
SELECT 
    v.nome as vendedor,
    EXTRACT(YEAR FROM AGE(CURRENT_DATE, v.data_contratacao)) as anos_experiencia,
    COUNT(vd.id) as total_vendas,
    SUM(vd.valor_total) as valor_total,
    ROUND(AVG(vd.valor_total), 2) as ticket_medio
FROM lab_workspace.vendas vd
JOIN lab_workspace.vendedores v ON vd.vendedor_id = v.id
GROUP BY v.nome, anos_experiencia
ORDER BY anos_experiencia DESC;

-- Correlação entre métodos de pagamento e valor médio
SELECT 
    metodo_pagamento,
    COUNT(*) as total_vendas,
    ROUND(AVG(valor_total), 2) as valor_medio,
    ROUND(AVG(parcelas), 2) as media_parcelas
FROM lab_workspace.vendas
GROUP BY metodo_pagamento
ORDER BY valor_medio DESC;
```

### 4.3 Análise de Segmentação

```sql
-- Segmentação de clientes por valor
WITH vendas_por_cliente AS (
    SELECT 
        c.id,
        c.nome,
        COUNT(v.id) as total_compras,
        SUM(v.valor_total) as valor_total,
        MAX(v.data_venda) as ultima_compra
    FROM lab_workspace.vendas v
    JOIN lab_workspace.clientes c ON v.cliente_id = c.id
    GROUP BY c.id, c.nome
)
SELECT 
    nome as cliente,
    total_compras,
    valor_total,
    ultima_compra,
    CASE 
        WHEN valor_total > 5000 THEN 'Premium'
        WHEN valor_total > 2000 THEN 'Regular'
        ELSE 'Básico'
    END as segmento,
    CASE 
        WHEN CURRENT_DATE - ultima_compra <= 30 THEN 'Ativo'
        WHEN CURRENT_DATE - ultima_compra <= 90 THEN 'Em risco'
        ELSE 'Inativo'
    END as status_atividade
FROM vendas_por_cliente
ORDER BY valor_total DESC;

-- Análise de produtos por margem de contribuição
SELECT 
    p.nome as produto,
    p.categoria,
    SUM(iv.quantidade) as quantidade_vendida,
    SUM(iv.quantidade * iv.preco_unitario) as receita_total,
    ROUND(
        SUM(iv.quantidade * iv.preco_unitario) / 
        NULLIF(SUM(iv.quantidade), 0),
    2) as preco_medio_venda,
    p.preco as preco_atual,
    ROUND(
        (p.preco - (SUM(iv.quantidade * iv.preco_unitario) / NULLIF(SUM(iv.quantidade), 0))) / 
        p.preco * 100,
    2) as variacao_percentual
FROM lab_workspace.itens_venda iv
JOIN lab_workspace.produtos p ON iv.produto_id = p.id
GROUP BY p.nome, p.categoria, p.preco
ORDER BY quantidade_vendida DESC;
```

## Parte 5: Análise Avançada

### 5.1 Detecção de Anomalias

```sql
-- Identificação de vendas atípicas (outliers)
WITH estatisticas_vendas AS (
    SELECT 
        AVG(valor_total) as media,
        STDDEV(valor_total) as desvio_padrao
    FROM lab_workspace.vendas
)
SELECT 
    v.id,
    v.data_venda,
    c.nome as cliente,
    vd.nome as vendedor,
    v.valor_total,
    ROUND((v.valor_total - e.media) / e.desvio_padrao, 2) as z_score
FROM lab_workspace.vendas v
JOIN lab_workspace.clientes c ON v.cliente_id = c.id
JOIN lab_workspace.vendedores vd ON v.vendedor_id = vd.id
CROSS JOIN estatisticas_vendas e
WHERE ABS((v.valor_total - e.media) / e.desvio_padrao) > 2
ORDER BY ABS((v.valor_total - e.media) / e.desvio_padrao) DESC;

-- Detecção de padrões incomuns de compra
SELECT 
    v.cliente_id,
    c.nome as cliente,
    COUNT(v.id) as total_compras,
    COUNT(DISTINCT v.data_venda) as dias_distintos,
    ROUND(COUNT(v.id)::NUMERIC / COUNT(DISTINCT v.data_venda), 2) as compras_por_dia,
    SUM(v.valor_total) as valor_total
FROM lab_workspace.vendas v
JOIN lab_workspace.clientes c ON v.cliente_id = c.id
GROUP BY v.cliente_id, c.nome
HAVING COUNT(v.id) > 10 AND COUNT(v.id)::NUMERIC / COUNT(DISTINCT v.data_venda) > 1.5
ORDER BY compras_por_dia DESC;
```

### 5.2 Análise Preditiva

```sql
-- Previsão simples de vendas futuras (média móvel)
WITH vendas_diarias AS (
    SELECT 
        data_venda,
        SUM(valor_total) as valor_diario
    FROM lab_workspace.vendas
    GROUP BY data_venda
    ORDER BY data_venda
)
SELECT 
    data_venda,
    valor_diario,
    ROUND(
        AVG(valor_diario) OVER (
            ORDER BY data_venda 
            ROWS BETWEEN 7 PRECEDING AND 1 PRECEDING
        ), 
    2) as previsao_media_movel,
    ROUND(
        valor_diario - AVG(valor_diario) OVER (
            ORDER BY data_venda 
            ROWS BETWEEN 7 PRECEDING AND 1 PRECEDING
        ), 
    2) as erro_previsao
FROM vendas_diarias
WHERE data_venda > (SELECT MIN(data_venda) + 7 FROM lab_workspace.vendas)
ORDER BY data_venda;

-- Tendência de crescimento por categoria
WITH vendas_mensais AS (
    SELECT 
        TO_CHAR(v.data_venda, 'YYYY-MM') as mes,
        p.categoria,
        SUM(iv.quantidade * iv.preco_unitario) as valor_total
    FROM lab_workspace.itens_venda iv
    JOIN lab_workspace.vendas v ON iv.venda_id = v.id
    JOIN lab_workspace.produtos p ON iv.produto_id = p.id
    GROUP BY mes, p.categoria
    ORDER BY mes, p.categoria
)
SELECT 
    categoria,
    CORR(
        EXTRACT(EPOCH FROM TO_DATE(mes, 'YYYY-MM')),
        valor_total
    ) as correlacao_temporal,
    CASE 
        WHEN CORR(
            EXTRACT(EPOCH FROM TO_DATE(mes, 'YYYY-MM')),
            valor_total
        ) > 0.7 THEN 'Forte crescimento'
        WHEN CORR(
            EXTRACT(EPOCH FROM TO_DATE(mes, 'YYYY-MM')),
            valor_total
        ) > 0.3 THEN 'Crescimento moderado'
        WHEN CORR(
            EXTRACT(EPOCH FROM TO_DATE(mes, 'YYYY-MM')),
            valor_total
        ) > -0.3 THEN 'Estável'
        WHEN CORR(
            EXTRACT(EPOCH FROM TO_DATE(mes, 'YYYY-MM')),
            valor_total
        ) > -0.7 THEN 'Declínio moderado'
        ELSE 'Forte declínio'
    END as tendencia
FROM vendas_mensais
GROUP BY categoria
ORDER BY correlacao_temporal DESC;
```

### 5.3 Análise de Coorte

```sql
-- Análise de coorte por mês de primeira compra
WITH primeira_compra AS (
    SELECT 
        cliente_id,
        MIN(TO_CHAR(data_venda, 'YYYY-MM')) as coorte
    FROM lab_workspace.vendas
    GROUP BY cliente_id
),
compras_mensais AS (
    SELECT 
        pc.coorte,
        TO_CHAR(v.data_venda, 'YYYY-MM') as mes,
        COUNT(DISTINCT v.cliente_id) as clientes_ativos,
        SUM(v.valor_total) as valor_total
    FROM lab_workspace.vendas v
    JOIN primeira_compra pc ON v.cliente_id = pc.cliente_id
    GROUP BY pc.coorte, mes
    ORDER BY pc.coorte, mes
)
SELECT 
    coorte,
    mes,
    clientes_ativos,
    valor_total,
    ROUND(
        valor_total / clientes_ativos,
    2) as valor_medio_por_cliente,
    ROUND(
        100.0 * clientes_ativos / (
            SELECT COUNT(DISTINCT cliente_id) 
            FROM primeira_compra 
            WHERE coorte = cm.coorte
        ),
    2) as retencao_percentual
FROM compras_mensais cm
ORDER BY coorte, mes;
```

## Parte 6: Visualização e Exportação

### 6.1 Preparação de Dados para Visualização

```sql
-- Dados para gráfico de barras (vendas por região)
SELECT 
    l.regiao,
    COUNT(v.id) as total_vendas,
    SUM(v.valor_total) as valor_total,
    ROUND(AVG(v.valor_total), 2) as ticket_medio
FROM lab_workspace.vendas v
JOIN lab_workspace.lojas l ON v.loja_id = l.id
GROUP BY l.regiao
ORDER BY valor_total DESC;

-- Dados para gráfico de linha (tendência temporal)
SELECT 
    TO_CHAR(data_venda, 'YYYY-MM-DD') as data,
    COUNT(*) as total_vendas,
    SUM(valor_total) as valor_total
FROM lab_workspace.vendas
WHERE data_venda >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY data
ORDER BY data;

-- Dados para gráfico de pizza (distribuição por categoria)
SELECT 
    p.categoria,
    COUNT(iv.venda_id) as total_vendas,
    SUM(iv.quantidade * iv.preco_unitario) as valor_total,
    ROUND(
        SUM(iv.quantidade * iv.preco_unitario) * 100.0 / 
        (SELECT SUM(quantidade * preco_unitario) FROM lab_workspace.itens_venda),
    2) as percentual
FROM lab_workspace.itens_venda iv
JOIN lab_workspace.produtos p ON iv.produto_id = p.id
GROUP BY p.categoria
ORDER BY valor_total DESC;
```

### 6.2 Exportação de Dados

```sql
-- Exportar resultados para CSV
COPY (
    SELECT 
        v.id,
        v.data_venda,
        c.nome as cliente,
        vd.nome as vendedor,
        l.nome as loja,
        v.valor_total,
        v.metodo_pagamento
    FROM lab_workspace.vendas v
    JOIN lab_workspace.clientes c ON v.cliente_id = c.id
    JOIN lab_workspace.vendedores vd ON v.vendedor_id = vd.id
    JOIN lab_workspace.lojas l ON v.loja_id = l.id
    ORDER BY v.data_venda DESC
) TO '/tmp/relatorio_vendas.csv' WITH CSV HEADER;

-- Exportar relatório de desempenho por vendedor
COPY (
    SELECT 
        vd.nome as vendedor,
        l.nome as loja,
        COUNT(v.id) as total_vendas,
        SUM(v.valor_total) as valor_total,
        ROUND(AVG(v.valor_total), 2) as ticket_medio,
        ROUND(
            SUM(v.valor_total) * 100.0 / NULLIF(vd.meta_mensal, 0),
        2) as percentual_meta
    FROM lab_workspace.vendas v
    JOIN lab_workspace.vendedores vd ON v.vendedor_id = vd.id
    JOIN lab_workspace.lojas l ON v.loja_id = l.id
    WHERE TO_CHAR(v.data_venda, 'YYYY-MM') = TO_CHAR(CURRENT_DATE, 'YYYY-MM')
    GROUP BY vd.nome, l.nome, vd.meta_mensal
    ORDER BY percentual_meta DESC
) TO '/tmp/desempenho_vendedores.csv' WITH CSV HEADER;
```

## Conclusão do Laboratório

```sql
-- Registrar conclusão do laboratório
UPDATE lab_workspace.lab_progress
SET 
    completion_time = CURRENT_TIMESTAMP,
    status = 'COMPLETED',
    notes = 'Laboratório de análise de dados concluído com sucesso. Foram realizadas análises de agregação, temporais, funções de janela, relatórios dinâmicos e análises avançadas.'
WHERE lab_name = 'Análise de Dados'
AND completion_time IS NULL;
```

## Desafios Adicionais

1. **Análise de RFM (Recência, Frequência, Monetário)**
   - Segmente clientes com base em padrões de compra
   - Identifique clientes de alto valor

2. **Análise de Cesta de Compras**
   - Descubra produtos frequentemente comprados juntos
   - Identifique oportunidades de cross-selling

3. **Previsão de Demanda**
   - Utilize técnicas de séries temporais
   - Projete vendas futuras por categoria

4. **Dashboard Interativo**
   - Crie visualizações dinâmicas
   - Implemente filtros e parâmetros

## Recursos Adicionais

- [PostgreSQL Window Functions](https://www.postgresql.org/docs/current/tutorial-window.html)
- [PostgreSQL Aggregation](https://www.postgresql.org/docs/current/functions-aggregate.html)
- [Data Analysis with SQL](https://mode.com/sql-tutorial/sql-data-analysis/)
- [Advanced SQL for Data Analysis](https://www.datacamp.com/courses/advanced-sql)

---

> **Nota**: Este laboratório fornece uma base sólida para análise de dados com SQL. As técnicas aprendidas aqui podem ser aplicadas em cenários reais de negócios para extrair insights valiosos e apoiar a tomada de decisões baseada em dados.