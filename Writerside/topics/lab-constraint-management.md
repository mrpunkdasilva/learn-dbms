# Laboratório: Gerenciamento de Constraints

## Objetivo
Praticar a implementação e gerenciamento de diferentes tipos de constraints no PostgreSQL, garantindo a integridade dos dados e regras de negócio.

## Ambiente de Teste

```sql
-- Schema para o laboratório
CREATE SCHEMA lab_constraints;

-- Tabelas para prática
CREATE TABLE lab_constraints.produtos (
    id SERIAL PRIMARY KEY,
    codigo VARCHAR(20) UNIQUE,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2),
    categoria_id INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE lab_constraints.categorias (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(50) UNIQUE,
    ativa BOOLEAN DEFAULT true
);
```

## Exercícios Práticos

### 1. Constraints Básicas

```sql
-- Primary Key
ALTER TABLE lab_constraints.produtos
    ADD CONSTRAINT pk_produtos PRIMARY KEY (id);

-- Unique Constraint
ALTER TABLE lab_constraints.produtos
    ADD CONSTRAINT uk_produtos_codigo UNIQUE (codigo);

-- Not Null
ALTER TABLE lab_constraints.produtos
    ALTER COLUMN nome SET NOT NULL;
```

### 2. Foreign Keys

```sql
-- Adicionar Foreign Key com diferentes ações
ALTER TABLE lab_constraints.produtos
    ADD CONSTRAINT fk_produtos_categoria
    FOREIGN KEY (categoria_id)
    REFERENCES lab_constraints.categorias(id)
    ON DELETE RESTRICT
    ON UPDATE CASCADE;
```

### 3. Check Constraints

```sql
-- Validação de preço
ALTER TABLE lab_constraints.produtos
    ADD CONSTRAINT ck_produtos_preco
    CHECK (preco > 0);

-- Validação de código
ALTER TABLE lab_constraints.produtos
    ADD CONSTRAINT ck_produtos_codigo
    CHECK (codigo ~ '^[A-Z]{2}\d{4}$');
```

### 4. Exclusion Constraints

```sql
-- Criar extensão btree_gist se necessário
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Exemplo de constraint de exclusão
CREATE TABLE lab_constraints.eventos (
    id SERIAL PRIMARY KEY,
    sala_id INTEGER,
    periodo tsrange,
    EXCLUDE USING gist (
        sala_id WITH =,
        periodo WITH &&
    )
);
```

## Tarefas Práticas

### Tarefa 1: Implementação de Regras de Negócio

```sql
-- Regra: Produto não pode ser deletado se tiver vendas
CREATE TABLE lab_constraints.vendas (
    id SERIAL PRIMARY KEY,
    produto_id INTEGER,
    quantidade INTEGER,
    CONSTRAINT fk_vendas_produto
    FOREIGN KEY (produto_id)
    REFERENCES lab_constraints.produtos(id)
    ON DELETE RESTRICT
);

-- Regra: Quantidade deve ser positiva
ALTER TABLE lab_constraints.vendas
    ADD CONSTRAINT ck_vendas_quantidade
    CHECK (quantidade > 0);
```

### Tarefa 2: Constraints Dinâmicas

```sql
-- Função para validação customizada
CREATE OR REPLACE FUNCTION lab_constraints.validar_preco()
RETURNS trigger AS $$
BEGIN
    IF NEW.preco < (
        SELECT MIN(preco) 
        FROM lab_constraints.produtos 
        WHERE categoria_id = NEW.categoria_id
    ) * 0.5 THEN
        RAISE EXCEPTION 'Preço muito baixo para a categoria';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger para aplicar validação
CREATE TRIGGER trg_validar_preco
    BEFORE INSERT OR UPDATE ON lab_constraints.produtos
    FOR EACH ROW
    EXECUTE FUNCTION lab_constraints.validar_preco();
```

### Tarefa 3: Gerenciamento de Constraints

```sql
-- Desabilitar temporariamente
ALTER TABLE lab_constraints.produtos
    DISABLE TRIGGER trg_validar_preco;

-- Reabilitar
ALTER TABLE lab_constraints.produtos
    ENABLE TRIGGER trg_validar_preco;

-- Remover constraint
ALTER TABLE lab_constraints.produtos
    DROP CONSTRAINT IF EXISTS ck_produtos_preco;
```

## Verificação de Constraints

```sql
-- Função para listar todas as constraints
CREATE OR REPLACE FUNCTION lab_constraints.listar_constraints()
RETURNS TABLE (
    tabela VARCHAR,
    constraint_name VARCHAR,
    constraint_type VARCHAR,
    definition TEXT
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        tc.table_schema || '.' || tc.table_name::VARCHAR,
        tc.constraint_name::VARCHAR,
        tc.constraint_type::VARCHAR,
        pg_get_constraintdef(pgc.oid)::TEXT
    FROM information_schema.table_constraints tc
    JOIN pg_constraint pgc 
        ON tc.constraint_name = pgc.conname
    WHERE tc.table_schema = 'lab_constraints'
    ORDER BY tc.table_name, tc.constraint_name;
END;
$$ LANGUAGE plpgsql;
```

## Boas Práticas

```ascii
╔════════════════════════════════════════════╗
║  DIRETRIZES DE CONSTRAINTS:               ║
║  ✓ Nomes descritivos                      ║
║  ✓ Documentação clara                     ║
║  ✓ Performance considerada                ║
║  ✓ Validações apropriadas                 ║
║  ✓ Manutenção planejada                   ║
╚════════════════════════════════════════════╝
```

## Troubleshooting

### Problemas Comuns

1. **Violação de Constraint**
   ```sql
   -- Verificar dados violando a constraint
   SELECT * FROM lab_constraints.produtos
   WHERE preco <= 0;
   ```

2. **Deadlocks em Foreign Keys**
   ```sql
   -- Ajustar isolamento
   SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
   ```

3. **Performance**
   ```sql
   -- Analisar impacto
   EXPLAIN ANALYZE
   SELECT * FROM lab_constraints.produtos
   WHERE codigo = 'AB1234';
   ```

## Scripts de Manutenção

```sql
-- Verificar integridade
CREATE OR REPLACE PROCEDURE lab_constraints.verificar_integridade()
AS $$
DECLARE
    v_constraint RECORD;
BEGIN
    FOR v_constraint IN (
        SELECT * FROM lab_constraints.listar_constraints()
    ) LOOP
        EXECUTE 'VALIDATE CONSTRAINT ' || v_constraint.constraint_name ||
                ' ON ' || v_constraint.tabela;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

## Critérios de Avaliação

1. **Implementação**
   - Constraints corretamente definidas
   - Regras de negócio implementadas
   - Validações funcionando

2. **Performance**
   - Impacto aceitável
   - Índices apropriados
   - Otimizações aplicadas

3. **Manutenibilidade**
   - Documentação clara
   - Nomes consistentes
   - Scripts de manutenção

## Próximos Passos

1. Explorar constraints avançadas
2. Implementar casos complexos
3. Otimizar performance
4. Automatizar testes

## Recursos Adicionais

- [PostgreSQL Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [Check Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-CHECK-CONSTRAINTS)
- [Exclusion Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-EXCLUSION)

---

> **Nota**: Este laboratório deve ser executado em um ambiente de teste para evitar impactos em dados de produção.