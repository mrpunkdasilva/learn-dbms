# DELETE: Removendo Dados com Precisão

```ascii
╔══════════════════════════════════════════════════════════════╗
║  DATA_MASTER >> Remoção de Dados                           ║
║  INSTRUTOR: DELETE_SPECIALIST                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Fundamentos do DELETE

DELETE_SPECIALIST explica: "O comando DELETE permite remover registros de forma precisa e controlada."

### Sintaxe Básica

```sql
-- Remoção simples
DELETE FROM tabela
WHERE condição;

-- Remoção com RETURNING
DELETE FROM produtos
WHERE estoque = 0
RETURNING id, nome;
```

## Formas de Deleção

### 1. Deleção Simples

```sql
-- Remoção por ID
DELETE FROM pedidos
WHERE id = 1001;

-- Remoção por condição
DELETE FROM produtos
WHERE validade < CURRENT_DATE;
```

### 2. Deleção com JOIN

DATA_SYNC_EXPERT demonstra: "Delete registros baseado em outras tabelas!"

```sql
-- Deleção com JOIN
DELETE FROM pedidos p
USING clientes c
WHERE p.cliente_id = c.id
AND c.status = 'inativo';

-- Deleção múltipla
DELETE FROM produtos
WHERE categoria_id IN (
    SELECT id 
    FROM categorias 
    WHERE descontinuada = true
);
```

### 3. Deleção Condicional

```sql
-- Usando subquery
DELETE FROM logs
WHERE created_at < (
    SELECT CURRENT_TIMESTAMP - INTERVAL '30 days'
);

-- Com EXISTS
DELETE FROM carrinho
WHERE EXISTS (
    SELECT 1 
    FROM produtos p
    WHERE carrinho.produto_id = p.id
    AND p.descontinuado = true
);
```

## Deleção em Massa

BULK_DELETE_MASTER compartilha: "Otimize deleções grandes!"

```sql
-- Deleção em lotes
DO $$
DECLARE
    batch_size INTEGER := 1000;
BEGIN
    WHILE EXISTS (
        SELECT 1 
        FROM logs 
        WHERE created_at < CURRENT_DATE - INTERVAL '1 year'
    ) LOOP
        DELETE FROM logs
        WHERE id IN (
            SELECT id 
            FROM logs 
            WHERE created_at < CURRENT_DATE - INTERVAL '1 year'
            LIMIT batch_size
        );
        COMMIT;
    END LOOP;
END $$;
```

## Soft Delete

ARCHITECTURE_SAGE apresenta: "Às vezes, é melhor não deletar de verdade!"

```sql
-- Implementando soft delete
ALTER TABLE produtos
ADD COLUMN deleted_at TIMESTAMP,
ADD COLUMN is_active BOOLEAN DEFAULT true;

-- Realizando soft delete
UPDATE produtos
SET 
    deleted_at = CURRENT_TIMESTAMP,
    is_active = false
WHERE id = 1001;

-- Consultando apenas ativos
SELECT * FROM produtos
WHERE is_active = true
OR is_active IS NULL;
```

## Boas Práticas

BEST_PRACTICES_SAGE recomenda:

### 1. Performance
- Use índices adequadamente
- Delete em lotes quando necessário
- Considere TRUNCATE para limpeza total
- Monitore locks e deadlocks

### 2. Integridade
- Verifique constraints
- Use transações quando necessário
- Considere impacto em dados relacionados
- Mantenha consistência referencial

### 3. Segurança
- Faça backup antes de deleções grandes
- Valide condições WHERE cuidadosamente
- Implemente soft delete quando apropriado
- Mantenha logs de deleções importantes

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DELETE:                        ║
║  □ WHERE adequado?                       ║
║  □ Impacto analisado?                    ║
║  □ Transação necessária?                 ║
║  □ Backup realizado?                     ║
║  □ Soft delete considerado?              ║
╚════════════════════════════════════════════╝
```

## Exemplos Práticos

### Sistema de E-commerce

```sql
-- Limpeza de carrinhos abandonados
DELETE FROM carrinhos
WHERE ultima_atualizacao < CURRENT_TIMESTAMP - INTERVAL '24 hours'
AND status = 'pendente';

-- Remoção de produtos descontinuados
DELETE FROM produtos p
WHERE NOT EXISTS (
    SELECT 1 
    FROM pedidos_itens pi
    WHERE pi.produto_id = p.id
    AND pi.created_at > CURRENT_DATE - INTERVAL '6 months'
)
AND p.estoque = 0;
```

## Troubleshooting Comum

PROBLEM_SOLVER apresenta soluções para problemas frequentes:

1. **Violação de Constraint**
```sql
-- Verificando dependências
SELECT 
    tc.table_schema, 
    tc.table_name, 
    kcu.column_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
    ON tc.constraint_name = kcu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
AND kcu.referenced_table_name = 'sua_tabela';

-- Deleção segura
BEGIN;
    DELETE FROM tabela_dependente
    WHERE chave_estrangeira IN (
        SELECT id FROM tabela_principal WHERE status = 'inativo'
    );
    
    DELETE FROM tabela_principal
    WHERE status = 'inativo';
COMMIT;
```

2. **Recuperação de Dados**
```sql
-- Backup antes de deletar
CREATE TABLE deleted_records AS
SELECT * FROM tabela
WHERE status = 'para_deletar';

-- Então execute a deleção
DELETE FROM tabela
WHERE status = 'para_deletar';
```

## Conclusão

DELETE_SPECIALIST conclui: "O DELETE é uma operação irreversível - use com cautela e sempre tenha um plano de backup."

> **Dica Final**: Sempre teste suas condições de DELETE com um SELECT primeiro para confirmar exatamente quais registros serão afetados.