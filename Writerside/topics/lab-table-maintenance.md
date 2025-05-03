# Laboratório: Manutenção de Tabelas

## Objetivo
Praticar operações de manutenção em tabelas de banco de dados, incluindo otimização de espaço, reorganização de dados e monitoramento de performance.

## Cenário
Você é o DBA responsável por um sistema que está em produção há vários anos. As tabelas cresceram significativamente e começaram a apresentar problemas de performance e fragmentação.

## Setup Inicial

```sql
-- Criar tabela de exemplo com fragmentação
CREATE TABLE lab_maintenance.pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER,
    produto_id INTEGER,
    quantidade INTEGER,
    valor DECIMAL(10,2),
    status VARCHAR(20),
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    data_atualizacao TIMESTAMP
);

-- Inserir dados de exemplo (muitos registros)
INSERT INTO lab_maintenance.pedidos (cliente_id, produto_id, quantidade, valor, status)
SELECT 
    floor(random() * 1000)::int,
    floor(random() * 500)::int,
    floor(random() * 10)::int + 1,
    (random() * 1000)::decimal(10,2),
    (ARRAY['PENDENTE', 'APROVADO', 'ENVIADO', 'ENTREGUE', 'CANCELADO'])[floor(random() * 5 + 1)]
FROM generate_series(1, 500000);

-- Simular atualizações frequentes para criar fragmentação
UPDATE lab_maintenance.pedidos
SET status = 'APROVADO', data_atualizacao = CURRENT_TIMESTAMP
WHERE id % 3 = 0;

UPDATE lab_maintenance.pedidos
SET status = 'ENVIADO', data_atualizacao = CURRENT_TIMESTAMP
WHERE id % 5 = 0;
```

## Exercícios

### 1. Análise de Fragmentação

```sql
-- Verificar estatísticas da tabela
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE relname = 'pedidos';

-- Verificar bloat (inchamento) da tabela
-- Requer extensão pgstattuple
CREATE EXTENSION IF NOT EXISTS pgstattuple;

SELECT * FROM pgstattuple('lab_maintenance.pedidos');
```

### 2. Operações de VACUUM

```sql
-- VACUUM básico
VACUUM lab_maintenance.pedidos;

-- VACUUM FULL (requer lock exclusivo)
VACUUM FULL lab_maintenance.pedidos;

-- VACUUM ANALYZE
VACUUM ANALYZE lab_maintenance.pedidos;

-- Verificar efeito do VACUUM
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    last_vacuum
FROM pg_stat_user_tables
WHERE relname = 'pedidos';
```

### 3. Reconstrução de Índices

```sql
-- Criar índices para a tabela
CREATE INDEX idx_pedidos_cliente ON lab_maintenance.pedidos(cliente_id);
CREATE INDEX idx_pedidos_status ON lab_maintenance.pedidos(status);
CREATE INDEX idx_pedidos_data ON lab_maintenance.pedidos(data_pedido);

-- Verificar fragmentação de índices
SELECT
    indexrelname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE relname = 'pedidos';

-- Reconstruir índice
REINDEX INDEX idx_pedidos_cliente;

-- Reconstruir todos os índices da tabela
REINDEX TABLE lab_maintenance.pedidos;
```

### 4. Monitoramento de Espaço

```sql
-- Verificar tamanho da tabela e índices
SELECT
    pg_size_pretty(pg_total_relation_size('lab_maintenance.pedidos')) as total_size,
    pg_size_pretty(pg_relation_size('lab_maintenance.pedidos')) as table_size,
    pg_size_pretty(pg_indexes_size('lab_maintenance.pedidos')) as indexes_size;

-- Verificar espaço por schema
SELECT
    schemaname,
    pg_size_pretty(sum(pg_relation_size(schemaname || '.' || relname))) as size
FROM pg_stat_user_tables
GROUP BY schemaname
ORDER BY sum(pg_relation_size(schemaname || '.' || relname)) DESC;
```

## Tarefas Práticas

### Tarefa 1: Otimização de Tabela Grande

1. Crie uma tabela com pelo menos 1 milhão de registros
2. Insira, atualize e exclua registros para criar fragmentação
3. Analise o estado da tabela
4. Aplique as operações de manutenção apropriadas
5. Compare o desempenho antes e depois

### Tarefa 2: Automação de Manutenção

```sql
-- Criar função para manutenção automática
CREATE OR REPLACE FUNCTION lab_maintenance.manter_tabela(tabela text)
RETURNS void AS $$
BEGIN
    EXECUTE 'VACUUM ANALYZE ' || tabela;
    RAISE NOTICE 'Manutenção concluída para %', tabela;
END;
$$ LANGUAGE plpgsql;

-- Agendar manutenção (simulação)
SELECT lab_maintenance.manter_tabela('lab_maintenance.pedidos');
```

## Boas Práticas

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE MANUTENÇÃO:                 ║
║  □ Monitorar crescimento de tabelas       ║
║  □ Configurar autovacuum apropriadamente  ║
║  □ Agendar VACUUM ANALYZE regularmente    ║
║  □ Reconstruir índices periodicamente     ║
║  □ Monitorar bloat de tabelas e índices   ║
║  □ Manter estatísticas atualizadas        ║
╚════════════════════════════════════════════╝
```

## Considerações de Performance

1. **Janelas de Manutenção**
   - Agendar operações pesadas para períodos de baixo uso
   - Considerar impacto em sistemas 24/7
   - Balancear frequência e duração

2. **Configurações do Autovacuum**
   ```sql
   -- Verificar configurações atuais
   SHOW autovacuum_vacuum_threshold;
   SHOW autovacuum_analyze_threshold;
   SHOW autovacuum_vacuum_scale_factor;
   SHOW autovacuum_analyze_scale_factor;
   ```

3. **Monitoramento Contínuo**
   - Implementar alertas para tabelas problemáticas
   - Acompanhar tendências de crescimento
   - Identificar padrões de acesso

## Recursos Adicionais

- [PostgreSQL VACUUM Documentation](https://www.postgresql.org/docs/current/sql-vacuum.html)
- [PostgreSQL Index Maintenance](https://www.postgresql.org/docs/current/sql-reindex.html)
- [Monitoring Database Activity](https://www.postgresql.org/docs/current/monitoring-stats.html)

---

> **Nota**: Este laboratório deve ser executado em um ambiente de teste para evitar impactos em dados de produção.