# Gerenciamento de Roles

```ascii
╔══════════════════════════════════════════════════════════════╗
║  ROLE_MASTER >> Gerenciamento de Roles e Grupos            ║
║  INSTRUTOR: ROLE_SPECIALIST                                ║
╚══════════════════════════════════════════════════════════════╝
```

## Conceitos Básicos

ROLE_SPECIALIST explica: "Roles são fundamentais para organizar e gerenciar permissões de forma eficiente!"

```sql
-- Criar role básica
CREATE ROLE nome_role;

-- Criar role com login
CREATE ROLE usuario_app WITH 
    LOGIN 
    PASSWORD 'senha_segura'
    VALID UNTIL '2024-12-31';
```

## Tipos de Roles

### 1. Roles de Aplicação

```sql
-- Role para aplicação web
CREATE ROLE app_web WITH LOGIN;
GRANT CONNECT ON DATABASE app_db TO app_web;
GRANT USAGE ON SCHEMA public TO app_web;
GRANT SELECT, INSERT, UPDATE ON tabela_usuarios TO app_web;
```

### 2. Roles de Grupo

```sql
-- Role para equipe de analistas
CREATE ROLE analistas;
GRANT SELECT ON ALL TABLES IN SCHEMA analytics TO analistas;
GRANT USAGE ON SCHEMA analytics TO analistas;

-- Atribuir usuários ao grupo
GRANT analistas TO analista1, analista2, analista3;
```

## Hierarquia de Roles

HIERARCHY_MASTER demonstra a estruturação:

```sql
-- Estrutura hierárquica
CREATE ROLE junior_dev;
CREATE ROLE senior_dev;
CREATE ROLE tech_lead;

-- Estabelecer hierarquia
GRANT junior_dev TO senior_dev;
GRANT senior_dev TO tech_lead;

-- Configurar permissões por nível
GRANT SELECT ON app_tables TO junior_dev;
GRANT INSERT, UPDATE ON app_tables TO senior_dev;
GRANT ALL PRIVILEGES ON app_tables TO tech_lead;
```

## Gerenciamento de Permissões

### 1. Atribuição de Privilégios

```sql
-- Privilégios básicos
GRANT SELECT, INSERT ON tabela TO role_name;

-- Privilégios administrativos
GRANT CREATE ON DATABASE app_db TO admin_role;
GRANT USAGE, CREATE ON SCHEMA public TO dev_role;
```

### 2. Revogação de Privilégios

```sql
-- Revogar privilégios
REVOKE INSERT, UPDATE ON dados_sensiveis FROM role_name;

-- Revogar membership
REVOKE analistas FROM ex_analista;
```

## Boas Práticas

SECURITY_EXPERT compartilha diretrizes:

### 1. Organização

```sql
-- Nomenclatura consistente
CREATE ROLE app_read;  -- Apenas leitura
CREATE ROLE app_write; -- Leitura e escrita
CREATE ROLE app_admin; -- Administração

-- Documentação
COMMENT ON ROLE app_read IS 'Role para acesso somente leitura';
```

### 2. Auditoria

```sql
-- Visualizar membros de role
SELECT r.rolname, m.member, u.usename
FROM pg_roles r
LEFT JOIN pg_auth_members m ON r.oid = m.roleid
LEFT JOIN pg_user u ON m.member = u.usesysid;

-- Verificar privilégios
SELECT * FROM information_schema.role_table_grants
WHERE grantee = 'role_name';
```

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE ROLES:                      ║
║  □ Nomenclatura padronizada?              ║
║  □ Hierarquia bem definida?               ║
║  □ Privilégios mínimos necessários?       ║
║  □ Documentação atualizada?               ║
║  □ Auditoria configurada?                 ║
╚════════════════════════════════════════════╝
```

## Cenários Comuns

IMPLEMENTATION_EXPERT apresenta soluções práticas:

### 1. Setup de Desenvolvimento

```sql
-- Role para desenvolvedores
CREATE ROLE dev_team;
GRANT CONNECT ON DATABASE dev_db TO dev_team;
GRANT USAGE, CREATE ON SCHEMA public TO dev_team;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO dev_team;
```

### 2. Setup de Produção

```sql
-- Role para aplicação em produção
CREATE ROLE prod_app WITH LOGIN;
GRANT CONNECT ON DATABASE prod_db TO prod_app;
GRANT USAGE ON SCHEMA public TO prod_app;
GRANT SELECT, INSERT ON tabela_clientes TO prod_app;
GRANT EXECUTE ON FUNCTION proc_negocio TO prod_app;
```

## Troubleshooting

PROBLEM_SOLVER apresenta soluções:

### 1. Diagnóstico

```sql
-- Verificar conflitos
SELECT r.rolname, 
       r.rolsuper, 
       r.rolinherit,
       r.rolcreaterole,
       r.rolcreatedb
FROM pg_roles r
WHERE r.rolname = 'role_problema';

-- Verificar sessões ativas
SELECT pid, usename, application_name
FROM pg_stat_activity
WHERE usename IN (
    SELECT m.member::regrole::text
    FROM pg_roles r
    JOIN pg_auth_members m ON r.oid = m.roleid
    WHERE r.rolname = 'role_problema'
);
```

### 2. Manutenção

```sql
-- Limpar roles não utilizadas
SELECT r.rolname
FROM pg_roles r
LEFT JOIN pg_auth_members m ON r.oid = m.roleid
WHERE m.roleid IS NULL
AND NOT r.rolcanlogin;

-- Atualizar expiração
ALTER ROLE usuario_temporario 
VALID UNTIL '2024-06-30';
```

## Conclusão

ROLE_SPECIALIST conclui: "Um bom gerenciamento de roles é fundamental para segurança e organização do banco de dados."

> **Dica Final**: Revise regularmente as roles e suas permissões, mantendo documentação atualizada e removendo acessos desnecessários.