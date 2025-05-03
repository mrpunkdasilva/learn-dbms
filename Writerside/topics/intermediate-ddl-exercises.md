# Exercícios Intermediários de DDL

## Exercício 1: Herança de Tabelas

### Descrição
Implemente uma estrutura de herança para diferentes tipos de produtos.

### Requisitos
- Tabela base de produtos
- Tabelas específicas para produtos físicos e digitais
- Constraints específicas para cada tipo

### Solução
```sql
CREATE TABLE produtos_base (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) NOT NULL,
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE produtos_fisicos (
    peso DECIMAL(10,2) NOT NULL,
    dimensoes VARCHAR(50),
    localizacao_estoque VARCHAR(50)
) INHERITS (produtos_base);

CREATE TABLE produtos_digitais (
    tamanho_arquivo VARCHAR(20),
    formato VARCHAR(20),
    url_download VARCHAR(255)
) INHERITS (produtos_base);
```

## Exercício 2: Particionamento de Tabelas

### Descrição
Crie uma tabela particionada para histórico de vendas.

### Requisitos
- Particionamento por intervalo de datas
- Partições mensais
- Índices apropriados para cada partição

### Solução
```sql
CREATE TABLE historico_vendas (
    id SERIAL,
    data_venda DATE NOT NULL,
    produto_id INTEGER,
    quantidade INTEGER,
    valor_total DECIMAL(10,2)
) PARTITION BY RANGE (data_venda);

CREATE TABLE vendas_202301 PARTITION OF historico_vendas
    FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');

CREATE TABLE vendas_202302 PARTITION OF historico_vendas
    FOR VALUES FROM ('2023-02-01') TO ('2023-03-01');

CREATE INDEX idx_vendas_202301_data ON vendas_202301(data_venda);
CREATE INDEX idx_vendas_202302_data ON vendas_202302(data_venda);
```

## Exercício 3: Domínios Personalizados

### Descrição
Defina domínios customizados para tipos de dados comuns.

### Requisitos
- Domínio para e-mail
- Domínio para telefone
- Domínio para CPF/CNPJ

### Solução
```sql
CREATE DOMAIN email_type AS VARCHAR(255)
    CHECK (VALUE ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$');

CREATE DOMAIN telefone_type AS VARCHAR(20)
    CHECK (VALUE ~ '^\+?[0-9]{10,14}$');

CREATE DOMAIN documento_type AS VARCHAR(14)
    CHECK (VALUE ~ '^\d{11}$' OR VALUE ~ '^\d{14}$');

CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    email email_type NOT NULL,
    telefone telefone_type,
    documento documento_type NOT NULL UNIQUE
);
```

## Exercício 4: Triggers e Funções

### Descrição
Implemente triggers para auditoria de alterações.

### Requisitos
- Tabela de auditoria
- Trigger para INSERT/UPDATE/DELETE
- Registro de usuário e timestamp

### Solução
```sql
CREATE TABLE auditoria (
    id SERIAL PRIMARY KEY,
    tabela_nome VARCHAR(50),
    operacao VARCHAR(20),
    registro_id INTEGER,
    dados_antigos JSONB,
    dados_novos JSONB,
    usuario_modificacao VARCHAR(50),
    data_modificacao TIMESTAMP
);

CREATE OR REPLACE FUNCTION fn_auditoria()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO auditoria (
        tabela_nome,
        operacao,
        registro_id,
        dados_antigos,
        dados_novos,
        usuario_modificacao,
        data_modificacao
    ) VALUES (
        TG_TABLE_NAME,
        TG_OP,
        COALESCE(NEW.id, OLD.id),
        CASE WHEN TG_OP IN ('UPDATE','DELETE') THEN to_jsonb(OLD) ELSE NULL END,
        CASE WHEN TG_OP IN ('UPDATE','INSERT') THEN to_jsonb(NEW) ELSE NULL END,
        current_user,
        current_timestamp
    );
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_produtos_auditoria
AFTER INSERT OR UPDATE OR DELETE ON produtos
FOR EACH ROW EXECUTE FUNCTION fn_auditoria();
```

## Exercício 5: Views Materializadas

### Descrição
Crie views materializadas para relatórios de vendas.

### Requisitos
- Agregações complexas
- Atualização programada
- Índices para performance

### Solução
```sql
CREATE MATERIALIZED VIEW mv_resumo_vendas AS
SELECT 
    p.categoria_id,
    c.nome as categoria,
    DATE_TRUNC('month', v.data_venda) as mes,
    COUNT(*) as total_vendas,
    SUM(v.quantidade) as quantidade_total,
    SUM(v.valor_total) as valor_total
FROM historico_vendas v
JOIN produtos p ON v.produto_id = p.id
JOIN categorias c ON p.categoria_id = c.id
GROUP BY p.categoria_id, c.nome, DATE_TRUNC('month', v.data_venda)
WITH DATA;

CREATE UNIQUE INDEX idx_mv_resumo_vendas 
ON mv_resumo_vendas (categoria_id, mes);

CREATE OR REPLACE FUNCTION refresh_mv_resumo_vendas()
RETURNS void AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY mv_resumo_vendas;
END;
$$ LANGUAGE plpgsql;
```

## Exercício 6: Schemas e Segurança

### Descrição
Configure schemas diferentes para separação de responsabilidades.

### Requisitos
- Schemas para diferentes módulos
- Controle de acesso granular
- Roles específicas

### Solução
```sql
-- Criação de schemas
CREATE SCHEMA vendas;
CREATE SCHEMA estoque;
CREATE SCHEMA relatorios;

-- Criação de roles
CREATE ROLE vendedor;
CREATE ROLE estoquista;
CREATE ROLE analista;

-- Permissões
GRANT USAGE ON SCHEMA vendas TO vendedor;
GRANT SELECT, INSERT ON ALL TABLES IN SCHEMA vendas TO vendedor;

GRANT USAGE ON SCHEMA estoque TO estoquista;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA estoque TO estoquista;

GRANT USAGE ON SCHEMA relatorios TO analista;
GRANT SELECT ON ALL TABLES IN SCHEMA relatorios TO analista;
```

## Exercício 7: Constraints Complexas

### Descrição
Implemente constraints avançadas com funções personalizadas.

### Requisitos
- Validação complexa de dados
- Constraints exclusivas condicionais
- Constraints entre tabelas

### Solução
```sql
-- Função para validação de estoque
CREATE OR REPLACE FUNCTION fn_validar_estoque()
RETURNS trigger AS $$
BEGIN
    IF NEW.quantidade < 0 THEN
        RAISE EXCEPTION 'Quantidade não pode ser negativa';
    END IF;
    
    IF NEW.quantidade < NEW.quantidade_minima THEN
        RAISE NOTICE 'Estoque abaixo do mínimo';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Constraint exclusiva condicional
ALTER TABLE produtos
ADD CONSTRAINT uk_produto_codigo
EXCLUDE USING gist (
    nome WITH =,
    codigo_barras WITH =
) WHERE (status = 'ativo');

-- Constraint entre tabelas
ALTER TABLE pedidos
ADD CONSTRAINT check_limite_credito
CHECK (
    valor_total <= (
        SELECT limite_credito 
        FROM clientes 
        WHERE id = cliente_id
    )
);
```

## Exercício 8: Extensões e Tipos Customizados

### Descrição
Utilize extensões PostgreSQL e crie tipos personalizados.

### Requisitos
- Extensão para geolocalização
- Tipo customizado para endereço
- Tipo customizado para dinheiro

### Solução
```sql
-- Instalação de extensão
CREATE EXTENSION IF NOT EXISTS postgis;

-- Tipo customizado para endereço
CREATE TYPE endereco_type AS (
    logradouro VARCHAR(100),
    numero VARCHAR(10),
    complemento VARCHAR(50),
    bairro VARCHAR(50),
    cidade VARCHAR(50),
    estado CHAR(2),
    cep VARCHAR(8)
);

-- Tipo customizado para moeda
CREATE TYPE moeda_type AS (
    valor DECIMAL(10,2),
    moeda CHAR(3)
);

-- Uso dos tipos
CREATE TABLE lojas (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100),
    endereco endereco_type,
    localizacao geography(Point),
    faturamento_mensal moeda_type
);
```

## Exercício 9: Otimização de Schema

### Descrição
Otimize a estrutura do banco de dados para performance.

### Requisitos
- Índices apropriados
- Estatísticas personalizadas
- Configuração de storage

### Solução
```sql
-- Índices para joins frequentes
CREATE INDEX idx_produtos_categoria_preco 
ON produtos (categoria_id, preco);

-- Índice parcial
CREATE INDEX idx_produtos_ativos 
ON produtos (nome) 
WHERE status = 'ativo';

-- Índice de texto
CREATE INDEX idx_produtos_busca 
ON produtos USING gin(to_tsvector('portuguese', nome || ' ' || descricao));

-- Estatísticas personalizadas
CREATE STATISTICS produtos_stats (dependencies) 
ON categoria_id, status, preco 
FROM produtos;

-- Configuração de storage
ALTER TABLE produtos SET (
    autovacuum_vacuum_scale_factor = 0.1,
    autovacuum_analyze_scale_factor = 0.05
);
```

## Exercício 10: Migração de Dados

### Descrição
Crie scripts para migração segura de dados.

### Requisitos
- Verificação de integridade
- Tratamento de erros
- Rollback em caso de falha

### Solução
```sql
DO $$
DECLARE
    v_count INTEGER;
BEGIN
    -- Início da transação
    BEGIN
        -- Criação de tabela temporária
        CREATE TEMP TABLE tmp_produtos AS 
        SELECT * FROM produtos_legado;
        
        GET DIAGNOSTICS v_count = ROW_COUNT;
        RAISE NOTICE 'Dados copiados: %', v_count;
        
        -- Validação dos dados
        PERFORM count(*) 
        FROM tmp_produtos 
        WHERE preco < 0;
        
        IF FOUND THEN
            RAISE EXCEPTION 'Dados inválidos encontrados';
        END IF;
        
        -- Inserção dos dados
        INSERT INTO produtos_novo
        SELECT * FROM tmp_produtos;
        
        -- Commit se tudo ok
        RAISE NOTICE 'Migração concluída com sucesso';
    EXCEPTION WHEN OTHERS THEN
        -- Rollback em caso de erro
        RAISE NOTICE 'Erro na migração: %', SQLERRM;
        RAISE;
    END;
END;
$$;
```

## Critérios de Avaliação

- Implementação correta das funcionalidades
- Uso adequado de recursos avançados
- Performance e escalabilidade
- Tratamento de erros
- Documentação do código

## Dicas de Estudo

- Consulte a documentação oficial do PostgreSQL
- Teste diferentes cenários
- Monitore a performance
- Mantenha scripts de rollback
- Documente decisões de design