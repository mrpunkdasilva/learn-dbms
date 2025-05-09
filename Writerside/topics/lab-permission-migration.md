# Laboratório: Migração de Permissões

## Objetivo
Planejar e executar a migração de permissões entre ambientes de banco de dados, garantindo a consistência e segurança durante o processo de transição.

## Cenário
Você é um DBA responsável por migrar um sistema de banco de dados de um ambiente de desenvolvimento para produção. Além da migração dos dados, você precisa garantir que todas as permissões sejam corretamente transferidas, mantendo a segurança e os controles de acesso.

## Setup Inicial

```sql
-- Conectar ao banco de dados como superusuário
-- psql -U postgres -d lab_dcl

-- Criar schemas para simular ambientes
CREATE SCHEMA IF NOT EXISTS dev;
CREATE SCHEMA IF NOT EXISTS homolog;
CREATE SCHEMA IF NOT EXISTS prod;

-- Registrar início do laboratório
INSERT INTO lab_workspace.lab_progress (lab_name)
VALUES ('Migração de Permissões');
```

## Parte 1: Preparação dos Ambientes

### 1.1 Criação de Estruturas nos Ambientes

```sql
-- Criar tabelas no ambiente de desenvolvimento
CREATE TABLE dev.clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    segmento VARCHAR(50),
    limite_credito DECIMAL(10,2),
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE dev.produtos (
    id SERIAL PRIMARY KEY,
    codigo VARCHAR(20) UNIQUE,
    nome VARCHAR(100) NOT NULL,
    descricao TEXT,
    preco DECIMAL(10,2) NOT NULL,
    estoque INTEGER DEFAULT 0,
    categoria VARCHAR(50)
);

CREATE TABLE dev.pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER REFERENCES dev.clientes(id),
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    valor_total DECIMAL(10,2),
    status VARCHAR(20) DEFAULT 'Pendente',
    observacoes TEXT
);

-- Criar tabelas no ambiente de homologação (estrutura idêntica)
CREATE TABLE homolog.clientes (LIKE dev.clientes INCLUDING ALL);
CREATE TABLE homolog.produtos (LIKE dev.produtos INCLUDING ALL);
CREATE TABLE homolog.pedidos (LIKE dev.pedidos INCLUDING ALL);
ALTER TABLE homolog.pedidos 
    DROP CONSTRAINT IF EXISTS pedidos_cliente_id_fkey,
    ADD CONSTRAINT pedidos_cliente_id_fkey 
    FOREIGN KEY (cliente_id) REFERENCES homolog.clientes(id);

-- Criar tabelas no ambiente de produção (estrutura idêntica)
CREATE TABLE prod.clientes (LIKE dev.clientes INCLUDING ALL);
CREATE TABLE prod.produtos (LIKE dev.produtos INCLUDING ALL);
CREATE TABLE prod.pedidos (LIKE dev.pedidos INCLUDING ALL);
ALTER TABLE prod.pedidos 
    DROP CONSTRAINT IF EXISTS pedidos_cliente_id_fkey,
    ADD CONSTRAINT pedidos_cliente_id_fkey 
    FOREIGN KEY (cliente_id) REFERENCES prod.clientes(id);
```

### 1.2 Configuração de Usuários e Roles

```sql
-- Criar roles para cada ambiente
CREATE ROLE dev_role;
CREATE ROLE homolog_role;
CREATE ROLE prod_role;

-- Criar roles funcionais
CREATE ROLE analista_role;
CREATE ROLE desenvolvedor_role;
CREATE ROLE dba_role;
CREATE ROLE suporte_role;

-- Criar usuários para cada ambiente
CREATE USER dev_user1 WITH PASSWORD 'dev123' IN ROLE desenvolvedor_role, dev_role;
CREATE USER dev_user2 WITH PASSWORD 'dev123' IN ROLE analista_role, dev_role;
CREATE USER homolog_user1 WITH PASSWORD 'hom123' IN ROLE analista_role, homolog_role;
CREATE USER prod_user1 WITH PASSWORD 'prod123' IN ROLE suporte_role, prod_role;
CREATE USER dba_user WITH PASSWORD 'dba123' IN ROLE dba_role;

-- Conceder permissões no ambiente de desenvolvimento
GRANT USAGE ON SCHEMA dev TO dev_role;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA dev TO desenvolvedor_role;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA dev TO analista_role;

-- Conceder permissões no ambiente de homologação
GRANT USAGE ON SCHEMA homolog TO homolog_role;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA homolog TO analista_role;

-- Conceder permissões no ambiente de produção
GRANT USAGE ON SCHEMA prod TO prod_role;
GRANT SELECT ON ALL TABLES IN SCHEMA prod TO suporte_role;

-- Conceder permissões administrativas
GRANT ALL PRIVILEGES ON SCHEMA dev, homolog, prod TO dba_role;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA dev, homolog, prod TO dba_role;
```

## Parte 2: Extração e Documentação de Permissões

### 2.1 Extração de Permissões Existentes

```sql
-- Criar tabela para documentar permissões
CREATE TABLE lab_workspace.permission_inventory (
    id SERIAL PRIMARY KEY,
    environment VARCHAR(20) NOT NULL,
    object_type VARCHAR(50) NOT NULL,
    object_name TEXT NOT NULL,
    grantee TEXT NOT NULL,
    privilege_type TEXT NOT NULL,
    is_grantable BOOLEAN,
    extracted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Função para extrair e documentar permissões
CREATE OR REPLACE FUNCTION lab_workspace.extract_permissions(env text)
RETURNS void AS $$
BEGIN
    -- Limpar permissões anteriores do mesmo ambiente
    DELETE FROM lab_workspace.permission_inventory
    WHERE environment = env;
    
    -- Inserir permissões de tabelas
    INSERT INTO lab_workspace.permission_inventory (
        environment, object_type, object_name, grantee, privilege_type, is_grantable
    )
    SELECT 
        env AS environment,
        'TABLE' AS object_type,
        table_schema || '.' || table_name AS object_name,
        grantee,
        privilege_type,
        is_grantable::boolean
    FROM information_schema.table_privileges
    WHERE table_schema = env;
    
    -- Inserir permissões de schemas
    INSERT INTO lab_workspace.permission_inventory (
        environment, object_type, object_name, grantee, privilege_type, is_grantable
    )
    SELECT 
        env AS environment,
        'SCHEMA' AS object_type,
        schema_name AS object_name,
        grantee,
        privilege_type,
        is_grantable::boolean
    FROM information_schema.usage_privileges
    WHERE object_name = env;
    
    -- Inserir permissões de colunas
    INSERT INTO lab_workspace.permission_inventory (
        environment, object_type, object_name, grantee, privilege_type, is_grantable
    )
    SELECT 
        env AS environment,
        'COLUMN' AS object_type,
        table_schema || '.' || table_name || '.' || column_name AS object_name,
        grantee,
        privilege_type,
        is_grantable::boolean
    FROM information_schema.column_privileges
    WHERE table_schema = env;
END;
$$ LANGUAGE plpgsql;

-- Extrair permissões de cada ambiente
SELECT lab_workspace.extract_permissions('dev');
SELECT lab_workspace.extract_permissions('homolog');
SELECT lab_workspace.extract_permissions('prod');
```

### 2.2 Análise de Permissões

```sql
-- Criar visão para análise de permissões
CREATE OR REPLACE VIEW lab_workspace.permission_analysis AS
WITH role_members AS (
    SELECT r.rolname AS role, u.usename AS username
    FROM pg_roles r
    JOIN pg_auth_members m ON r.oid = m.roleid
    JOIN pg_roles u ON m.member = u.oid
    WHERE u.rolcanlogin
    
    UNION
    
    SELECT r.rolname AS role, r.rolname AS username
    FROM pg_roles r
    WHERE r.rolcanlogin
)
SELECT 
    pi.environment,
    pi.object_type,
    pi.object_name,
    pi.grantee,
    pi.privilege_type,
    array_agg(DISTINCT rm.username) AS affected_users
FROM lab_workspace.permission_inventory pi
LEFT JOIN role_members rm ON pi.grantee = rm.role
GROUP BY pi.environment, pi.object_type, pi.object_name, pi.grantee, pi.privilege_type;

-- Consultar análise
SELECT * FROM lab_workspace.permission_analysis
ORDER BY environment, object_type, object_name;
```

## Parte 3: Planejamento da Migração

### 3.1 Criação do Plano de Migração

```sql
-- Criar tabela para plano de migração
CREATE TABLE lab_workspace.migration_plan (
    id SERIAL PRIMARY KEY,
    source_environment VARCHAR(20) NOT NULL,
    target_environment VARCHAR(20) NOT NULL,
    object_type VARCHAR(50) NOT NULL,
    object_name TEXT NOT NULL,
    grantee TEXT NOT NULL,
    privilege_type TEXT NOT NULL,
    migration_status VARCHAR(20) DEFAULT 'Pending',
    migration_date TIMESTAMP,
    migration_notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Função para gerar plano de migração
CREATE OR REPLACE FUNCTION lab_workspace.generate_migration_plan(
    source_env text,
    target_env text
)
RETURNS void AS $$
BEGIN
    -- Limpar plano anterior para os mesmos ambientes
    DELETE FROM lab_workspace.migration_plan
    WHERE source_environment = source_env AND target_environment = target_env;
    
    -- Inserir permissões a serem migradas
    INSERT INTO lab_workspace.migration_plan (
        source_environment,
        target_environment,
        object_type,
        object_name,
        grantee,
        privilege_type,
        migration_notes
    )
    SELECT 
        source_env,
        target_env,
        pi.object_type,
        -- Substituir o schema no nome do objeto
        REPLACE(pi.object_name, source_env || '.', target_env || '.') AS object_name,
        pi.grantee,
        pi.privilege_type,
        'Migrado automaticamente do plano'
    FROM lab_workspace.permission_inventory pi
    WHERE pi.environment = source_env;
END;
$$ LANGUAGE plpgsql;

-- Gerar plano de migração de dev para homolog
SELECT lab_workspace.generate_migration_plan('dev', 'homolog');

-- Gerar plano de migração de homolog para prod
SELECT lab_workspace.generate_migration_plan('homolog', 'prod');
```

### 3.2 Revisão e Ajuste do Plano

```sql
-- Ajustar plano para remover permissões sensíveis em produção
UPDATE lab_workspace.migration_plan
SET migration_status = 'Rejected', 
    migration_notes = 'Permissão de DELETE não permitida em produção'
WHERE target_environment = 'prod' 
AND privilege_type = 'DELETE';

-- Ajustar plano para adicionar permissões específicas
INSERT INTO lab_workspace.migration_plan (
    source_environment,
    target_environment,
    object_type,
    object_name,
    grantee,
    privilege_type,
    migration_status,
    migration_notes
)
VALUES 
('homolog', 'prod', 'TABLE', 'prod.clientes', 'suporte_role', 'SELECT', 'Pending', 'Permissão adicional para suporte'),
('homolog', 'prod', 'TABLE', 'prod.pedidos', 'suporte_role', 'SELECT', 'Pending', 'Permissão adicional para suporte');

-- Verificar plano final
SELECT * FROM lab_workspace.migration_plan
WHERE migration_status = 'Pending'
ORDER BY target_environment, object_type, object_name;
```

## Parte 4: Execução da Migração

### 4.1 Geração de Scripts de Migração

```sql
-- Função para gerar script SQL de migração
CREATE OR REPLACE FUNCTION lab_workspace.generate_migration_script(
    target_env text
)
RETURNS TABLE (
    script_order int,
    sql_command text
) AS $$
BEGIN
    RETURN QUERY
    
    -- Scripts para permissões de schema
    SELECT 
        1 AS script_order,
        'GRANT ' || privilege_type || ' ON SCHEMA ' || 
        SPLIT_PART(object_name, '.', 1) || ' TO ' || grantee || ';' AS sql_command
    FROM lab_workspace.migration_plan
    WHERE target_environment = target_env
    AND migration_status = 'Pending'
    AND object_type = 'SCHEMA'
    
    UNION ALL
    
    -- Scripts para permissões de tabela
    SELECT 
        2 AS script_order,
        'GRANT ' || privilege_type || ' ON TABLE ' || 
        object_name || ' TO ' || grantee || ';' AS sql_command
    FROM lab_workspace.migration_plan
    WHERE target_environment = target_env
    AND migration_status = 'Pending'
    AND object_type = 'TABLE'
    
    UNION ALL
    
    -- Scripts para permissões de coluna
    SELECT 
        3 AS script_order,
        'GRANT ' || privilege_type || ' (' || 
        SPLIT_PART(object_name, '.', 3) || ') ON TABLE ' || 
        SPLIT_PART(object_name, '.', 1) || '.' || 
        SPLIT_PART(object_name, '.', 2) || ' TO ' || grantee || ';' AS sql_command
    FROM lab_workspace.migration_plan
    WHERE target_environment = target_env
    AND migration_status = 'Pending'
    AND object_type = 'COLUMN'
    
    ORDER BY script_order, sql_command;
END;
$$ LANGUAGE plpgsql;

-- Gerar script para homologação
SELECT * FROM lab_workspace.generate_migration_script('homolog');

-- Gerar script para produção
SELECT * FROM lab_workspace.generate_migration_script('prod');
```

### 4.2 Execução e Validação

```sql
-- Função para executar migração
CREATE OR REPLACE FUNCTION lab_workspace.execute_migration(
    target_env text
)
RETURNS TABLE (
    command_id int,
    sql_command text,
    execution_status text
) AS $$
DECLARE
    cmd record;
    cmd_count int := 0;
    cmd_status text;
BEGIN
    -- Atualizar status para "Em Progresso"
    UPDATE lab_workspace.migration_plan
    SET migration_status = 'In Progress'
    WHERE target_environment = target_env
    AND migration_status = 'Pending';
    
    -- Executar cada comando
    FOR cmd IN SELECT * FROM lab_workspace.generate_migration_script(target_env) LOOP
        cmd_count := cmd_count + 1;
        BEGIN
            EXECUTE cmd.sql_command;
            cmd_status := 'Success';
            
            command_id := cmd_count;
            sql_command := cmd.sql_command;
            execution_status := cmd_status;
            RETURN NEXT;
        EXCEPTION WHEN OTHERS THEN
            cmd_status := 'Error: ' || SQLERRM;
            
            command_id := cmd_count;
            sql_command := cmd.sql_command;
            execution_status := cmd_status;
            RETURN NEXT;
        END;
    END LOOP;
    
    -- Atualizar status para "Concluído"
    UPDATE lab_workspace.migration_plan
    SET 
        migration_status = 'Completed',
        migration_date = CURRENT_TIMESTAMP
    WHERE target_environment = target_env
    AND migration_status = 'In Progress';
    
    RETURN;
END;
$$ LANGUAGE plpgsql;

-- Executar migração para homologação
SELECT * FROM lab_workspace.execute_migration('homolog');

-- Executar migração para produção
SELECT * FROM lab_workspace.execute_migration('prod');
```

## Parte 5: Verificação e Documentação

### 5.1 Verificação de Permissões Migradas

```sql
-- Extrair permissões atualizadas
SELECT lab_workspace.extract_permissions('homolog');
SELECT lab_workspace.extract_permissions('prod');

-- Comparar permissões planejadas vs. implementadas
CREATE OR REPLACE VIEW lab_workspace.migration_verification AS
SELECT 
    mp.target_environment,
    mp.object_type,
    mp.object_name,
    mp.grantee,
    mp.privilege_type,
    mp.migration_status,
    CASE 
        WHEN pi.id IS NOT NULL THEN 'Verificado'
        ELSE 'Não Encontrado'
    END AS verification_status
FROM lab_workspace.migration_plan mp
LEFT JOIN lab_workspace.permission_inventory pi ON
    mp.target_environment = pi.environment AND
    mp.object_type = pi.object_type AND
    mp.object_name = pi.object_name AND
    mp.grantee = pi.grantee AND
    mp.privilege_type = pi.privilege_type
WHERE mp.migration_status = 'Completed';

-- Consultar verificação
SELECT * FROM lab_workspace.migration_verification
ORDER BY target_environment, verification_status DESC, object_type, object_name;
```

### 5.2 Documentação Final

```sql
-- Criar relatório final de migração
CREATE OR REPLACE VIEW lab_workspace.migration_report AS
SELECT 
    target_environment,
    COUNT(*) AS total_permissions,
    COUNT(*) FILTER (WHERE migration_status = 'Completed') AS migrated_permissions,
    COUNT(*) FILTER (WHERE migration_status = 'Rejected') AS rejected_permissions,
    COUNT(*) FILTER (WHERE migration_status = 'Completed' AND verification_status = 'Verificado') AS verified_permissions,
    COUNT(*) FILTER (WHERE migration_status = 'Completed' AND verification_status = 'Não Encontrado') AS missing_permissions
FROM lab_workspace.migration_verification
GROUP BY target_environment;

-- Consultar relatório
SELECT * FROM lab_workspace.migration_report;
```

## Verificações e Testes

1. Verificar permissões como diferentes usuários:
   ```
   psql -U dev_user1 -d lab_dcl
   psql -U homolog_user1 -d lab_dcl
   psql -U prod_user1 -d lab_dcl
   ```

2. Testar operações em cada ambiente:
   ```sql
   -- Como dev_user1
   INSERT INTO dev.clientes (nome, email) VALUES ('Cliente Dev', 'dev@email.com');
   
   -- Como homolog_user1
   INSERT INTO homolog.clientes (nome, email) VALUES ('Cliente Homolog', 'homolog@email.com');
   
   -- Como prod_user1
   SELECT * FROM prod.clientes;
   INSERT INTO prod.clientes (nome, email) VALUES ('Cliente Prod', 'prod@email.com'); -- Deve falhar
   ```

3. Verificar inventário de permissões:
   ```sql
   SELECT * FROM lab_workspace.permission_inventory
   WHERE environment = 'prod'
   ORDER BY object_type, object_name;
   ```

## Conclusão

Neste laboratório, você implementou:

- Um processo estruturado para migração de permissões entre ambientes
- Ferramentas para extração e documentação de permissões existentes
- Mecanismos para planejamento e revisão de migrações
- Scripts automatizados para execução de migrações
- Métodos para verificação e validação de permissões migradas

Esta abordagem sistemática para migração de permissões oferece vários benefícios:

1. **Consistência**: Garante que todas as permissões sejam corretamente transferidas entre ambientes
2. **Segurança**: Mantém os controles de acesso e privilégios definidos
3. **Eficiência**: Automatiza o processo de migração, reduzindo erros humanos
4. **Documentação**: Fornece um registro detalhado das permissões em cada ambiente
5. **Flexibilidade**: Permite ajustes e revisões no plano de migração antes da execução
