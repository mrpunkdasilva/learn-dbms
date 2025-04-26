# DROP Statements: Removendo Objetos do Banco de Dados

```ascii
╔══════════════════════════════════════════════════════════════╗
║  CLEANUP_MASTER >> Remoção de Objetos                      ║
║  INSTRUTOR: DATABASE_CLEANER                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Fundamentos do DROP

DATABASE_CLEANER alerta: "O comando DROP é poderoso e irreversível. Use com extrema cautela!"

## DROP TABLE

SCHEMA_CLEANER apresenta: "Vamos explorar como remover tabelas de forma segura."

```sql
-- Remoção simples
DROP TABLE IF EXISTS produtos;

-- Remoção com CASCADE
DROP TABLE clientes CASCADE;

-- Remoção múltipla
DROP TABLE IF EXISTS 
    temp_logs, 
    old_backups, 
    test_data;
```

## DROP DATABASE

DATABASE_MASTER adverte: "Este é o comando mais perigoso - use com extrema cautela!"

```sql
-- Removendo banco de dados
DROP DATABASE IF EXISTS test_db;

-- Forçando desconexão de usuários
SELECT pg_terminate_backend(pid) 
FROM pg_stat_activity 
WHERE datname = 'test_db';
```

## DROP INDEX

INDEX_MASTER explica: "Índices desnecessários podem prejudicar a performance."

```sql
-- Removendo índice
DROP INDEX IF EXISTS idx_produtos_nome;

-- Removendo múltiplos índices
DROP INDEX idx_temp1, idx_temp2;

-- Removendo índice concorrentemente
DROP INDEX CONCURRENTLY idx_logs;
```

## DROP CONSTRAINT

CONSTRAINT_GUARDIAN orienta: "Remova constraints com cuidado para manter a integridade."

```sql
-- Removendo constraint
ALTER TABLE pedidos
DROP CONSTRAINT fk_cliente;

-- Removendo PRIMARY KEY
ALTER TABLE produtos
DROP CONSTRAINT produtos_pkey;

-- Removendo UNIQUE constraint
ALTER TABLE usuarios
DROP CONSTRAINT uq_email;
```

## Cenários Práticos

PRACTICAL_MASTER apresenta: "Vamos ver situações reais de limpeza de banco."

### Limpeza de Ambiente de Teste

```sql
-- Removendo objetos temporários
DROP TABLE IF EXISTS temp_import;
DROP INDEX IF EXISTS idx_temp_search;
DROP VIEW IF EXISTS vw_test_report;

-- Removendo schema de teste
DROP SCHEMA test_env CASCADE;
```

### Refatoração de Sistema

```sql
-- Removendo estruturas antigas
DROP TABLE IF EXISTS legacy_customers CASCADE;
DROP VIEW IF EXISTS old_reports;
DROP TRIGGER audit_trigger ON transactions;
```

## Boas Práticas

SAFETY_GUARDIAN compartilha diretrizes essenciais:

### 1. Segurança
- Sempre faça backup antes
- Use IF EXISTS para evitar erros
- Verifique dependências
- Documente todas as remoções

### 2. Performance
- Considere o momento adequado
- Avalie impacto em outros objetos
- Use CONCURRENTLY quando possível
- Monitore locks e conexões

### 3. Manutenção
- Mantenha scripts de recreação
- Atualize documentação
- Comunique alterações
- Mantenha histórico de mudanças

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE REMOÇÃO:                    ║
║  □ Backup realizado?                      ║
║  □ Dependências verificadas?              ║
║  □ Scripts de recreação prontos?          ║
║  □ Equipe notificada?                     ║
║  □ Janela de manutenção definida?        ║
║  □ Impacto analisado?                     ║
╚════════════════════════════════════════════╝
```

## Troubleshooting

ERROR_HANDLER apresenta soluções para problemas comuns:

### 1. Objetos com Dependências

```sql
-- Verificando dependências
SELECT DISTINCT dependent_ns.nspname as dependent_schema,
                dependent_view.relname as dependent_view
FROM pg_depend 
JOIN pg_class dependent_view ON dependent_view.oid = pg_depend.objid
JOIN pg_namespace dependent_ns ON dependent_ns.oid = dependent_view.relnamespace
WHERE refobjid = 'sua_tabela'::regclass;
```

### 2. Objetos Bloqueados

```sql
-- Identificando bloqueios
SELECT blocked_locks.pid AS blocked_pid,
       blocking_locks.pid AS blocking_pid
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
WHERE NOT blocked_locks.granted;
```

## Conclusão

DATABASE_CLEANER conclui: "O DROP é uma ferramenta poderosa para manutenção do banco de dados, mas deve ser usado com responsabilidade e planejamento adequado."

> **Dica Final**: Sempre mantenha scripts de recriação para objetos importantes que precisam ser removidos. Em caso de necessidade, você poderá restaurá-los facilmente.