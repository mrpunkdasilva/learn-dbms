# Exercícios Avançados de DML

## Exercício 1: Operações de Sincronização de Dados

### Descrição
Pratique operações de sincronização entre tabelas e esquemas.

### Requisitos
- Sincronização bidirecional
- Detecção e resolução de conflitos
- Sincronização seletiva
- Registro de auditoria

### Solução
```sql
-- Sincronização bidirecional
WITH 
    origem_alteracoes AS (
        SELECT id, nome, preco, estoque, ultima_atualizacao
        FROM pratica.produtos_origem
        WHERE ultima_atualizacao > (SELECT MAX(ultima_sincronizacao) FROM pratica.controle_sync)
    ),
    destino_alteracoes AS (
        SELECT id, nome, preco, estoque, ultima_atualizacao
        FROM pratica.produtos_destino
        WHERE ultima_atualizacao > (SELECT MAX(ultima_sincronizacao) FROM pratica.controle_sync)
    ),
    conflitos AS (
        SELECT o.id
        FROM origem_alteracoes o
        JOIN destino_alteracoes d ON o.id = d.id
    ),
    atualizacoes_origem AS (
        UPDATE pratica.produtos_destino pd
        SET 
            nome = po.nome,
            preco = po.preco,
            estoque = po.estoque,
            ultima_atualizacao = CURRENT_TIMESTAMP
        FROM pratica.produtos_origem po
        WHERE pd.id = po.id
        AND po.ultima_atualizacao > pd.ultima_atualizacao
        AND po.id NOT IN (SELECT id FROM conflitos)
        RETURNING pd.id
    ),
    atualizacoes_destino AS (
        UPDATE pratica.produtos_origem po
        SET 
            nome = pd.nome,
            preco = pd.preco,
            estoque = pd.estoque,
            ultima_atualizacao = CURRENT_TIMESTAMP
        FROM pratica.produtos_destino pd
        WHERE po.id = pd
        AND pd.ultima_atualizacao > po.ultima_atualizacao
        AND pd.id NOT IN (SELECT id FROM conflitos)
        RETURNING po.id
    ),
    insercoes_origem AS (
        INSERT INTO pratica.produtos_destino (id, nome, preco, estoque, ultima_atualizacao)
        SELECT id, nome, preco, estoque, ultima_atualizacao
        FROM origem_alteracoes
        WHERE id NOT IN (SELECT id FROM destino_alteracoes)
        RETURNING id
    ),
    insercoes_destino AS (
        INSERT INTO pratica.produtos_origem (id, nome, preco, estoque, ultima_atualizacao)
        SELECT id, nome, preco, estoque, ultima_atualizacao
        FROM destino_alteracoes
        WHERE id NOT IN (SELECT id FROM origem_alteracoes)
        RETURNING id
    )
SELECT 
    'Atualizou produtos origem' as operacao,
    atualizacoes_origem
FROM atualizacoes_origem
UNION ALL
SELECT 
    'Atualizou produtos destino' as operacao,
    atualizacoes_destino
FROM atualizacoes_destino
UNION ALL
SELECT 
    'Inseriu produtos origem' as operacao,
    insercoes_origem
FROM insercoes_origem
UNION ALL
SELECT 
    'Inseriu produtos destino' as operacao,
    insercoes_destino
FROM insercoes_destino;

-- Atualização do controle de sincronização
UPDATE pratica.controle_sync
SET ultima_sincronizacao = CURRENT_TIMESTAMP;
```

## Exercício 2: Manipulação de Dados JSON/JSONB

### Descrição
Trabalhe com armazenamento e manipulação de dados em formato JSON.

### Requisitos
- Armazenamento de estruturas complexas
- Consultas em dados JSON
- Modificação de estruturas JSON
- Indexação de campos JSON

### Solução
```sql
-- Tabela com dados JSON
CREATE TABLE pratica.configuracoes (
    id SERIAL PRIMARY KEY,
    aplicacao VARCHAR(50) NOT NULL,
    config JSONB NOT NULL,
    ultima_modificacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Inserção de dados JSON
INSERT INTO pratica.configuracoes (aplicacao, config)
VALUES (
    'sistema_vendas',
    '{
        "database": {
            "host": "db.example.com",
            "port": 5432,
            "credentials": {
                "username": "app_user",
                "password": "encrypted:abc123"
            }
        },
        "features": ["vendas", "estoque", "relatorios"],
        "limits": {
            "max_users": 100,
            "max_transactions": 10000
        },
        "logging": {
            "level": "info",
            "retention_days": 30
        }
    }'::jsonb
);

-- Consultas em dados JSON
SELECT 
    aplicacao,
    config->'database'->>'host' as db_host,
    config->'database'->>'port' as db_port,
    config->'features' as recursos_ativos,
    jsonb_array_length(config->'features') as total_recursos
FROM pratica.configuracoes
WHERE config->'limits'->>'max_users' = '100';

-- Atualização de dados JSON
UPDATE pratica.configuracoes
SET 
    config = jsonb_set(
        jsonb_set(
            config, 
            '{database,host}', 
            '"new-db.example.com"'
        ),
        '{limits,max_users}', 
        '200'
    ),
    ultima_modificacao = CURRENT_TIMESTAMP
WHERE aplicacao = 'sistema_vendas';

-- Adição de elementos em arrays JSON
UPDATE pratica.configuracoes
SET 
    config = jsonb_set(
        config, 
        '{features}', 
        config->'features' || '"financeiro"'::jsonb
    ),
    ultima_modificacao = CURRENT_TIMESTAMP
WHERE aplicacao = 'sistema_vendas';

-- Índice para consultas em JSON
CREATE INDEX idx_config_features ON pratica.configuracoes 
USING gin (config->'features');
```

## Exercício 3: Operações com Window Functions

### Descrição
Utilize window functions para análises avançadas de dados.

### Requisitos
- Cálculo de métricas por janela
- Ranking e particionamento
- Análises cumulativas
- Comparações com períodos anteriores

### Solução
```sql
-- Tabela de vendas para análise
CREATE TABLE IF NOT EXISTS pratica.vendas_mensais (
    id SERIAL PRIMARY KEY,
    produto_id INTEGER NOT NULL,
    categoria_id INTEGER NOT NULL,
    regiao VARCHAR(50) NOT NULL,
    data_venda DATE NOT NULL,
    quantidade INTEGER NOT NULL,
    valor_total DECIMAL(12,2) NOT NULL
);

-- Ranking de produtos por região
SELECT 
    regiao,
    produto_id,
    SUM(valor_total) as total_vendas,
    RANK() OVER (PARTITION BY regiao ORDER BY SUM(valor_total) DESC) as ranking_regional,
    DENSE_RANK() OVER (ORDER BY SUM(valor_total) DESC) as ranking_geral
FROM pratica.vendas_mensais
GROUP BY regiao, produto_id;

-- Análise de tendências com janelas móveis
SELECT 
    data_venda,
    categoria_id,
    SUM(valor_total) as vendas_diarias,
    AVG(SUM(valor_total)) OVER (
        PARTITION BY categoria_id 
        ORDER BY data_venda 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as media_movel_7dias,
    SUM(SUM(valor_total)) OVER (
        PARTITION BY categoria_id 
        ORDER BY data_venda 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as vendas_acumuladas
FROM pratica.vendas_mensais
GROUP BY data_venda, categoria_id
ORDER BY categoria_id, data_venda;

-- Comparação com período anterior
SELECT 
    EXTRACT(YEAR FROM data_venda) as ano,
    EXTRACT(MONTH FROM data_venda) as mes,
    SUM(valor_total) as vendas_mes,
    LAG(SUM(valor_total), 1) OVER (
        PARTITION BY EXTRACT(MONTH FROM data_venda) 
        ORDER BY EXTRACT(YEAR FROM data_venda)
    ) as vendas_mes_ano_anterior,
    SUM(valor_total) - LAG(SUM(valor_total), 1) OVER (
        PARTITION BY EXTRACT(MONTH FROM data_venda) 
        ORDER BY EXTRACT(YEAR FROM data_venda)
    ) as diferenca,
    CASE 
        WHEN LAG(SUM(valor_total), 1) OVER (
            PARTITION BY EXTRACT(MONTH FROM data_venda) 
            ORDER BY EXTRACT(YEAR FROM data_venda)
        ) > 0 THEN
            ROUND(
                (SUM(valor_total) - LAG(SUM(valor_total), 1) OVER (
                    PARTITION BY EXTRACT(MONTH FROM data_venda) 
                    ORDER BY EXTRACT(YEAR FROM data_venda)
                )) / LAG(SUM(valor_total), 1) OVER (
                    PARTITION BY EXTRACT(MONTH FROM data_venda) 
                    ORDER BY EXTRACT(YEAR FROM data_venda)
                ) * 100
            )
        ELSE NULL
    END as variacao_percentual
FROM pratica.vendas_mensais
GROUP BY ano, mes
ORDER BY ano, mes;
```

## Exercício 4: Operações com Common Table Expressions (CTEs)

### Descrição
Utilize CTEs para consultas complexas e recursivas.

### Requisitos
- CTEs para modularização de consultas
- CTEs recursivas para estruturas hierárquicas
- CTEs para operações em múltiplas etapas
- Análise de caminhos em grafos

### Solução
```sql
-- Tabela de categorias hierárquicas
CREATE TABLE IF NOT EXISTS pratica.categorias (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    categoria_pai_id INTEGER REFERENCES pratica.categorias(id)
);

-- CTE recursiva para hierarquia de categorias
WITH RECURSIVE hierarquia AS (
    -- Caso base: categorias raiz
    SELECT 
        id, 
        nome, 
        categoria_pai_id, 
        1 as nivel,
        ARRAY[id] as caminho,
        nome as caminho_nome
    FROM pratica.categorias
    WHERE categoria_pai_id IS NULL
    
    UNION ALL
    
    -- Caso recursivo: categorias filhas
    SELECT 
        c.id, 
        c.nome, 
        c.categoria_pai_id, 
        h.nivel + 1,
        h.caminho || c.id,
        h.caminho_nome || ' > ' || c.nome
    FROM pratica.categorias c
    JOIN hierarquia h ON c.categoria_pai_id = h.id
)
SELECT 
    id,
    nome,
    nivel,
    caminho,
    caminho_nome
FROM hierarquia
ORDER BY caminho;

-- CTE para análise de vendas em múltiplas etapas
WITH vendas_mensais AS (
    SELECT 
        EXTRACT(YEAR FROM data_venda) as ano,
        EXTRACT(MONTH FROM data_venda) as mes,
        categoria_id,
        SUM(valor_total) as total_vendas
    FROM pratica.vendas_mensais
    GROUP BY ano, mes, categoria_id
),
ranking_mensal AS (
    SELECT 
        ano,
        mes,
        categoria_id,
        total_vendas,
        RANK() OVER (PARTITION BY ano, mes ORDER BY total_vendas DESC) as ranking
    FROM vendas_mensais
),
top_categorias AS (
    SELECT 
        ano,
        mes,
        categoria_id,
        total_vendas
    FROM ranking_mensal
    WHERE ranking <= 3
)
SELECT 
    tc.ano,
    tc.mes,
    tc.categoria_id,
    c.nome as categoria,
    tc.total_vendas,
    ROUND(tc.total_vendas / SUM(vm.total_vendas) OVER (PARTITION BY tc.ano, tc.mes) * 100, 2) as percentual_total
FROM top_categorias tc
JOIN pratica.categorias c ON tc.categoria_id = c.id
JOIN vendas_mensais vm ON tc.ano = vm.ano AND tc.mes = vm.mes
ORDER BY tc.ano, tc.mes, tc.total_vendas DESC;
```

## Exercício 5: Operações com Pivotamento de Dados

### Descrição
Transforme dados de formato linha para coluna e vice-versa.

### Requisitos
- Pivotamento dinâmico de linhas para colunas
- Despivotamento de colunas para linhas
- Agregações em pivotamento
- Formatação de relatórios

### Solução
```sql
-- Tabela de vendas por região e produto
CREATE TABLE IF NOT EXISTS pratica.vendas_regiao (
    id SERIAL PRIMARY KEY,
    data_venda DATE NOT NULL,
    regiao VARCHAR(50) NOT NULL,
    produto_id INTEGER NOT NULL,
    quantidade INTEGER NOT NULL,
    valor_total DECIMAL(12,2) NOT NULL
);

-- Pivotamento de linhas para colunas (vendas por região)
SELECT 
    data_venda,
    produto_id,
    SUM(CASE WHEN regiao = 'Norte' THEN valor_total ELSE 0 END) as norte,
    SUM(CASE WHEN regiao = 'Sul' THEN valor_total ELSE 0 END) as sul,
    SUM(CASE WHEN regiao = 'Leste' THEN valor_total ELSE 0 END) as leste,
    SUM(CASE WHEN regiao = 'Oeste' THEN valor_total ELSE 0 END) as oeste,
    SUM(valor_total) as total_geral
FROM pratica.vendas_regiao
GROUP BY data_venda, produto_id
ORDER BY data_venda, produto_id;

-- Pivotamento dinâmico usando crosstab (PostgreSQL)
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT * FROM crosstab(
    'SELECT 
        produto_id, 
        regiao, 
        SUM(valor_total)
     FROM pratica.vendas_regiao
     WHERE data_venda BETWEEN ''2023-01-01'' AND ''2023-01-31''
     GROUP BY produto_id, regiao
     ORDER BY 1, 2',
    'SELECT DISTINCT regiao FROM pratica.vendas_regiao ORDER BY 1'
) AS ct (
    produto_id INTEGER,
    norte DECIMAL,
    sul DECIMAL,
    leste DECIMAL,
    oeste DECIMAL
);

-- Despivotamento de colunas para linhas
CREATE TABLE IF NOT EXISTS pratica.relatorio_mensal (
    produto_id INTEGER NOT NULL,
    mes DATE NOT NULL,
    norte DECIMAL(12,2),
    sul DECIMAL(12,2),
    leste DECIMAL(12,2),
    oeste DECIMAL(12,2),
    PRIMARY KEY (produto_id, mes)
);

-- Despivotamento (PostgreSQL)
SELECT 
    produto_id,
    mes,
    regiao,
    valor
FROM pratica.relatorio_mensal
CROSS JOIN LATERAL (
    VALUES 
        ('Norte', norte),
        ('Sul', sul),
        ('Leste', leste),
        ('Oeste', oeste)
) as regioes(regiao, valor)
WHERE valor > 0
ORDER BY produto_id, mes, regiao;
```

## Exercício 6: Operações com Dados Temporais

### Descrição
Trabalhe com análises temporais avançadas.

### Requisitos
- Análise de séries temporais
- Cálculo de períodos de negócio
- Detecção de padrões temporais
- Agregações por intervalos customizados

### Solução
```sql
-- Tabela de eventos temporais
CREATE TABLE IF NOT EXISTS pratica.eventos_sistema (
    id SERIAL PRIMARY KEY,
    tipo_evento VARCHAR(50) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    usuario_id INTEGER,
    duracao_ms INTEGER,
    detalhes JSONB
);

-- Análise de distribuição por hora do dia
SELECT 
    EXTRACT(HOUR FROM timestamp) as hora,
    COUNT(*) as total_eventos,
    AVG(duracao_ms) as duracao_media,
    COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () as percentual
FROM pratica.eventos_sistema
WHERE timestamp >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY hora
ORDER BY hora;

-- Cálculo de tempo entre eventos (lag/lead)
SELECT 
    id,
    tipo_evento,
    timestamp,
    usuario_id,
    EXTRACT(EPOCH FROM (timestamp - LAG(timestamp) OVER (
        PARTITION BY usuario_id 
        ORDER BY timestamp
    ))) as segundos_desde_ultimo_evento,
    EXTRACT(EPOCH FROM (LEAD(timestamp) OVER (
        PARTITION BY usuario_id 
        ORDER BY timestamp
    ) - timestamp)) as segundos_ate_proximo_evento
FROM pratica.eventos_sistema
WHERE usuario_id = 123
ORDER BY timestamp;

-- Agregação por intervalos customizados (15 minutos)
SELECT 
    DATE_TRUNC('day', timestamp) as dia,
    (EXTRACT(HOUR FROM timestamp) * 4 + 
     FLOOR(EXTRACT(MINUTE FROM timestamp) / 15)) * 15 as minuto_do_dia,
    COUNT(*) as total_eventos,
    SUM(duracao_ms) as duracao_total
FROM pratica.eventos_sistema
WHERE timestamp >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY dia, minuto_do_dia
ORDER BY dia, minuto_do_dia;

-- Detecção de padrões temporais (3 falhas em 5 minutos)
WITH eventos_falha AS (
    SELECT 
        timestamp,
        usuario_id,
        tipo_evento,
        COUNT(*) OVER (
            PARTITION BY usuario_id 
            ORDER BY timestamp 
            RANGE BETWEEN INTERVAL '5 minutes' PRECEDING AND CURRENT ROW
        ) as falhas_recentes
    FROM pratica.eventos_sistema
    WHERE tipo_evento = 'login_falha'
)
SELECT DISTINCT
    usuario_id,
    MIN(timestamp) OVER (PARTITION BY usuario_id) as primeira_falha,
    COUNT(*) OVER (PARTITION BY usuario_id) as total_falhas
FROM eventos_falha
WHERE falhas_recentes >= 3
ORDER BY usuario_id;
```

## Exercício 7: Operações com Dados Geoespaciais

### Descrição
Trabalhe com dados geoespaciais para análises baseadas em localização.

### Requisitos
- Armazenamento de coordenadas
- Cálculo de distâncias
- Consultas por proximidade
- Análises de áreas

### Solução
```sql
-- Extensão PostGIS (PostgreSQL)
CREATE EXTENSION IF NOT EXISTS postgis;

-- Tabela de locais
CREATE TABLE IF NOT EXISTS pratica.locais (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    tipo VARCHAR(50) NOT NULL,
    geom GEOMETRY(Point, 4326) NOT NULL
);

-- Inserção de locais
INSERT INTO pratica.locais (nome, tipo, geom)
VALUES 
    ('Parque Central', 'Parque', ST_SetSRID(ST_MakePoint(-46.6333, -23.5505), 4326)),
    ('Museu de Arte', 'Museu', ST_SetSRID(ST_MakePoint(-46.6358, -23.5515), 4326)),
    ('Teatro Municipal', 'Teatro', ST_SetSRID(ST_MakePoint(-46.6345, -23.5520), 4326));

-- Consulta de locais próximos
SELECT 
    nome,
    tipo,
    ST_Distance(geom, ST_SetSRID(ST_MakePoint(-46.6333, -23.5505), 4326)) as distancia_km
FROM pratica.locais
ORDER BY distancia_km;

-- Cálculo de área de um polígono
CREATE TABLE IF NOT EXISTS pratica.poligonos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    geom GEOMETRY(Polygon, 4326) NOT NULL
);

INSERT INTO pratica.poligonos (nome, geom)
VALUES (
    'Área Turística',
    ST_SetSRID(ST_GeomFromText('POLYGON((-46.6333 -23.5505, -46.6358 -23.5515, -46.6345 -23.5520, -46.6333 -23.5505))'), 4326)
);

SELECT 
    nome,
    ST_Area(geom) as area_km2
FROM pratica.poligonos;
```
