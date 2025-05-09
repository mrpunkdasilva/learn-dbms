# Laboratório: Auditoria de Segurança

## Objetivo
Implementar e configurar um sistema abrangente de auditoria de segurança em um banco de dados PostgreSQL, identificando atividades suspeitas e garantindo conformidade com políticas de segurança.

## Cenário
Você é um especialista em segurança de dados em uma instituição financeira que precisa implementar controles de auditoria rigorosos para atender a requisitos regulatórios e de compliance. Sua tarefa é configurar um sistema de auditoria que capture todas as atividades relevantes no banco de dados.

## Setup Inicial

```sql
-- Conectar ao banco de dados como superusuário
-- psql -U postgres -d lab_dcl

-- Verificar extensões disponíveis
SELECT name, default_version, installed_version
FROM pg_available_extensions
WHERE name IN ('pgaudit', 'pg_stat_statements');

-- Registrar início do laboratório
INSERT INTO lab_workspace.lab_progress (lab_name)
VALUES ('Auditoria de Segurança');
```

## Parte 1: Configuração de Logs Básicos

### 1.1 Configuração de Parâmetros de Log

```sql
-- Verificar configurações atuais
SHOW log_destination;
SHOW logging_collector;
SHOW log_directory;
SHOW log_filename;
SHOW log_statement;

-- Configurar parâmetros (em postgresql.conf ou via ALTER SYSTEM)
-- Nota: Em um ambiente real, você alteraria o postgresql.conf
-- Aqui usamos ALTER SYSTEM para demonstração

ALTER SYSTEM SET log_destination = 'csvlog';
ALTER SYSTEM SET logging_collector = 'on';
ALTER SYSTEM SET log_directory = 'pg_log';
ALTER SYSTEM SET log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log';
ALTER SYSTEM SET log_rotation_age = '1d';
ALTER SYSTEM SET log_rotation_size = '10MB';

-- Configurar o que será logado
ALTER SYSTEM SET log_statement = 'mod';  -- Loga todas as modificações (INSERT, UPDATE, DELETE, etc.)
ALTER SYSTEM SET log_min_duration_statement = '1000';  -- Loga queries que demoram mais de 1 segundo
ALTER SYSTEM SET log_connections = 'on';
ALTER SYSTEM SET log_disconnections = 'on';
ALTER SYSTEM SET log_duration = 'on';

-- Aplicar alterações (requer reinicialização do servidor)
-- SELECT pg_reload_conf();
```

### 1.2 Criação de Tabelas de Auditoria

```sql
-- Criar schema dedicado para auditoria
CREATE SCHEMA IF NOT EXISTS audit;

-- Tabela para registro de atividades
CREATE TABLE audit.activity_log (
    id SERIAL PRIMARY KEY,
    event_time TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    username TEXT NOT NULL,
    database_name TEXT NOT NULL,
    schema_name TEXT NOT NULL,
    table_name TEXT,
    action_type TEXT NOT NULL,
    query TEXT,
    old_data JSONB,
    new_data JSONB,
    client_ip TEXT,
    application_name TEXT
);

-- Tabela para registro de logins
CREATE TABLE audit.login_attempts (
    id SERIAL PRIMARY KEY,
    event_time TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    username TEXT NOT NULL,
    success BOOLEAN NOT NULL,
    client_ip TEXT,
    connection_type TEXT,
    application_name TEXT,
    details TEXT
);

-- Tabela para registro de alterações de permissões
CREATE TABLE audit.permission_changes (
    id SERIAL PRIMARY KEY,
    event_time TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    username TEXT NOT NULL,
    action_type TEXT NOT NULL,
    object_type TEXT NOT NULL,
    object_name TEXT NOT NULL,
    grantee TEXT,
    permission TEXT,
    query TEXT
);
```

## Parte 2: Implementação de Triggers de Auditoria

### 2.1 Função Genérica de Auditoria

```sql
-- Criar função para auditoria de alterações em tabelas
CREATE OR REPLACE FUNCTION audit.log_table_changes()
RETURNS TRIGGER AS $$
DECLARE
    old_data JSONB := NULL;
    new_data JSONB := NULL;
BEGIN
    IF TG_OP = 'DELETE' THEN
        old_data := row_to_json(OLD)::JSONB;
    ELSIF TG_OP = 'UPDATE' THEN
        old_data := row_to_json(OLD)::JSONB;
        new_data := row_to_json(NEW)::JSONB;
    ELSIF TG_OP = 'INSERT' THEN
        new_data := row_to_json(NEW)::JSONB;
    END IF;

    INSERT INTO audit.activity_log (
        username,
        database_name,
        schema_name,
        table_name,
        action_type,
        query,
        old_data,
        new_data,
        client_ip,
        application_name
    ) VALUES (
        current_user,
        current_database(),
        TG_TABLE_SCHEMA,
        TG_TABLE_NAME,
        TG_OP,
        current_query(),
        old_data,
        new_data,
        inet_client_addr(),
        current_setting('application_name')
    );

    RETURN NULL;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### 2.2 Aplicação de Triggers nas Tabelas

```sql
-- Aplicar triggers nas tabelas principais
CREATE TRIGGER audit_clientes
AFTER INSERT OR UPDATE OR DELETE ON lab_workspace.clientes
FOR EACH ROW EXECUTE FUNCTION audit.log_table_changes();

CREATE TRIGGER audit_produtos
AFTER INSERT OR UPDATE OR DELETE ON lab_workspace.produtos
FOR EACH ROW EXECUTE FUNCTION audit.log_table_changes();

CREATE TRIGGER audit_pedidos
AFTER INSERT OR UPDATE OR DELETE ON lab_workspace.pedidos
FOR EACH ROW EXECUTE FUNCTION audit.log_table_changes();

CREATE TRIGGER audit_itens_pedido
AFTER INSERT OR UPDATE OR DELETE ON lab_workspace.itens_pedido
FOR EACH ROW EXECUTE FUNCTION audit.log_table_changes();

CREATE TRIGGER audit_usuarios
AFTER INSERT OR UPDATE OR DELETE ON lab_workspace.usuarios
FOR EACH ROW EXECUTE FUNCTION audit.log_table_changes();
```

## Parte 3: Monitoramento de Atividades Sensíveis

### 3.1 Auditoria de Permissões

```sql
-- Função para auditoria de alterações de permissões
CREATE OR REPLACE FUNCTION audit.log_permission_changes()
RETURNS event_trigger AS $$
DECLARE
    obj record;
    query text;
BEGIN
    query := current_query();
    
    IF query ~* 'grant|revoke' THEN
        INSERT INTO audit.permission_changes (
            username,
            action_type,
            object_type,
            object_name,
            query
        ) VALUES (
            current_user,
            CASE 
                WHEN query ~* 'grant' THEN 'GRANT'
                WHEN query ~* 'revoke' THEN 'REVOKE'
                ELSE 'UNKNOWN'
            END,
            CASE 
                WHEN query ~* 'table' THEN 'TABLE'
                WHEN query ~* 'function' THEN 'FUNCTION'
                WHEN query ~* 'schema' THEN 'SCHEMA'
                WHEN query ~* 'sequence' THEN 'SEQUENCE'
                ELSE 'UNKNOWN'
            END,
            regexp_replace(query, '.*(?:on|ON)\s+([^\s]+).*', '\1'),
            query
        );
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Criar event trigger para capturar alterações de permissões
CREATE EVENT TRIGGER permission_audit ON ddl_command_end
WHEN TAG IN ('GRANT', 'REVOKE')
EXECUTE FUNCTION audit.log_permission_changes();
```

### 3.2 Auditoria de Logins

```sql
-- Função para simular auditoria de logins
-- Nota: Em um ambiente real, isso seria implementado via configuração do PostgreSQL
-- ou usando extensões como pgaudit

CREATE OR REPLACE FUNCTION audit.simulate_login_audit()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit.login_attempts (
        username,
        success,
        client_ip,
        application_name
    ) VALUES (
        NEW.username,
        TRUE,
        inet_client_addr(),
        current_setting('application_name')
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Aplicar trigger na tabela de usuários para simular login
-- (apenas para demonstração - em ambiente real seria diferente)
CREATE TRIGGER simulate_login
AFTER UPDATE OF ultimo_acesso ON lab_workspace.usuarios
FOR EACH ROW EXECUTE FUNCTION audit.simulate_login_audit();

-- Função para registrar login manualmente (para demonstração)
CREATE OR REPLACE FUNCTION lab_workspace.register_login(username text)
RETURNS void AS $$
BEGIN
    UPDATE lab_workspace.usuarios
    SET ultimo_acesso = CURRENT_TIMESTAMP
    WHERE username = register_login.username;
END;
$$ LANGUAGE plpgsql;
```

## Parte 4: Análise e Relatórios de Auditoria

### 4.1 Visões para Análise

```sql
-- Visão para atividades por usuário
CREATE OR REPLACE VIEW audit.user_activity AS
SELECT 
    username,
    COUNT(*) AS total_actions,
    COUNT(*) FILTER (WHERE action_type = 'INSERT') AS inserts,
    COUNT(*) FILTER (WHERE action_type = 'UPDATE') AS updates,
    COUNT(*) FILTER (WHERE action_type = 'DELETE') AS deletes,
    MIN(event_time) AS first_activity,
    MAX(event_time) AS last_activity
FROM audit.activity_log
GROUP BY username
ORDER BY total_actions DESC;

-- Visão para atividades por tabela
CREATE OR REPLACE VIEW audit.table_activity AS
SELECT 
    schema_name,
    table_name,
    COUNT(*) AS total_actions,
    COUNT(*) FILTER (WHERE action_type = 'INSERT') AS inserts,
    COUNT(*) FILTER (WHERE action_type = 'UPDATE') AS updates,
    COUNT(*) FILTER (WHERE action_type = 'DELETE') AS deletes,
    MIN(event_time) AS first_activity,
    MAX(event_time) AS last_activity
FROM audit.activity_log
GROUP BY schema_name, table_name
ORDER BY total_actions DESC;

-- Visão para alterações sensíveis
CREATE OR REPLACE VIEW audit.sensitive_changes AS
SELECT 
    a.id,
    a.event_time,
    a.username,
    a.table_name,
    a.action_type,
    a.old_data,
    a.new_data
FROM audit.activity_log a
WHERE 
    (a.table_name = 'clientes' AND 
     (a.old_data->>'limite_credito' IS DISTINCT FROM a.new_data->>'limite_credito' OR
      a.old_data->>'dados_sensíveis' IS DISTINCT FROM a.new_data->>'dados_sensíveis'))
    OR
    (a.table_name = 'usuarios' AND a.action_type IN ('INSERT', 'DELETE'))
ORDER BY a.event_time DESC;
```

### 4.2 Funções para Relatórios

```sql
-- Função para relatório de atividades em um período
CREATE OR REPLACE FUNCTION audit.activity_report(
    start_time timestamp with time zone,
    end_time timestamp with time zone
)
RETURNS TABLE (
    username text,
    action_type text,
    table_name text,
    count bigint
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        a.username,
        a.action_type,
        a.table_name,
        COUNT(*) AS count
    FROM audit.activity_log a
    WHERE a.event_time BETWEEN start_time AND end_time
    GROUP BY a.username, a.action_type, a.table_name
    ORDER BY count DESC;
END;
$$ LANGUAGE plpgsql;

-- Função para detectar atividades suspeitas
CREATE OR REPLACE FUNCTION audit.detect_suspicious_activity()
RETURNS TABLE (
    id integer,
    event_time timestamp with time zone,
    username text,
    action_type text,
    table_name text,
    suspicion_reason text
) AS $$
BEGIN
    RETURN QUERY
    
    -- Atividades fora do horário comercial
    SELECT 
        a.id,
        a.event_time,
        a.username,
        a.action_type,
        a.table_name,
        'Atividade fora do horário comercial' AS suspicion_reason
    FROM audit.activity_log a
    WHERE 
        EXTRACT(HOUR FROM a.event_time) < 8 OR
        EXTRACT(HOUR FROM a.event_time) > 18
        
    UNION ALL
    
    -- Volume anormal de operações
    SELECT 
        a.id,
        a.event_time,
        a.username,
        a.action_type,
        a.table_name,
        'Volume anormal de operações' AS suspicion_reason
    FROM audit.activity_log a
    JOIN (
        SELECT 
            username, 
            table_name, 
            COUNT(*) AS action_count
        FROM audit.activity_log
        WHERE event_time > CURRENT_TIMESTAMP - INTERVAL '1 hour'
        GROUP BY username, table_name
        HAVING COUNT(*) > 50
    ) high_volume ON a.username = high_volume.username AND a.table_name = high_volume.table_name
    
    UNION ALL
    
    -- Alterações em dados sensíveis
    SELECT 
        a.id,
        a.event_time,
        a.username,
        a.action_type,
        a.table_name,
        'Alteração em dados sensíveis' AS suspicion_reason
    FROM audit.activity_log a
    WHERE 
        a.table_name = 'clientes' AND
        a.action_type = 'UPDATE' AND
        a.old_data->>'limite_credito' IS DISTINCT FROM a.new_data->>'limite_credito'
    
    ORDER BY event_time DESC;
END;
$$ LANGUAGE plpgsql;
```

## Verificações e Testes

1. Gerar atividades para teste:
   ```sql
   -- Inserir dados de teste
   INSERT INTO lab_workspace.clientes (nome, email, segmento, limite_credito)
   VALUES ('Cliente Auditoria', 'auditoria@email.com', 'Varejo', 1000.00);
   
   UPDATE lab_workspace.clientes
   SET limite_credito = 5000.00
   WHERE email = 'auditoria@email.com';
   
   DELETE FROM lab_workspace.clientes
   WHERE email = 'auditoria@email.com';
   
   -- Simular login
   SELECT lab_workspace.register_login('app_user');
   ```

2. Verificar logs de auditoria:
   ```sql
   -- Verificar logs de atividade
   SELECT * FROM audit.activity_log ORDER BY event_time DESC LIMIT 10;
   
   -- Verificar logs de login
   SELECT * FROM audit.login_attempts ORDER BY event_time DESC LIMIT 10;
   
   -- Verificar logs de permissões
   SELECT * FROM audit.permission_changes ORDER BY event_time DESC LIMIT 10;
   ```

3. Executar relatórios:
   ```sql
   -- Relatório de atividades do dia
   SELECT * FROM audit.activity_report(
       CURRENT_DATE,
       CURRENT_DATE + INTERVAL '1 day'
   );
   
   -- Verificar atividades suspeitas
   SELECT * FROM audit.detect_suspicious_activity();
   
   -- Verificar visões de resumo
   SELECT * FROM audit.user_activity;
   SELECT * FROM audit.table_activity;
   SELECT * FROM audit.sensitive_changes;
   ```

## Conclusão

Neste laboratório, você implementou:

- Um sistema abrangente de auditoria para banco de dados PostgreSQL
- Captura automática de alterações em tabelas via triggers
- Monitoramento de atividades sensíveis como alterações de permissões
- Relatórios e análises para identificação de atividades suspeitas

Estas técnicas são essenciais para:

1. **Conformidade regulatória**: Atender requisitos como GDPR, PCI-DSS, SOX, etc.
2. **Segurança**: Detectar e investigar atividades maliciosas ou não autorizadas
3. **Forense digital**: Reconstruir eventos em caso de incidentes de segurança
4. **Governança de dados**: Manter controle sobre quem acessa e modifica dados sensíveis

## Próximos Passos

1. Implementar retenção e arquivamento de logs de auditoria
2. Configurar alertas automáticos para atividades suspeitas
3. Integrar com sistemas SIEM (Security Information and Event Management)
4. Implementar criptografia para dados de auditoria sensíveis

---

> **Nota**: Registre a conclusão do laboratório:
> ```sql
> UPDATE lab_workspace.lab_progress 
> SET completion_time = CURRENT_TIMESTAMP, status = 'COMPLETED' 
> WHERE lab_name = 'Auditoria de Segurança';
> ```