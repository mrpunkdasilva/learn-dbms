# Laboratório: Gerenciamento de Usuários

## Objetivo
Praticar a criação, configuração e gerenciamento de usuários em um banco de dados PostgreSQL, aplicando boas práticas de segurança e controle de acesso.

## Cenário
Você é um administrador de banco de dados em uma empresa que está implementando um novo sistema de gerenciamento de permissões. Sua tarefa é configurar usuários com diferentes níveis de acesso de acordo com suas funções na organização.

## Setup Inicial

```sql
-- Conectar ao banco de dados como superusuário
-- psql -U postgres -d lab_dcl

-- Verificar usuários existentes
SELECT usename, usecreatedb, usesuper, passwd IS NOT NULL AS has_password
FROM pg_catalog.pg_user;

-- Registrar início do laboratório
INSERT INTO lab_workspace.lab_progress (lab_name)
VALUES ('Gerenciamento de Usuários');
```

## Parte 1: Criação de Usuários Básicos

### 1.1 Criação de Usuários

```sql
-- Criar usuário básico
CREATE USER app_user WITH PASSWORD 'app123';

-- Criar usuário com data de expiração
CREATE USER temp_user WITH 
    PASSWORD 'temp456' 
    VALID UNTIL '2023-12-31 23:59:59';

-- Criar usuário sem privilégios de login
CREATE USER app_service WITH 
    PASSWORD 'srv789' 
    NOLOGIN;

-- Verificar usuários criados
SELECT usename, usesuper, usecreatedb, valuntil 
FROM pg_catalog.pg_user 
WHERE usename IN ('app_user', 'temp_user', 'app_service');
```

### 1.2 Configuração de Parâmetros

```sql
-- Alterar configurações de usuário
ALTER USER app_user SET search_path TO lab_workspace, public;
ALTER USER app_user SET statement_timeout = '30s';
ALTER USER app_user SET idle_in_transaction_session_timeout = '60s';

-- Verificar configurações
SELECT usename, useconfig 
FROM pg_catalog.pg_user 
WHERE usename = 'app_user';
```

## Parte 2: Atribuição de Permissões

### 2.1 Permissões Básicas

```sql
-- Conceder permissões de conexão
GRANT CONNECT ON DATABASE lab_dcl TO app_user;
GRANT CONNECT ON DATABASE lab_dcl TO temp_user;

-- Conceder permissões de uso no schema
GRANT USAGE ON SCHEMA lab_workspace TO app_user;
GRANT USAGE ON SCHEMA lab_workspace TO temp_user;

-- Conceder permissões de leitura para app_user
GRANT SELECT ON ALL TABLES IN SCHEMA lab_workspace TO app_user;

-- Conceder permissões específicas para temp_user
GRANT SELECT ON lab_workspace.clientes TO temp_user;
GRANT SELECT ON lab_workspace.produtos TO temp_user;
```

### 2.2 Permissões Avançadas

```sql
-- Conceder permissões de modificação em tabelas específicas
GRANT INSERT, UPDATE ON lab_workspace.clientes TO app_user;
GRANT INSERT, UPDATE, DELETE ON lab_workspace.pedidos TO app_user;
GRANT INSERT, UPDATE, DELETE ON lab_workspace.itens_pedido TO app_user;

-- Conceder permissões em nível de coluna
GRANT UPDATE (nome, email, status) ON lab_workspace.clientes TO app_user;
GRANT UPDATE (status, observacoes) ON lab_workspace.pedidos TO temp_user;

-- Revogar permissões sensíveis
REVOKE UPDATE (limite_credito, dados_sensíveis) ON lab_workspace.clientes FROM app_user;
```

## Parte 3: Gerenciamento de Senhas e Políticas

### 3.1 Alteração de Senhas

```sql
-- Alterar senha de usuário
ALTER USER app_user WITH PASSWORD 'novasenha123';

-- Forçar expiração de senha
ALTER USER temp_user VALID UNTIL '2023-10-31 23:59:59';

-- Desabilitar usuário temporariamente
ALTER USER temp_user WITH NOLOGIN;
```

### 3.2 Implementação de Políticas

```sql
-- Criar função para registrar acessos
CREATE OR REPLACE FUNCTION lab_workspace.log_user_access()
RETURNS trigger AS $$
BEGIN
    INSERT INTO lab_workspace.log_acessos (
        usuario_id,
        operacao,
        tabela_acessada,
        detalhes
    ) VALUES (
        (SELECT id FROM lab_workspace.usuarios WHERE username = current_user),
        TG_OP,
        TG_TABLE_NAME,
        'Acesso em ' || now()
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Criar trigger para auditoria
CREATE TRIGGER log_clientes_access
AFTER INSERT OR UPDATE OR DELETE ON lab_workspace.clientes
FOR EACH STATEMENT EXECUTE FUNCTION lab_workspace.log_user_access();
```

## Parte 4: Monitoramento e Auditoria

### 4.1 Visualização de Atividades

```sql
-- Consultar atividades de usuários
SELECT usename, application_name, client_addr, backend_start, state
FROM pg_stat_activity
WHERE usename IN ('app_user', 'temp_user');

-- Consultar permissões concedidas
SELECT grantee, table_schema, table_name, privilege_type
FROM information_schema.table_privileges
WHERE grantee IN ('app_user', 'temp_user')
ORDER BY grantee, table_schema, table_name;
```

### 4.2 Relatórios de Segurança

```sql
-- Criar visão para relatório de permissões
CREATE OR REPLACE VIEW lab_workspace.user_permissions_report AS
SELECT 
    u.usename AS username,
    s.nspname AS schema,
    t.relname AS table,
    array_agg(p.privilege_type) AS privileges
FROM pg_user u
CROSS JOIN pg_namespace s
JOIN pg_class t ON t.relnamespace = s.oid
JOIN information_schema.table_privileges p ON 
    p.table_schema = s.nspname AND 
    p.table_name = t.relname AND 
    p.grantee = u.usename
WHERE s.nspname = 'lab_workspace'
GROUP BY u.usename, s.nspname, t.relname
ORDER BY u.usename, s.nspname, t.relname;

-- Consultar relatório
SELECT * FROM lab_workspace.user_permissions_report;
```

## Verificações e Testes

1. Conecte-se como cada usuário e verifique suas permissões:
   ```
   psql -U app_user -d lab_dcl
   ```

2. Tente executar operações permitidas e não permitidas:
   ```sql
   -- Como app_user
   SELECT * FROM lab_workspace.clientes;
   UPDATE lab_workspace.clientes SET nome = 'Teste' WHERE id = 1;
   UPDATE lab_workspace.clientes SET limite_credito = 5000 WHERE id = 1; -- Deve falhar
   ```

3. Verifique os logs de auditoria:
   ```sql
   SELECT * FROM lab_workspace.log_acessos ORDER BY data_hora DESC;
   ```

## Conclusão

Neste laboratório, você praticou:

- Criação e configuração de usuários
- Atribuição de permissões em diferentes níveis
- Gerenciamento de senhas e políticas de segurança
- Monitoramento e auditoria de atividades

Estas habilidades são fundamentais para implementar um sistema de controle de acesso robusto e seguro em ambientes PostgreSQL.

## Próximos Passos

1. Explore a criação de roles para agrupar permissões
2. Implemente políticas de segurança mais avançadas
3. Configure autenticação externa (LDAP, SSO)
4. Desenvolva um sistema de auditoria mais abrangente

---

> **Nota**: Registre a conclusão do laboratório:
> ```sql
> UPDATE lab_workspace.lab_progress 
> SET completion_time = CURRENT_TIMESTAMP, status = 'COMPLETED' 
> WHERE lab_name = 'Gerenciamento de Usuários';
> ```