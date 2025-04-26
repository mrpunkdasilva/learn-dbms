# MERGE: Sincronizando Dados com Precisão

```ascii
╔══════════════════════════════════════════════════════════════╗
║  DATA_SYNC_MASTER >> Sincronização de Dados                ║
║  INSTRUTOR: MERGE_SPECIALIST                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Fundamentos do MERGE

MERGE_SPECIALIST explica: "O comando MERGE é uma ferramenta poderosa para sincronização de dados, combinando INSERT, UPDATE e DELETE em uma única operação."

### Sintaxe Básica

```sql
MERGE INTO tabela_destino d
USING tabela_origem o
ON (d.id = o.id)
WHEN MATCHED THEN
    UPDATE SET coluna = o.valor
WHEN NOT MATCHED THEN
    INSERT (coluna) VALUES (o.valor);
```

## Padrões de Uso

### 1. Sincronização Completa

```sql
-- Sincronização com todas as ações
MERGE INTO produtos_destino d
USING produtos_origem o
ON (d.codigo = o.codigo)
WHEN MATCHED THEN
    UPDATE SET 
        nome = o.nome,
        preco = o.preco,
        updated_at = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (codigo, nome, preco, created_at)
    VALUES (o.codigo, o.nome, o.preco, CURRENT_TIMESTAMP)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

### 2. Atualização Condicional

```sql
-- Merge com condições específicas
MERGE INTO estoque e
USING novos_dados n
ON (e.produto_id = n.produto_id)
WHEN MATCHED AND e.quantidade != n.quantidade THEN
    UPDATE SET 
        quantidade = n.quantidade,
        ultima_atualizacao = CURRENT_TIMESTAMP
WHEN NOT MATCHED AND n.quantidade > 0 THEN
    INSERT (produto_id, quantidade, ultima_atualizacao)
    VALUES (n.produto_id, n.quantidade, CURRENT_TIMESTAMP);
```

## Casos de Uso

DATA_SYNC_EXPERT apresenta: "Cenários comuns para MERGE"

### 1. Dimensões Slowly Changing

```sql
-- Dimensão Tipo 2
MERGE INTO dim_clientes d
USING stg_clientes s
ON (d.cliente_id = s.cliente_id AND d.atual = true)
WHEN MATCHED AND (
    d.nome != s.nome OR 
    d.endereco != s.endereco
) THEN
    UPDATE SET 
        atual = false,
        data_fim = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (cliente_id, nome, endereco, data_inicio, data_fim, atual)
    VALUES (s.cliente_id, s.nome, s.endereco, CURRENT_TIMESTAMP, NULL, true);
```

### 2. Sincronização de Cache

```sql
-- Atualização de cache
MERGE INTO cache_produtos c
USING produtos_atualizados p
ON (c.produto_id = p.id)
WHEN MATCHED AND c.hash != p.hash THEN
    UPDATE SET 
        dados = p.dados,
        hash = p.hash,
        atualizado_em = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (produto_id, dados, hash, atualizado_em)
    VALUES (p.id, p.dados, p.hash, CURRENT_TIMESTAMP);
```

## Boas Práticas

BEST_PRACTICES_SAGE compartilha:

### 1. Performance
- Use índices apropriados
- Considere o volume de dados
- Faça em lotes quando necessário
- Monitore tempos de execução

### 2. Integridade
- Use transações
- Valide dados antes do merge
- Mantenha consistência
- Implemente logging

### 3. Manutenção
- Documente regras de merge
- Mantenha histórico de execuções
- Implemente recuperação de erros
- Monitore resultados

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST MERGE:                         ║
║  □ Dados validados?                      ║
║  □ Índices verificados?                  ║
║  □ Transação configurada?                ║
║  □ Logging implementado?                 ║
║  □ Recuperação planejada?                ║
╚════════════════════════════════════════════╝
```

## Padrões Avançados

PATTERN_MASTER demonstra: "Técnicas avançadas de MERGE"

### 1. Merge com Staging

```sql
-- Usando tabela temporária
WITH staging AS (
    SELECT * FROM novos_dados
    WHERE data_processamento = CURRENT_DATE
)
MERGE INTO produtos_final p
USING staging s
ON (p.id = s.id)
WHEN MATCHED THEN
    UPDATE SET 
        dados = s.dados,
        atualizado_em = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (id, dados, criado_em)
    VALUES (s.id, s.dados, CURRENT_TIMESTAMP);
```

### 2. Merge com Auditoria

```sql
-- Mantendo histórico de alterações
MERGE INTO produtos p
USING atualizacoes a
ON (p.id = a.id)
WHEN MATCHED THEN
    UPDATE SET 
        nome = a.nome,
        preco = a.preco
    -- Registro de auditoria
    INSERT INTO produtos_audit (
        produto_id, campo, valor_antigo, valor_novo, data_alteracao
    )
    VALUES 
        (p.id, 'nome', p.nome, a.nome, CURRENT_TIMESTAMP),
        (p.id, 'preco', p.preco, a.preco, CURRENT_TIMESTAMP)
WHEN NOT MATCHED THEN
    INSERT (id, nome, preco)
    VALUES (a.id, a.nome, a.preco);
```

## Troubleshooting

ERROR_HANDLER apresenta soluções para problemas comuns:

1. **Conflitos de Chave**
```sql
-- Verificação prévia
SELECT COUNT(*), id 
FROM tabela_origem 
GROUP BY id 
HAVING COUNT(*) > 1;

-- Merge com resolução de duplicatas
MERGE INTO destino d
USING (
    SELECT DISTINCT ON (id) *
    FROM origem
    ORDER BY id, ultima_atualizacao DESC
) o
ON (d.id = o.id)
...
```

2. **Performance**
```sql
-- Merge em lotes
DO $$
DECLARE
    batch_size INTEGER := 1000;
    total_rows INTEGER;
BEGIN
    SELECT COUNT(*) INTO total_rows FROM origem;
    
    FOR i IN 0..total_rows/batch_size LOOP
        MERGE INTO destino d
        USING (
            SELECT *
            FROM origem
            OFFSET i * batch_size
            LIMIT batch_size
        ) o
        ON (d.id = o.id)
        ...;
        
        COMMIT;
    END LOOP;
END $$;
```

## Conclusão

MERGE_SPECIALIST conclui: "O MERGE é uma ferramenta poderosa para sincronização de dados, mas requer planejamento cuidadoso e atenção aos detalhes."

> **Dica Final**: Sempre teste seu MERGE com um conjunto pequeno de dados antes de executar em produção, e mantenha backups atualizados.