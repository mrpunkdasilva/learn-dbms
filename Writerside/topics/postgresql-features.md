# Características do PostgreSQL

O PostgreSQL é conhecido por seu rico conjunto de recursos que o tornam adequado para uma ampla variedade de casos de uso, desde aplicações simples até sistemas empresariais complexos. Nesta seção, exploraremos em detalhes as características que fazem do PostgreSQL um dos sistemas de gerenciamento de banco de dados mais poderosos e versáteis disponíveis.

## Conformidade com Padrões

O PostgreSQL tem um forte compromisso com a conformidade com padrões SQL:

- **Compatibilidade com SQL**: Implementa uma grande parte do padrão SQL:2016
- **Conformidade ACID**: Garante Atomicidade, Consistência, Isolamento e Durabilidade
- **Transações**: Suporte completo a transações com diferentes níveis de isolamento

```sql
-- Exemplo de transação
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE user_id = 2;
COMMIT;
```

## Tipos de Dados Avançados

O PostgreSQL oferece uma ampla variedade de tipos de dados nativos:

### Tipos Numéricos
- **Inteiros**: smallint, integer, bigint
- **Ponto flutuante**: real, double precision
- **Precisão arbitrária**: numeric, decimal
- **Serial**: smallserial, serial, bigserial (para colunas de autoincremento)

```sql
CREATE TABLE example (
    id serial PRIMARY KEY,
    small_num smallint,
    regular_num integer,
    big_num bigint,
    floating_num real,
    double_num double precision,
    precise_num numeric(10,2)
);
```

### Tipos de Texto
- **Comprimento fixo**: char(n)
- **Comprimento variável**: varchar(n), text
- **Suporte a Unicode**: Completo suporte a UTF-8

### Tipos Binários
- **bytea**: Para armazenamento de dados binários

### Tipos de Data/Hora
- **date**: Apenas data
- **time**: Hora do dia
- **timestamp**: Data e hora
- **timestamptz**: Data e hora com fuso horário
- **interval**: Períodos de tempo
- **Suporte a fusos horários**: Conversão automática entre fusos horários

```sql
CREATE TABLE events (
    id serial PRIMARY KEY,
    event_date date,
    event_time time,
    event_timestamp timestamp,
    event_timestamptz timestamptz,
    duration interval
);
```

### Tipos Booleanos
- **boolean**: true/false

### Tipos Geométricos
- **point**: Ponto no plano
- **line**: Linha infinita
- **lseg**: Segmento de linha finito
- **box**: Retângulo
- **path**: Caminho aberto ou fechado
- **polygon**: Polígono
- **circle**: Círculo

```sql
CREATE TABLE geo_objects (
    id serial PRIMARY KEY,
    location point,
    boundary box,
    area polygon,
    radius circle
);
```

### Tipos de Rede
- **cidr**: Redes IPv4 e IPv6
- **inet**: Endereços IPv4 e IPv6
- **macaddr**: Endereços MAC
- **macaddr8**: Endereços MAC (formato EUI-64)

### Tipos de Array
- Arrays de qualquer tipo de dados
- Arrays multidimensionais
- Funções e operadores específicos para arrays

```sql
CREATE TABLE matrix (
    id serial PRIMARY KEY,
    numbers integer[],
    grid integer[][],
    names text[]
);

INSERT INTO matrix (numbers, grid, names)
VALUES 
    ('{1,2,3}', '{{1,2},{3,4}}', '{"John", "Jane", "Bob"}');
```

### Tipos JSON
- **json**: Armazenamento de dados JSON
- **jsonb**: Armazenamento binário de JSON (mais eficiente para processamento)
- Operadores e funções para manipulação de JSON

```sql
CREATE TABLE documents (
    id serial PRIMARY KEY,
    data json,
    metadata jsonb
);

INSERT INTO documents (data, metadata)
VALUES 
    ('{"title": "Document 1", "content": "Sample content"}', 
     '{"author": "John Doe", "tags": ["sample", "example"]}');
```

### Tipos de Intervalo (Range)
- **int4range**: Intervalo de inteiros
- **int8range**: Intervalo de bigint
- **numrange**: Intervalo de numeric
- **tsrange**: Intervalo de timestamp
- **tstzrange**: Intervalo de timestamp com fuso horário
- **daterange**: Intervalo de datas

```sql
CREATE TABLE reservations (
    id serial PRIMARY KEY,
    room_id integer,
    reserved_during daterange,
    EXCLUDE USING gist (room_id WITH =, reserved_during WITH &&)
);
```

### Outros Tipos Especializados
- **uuid**: Identificadores únicos universais
- **xml**: Dados XML
- **money**: Valores monetários
- **tsvector** e **tsquery**: Para pesquisa de texto completo
- **pg_lsn**: Log Sequence Number do PostgreSQL

### Tipos Personalizados
- **Tipos compostos**: Definidos pelo usuário
- **Tipos enum**: Conjuntos de valores estáticos
- **Tipos domain**: Tipos com restrições

```sql
-- Tipo enum
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');

-- Tipo domain
CREATE DOMAIN positive_int AS integer CHECK (VALUE > 0);

-- Tipo composto
CREATE TYPE address AS (
    street text,
    city text,
    zip text
);

-- Usando tipos personalizados
CREATE TABLE person (
    id serial PRIMARY KEY,
    current_mood mood,
    age positive_int,
    home_address address
);
```

## Extensibilidade

Uma das características mais poderosas do PostgreSQL é sua arquitetura extensível:

### Sistema de Extensões
- Mecanismo formal para adicionar funcionalidades
- Centenas de extensões disponíveis
- Instalação simplificada com CREATE EXTENSION

```sql
-- Instalando a extensão PostGIS para suporte geoespacial
CREATE EXTENSION postgis;

-- Instalando a extensão pgcrypto para funções criptográficas
CREATE EXTENSION pgcrypto;
```

### Extensões Populares
- **PostGIS**: Suporte a dados geoespaciais
- **TimescaleDB**: Otimizado para séries temporais
- **pgvector**: Pesquisa de similaridade vetorial
- **pg_stat_statements**: Análise de desempenho de consultas
- **hstore**: Armazenamento de pares chave-valor
- **ltree**: Suporte a dados hierárquicos
- **pg_trgm**: Pesquisa de similaridade de texto
- **uuid-ossp**: Geração de UUIDs
- **tablefunc**: Funções que retornam tabelas
- **pgcrypto**: Funções criptográficas

### Linguagens Procedurais
- **PL/pgSQL**: Similar ao PL/SQL do Oracle
- **PL/Python**: Programação em Python
- **PL/Perl**: Programação em Perl
- **PL/Tcl**: Programação em Tcl
- **PL/Java**: Programação em Java
- **PL/R**: Programação em R
- **PL/v8**: JavaScript (via V8)
- **Outras linguagens** através de extensões

```sql
-- Função em PL/pgSQL
CREATE FUNCTION add_numbers(a integer, b integer) 
RETURNS integer AS $$
BEGIN
    RETURN a + b;
END;
$$ LANGUAGE plpgsql;

-- Função em PL/Python
CREATE FUNCTION pymax(a integer, b integer)
RETURNS integer AS $$
    return max(a, b)
$$ LANGUAGE plpython3u;
```

### Tipos de Índice
- **B-tree**: Índice padrão para comparações de ordem
- **Hash**: Para igualdade exata
- **GiST**: Índice de pesquisa generalizado
- **SP-GiST**: Índice de particionamento espacial
- **GIN**: Índice invertido generalizado
- **BRIN**: Block Range INdex (para tabelas muito grandes)
- **Bloom**: Filtro de Bloom para múltiplas colunas

```sql
-- Índice B-tree (padrão)
CREATE INDEX idx_customer_name ON customers(name);

-- Índice Hash
CREATE INDEX idx_customer_id_hash ON customers USING hash (customer_id);

-- Índice GIN para arrays
CREATE INDEX idx_tags ON documents USING gin (tags);

-- Índice BRIN para tabelas grandes ordenadas
CREATE INDEX idx_timestamp ON logs USING brin (created_at);
```

## Recursos Avançados de SQL

O PostgreSQL estende o SQL padrão com muitos recursos poderosos:

### Consultas Avançadas
- **Common Table Expressions (CTEs)**: Consultas temporárias nomeadas
- **CTEs Recursivas**: Para consultas hierárquicas
- **Funções de Janela (Window Functions)**: Para análise e agregação
- **GROUPING SETS, CUBE, ROLLUP**: Para agregações múltiplas
- **LATERAL JOINs**: Para subconsultas correlacionadas
- **Tabelas Derivadas**: Subconsultas no FROM

```sql
-- CTE (Common Table Expression)
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region
)
SELECT region, total_sales
FROM regional_sales
WHERE total_sales > 1000;

-- CTE Recursiva
WITH RECURSIVE subordinates AS (
    SELECT employee_id, manager_id, name
    FROM employees
    WHERE employee_id = 2
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name
    FROM employees e
    JOIN subordinates s ON s.employee_id = e.manager_id
)
SELECT * FROM subordinates;

-- Função de Janela
SELECT 
    department,
    employee_name,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as salary_rank
FROM employees;

-- GROUPING SETS
SELECT region, product, SUM(sales)
FROM sales
GROUP BY GROUPING SETS ((region), (product), (region, product), ());
```

### Manipulação de Dados
- **UPSERT**: INSERT com ON CONFLICT
- **RETURNING**: Retorna dados modificados
- **Operações em Massa**: Eficientes para grandes volumes

```sql
-- UPSERT com ON CONFLICT
INSERT INTO customers (id, name, email)
VALUES (1, 'John Doe', 'john@example.com')
ON CONFLICT (id) DO UPDATE
SET name = EXCLUDED.name, email = EXCLUDED.email;

-- RETURNING para obter dados inseridos
INSERT INTO users (name, email)
VALUES ('Jane Smith', 'jane@example.com')
RETURNING id, created_at;
```

### Restrições Avançadas
- **CHECK**: Validações personalizadas
- **EXCLUSION**: Restrições baseadas em operadores
- **DEFERRABLE**: Restrições que podem ser adiadas até o final da transação

```sql
-- Restrição CHECK
CREATE TABLE products (
    id serial PRIMARY KEY,
    name text,
    price numeric CHECK (price > 0),
    discounted_price numeric,
    CHECK (discounted_price IS NULL OR discounted_price < price)
);

-- Restrição EXCLUSION
CREATE TABLE reservations (
    room_id integer,
    during tsrange,
    EXCLUDE USING gist (room_id WITH =, during WITH &&)
);
```

### Objetos de Esquema
- **Schemas**: Namespaces para objetos
- **Herança de Tabelas**: Tabelas podem herdar de outras
- **Particionamento**: Divisão de tabelas grandes
- **Visualizações**: Regulares e materializadas
- **Regras**: Reescrita de consultas

```sql
-- Criando um schema
CREATE SCHEMA analytics;

-- Tabela com herança
CREATE TABLE cities (
    name text,
    population integer,
    elevation integer
);

CREATE TABLE capitals (
    state char(2)
) INHERITS (cities);

-- Particionamento de tabela
CREATE TABLE measurements (
    city_id int not null,
    logdate date not null,
    peaktemp int,
    unitsales int
) PARTITION BY RANGE (logdate);

CREATE TABLE measurements_y2020 PARTITION OF measurements
    FOR VALUES FROM ('2020-01-01') TO ('2021-01-01');

CREATE TABLE measurements_y2021 PARTITION OF measurements
    FOR VALUES FROM ('2021-01-01') TO ('2022-01-01');
```

## Controle de Concorrência

O PostgreSQL implementa um sofisticado sistema de controle de concorrência:

### MVCC (Multiversion Concurrency Control)
- Permite leituras e escritas simultâneas
- Cada transação vê um snapshot consistente dos dados
- Elimina a maioria dos bloqueios de leitura

### Níveis de Isolamento de Transação
- **READ UNCOMMITTED**: Na prática, igual a READ COMMITTED no PostgreSQL
- **READ COMMITTED**: Cada comando vê apenas dados confirmados
- **REPEATABLE READ**: A transação vê apenas dados confirmados antes do início
- **SERIALIZABLE**: Garante que transações concorrentes tenham o mesmo efeito que se fossem executadas sequencialmente

```sql
-- Definindo o nível de isolamento
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- operações da transação
COMMIT;
```

### Sistema de Bloqueio
- Bloqueios em nível de tabela e linha
- Bloqueios consultivos (advisory locks)
- Detecção automática de deadlock

```sql
-- Bloqueio explícito
BEGIN;
LOCK TABLE accounts IN SHARE MODE;
-- operações
COMMIT;

-- Bloqueio consultivo
SELECT pg_advisory_lock(123);
-- operações críticas
SELECT pg_advisory_unlock(123);
```

## Alta Disponibilidade e Replicação

O PostgreSQL oferece várias opções para alta disponibilidade:

### Replicação Física
- **Streaming Replication**: Replicação em tempo real baseada em WAL
- **Cascading Replication**: Réplicas podem servir como origem para outras réplicas
- **Synchronous Replication**: Confirmações só retornam após dados serem replicados

### Replicação Lógica
- Replicação baseada em mudanças de dados em nível lógico
- Permite replicação seletiva de tabelas
- Suporta replicação entre versões diferentes do PostgreSQL

### Recuperação de Falhas
- **Write-Ahead Logging (WAL)**: Registro de todas as mudanças
- **Point-in-Time Recovery**: Recuperação para qualquer momento no passado
- **Arquivamento Contínuo**: Armazenamento de longo prazo de WAL

## Segurança

O PostgreSQL inclui recursos abrangentes de segurança:

### Autenticação
- Senha (MD5, SCRAM-SHA-256)
- Certificado SSL/TLS
- GSSAPI/Kerberos
- LDAP
- RADIUS
- PAM
- Autenticação baseada em host (pg_hba.conf)

### Autorização
- Sistema de roles (funções) flexível
- Privilégios granulares
- Segurança em nível de linha (Row-Level Security)
- Políticas de segurança

```sql
-- Criando roles
CREATE ROLE app_read WITH LOGIN PASSWORD 'secret';
CREATE ROLE app_write WITH LOGIN PASSWORD 'secret';

-- Concedendo privilégios
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_read;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_write;

-- Segurança em nível de linha
CREATE TABLE accounts (
    id serial PRIMARY KEY,
    owner_id integer,
    balance numeric
);

ALTER TABLE accounts ENABLE ROW LEVEL SECURITY;

CREATE POLICY account_owner ON accounts
    USING (owner_id = current_user_id());
```

### Criptografia
- Conexões criptografadas via SSL/TLS
- Criptografia de dados via pgcrypto
- Criptografia de senhas

## Monitoramento e Diagnóstico

O PostgreSQL fornece ferramentas extensivas para monitoramento:

### Estatísticas
- Visões de estatísticas do sistema
- Coletor de estatísticas
- Monitoramento de atividade

```sql
-- Consultas ativas
SELECT * FROM pg_stat_activity;

-- Estatísticas de tabelas
SELECT * FROM pg_stat_user_tables;

-- Estatísticas de índices
SELECT * FROM pg_stat_user_indexes;
```

### Logging
- Configuração flexível de log
- Análise de consultas lentas
- Logging de erros e avisos

### Ferramentas de Diagnóstico
- EXPLAIN para análise de planos de consulta
- EXPLAIN ANALYZE para execução e análise
- pg_stat_statements para análise de desempenho

```sql
-- Analisando o plano de execução
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

-- Executando e analisando
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 123;
```

## Conclusão

O PostgreSQL oferece um conjunto impressionante de recursos que o tornam adequado para uma ampla variedade de casos de uso, desde aplicações simples até sistemas empresariais complexos. Sua combinação de conformidade com padrões, extensibilidade, tipos de dados avançados e recursos de segurança robustos o posicionam como uma escolha excelente para desenvolvedores e organizações que precisam de um sistema de banco de dados confiável e poderoso.

Nos próximos capítulos, exploraremos como instalar, configurar e utilizar efetivamente esses recursos para criar aplicações de banco de dados eficientes e robustas.

---

Continue para [PostgreSQL vs. Outros Bancos de Dados](postgresql-vs-others.md) para entender como o PostgreSQL se compara a outros sistemas populares de gerenciamento de banco de dados.