# Laboratório: Evolução de Schema

## Objetivo
Praticar alterações de schema em um banco de dados em produção, aplicando boas práticas de versionamento e migração.

## Cenário
Você é o DBA responsável por um sistema de e-commerce em crescimento que precisa evoluir seu schema para acomodar novos requisitos de negócio.

## Setup Inicial

```sql
-- Schema inicial
CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) NOT NULL,
    estoque INT DEFAULT 0
);

CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INT REFERENCES clientes(id),
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Exercícios

### 1. Adição de Novas Features

```sql
-- TODO: Adicionar suporte para categorias de produtos
-- 1.1 Crie uma tabela de categorias
-- 1.2 Adicione uma coluna categoria_id em produtos
-- 1.3 Estabeleça a foreign key apropriada
```

### 2. Modificação de Estrutura Existente

```sql
-- TODO: Expandir informações de cliente
-- 2.1 Adicione colunas para telefone e endereço
-- 2.2 Torne o telefone único e não nulo
-- 2.3 Migre dados existentes adequadamente
```

### 3. Gestão de Constraints

```sql
-- TODO: Implementar regras de negócio
-- 3.1 Adicione check constraint para preço mínimo
-- 3.2 Crie unique constraint para SKU de produtos
-- 3.3 Implemente trigger para log de alterações
```

## Desafios Extras

### 1. Versionamento

```sql
-- Implemente um sistema de controle de versão de schema
CREATE TABLE schema_version (
    version INT PRIMARY KEY,
    applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    description TEXT
);

-- TODO: Crie scripts de up/down para cada alteração
```

### 2. Zero Downtime

Desenvolva estratégias para:
- Adição de colunas não-nulas
- Renomeação de colunas
- Divisão de tabelas

### 3. Rollback Plan

```sql
-- TODO: Para cada alteração, crie um plano de rollback
-- Exemplo:
BEGIN;
    -- Alterações
    ALTER TABLE produtos ADD COLUMN categoria_id INT;
    
    -- Verificação
    SELECT COUNT(*) FROM produtos WHERE categoria_id IS NULL;
    
    -- Decisão de commit/rollback
    -- COMMIT ou ROLLBACK;
END;
```

## Critérios de Avaliação

1. **Segurança de Dados**
   - Backup antes das alterações
   - Validação de integridade
   - Tratamento de erros

2. **Performance**
   - Tempo de execução
   - Impacto em produção
   - Uso de recursos

3. **Manutenibilidade**
   - Documentação
   - Versionamento
   - Scripts de rollback

## Entrega

Crie um arquivo SQL contendo:
1. Scripts de alteração
2. Scripts de rollback
3. Documentação das mudanças
4. Logs de teste

## Dicas

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE ALTERAÇÕES:                 ║
║  □ Backup realizado?                      ║
║  □ Scripts testados em homologação?       ║
║  □ Plano de rollback preparado?           ║
║  □ Janela de manutenção definida?         ║
║  □ Stakeholders notificados?              ║
╚════════════════════════════════════════════╝
```

## Recursos Adicionais

- [Documentação PostgreSQL sobre ALTER](https://www.postgresql.org/docs/current/sql-alter.html)
- [Práticas de Schema Migration](https://flywaydb.org/documentation/concepts/migrations)
- [Zero Downtime Deployments](https://www.postgresql.org/docs/current/monitoring-stats.html)

## Próximos Passos

1. Implemente monitoramento de performance
2. Automatize testes de migração
3. Crie documentação detalhada
4. Prepare apresentação dos resultados
