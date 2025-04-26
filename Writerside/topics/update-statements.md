# UPDATE: Modificando Dados

```ascii
╔══════════════════════════════════════════════════════════════╗
║  DATA_MASTER >> Atualização de Dados                       ║
║  INSTRUTOR: UPDATE_SPECIALIST                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Fundamentos do UPDATE

UPDATE_SPECIALIST explica: "O comando UPDATE permite modificar dados existentes de forma precisa e controlada."

### Sintaxe Básica

```sql
-- Atualização simples
UPDATE tabela
SET coluna = valor
WHERE condição;

-- Múltiplas colunas
UPDATE tabela
SET 
    coluna1 = valor1,
    coluna2 = valor2
WHERE condição;
```

## Formas de Atualização

### 1. Valores Diretos

```sql
-- Atualização simples
UPDATE produtos
SET 
    preco = 999.99,
    ultima_atualizacao = CURRENT_TIMESTAMP
WHERE id = 1001;

-- Com expressões
UPDATE produtos
SET preco = preco * 1.1
WHERE categoria = 'Eletrônicos';
```

### 2. Atualização com JOIN

DATA_SYNC_EXPERT demonstra: "Atualize baseado em outras tabelas!"

```sql
-- Atualização com JOIN
UPDATE pedidos p
SET status = 'aprovado'
FROM clientes c
WHERE p.cliente_id = c.id
AND c.categoria = 'VIP';

-- Atualização múltipla
UPDATE produtos p
SET 
    preco = p.preco * t.fator_ajuste,
    atualizado_em = CURRENT_TIMESTAMP
FROM tabela_ajustes t
WHERE p.categoria = t.categoria;
```

### 3. Atualizações Condicionais

```sql
-- Usando CASE
UPDATE funcionarios
SET salario = CASE
    WHEN tempo_servico < 2 THEN salario * 1.05
    WHEN tempo_servico < 5 THEN salario * 1.07
    ELSE salario * 1.10
END;

-- Com COALESCE
UPDATE clientes
SET 
    telefone = COALESCE(novo_telefone, telefone),
    email = COALESCE(novo_email, email)
FROM atualizacoes_clientes
WHERE clientes.id = atualizacoes_clientes.cliente_id;
```

## Tratamento de Erros

ERROR_HANDLER apresenta: "Lidando com atualizações seguras"

```sql
-- Com validação
UPDATE produtos
SET preco = novo_preco
WHERE id = produto_id
AND novo_preco > 0
RETURNING id, nome, preco as novo_preco;

-- Dentro de transação
BEGIN;
    UPDATE contas
    SET saldo = saldo - 100
    WHERE id = 1;
    
    UPDATE contas
    SET saldo = saldo + 100
    WHERE id = 2;
    
    -- Verificação
    IF EXISTS (SELECT 1 FROM contas WHERE saldo < 0) THEN
        ROLLBACK;
    ELSE
        COMMIT;
    END IF;
```

## Atualizações em Massa

BULK_UPDATE_MASTER compartilha: "Otimize atualizações grandes!"

```sql
-- Atualização em lotes
UPDATE produtos
SET status = 'inativo'
WHERE id IN (
    SELECT id 
    FROM produtos
    WHERE ultima_venda < CURRENT_DATE - INTERVAL '1 year'
    LIMIT 1000
);

-- Com tabela temporária
CREATE TEMP TABLE updates_temp AS
SELECT id, novo_preco
FROM calculos_precos;

UPDATE produtos p
SET preco = t.novo_preco
FROM updates_temp t
WHERE p.id = t.id;
```

## Boas Práticas

BEST_PRACTICES_SAGE recomenda:

### 1. Performance
- Use índices adequadamente
- Atualize em lotes quando possível
- Evite triggers desnecessários
- Monitore locks e deadlocks

### 2. Integridade
- Valide dados antes da atualização
- Use transações quando necessário
- Mantenha consistência referencial
- Considere impacto em constraints

### 3. Manutenção
- Mantenha logs de alterações
- Documente atualizações críticas
- Implemente rollback strategy
- Monitore impacto em índices

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST UPDATE:                        ║
║  □ WHERE adequado?                       ║
║  □ Dados validados?                      ║
║  □ Transação necessária?                 ║
║  □ Índices considerados?                 ║
║  □ Backup realizado?                     ║
╚════════════════════════════════════════════╝
```

## Exemplos Práticos

### Sistema de E-commerce

```sql
-- Atualização de estoque após venda
UPDATE produtos p
SET 
    estoque = p.estoque - v.quantidade,
    ultima_venda = CURRENT_TIMESTAMP,
    total_vendas = p.total_vendas + 1
FROM vendas v
WHERE p.id = v.produto_id
AND v.status = 'confirmado';

-- Atualização de status de pedidos
UPDATE pedidos p
SET 
    status = 'em_transito',
    atualizado_em = CURRENT_TIMESTAMP,
    tracking_code = e.codigo_rastreio
FROM entregas e
WHERE p.id = e.pedido_id
AND p.status = 'aprovado'
AND e.status = 'coletado';
```

## Troubleshooting Comum

PROBLEM_SOLVER apresenta soluções para problemas frequentes:

1. **Deadlocks em Atualizações**
```sql
-- Identificando locks
SELECT relation::regclass, mode, granted
FROM pg_locks
WHERE NOT granted;

-- Atualizando com timeout
SET lock_timeout = '10s';
UPDATE tabela_concorrida
SET status = 'processado'
WHERE id = 1001;
```

2. **Atualizações Órfãs**
```sql
-- Verificando integridade
UPDATE pedidos p
SET status = 'cancelado'
WHERE EXISTS (
    SELECT 1 
    FROM clientes c
    WHERE p.cliente_id = c.id
    AND c.status = 'inativo'
);
```

## Conclusão

UPDATE_SPECIALIST conclui: "O UPDATE é uma operação poderosa que requer atenção aos detalhes. Sempre valide suas condições e considere o impacto das alterações."

> **Dica Final**: Sempre teste suas atualizações com SELECT primeiro para confirmar o conjunto de registros afetados.