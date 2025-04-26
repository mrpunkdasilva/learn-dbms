# REVOKE: Revogando Privilégios

```ascii
╔══════════════════════════════════════════════════════════════╗
║  SECURITY_MASTER >> Gerenciamento de Revogações            ║
║  INSTRUTOR: REVOKE_SPECIALIST                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Sintaxe Básica

REVOKE_SPECIALIST explica: "O comando REVOKE é essencial para manter a segurança e controle de acesso!"

```sql
-- Sintaxe básica
REVOKE privilégio ON objeto FROM destinatário;

-- Múltiplos privilégios
REVOKE SELECT, INSERT, UPDATE ON tabela FROM usuario;

-- Todos os privilégios
REVOKE ALL PRIVILEGES ON tabela FROM usuario;
```

## Tipos de Revogação

SECURITY_MASTER apresenta os principais cenários:

### 1. Revogação de Tabela

```sql
-- Privilégios básicos
REVOKE SELECT ON clientes FROM analista;
REVOKE INSERT, UPDATE ON produtos FROM vendedor;
REVOKE DELETE ON temp_dados FROM usuario;

-- Privilégios em colunas específicas
REVOKE SELECT (senha, dados_sensíveis) ON usuarios FROM suporte;
REVOKE UPDATE (salario) ON funcionarios FROM gerente;
```

### 2. Revogação de Schema

```sql
-- Acesso ao schema
REVOKE USAGE ON SCHEMA confidencial FROM usuario;
REVOKE CREATE ON SCHEMA public FROM dev_team;

-- Todos os objetos do schema
REVOKE SELECT ON ALL TABLES IN SCHEMA dados_sensiveis FROM grupo;
```

### 3. Revogação de Database

```sql
-- Privilégios de banco
REVOKE CONNECT ON DATABASE prod_db FROM test_user;
REVOKE ALL PRIVILEGES ON DATABASE analytics FROM old_user;
REVOKE TEMPORARY ON DATABASE temp_db FROM temp_user;
```

## Cenários Comuns

IMPLEMENTATION_EXPERT demonstra casos práticos:

### 1. Desativação de Usuário

```sql
-- Remoção completa de acessos
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM usuario_desativado;
REVOKE ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public FROM usuario_desativado;
REVOKE ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public FROM usuario_desativado;
REVOKE USAGE ON SCHEMA public FROM usuario_desativado;
```

### 2. Ajuste de Permissões

```sql
-- Refinamento de acesso
REVOKE INSERT, UPDATE, DELETE ON dados_financeiros FROM analista;
GRANT SELECT ON dados_financeiros TO analista;
```

## Boas Práticas

SECURITY_GUARDIAN compartilha diretrizes:

### 1. Revogação Segura

```sql
-- Verificar antes de revogar
SELECT * FROM information_schema.role_table_grants
WHERE grantee = 'usuario_alvo';

-- Revogar com cautela
BEGIN;
    REVOKE critical_role FROM usuario;
    -- Verificar impacto
    SELECT current_user, session_user;
COMMIT;
```

### 2. Auditoria de Revogações

```sql
-- Logging de revogações
CREATE TABLE revoke_audit_log (
    id SERIAL PRIMARY KEY,
    revoked_from VARCHAR(100),
    revoked_privilege VARCHAR(50),
    revoked_object VARCHAR(100),
    revoked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    revoked_by VARCHAR(100)
);

-- Trigger para auditoria
CREATE TRIGGER log_revokes
AFTER REVOKE ON *.* 
FOR EACH STATEMENT 
EXECUTE FUNCTION log_revoke_changes();
```

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE REVOGAÇÃO:                  ║
║  □ Impacto analisado?                     ║
║  □ Backup de permissões feito?            ║
║  □ Usuários afetados notificados?         ║
║  □ Auditoria configurada?                 ║
║  □ Plano de contingência pronto?          ║
╚════════════════════════════════════════════╝
```

## Troubleshooting

PROBLEM_SOLVER apresenta soluções:

### 1. Verificação de Impacto

```sql
-- Analisar dependências
SELECT r.rolname, m.member
FROM pg_roles r
JOIN pg_auth_members m ON r.oid = m.roleid
WHERE r.rolname = 'role_a_revogar';

-- Verificar sessões ativas
SELECT pid, usename, application_name
FROM pg_stat_activity
WHERE usename = 'usuario_alvo';
```

### 2. Recuperação de Acesso

```sql
-- Backup de privilégios
CREATE TABLE privilege_backup AS
SELECT * FROM information_schema.role_table_grants
WHERE grantee = 'usuario_alvo';

-- Restauração de privilégios
-- (Gerar scripts de GRANT baseados no backup)
```

## Conclusão

REVOKE_SPECIALIST conclui: "REVOKE é uma operação crítica para segurança - use com planejamento e cautela."

> **Dica Final**: Sempre documente as revogações realizadas e mantenha um histórico das alterações de permissões para referência futura.