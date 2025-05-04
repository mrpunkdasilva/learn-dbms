# Laboratório: Transformação de Dados

## Objetivo
Desenvolver habilidades de transformação e manipulação de dados utilizando SQL, incluindo conversão de formatos, limpeza, normalização e enriquecimento de dados.

## Cenário
Você é um engenheiro de dados em uma empresa que precisa preparar dados brutos para análise, integrando múltiplas fontes e garantindo a qualidade dos dados.

## Setup Inicial

```sql
-- Criar tabelas para dados brutos
CREATE TABLE lab_workspace.dados_brutos_clientes (
    id VARCHAR(20),
    nome_completo TEXT,
    email TEXT,
    telefone VARCHAR(50),
    endereco TEXT,
    data_cadastro VARCHAR(30),
    status VARCHAR(20),
    observacoes TEXT
);

CREATE TABLE lab_workspace.dados_brutos_vendas (
    id_venda VARCHAR(20),
    data_hora VARCHAR(30),
    cliente_id VARCHAR(20),
    itens TEXT,
    valor_total VARCHAR(20),
    desconto VARCHAR(20),
    forma_pagamento TEXT,
    status_pagamento TEXT,
    status_entrega TEXT
);

CREATE TABLE lab_workspace.dados_brutos_produtos (
    codigo VARCHAR(20),
    nome TEXT,
    descricao TEXT,
    categoria TEXT,
    subcategoria TEXT,
    fornecedor TEXT,
    preco VARCHAR(20),
    estoque VARCHAR(10),
    unidade_medida VARCHAR(10),
    dimensoes TEXT,
    peso VARCHAR(20),
    data_cadastro VARCHAR(30)
);

-- Carregar dados de exemplo com problemas comuns
INSERT INTO lab_workspace.dados_brutos_clientes
VALUES 
    ('CLI-001', 'Silva, João Carlos', 'joao.silva@email.com', '(11) 98765-4321', 'Rua das Flores, 123 - São Paulo, SP', '15/03/2022', 'ATIVO', NULL),
    ('CLI-002', 'Maria Souza', 'maria.souza@email.com', '11987654322', 'Av. Paulista, 1000, Apto 501, São Paulo - SP', '2022-04-20', 'ativo', 'Cliente VIP'),
    ('CLI-003', 'PEDRO SANTOS', 'pedro@email', '(21)87654323', 'Rua Ipanema 45, Rio de Janeiro RJ', '05/05/2022', 'INATIVO', 'Bloqueado por inadimplência'),
    ('CLI-004', 'Ana Paula  Oliveira', 'ana.oliveira@email.com', '(31) 9876-54324', 'Av. Amazonas, 500 - Belo Horizonte - MG', '10/06/2022', 'ATIVO', NULL),
    ('CLI-005', 'Carlos Eduardo da Silva', 'carlos@email.com.br', '(41)998765325', 'R. XV de Novembro, 200 - Curitiba PR', '2022/07/15', 'PENDENTE', 'Aguardando confirmação de email');

INSERT INTO lab_workspace.dados_brutos_vendas
VALUES 
    ('V-2022-001', '15/04/2022 14:30:25', 'CLI-001', 'PROD-001:2,PROD-003:1', 'R$ 1.299,90', '10%', 'CARTÃO DE CRÉDITO', 'APROVADO', 'ENTREGUE'),
    ('V-2022-002', '2022-05-20 09:45:12', 'CLI-002', 'PROD-002:1', '2499.90', '0', 'PIX', 'APROVADO', 'EM TRANSPORTE'),
    ('V-2022-003', '25/05/2022 18:20:45', 'CLI-001', 'PROD-005:3,PROD-004:1', 'R$1.549,97', 'R$ 100,00', 'BOLETO', 'APROVADO', 'PENDENTE'),
    ('V-2022-004', '30/05/2022 10:15:30', 'CLI-003', 'PROD-001:1', 'R$ 599,90', 'R$ 0,00', 'DÉBITO', 'APROVADO', 'ENTREGUE'),
    ('V-2022-005', '2022-06-10T08:45:12Z', 'CLI-002', 'PROD-003:2', '1400', '5%', 'CRÉDITO', 'APROVADO', 'ENTREGUE');

INSERT INTO lab_workspace.dados_brutos_produtos
VALUES 
    ('PROD-001', 'Smartphone XYZ', 'Smartphone com 128GB de memória', 'Eletrônicos', 'Celulares', 'Tech Inc.', 'R$ 599,90', '15', 'un', '15x7x1 cm', '180g', '10/01/2022'),
    ('PROD-002', 'Notebook Ultra', 'Notebook com processador i7', 'Eletrônicos', 'Computadores', 'Tech Inc.', '2499.90', '8', 'un', NULL, '1,5 kg', '2022-01-15'),
    ('PROD-003', 'Fone de Ouvido Bluetooth', 'Fone sem fio com cancelamento de ruído', 'Eletrônicos', 'Acessórios', 'AudioTech', 'R$699,95', '25', 'un', '18x18x5 cm', '200 g', '20/01/2022'),
    ('PROD-004', 'Mouse Gamer', 'Mouse com 6 botões e LED RGB', 'Eletrônicos', 'Acessórios', 'GamerTech', 'R$ 149,99', '30', 'un', '12x7x4 cm', '100g', '25/01/2022'),
    ('PROD-005', 'Teclado Mecânico', 'Teclado com switches blue', 'Eletrônicos', 'Acessórios', 'GamerTech', 'R$ 299,99', '20', 'un', '44x14x4 cm', '850 g', '2022/01/30');

-- Criar tabela de progresso do laboratório
CREATE TABLE IF NOT EXISTS lab_workspace.lab_progress (
    lab_name VARCHAR(100) PRIMARY KEY,
    start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completion_time TIMESTAMP,
    status VARCHAR(20) DEFAULT 'IN_PROGRESS',
    notes TEXT
);

-- Registrar início do laboratório
INSERT INTO lab_workspace.lab_progress (lab_name)
VALUES ('Transformação de Dados');
```

## Parte 1: Limpeza e Padronização

### 1.1 Padronização de Nomes

```sql
-- Criar tabela normalizada para clientes
CREATE TABLE lab_workspace.clientes (
    id VARCHAR(20) PRIMARY KEY,
    nome_primeiro VARCHAR(50),
    nome_ultimo VARCHAR(50),
    email VARCHAR(100),
    telefone VARCHAR(20),
    data_cadastro DATE,
    status VARCHAR(20),
    observacoes TEXT
);

-- Padronizar e separar nomes
INSERT INTO lab_workspace.clientes (
    id,
    nome_primeiro,
    nome_ultimo,
    email,
    telefone,
    data_cadastro,
    status,
    observacoes
)
SELECT
    id,
    -- Padronizar e extrair primeiro nome
    CASE
        WHEN nome_completo LIKE '%,%' THEN 
            TRIM(SPLIT_PART(nome_completo, ',', 2))
        ELSE
            TRIM(SPLIT_PART(nome_completo, ' ', 1))
    END AS nome_primeiro,
    
    -- Padronizar e extrair último nome
    CASE
        WHEN nome_completo LIKE '%,%' THEN 
            TRIM(SPLIT_PART(nome_completo, ',', 1))
        ELSE
            CASE
                WHEN ARRAY_LENGTH(STRING_TO_ARRAY(nome_completo, ' '), 1) > 1 THEN
                    TRIM(SPLIT_PART(nome_completo, ' ', ARRAY_LENGTH(STRING_TO_ARRAY(nome_completo, ' '), 1)))
                ELSE
                    NULL
            END
    END AS nome_ultimo,
    
    -- Manter email original por enquanto
    email,
    
    -- Padronizar telefone (remover formatação)
    REGEXP_REPLACE(telefone, '[^0-9]', '', 'g') AS telefone,
    
    -- Converter data para formato padrão
    CASE
        WHEN data_cadastro ~ '^[0-9]{2}/[0-9]{2}/[0-9]{4}$' THEN 
            TO_DATE(data_cadastro, 'DD/MM/YYYY')
        WHEN data_cadastro ~ '^[0-9]{4}-[0-9]{2}-[0-9]{2}$' THEN 
            TO_DATE(data_cadastro, 'YYYY-MM-DD')
        WHEN data_cadastro ~ '^[0-9]{4}/[0-9]{2}/[0-9]{2}$' THEN 
            TO_DATE(data_cadastro, 'YYYY/MM/DD')
        ELSE NULL
    END AS data_cadastro,
    
    -- Padronizar status (primeira letra maiúscula, resto minúscula)
    INITCAP(LOWER(status)) AS status,
    
    observacoes
FROM lab_workspace.dados_brutos_clientes;

-- Verificar resultados
SELECT * FROM lab_workspace.clientes;
```

### 1.2 Validação e Correção de Dados

```sql
-- Validar e corrigir emails
UPDATE lab_workspace.clientes
SET email = 
    CASE
        WHEN email NOT LIKE '%@%.%' THEN email || '@dominio.com'
        ELSE email
    END
WHERE email NOT LIKE '%@%.%';

-- Validar e padronizar telefones
UPDATE lab_workspace.clientes
SET telefone = 
    CASE
        WHEN LENGTH(telefone) = 10 THEN 
            SUBSTRING(telefone, 1, 2) || '9' || SUBSTRING(telefone, 3)
        ELSE telefone
    END
WHERE LENGTH(telefone) = 10;

-- Verificar registros com problemas
SELECT id, email, telefone
FROM lab_workspace.clientes
WHERE 
    email !~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$' OR
    telefone !~ '^[0-9]{10,11}$';
```

### 1.3 Normalização de Endereços

```sql
-- Criar tabela normalizada para endereços
CREATE TABLE lab_workspace.enderecos (
    id SERIAL PRIMARY KEY,
    cliente_id VARCHAR(20) REFERENCES lab_workspace.clientes(id),
    logradouro VARCHAR(100),
    numero VARCHAR(20),
    complemento VARCHAR(50),
    bairro VARCHAR(50),
    cidade VARCHAR(50),
    estado CHAR(2),
    cep VARCHAR(8)
);

-- Extrair e normalizar componentes de endereço
INSERT INTO lab_workspace.enderecos (
    cliente_id,
    logradouro,
    numero,
    complemento,
    cidade,
    estado
)
SELECT
    id AS cliente_id,
    -- Extrair logradouro (até o número)
    REGEXP_REPLACE(
        SPLIT_PART(
            SPLIT_PART(endereco, ',', 1), 
            ' - ', 
            1
        ),
        '[0-9]+$', 
        ''
    ) AS logradouro,
    
    -- Extrair número
    REGEXP_REPLACE(
        SPLIT_PART(
            SPLIT_PART(endereco, ',', 1), 
            ' - ', 
            1
        ),
        '^.*?([0-9]+)$', 
        '\1'
    ) AS numero,
    
    -- Extrair complemento (se existir)
    CASE
        WHEN endereco LIKE '%Apto%' THEN 
            REGEXP_REPLACE(
                SPLIT_PART(endereco, ',', 2),
                '.*Apto ([0-9]+).*',
                'Apto \1'
            )
        ELSE NULL
    END AS complemento,
    
    -- Extrair cidade
    TRIM(SPLIT_PART(
        SPLIT_PART(endereco, ' - ', 2),
        '-',
        1
    )) AS cidade,
    
    -- Extrair estado
    TRIM(REGEXP_REPLACE(
        SPLIT_PART(endereco, ' - ', 2),
        '.*([A-Z]{2})$',
        '\1'
    )) AS estado
FROM lab_workspace.dados_brutos_clientes;

-- Verificar resultados
SELECT c.id, c.nome_primeiro, c.nome_ultimo, e.logradouro, e.numero, e.cidade, e.estado
FROM lab_workspace.clientes c
JOIN lab_workspace.enderecos e ON c.id = e.cliente_id;
```

## Parte 2: Conversão de Tipos e Formatos

### 2.1 Normalização de Produtos

```sql
-- Criar tabela normalizada para produtos
CREATE TABLE lab_workspace.produtos (
    id VARCHAR(20) PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    descricao TEXT,
    categoria VARCHAR(50),
    subcategoria VARCHAR(50),
    fornecedor VARCHAR(50),
    preco DECIMAL(10,2) NOT NULL,
    estoque INTEGER NOT NULL,
    unidade_medida VARCHAR(10),
    peso DECIMAL(10,2),
    peso_unidade VARCHAR(5),
    data_cadastro DATE
);

-- Converter e normalizar dados de produtos
INSERT INTO lab_workspace.produtos (
    id,
    nome,
    descricao,
    categoria,
    subcategoria,
    fornecedor,
    preco,
    estoque,
    unidade_medida,
    peso,
    peso_unidade,
    data_cadastro
)
SELECT
    codigo AS id,
    nome,
    descricao,
    categoria,
    subcategoria,
    fornecedor,
    
    -- Converter preço para decimal
    CASE
        WHEN preco LIKE 'R$%' THEN 
            REPLACE(REPLACE(REPLACE(preco, 'R$', ''), '.', ''), ',', '.')::DECIMAL(10,2)
        ELSE
            REPLACE(preco, ',', '.')::DECIMAL(10,2)
    END AS preco,
    
    -- Converter estoque para inteiro
    estoque::INTEGER,
    
    unidade_medida,
    
    -- Extrair valor numérico do peso
    CASE
        WHEN peso LIKE '%kg%' THEN 
            REPLACE(REPLACE(peso, 'kg', ''), ',', '.')::DECIMAL(10,2) * 1000
        WHEN peso LIKE '%g%' THEN 
            REPLACE(REPLACE(peso, 'g', ''), ',', '.')::DECIMAL(10,2)
        ELSE NULL
    END AS peso,
    
    -- Extrair unidade do peso
    'g' AS peso_unidade,
    
    -- Converter data para formato padrão
    CASE
        WHEN data_cadastro ~ '^[0-9]{2}/[0-9]{2}/[0-9]{4}$' THEN 
            TO_DATE(data_cadastro, 'DD/MM/YYYY')
        WHEN data_cadastro ~ '^[0-9]{4}-[0-9]{2}-[0-9]{2}$' THEN 
            TO_DATE(data_cadastro, 'YYYY-MM-DD')
        WHEN data_cadastro ~ '^[0-9]{4}/[0-9]{2}/[0-9]{2}$' THEN 
            TO_DATE(data_cadastro, 'YYYY/MM/DD')
        ELSE NULL
    END AS data_cadastro
FROM lab_workspace.dados_brutos_produtos;

-- Verificar resultados
SELECT id, nome, preco, estoque, peso, data_cadastro
FROM lab_workspace.produtos;
```

### 2.2 Normalização de Vendas

```sql
-- Criar tabela normalizada para vendas
CREATE TABLE lab_workspace.vendas (
    id VARCHAR(20) PRIMARY KEY,
    data_hora TIMESTAMP NOT NULL,
    cliente_id VARCHAR(20) REFERENCES lab_workspace.clientes(id),
    valor_total DECIMAL(10,2) NOT NULL,
    desconto DECIMAL(10,2),
    forma_pagamento VARCHAR(50),
    status_pagamento VARCHAR(20),
    status_entrega VARCHAR(20)
);

-- Converter e normalizar dados de vendas
INSERT INTO lab_workspace.vendas (
    id,
    data_hora,
    cliente_id,
    valor_total,
    desconto,
    forma_pagamento,
    status_pagamento,
    status_entrega
)
SELECT
    id_venda AS id,
    
    -- Converter data_hora para timestamp
    CASE
        WHEN data_hora ~ '^[0-9]{2}/[0-9]{2}/[0-9]{4} [0-9]{2}:[0-9]{2}:[0-9]{2}$' THEN 
            TO_TIMESTAMP(data_hora, 'DD/MM/YYYY HH24:MI:SS')
        WHEN data_hora ~ '^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}$' THEN 
            TO_TIMESTAMP(data_hora, 'YYYY-MM-DD HH24:MI:SS')
        ELSE
            TO_TIMESTAMP(data_hora, 'YYYY-MM-DDTHH24:MI:SSZ')
    END AS data_hora,
    
    cliente_id,
    
    -- Converter valor_total para decimal
    CASE
        WHEN valor_total LIKE 'R$%' THEN 
            REPLACE(REPLACE(REPLACE(valor_total, 'R$', ''), '.', ''), ',', '.')::DECIMAL(10,2)
        ELSE
            REPLACE(valor_total, ',', '.')::DECIMAL(10,2)
    END AS valor_total,
    
    -- Converter desconto para decimal
    CASE
        WHEN desconto LIKE '%%' THEN 
            REPLACE(desconto, '%', '')::DECIMAL(10,2) / 100
        ELSE
            REPLACE(desconto, ',', '.')::DECIMAL(10,2)
    END AS desconto,
    
    forma_pagamento,
    status_pagamento,
    status_entrega
FROM lab_workspace.dados_brutos_vendas;

-- Verificar resultados
SELECT id, data_hora, cliente_id, valor_total, desconto, forma_pagamento, status_pagamento, status_entrega
FROM lab_workspace.vendas;
```

## Parte 3: Enriquecimento de Dados

### 3.1 Adicionar Informações de Geolocalização

```sql
-- Criar tabela para armazenar informações de geolocalização
CREATE TABLE lab_workspace.geolocalizacao (
    cidade VARCHAR(50),
    estado CHAR(2),
    pais VARCHAR(50),
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6)
);

-- Inserir dados de geolocalização (exemplo)
INSERT INTO lab_workspace.geolocalizacao (cidade, estado, pais, latitude, longitude)
VALUES 
    ('São Paulo', 'SP', 'Brasil', -23.550520, -46.633309),
    ('Rio de Janeiro', 'RJ', 'Brasil', -22.906847, -43.172897),
    ('Belo Horizonte', 'MG', 'Brasil', -19.815733, -43.954234),
    ('Curitiba', 'PR', 'Brasil', -25.428408, -49.273388);

-- Adicionar colunas para geolocalização na tabela de endereços
ALTER TABLE lab_workspace.enderecos
ADD COLUMN latitude DECIMAL(9,6),
ADD COLUMN longitude DECIMAL(9,6);

-- Atualizar endereços com informações de geolocalização
UPDATE lab_workspace.enderecos e
SET latitude = g.latitude,
    longitude = g.longitude
FROM lab_workspace.geolocalizacao g
WHERE e.cidade = g.cidade AND e.estado = g.estado;

-- Verificar resultados
SELECT c.id, c.nome_primeiro, c.nome_ultimo, e.logradouro, e.numero, e.cidade, e.estado, e.latitude, e.longitude
FROM lab_workspace.clientes c
JOIN lab_workspace.enderecos e ON c.id = e.cliente_id;
```

### 3.2 Adicionar Informações de Produtos

```sql
-- Criar tabela para armazenar informações adicionais de produtos
CREATE TABLE lab_workspace.produtos_info (
    produto_id VARCHAR(20) PRIMARY KEY,
    marca VARCHAR(50),
    modelo VARCHAR(50),
    garantia VARCHAR(50),
    fabricante VARCHAR(50)
);

-- Inserir dados de informações de produtos (exemplo)
INSERT INTO lab_workspace.produtos_info (produto_id, marca, modelo, garantia, fabricante)
VALUES 
    ('PROD-001', 'Tech Inc.', 'XYZ', '1 ano', 'Tech Inc.'),
    ('PROD-002', 'Tech Inc.', 'Ultra', '2 anos', 'Tech Inc.'),
    ('PROD-003', 'AudioTech', 'Bluetooth', '1 ano', 'AudioTech'),
    ('PROD-004', 'GamerTech', 'Gamer', '1 ano', 'GamerTech'),
    ('PROD-005', 'GamerTech', 'Mechanical', '2 anos', 'GamerTech');

-- Adicionar colunas para informações de produtos na tabela de produtos
ALTER TABLE lab_workspace.produtos
ADD COLUMN marca VARCHAR(50),
ADD COLUMN modelo VARCHAR(50),
ADD COLUMN garantia VARCHAR(50),
ADD COLUMN fabricante VARCHAR(50);

-- Atualizar produtos com informações adicionais
UPDATE lab_workspace.produtos p
SET marca = pi.marca,
    modelo = pi.modelo,
    garantia = pi.garantia,
    fabricante = pi.fabricante
FROM lab_workspace.produtos_info pi
WHERE p.id = pi.produto_id;

-- Verificar resultados
SELECT p.id, p.nome, p.preco, p.estoque, p.peso, p.data_cadastro, p.marca, p.modelo, p.garantia, p.fabricante
FROM lab_workspace.produtos p;
```

## Parte 4: Análise e Visualização

### 4.1 Análise de Vendas

```sql
-- Calcular o total de vendas por cliente
SELECT c.id, c.nome_primeiro, c.nome_ultimo, SUM(v.valor_total) AS total_vendas
FROM lab_workspace.clientes c
JOIN lab_workspace.vendas v ON c.id = v.cliente_id
GROUP BY c.id, c.nome_primeiro, c.nome_ultimo
ORDER BY total_vendas DESC;

-- Calcular o total de vendas por produto
SELECT p.id, p.nome, SUM(v.valor_total) AS total_vendas
FROM lab_workspace.produtos p
JOIN lab_workspace.vendas v ON p.id = v.itens
GROUP BY p.id, p.nome
ORDER BY total_vendas DESC;

-- Calcular o total de vendas por mês
SELECT TO_CHAR(v.data_hora, 'YYYY-MM') AS mes, SUM(v.valor_total) AS total_vendas
FROM lab_workspace.vendas v
GROUP BY TO_CHAR(v.data_hora, 'YYYY-MM')
ORDER BY mes;
```

### 4.2 Visualização de Dados

Para visualizar os dados, você pode usar ferramentas como Tableau, Power BI ou Google Data Studio. Aqui estão algumas ideias de visualizações:

1. Gráfico de barras para o total de vendas por cliente.
2. Gráfico de pizza para a distribuição de vendas por produto.
3. Gráfico de linha para o total de vendas ao longo do tempo.
4. Mapa para visualizar a distribuição geográfica dos clientes.

Lembre-se de que a visualização de dados é uma parte importante da análise e pode ajudar a identificar tendências e padrões nos dados.

## Conclusão

Neste laboratório, você aprendeu a transformar e manipular dados brutos utilizando SQL, incluindo limpeza, normalização, conversão de formatos e enriquecimento de dados. Você também explorou técnicas de análise e visualização de dados para obter insights valiosos.

## Próximos Passos

Para aprofundar seus conhecimentos, você pode:

1. Explorar mais técnicas de manipulação de dados em SQL.
2. Aprender sobre ferramentas de análise e visualização de dados.
3. Trabalhar com conjuntos de dados maiores e mais complexos.
4. Experimentar diferentes abordagens de limpeza e normalização de dados.

Boa sorte! Se tiver alguma dúvida, sinta-se à vontade para perguntar.
