# Exercícios Intermediários de DCL

## Exercício 1: Hierarquia de Roles

### Descrição
Pratique a criação de uma hierarquia de roles para organizar permissões.

### Requisitos
- Criar estrutura hierárquica de roles
- Conceder permissões em diferentes níveis
- Herança de permissões
- Verificar permissões efetivas

### Solução
```sql
-- Criar estrutura hierárquica de roles
CREATE ROLE funcionario_base;
CREATE ROLE vendedor;
CREATE ROLE gerente_vendas;
CREATE ROLE diretor_comercial;

-- Estabelecer hierarquia
GRANT funcionario_base TO vendedor;
GRANT vendedor TO gerente_vendas;
GRANT gerente_vendas TO diretor_comercial;

-- Conceder permissões em diferentes níveis
GRANT SELECT ON aplicacao.produtos TO funcionario_base;
GRANT SELECT, INSERT ON aplicacao.pedidos TO vendedor;
GRANT UPDATE ON aplicacao.clientes TO gerente_vendas;
GRANT ALL PRIVILEGES ON SCHEMA aplicacao TO diretor_comercial;

-- Atribuir roles a usuários
GRANT vendedor TO usuario_vendas;
GRANT gerente_vendas TO usuario_gerente;
GRANT diretor_comercial TO usuario_diretor;

-- Verificar permissões efetivas
SELECT r.rolname, u.usename, 
       has_table_privilege(u.usename, 'aplicacao.produtos', 'SELECT') as pode_consultar,
       has_table_privilege(u.usename, 'aplicacao.pedidos', 'INSERT') as pode_inserir,
       has_table_privilege(u.usename, 'aplicacao.clientes', 'UPDATE') as pode_atualizar
FROM pg_roles r
JOIN pg_user u ON r.rolname = u.usename
WHERE r.rolname IN ('usuario_vendas', 'usuario_gerente', 'usuario_diretor');
```

## Exercício 2: Permissões Baseadas em Função

### Descrição
Implemente um sistema de permissões baseado nas funções dos usuários na organização.

### Requisitos
- Criar roles para departamentos
- Criar roles para níveis hierárquicos
- Implementar matriz de permissões
- Gerenciar usuários com múltiplas funções

### Solução
```sql
-- Criar roles para departamentos
CREATE ROLE dept_vendas;
CREATE ROLE dept_marketing;
CREATE ROLE dept_financeiro;
CREATE ROLE dept_ti;

-- Criar roles para níveis hierárquicos
CREATE ROLE nivel_operacional;
CREATE ROLE nivel_tatico;
CREATE ROLE nivel_estrategico;

-- Implementar matriz de permissões para departamentos
-- Vendas
GRANT SELECT, INSERT ON aplicacao.clientes TO dept_vendas;
GRANT SELECT, INSERT ON aplicacao.pedidos TO dept_vendas;
GRANT SELECT ON aplicacao.produtos TO dept_vendas;

-- Marketing
GRANT SELECT ON aplicacao.clientes TO dept_marketing;
GRANT SELECT ON aplicacao.pedidos TO dept_marketing;
GRANT SELECT ON relatorios.vendas_mensais TO dept_marketing;

-- Financeiro
GRANT SELECT ON aplicacao.pedidos TO dept_financeiro;
GRANT SELECT ON aplicacao.clientes TO dept_financeiro;
GRANT SELECT, UPDATE ON relatorios.vendas_mensais TO dept_financeiro;

-- TI
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA aplicacao TO dept_ti;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA relatorios TO dept_ti;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA administracao TO dept_ti;

-- Implementar matriz de permissões para níveis
-- Operacional
GRANT SELECT ON aplicacao.produtos TO nivel_operacional;
GRANT SELECT ON aplicacao.clientes TO nivel_operacional;

-- Tático
GRANT SELECT, INSERT, UPDATE ON aplicacao.produtos TO nivel_tatico;
GRANT SELECT, INSERT, UPDATE ON aplicacao.clientes TO nivel_tatico;
GRANT SELECT ON relatorios.vendas_mensais TO nivel_tatico;

-- Estratégico
GRANT ALL PRIVILEGES ON SCHEMA relatorios TO nivel_estrategico;
GRANT SELECT ON ALL TABLES IN SCHEMA aplicacao TO nivel_estrategico;

-- Gerenciar usuários com múltiplas funções
CREATE USER maria WITH PASSWORD 'senha123';
CREATE USER joao WITH PASSWORD 'senha123';
CREATE USER carlos WITH PASSWORD 'senha123';

-- Maria: Vendas + Operacional
GRANT dept_vendas TO maria;
GRANT nivel_operacional TO maria;

-- João: Marketing + Tático
GRANT dept_marketing TO joao;
GRANT nivel_tatico TO joao;

-- Carlos: TI + Estratégico
GRANT dept_ti TO carlos;
GRANT nivel_estrategico TO carlos;
```

## Exercício 3: Permissões com Condições

### Descrição
Implemente permissões com condições específicas usando roles e esquemas.

### Requisitos
- Criar roles com permissões condicionais
- Implementar permissões por horário de acesso
- Implementar permissões por tipo de operação
- Verificar efetividade das permissões

### Solução
```sql
-- Criar roles para diferentes condições
CREATE ROLE acesso_horario_comercial;
CREATE ROLE acesso_somente_leitura;
CREATE ROLE acesso_manutencao;

-- Conceder permissões básicas
GRANT CONNECT ON DATABASE seguranca_db TO acesso_horario_comercial;
GRANT USAGE ON SCHEMA aplicacao TO acesso_horario_comercial;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA aplicacao TO acesso_horario_comercial;

GRANT CONNECT ON DATABASE seguranca_db TO acesso_somente_leitura;
GRANT USAGE ON SCHEMA aplicacao, relatorios TO acesso_somente_leitura;
GRANT SELECT ON ALL TABLES IN SCHEMA aplicacao, relatorios TO acesso_somente_leitura;

GRANT CONNECT ON DATABASE seguranca_db TO acesso_manutencao;
GRANT USAGE ON SCHEMA aplicacao TO acesso_manutencao;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA aplicacao TO acesso_manutencao;

-- Criar usuários para teste
CREATE USER usuario_dia WITH PASSWORD 'senha123';
CREATE USER usuario_consulta WITH PASSWORD 'senha123';
CREATE USER usuario_manutencao WITH PASSWORD 'senha123';

-- Atribuir roles aos usuários
GRANT acesso_horario_comercial TO usuario_dia;
GRANT acesso_somente_leitura TO usuario_consulta;
GRANT acesso_manutencao TO usuario_manutencao;

-- Criar tabela para controle de acesso
CREATE TABLE administracao.controle_acesso (
    id SERIAL PRIMARY KEY,
    role_name VARCHAR(50),
    horario_inicio TIME,
    horario_fim TIME,
    dias_semana VARCHAR(20),
    ativo BOOLEAN DEFAULT TRUE
);

-- Inserir regras de controle
INSERT INTO administracao.controle_acesso 
(role_name, horario_inicio, horario_fim, dias_semana)
VALUES 
('acesso_horario_comercial', '08:00:00', '18:00:00', 'Mon-Fri'),
('acesso_manutencao', '22:00:00', '06:00:00', 'Mon-Sun');

-- Criar função para verificar permissão de acesso
CREATE OR REPLACE FUNCTION administracao.verificar_acesso_permitido(p_role VARCHAR)
RETURNS BOOLEAN AS $$
DECLARE
    v_permitido BOOLEAN := FALSE;
    v_regra RECORD;
    v_hora_atual TIME := CURRENT_TIME;
    v_dia_atual VARCHAR := to_char(CURRENT_DATE, 'Dy');
BEGIN
    -- Verificar se existe regra para a role
    SELECT * INTO v_regra
    FROM administracao.controle_acesso
    WHERE role_name = p_role AND ativo = TRUE;
    
    IF NOT FOUND THEN
        -- Se não há regra específica, acesso é permitido
        RETURN TRUE;
    END IF;
    
    -- Verificar horário
    IF (v_hora_atual BETWEEN v_regra.horario_inicio AND v_regra.horario_fim) THEN
        -- Verificar dia da semana
        IF v_regra.dias_semana = 'Mon-Sun' OR 
           position(v_dia_atual in v_regra.dias_semana) > 0 THEN
            v_permitido := TRUE;
        END IF;
    END IF;
    
    -- Registrar tentativa de acesso
    INSERT INTO administracao.log_acessos
    (username, tabela_acessada, tipo_operacao)
    VALUES (
        current_user,
        'Verificação de acesso',
        CASE WHEN v_permitido THEN 'ACESSO_PERMITIDO' ELSE 'ACESSO_NEGADO' END
    );
    
    RETURN v_permitido;
END;
$$ LANGUAGE plpgsql;

-- Instruções para uso (comentadas)
/*
-- Como administrador, criar regra para verificar acesso antes de operações
CREATE OR REPLACE RULE check_acesso_clientes AS
ON SELECT TO aplicacao.clientes
WHERE NOT administracao.verificar_acesso_permitido(current_user)
DO INSTEAD NOTHING;

-- Testar acesso em diferentes horários
-- (executar como diferentes usuários)
SELECT * FROM aplicacao.clientes;
*/
```

## Exercício 4: Auditoria de Permissões

### Descrição
Implemente um sistema para auditar e monitorar permissões e acessos.

### Requisitos
- Criar tabelas de auditoria
- Registrar alterações de permissões
- Criar relatórios de auditoria
- Detectar anomalias de permissões

### Solução
```sql
-- Criar tabelas de auditoria
CREATE TABLE administracao.auditoria_permissoes (
    id SERIAL PRIMARY KEY,
    data_hora TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_executor VARCHAR(50),
    comando TEXT,
    objeto_afetado TEXT,
    usuario_afetado VARCHAR(50),
    tipo_operacao VARCHAR(20)
);

-- Criar função para registrar alterações de permissões manualmente
CREATE OR REPLACE FUNCTION administracao.registrar_alteracao_permissao(
    p_comando TEXT,
    p_objeto TEXT,
    p_usuario TEXT,
    p_tipo VARCHAR
) RETURNS void AS $$
BEGIN
    INSERT INTO administracao.auditoria_permissoes
    (usuario_executor, comando, objeto_afetado, usuario_afetado, tipo_operacao)
    VALUES (
        current_user,
        p_comando,
        p_objeto,
        p_usuario,
        p_tipo
    );
END;
$$ LANGUAGE plpgsql;

-- Criar relatório de auditoria
CREATE OR REPLACE VIEW administracao.relatorio_permissoes AS
SELECT 
    u.usename AS usuario,
    n.nspname AS schema,
    c.relname AS tabela,
    c.relkind AS tipo_objeto,
    string_agg(p.privilege_type, ', ') AS privilegios
FROM pg_user u
CROSS JOIN pg_class c
JOIN pg_namespace n ON c.relnamespace = n.oid
JOIN information_schema.table_privileges p ON 
    p.grantee = u.usename AND 
    p.table_schema = n.nspname AND 
    p.table_name = c.relname
WHERE n.nspname NOT LIKE 'pg_%' AND n.nspname != 'information_schema'
GROUP BY u.usename, n.nspname, c.relname, c.relkind
ORDER BY u.usename, n.nspname, c.relname;

-- Detectar anomalias de permissões
CREATE OR REPLACE VIEW administracao.anomalias_permissoes AS
-- Usuários com permissões diretas (não via roles)
SELECT 
    p.grantee AS usuario,
    p.table_schema || '.' || p.table_name AS objeto,
    p.privilege_type AS privilegio,
    'Permissão direta' AS tipo_anomalia
FROM information_schema.table_privileges p
WHERE p.grantee NOT IN (
    SELECT rolname FROM pg_roles WHERE rolcanlogin = false
)
AND p.table_schema NOT LIKE 'pg_%'
AND p.table_schema != 'information_schema'

UNION ALL

-- Usuários com permissões excessivas
SELECT 
    p.grantee AS usuario,
    p.table_schema || '.' || p.table_name AS objeto,
    string_agg(p.privilege_type, ', ') AS privilegio,
    'Permissões excessivas' AS tipo_anomalia
FROM information_schema.table_privileges p
WHERE p.privilege_type IN ('DELETE', 'TRUNCATE')
AND p.table_schema = 'aplicacao'
GROUP BY p.grantee, p.table_schema, p.table_name;
```

## Exercício 5: Gerenciamento de Permissões em Massa

### Descrição
Implemente scripts para gerenciar permissões em massa para múltiplos objetos e usuários.

### Requisitos
- Conceder permissões em massa
- Revogar permissões em massa
- Sincronizar permissões entre ambientes
- Documentar permissões concedidas

### Solução
```sql
-- Função para conceder permissões em massa
CREATE OR REPLACE FUNCTION administracao.conceder_permissoes_em_massa(
    p_schema TEXT,
    p_role TEXT,
    p_privilegios TEXT
) RETURNS void AS $$
DECLARE
    v_tabela TEXT;
BEGIN
    FOR v_tabela IN 
        SELECT table_name 
        FROM information_schema.tables 
        WHERE table_schema = p_schema
        AND table_type = 'BASE TABLE'
    LOOP
        EXECUTE 'GRANT ' || p_privilegios || ' ON ' || 
                p_schema || '.' || v_tabela || 
                ' TO ' || p_role;
                
        -- Registrar na auditoria
        INSERT INTO administracao.auditoria_permissoes
        (usuario_executor, comando, objeto_afetado, usuario_afetado, tipo_operacao)
        VALUES (
            current_user,
            'GRANT ' || p_privilegios || ' ON ' || p_schema || '.' || v_tabela || ' TO ' || p_role,
            p_schema || '.' || v_tabela,
            p_role,
            'GRANT_MASSA'
        );
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Função para revogar permissões em massa
CREATE OR REPLACE FUNCTION administracao.revogar_permissoes_em_massa(
    p_schema TEXT,
    p_role TEXT,
    p_privilegios TEXT DEFAULT 'ALL PRIVILEGES'
) RETURNS void AS $$
DECLARE
    v_tabela TEXT;
BEGIN
    FOR v_tabela IN 
        SELECT table_name 
        FROM information_schema.tables 
        WHERE table_schema = p_schema
        AND table_type = 'BASE TABLE'
    LOOP
        EXECUTE 'REVOKE ' || p_privilegios || ' ON ' || 
                p_schema || '.' || v_tabela || 
                ' FROM ' || p_role;
                
        -- Registrar na auditoria
        INSERT INTO administracao.auditoria_permissoes
        (usuario_executor, comando, objeto_afetado, usuario_afetado, tipo_operacao)
        VALUES (
            current_user,
            'REVOKE ' || p_privilegios || ' ON ' || p_schema || '.' || v_tabela || ' FROM ' || p_role,
            p_schema || '.' || v_tabela,
            p_role,
            'REVOKE_MASSA'
        );
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Função para exportar permissões (para sincronização entre ambientes)
CREATE OR REPLACE FUNCTION administracao.exportar_permissoes(
    p_schema TEXT DEFAULT NULL
) RETURNS TABLE (comando TEXT) AS $$
BEGIN
    RETURN QUERY
    WITH perms AS (
        SELECT 
            'GRANT ' || string_agg(privilege_type, ', ') || 
            ' ON ' || table_schema || '.' || table_name || 
            ' TO ' || grantee || ';' AS cmd
        FROM information_schema.table_privileges
        WHERE (p_schema IS NULL OR table_schema = p_schema)
        AND table_schema NOT LIKE 'pg_%'
        AND table_schema != 'information_schema'
        GROUP BY table_schema, table_name, grantee
        
        UNION ALL
        
        SELECT 
            'GRANT ' || string_agg(privilege_type, ', ') || 
            ' ON SCHEMA ' || table_schema || 
            ' TO ' || grantee || ';' AS cmd
        FROM information_schema.schema_privileges
        WHERE (p_schema IS NULL OR table_schema = p_schema)
        AND table_schema NOT LIKE 'pg_%'
        AND table_schema != 'information_schema'
        GROUP BY table_schema, grantee
    )
    SELECT cmd FROM perms
    ORDER BY cmd;
END;
$$ LANGUAGE plpgsql;
```
