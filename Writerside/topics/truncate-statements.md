# TRUNCATE: Limpeza Rápida de Dados

```ascii
╔══════════════════════════════════════════════════════════════╗
║  DATA_CLEANER >> Limpeza Eficiente de Dados               ║
║  INSTRUTOR: TRUNCATE_SPECIALIST                           ║
╚══════════════════════════════════════════════════════════════╝
```

## Fundamentos do TRUNCATE

TRUNCATE_SPECIALIST explica: "TRUNCATE é a forma mais rápida de remover todos os dados de uma tabela!"

```sql
-- Sintaxe básica
TRUNCATE TABLE logs;

-- Múltiplas tabelas
TRUNCATE TABLE 
    temp_data, 
    staging_area, 
    import_buffer;

-- Com RESTART IDENTITY
TRUNCATE TABLE produtos RESTART IDENTITY;

-- Com CASCADE
TRUNCATE TABLE clientes CASCADE;
```

## TRUNCATE vs DELETE

PERFORMANCE_GURU compara: "Entenda quando usar cada um!"

| TRUNCATE | DELETE |
|----------|--------|
| Mais rápido | Mais flexível |
| Não é transacional* | Transacional |
| Reseta sequências | Mantém sequências |
| Não dispara triggers | Dispara triggers |

## Cenários de Uso

DATA_ARCHITECT apresenta: "Situações ideais para TRUNCATE"

### 1. Limpeza de Ambiente

```sql
-- Limpando tabelas temporárias
TRUNCATE TABLE 
    temp_imports,
    staging_area,
    etl_buffer
RESTART IDENTITY;
```

### 2. Reset de Desenvolvimento

```sql
-- Resetando ambiente de testes
TRUNCATE TABLE 
    test_data,
    test_logs,
    test_metrics
CASCADE;
```

## Boas Práticas

SAFETY_MASTER compartilha diretrizes essenciais:

### 1. Segurança
- Faça backup antes
- Verifique dependências
- Use em ambiente correto
- Documente operações

### 2. Performance
- Considere locks
- Avalie impacto
- Planeje janela de execução
- Monitore espaço em disco

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE TRUNCATE:                   ║
║  □ Backup realizado?                      ║
║  □ Ambiente correto?                      ║
║  □ Dependências verificadas?              ║
║  □ Equipe notificada?                     ║
║  □ Janela de manutenção definida?        ║
╚════════════════════════════════════════════╝
```

## Troubleshooting

ERROR_HANDLER apresenta soluções comuns:

### 1. Bloqueios

```sql
-- Verificando locks
SELECT relation::regclass, mode, granted
FROM pg_locks
WHERE relation IN (
    SELECT oid 
    FROM pg_class 
    WHERE relname = 'sua_tabela'
);
```

### 2. Dependências

```sql
-- Verificando referências
SELECT 
    tc.table_schema, 
    tc.table_name, 
    kcu.column_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
    ON tc.constraint_name = kcu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
AND kcu.referenced_table_name = 'sua_tabela';
```

## Padrões de Uso

PATTERN_MASTER apresenta: "Padrões comuns de utilização"

### 1. Limpeza Periódica

```sql
-- Procedure de limpeza
CREATE PROCEDURE limpar_logs()
LANGUAGE SQL
AS $$
    TRUNCATE TABLE 
        system_logs,
        audit_logs,
        error_logs;
$$;
```

### 2. Reset de Ambiente

```sql
-- Script de reset
BEGIN;
    -- Desativa foreign key checks
    SET CONSTRAINTS ALL DEFERRED;
    
    -- Limpa todas as tabelas
    TRUNCATE TABLE 
        tabela1,
        tabela2,
        tabela3
    CASCADE;
    
    -- Reativa constraints
    SET CONSTRAINTS ALL IMMEDIATE;
COMMIT;
```

## Conclusão

TRUNCATE_SPECIALIST conclui: "TRUNCATE é uma ferramenta poderosa para limpeza de dados, mas deve ser usada com conhecimento e cautela."

> **Dica Final**: Sempre tenha um plano de recuperação antes de executar TRUNCATE em tabelas importantes. A operação é rápida, mas irreversível!