# GRANT: Concedendo Privilégios

```ascii
╔══════════════════════════════════════════════════════════════╗
║  PERMISSION_MASTER >> Gerenciamento de Privilégios         ║
║  INSTRUTOR: GRANT_SPECIALIST                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Sintaxe Básica

GRANT_SPECIALIST explica: "O comando GRANT é fundamental para controle de acesso!"

```sql
-- Sintaxe básica
GRANT privilégio ON objeto TO destinatário;

-- Múltiplos privilégios
GRANT SELECT, INSERT, UPDATE ON tabela TO usuario;

-- Todos os privilégios
GRANT ALL PRIVILEGES ON tabela TO admin;
```

## Tipos de Privilégios

PRIVILEGE_MASTER apresenta os principais privilégios:

### 1. Privilégios de Tabela

```sql
-- Privilégios básicos
GRANT SELECT ON clientes TO analista;
GRANT INSERT, UPDATE ON produtos TO vendedor;
GRANT DELETE ON temp_dados TO admin;

-- Privilégios em colunas específicas
GRANT SELECT (nome, email) ON usuarios TO suporte;
GRANT UPDATE (preco, estoque) ON produtos TO gerente;
```

### 2. Privilégios de Schema

```sql
-- Acesso ao schema
GRANT USAGE ON SCHEMA public TO usuario;
GRANT CREATE ON SCHEMA analytics TO dev_team;

-- Todos os objetos do schema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO leitor;
```

### 3. Privilégios de Database

```sql
-- Privilégios de banco
GRANT CONNECT ON DATABASE app_db TO app_user;
GRANT TEMPORARY ON DATABASE temp_db TO etl_user;
GRANT ALL PRIVILEGES ON DATABASE prod_db TO admin;
```

## Cenários Comuns

IMPLEMENTATION_EXPERT demonstra casos práticos:

### 1. Setup de Aplicação

```sql
-- Usuário de aplicação
GRANT CONNECT ON DATABASE app_db TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE ON tabela_clientes TO app_user;
GRANT EXECUTE ON FUNCTION proc_negocio TO app_user;
```

### 2. Setup de Analista

```sql
-- Analista de dados
GRANT CONNECT ON DATABASE analytics_db TO analista;
GRANT USAGE ON SCHEMA relatorios TO analista;
GRANT SELECT ON ALL TABLES IN SCHEMA relatorios TO analista;
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA relatorios TO analista;
```

## Boas Práticas

SECURITY_GUARDIAN compartilha diretrizes:

### 1. Princípio do Menor Privilégio

```sql
-- Privilégios específicos
GRANT SELECT ON vw_relatorio TO analista;
GRANT INSERT ON log_acesso TO app_user;

-- Evitar privilégios excessivos
REVOKE ALL ON ALL TABLES IN SCHEMA public FROM PUBLIC;
```

### 2. Gerenciamento via Roles

```sql
-- Criar e configurar role
CREATE ROLE vendedores;
GRANT SELECT, INSERT ON pedidos TO vendedores;
GRANT UPDATE (status) ON pedidos TO vendedores;

-- Atribuir role a usuários
GRANT vendedores TO usuario1, usuario2;
```

### 3. Auditoria

```sql
-- Visualizar privilégios
SELECT grantee, privilege_type, table_name
FROM information_schema.role_table_grants
WHERE table_schema = 'public';

-- Logging de alterações
CREATE TRIGGER audit_grants
AFTER GRANT ON *.* 
FOR EACH STATEMENT 
EXECUTE FUNCTION log_grant_changes();
```

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE CONCESSÃO:                  ║
║  □ Privilégios mínimos necessários?       ║
║  □ Role apropriada existe?                ║
║  □ Escopo bem definido?                   ║
║  □ Auditoria configurada?                 ║
║  □ Documentação atualizada?               ║
╚════════════════════════════════════════════╝
```

## Troubleshooting

PROBLEM_SOLVER apresenta soluções:

### 1. Verificação de Privilégios

```sql
-- Verificar privilégios concedidos
SELECT * FROM information_schema.table_privileges
WHERE grantee = 'usuario';

-- Verificar roles do usuário
SELECT r.rolname
FROM pg_roles r
JOIN pg_auth_members m ON m.roleid = r.oid
WHERE m.member = 'usuario'::regrole;
```

### 2. Resolução de Problemas

```sql
-- Revogar e reconceder privilégios
REVOKE ALL PRIVILEGES ON tabela FROM usuario;
GRANT SELECT, INSERT ON tabela TO usuario;

-- Verificar conflitos
SELECT * FROM information_schema.role_table_grants
WHERE table_name = 'tabela_problema';
```

## Conclusão

GRANT_SPECIALIST conclui: "GRANT é uma ferramenta poderosa para segurança. Use-a com sabedoria e mantenha documentação clara dos privilégios concedidos."

> **Dica Final**: Revise periodicamente os privilégios concedidos e mantenha um registro de todas as alterações de permissões para auditoria futura.