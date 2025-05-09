# Exercícios Básicos de DCL

## Exercício 1: Concessão de Privilégios Básicos

### Descrição
Pratique a concessão de privilégios básicos para usuários em diferentes tabelas.

### Requisitos
- Conceder privilégios de SELECT
- Conceder privilégios de INSERT/UPDATE
- Conceder privilégios de DELETE
- Verificar privilégios concedidos

### Solução
```sql
-- Conceder privilégio de SELECT
GRANT SELECT ON aplicacao.produtos TO app_user;

-- Conceder privilégios de INSERT e UPDATE
GRANT INSERT, UPDATE ON aplicacao.pedidos TO app_user;

-- Conceder privilégio de DELETE com restrição
GRANT DELETE ON aplicacao.pedidos TO admin_user;

-- Verificar privilégios concedidos
SELECT grantee, table_schema, table_name, privilege_type
FROM information_schema.table_privileges
WHERE grantee = 'app_user';
```

## Exercício 2: Revogação de Privilégios

### Descrição
Pratique a revogação de privilégios previamente concedidos.

### Requisitos
- Revogar privilégios específicos
- Revogar todos os privilégios
- Revogar privilégios de múltiplos objetos
- Verificar após revogação

### Solução
```sql
-- Revogar privilégio específico
REVOKE INSERT ON aplicacao.pedidos FROM app_user;

-- Revogar todos os privilégios de uma tabela
REVOKE ALL PRIVILEGES ON aplicacao.clientes FROM app_user;

-- Revogar privilégios de múltiplas tabelas
REVOKE SELECT ON aplicacao.produtos, aplicacao.pedidos FROM app_user;

-- Verificar após revogação
SELECT grantee, table_schema, table_name, privilege_type
FROM information_schema.table_privileges
WHERE grantee = 'app_user';
```

## Exercício 3: Privilégios em Nível de Schema

### Descrição
Pratique a concessão e revogação de privilégios em nível de schema.

### Requisitos
- Conceder USAGE em schema
- Conceder privilégios em todas as tabelas
- Conceder privilégios em tabelas futuras
- Revogar privilégios em nível de schema

### Solução
```sql
-- Conceder USAGE em schema
GRANT USAGE ON SCHEMA relatorios TO relatorio_user;

-- Conceder privilégios em todas as tabelas existentes
GRANT SELECT ON ALL TABLES IN SCHEMA relatorios TO relatorio_user;

-- Conceder privilégios em tabelas futuras
ALTER DEFAULT PRIVILEGES IN SCHEMA relatorios
GRANT SELECT ON TABLES TO relatorio_user;

-- Revogar privilégios em nível de schema
REVOKE ALL PRIVILEGES ON SCHEMA administracao FROM relatorio_user;
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA administracao FROM relatorio_user;
```

## Exercício 4: Criação e Gerenciamento de Roles

### Descrição
Pratique a criação e gerenciamento básico de roles.

### Requisitos
- Criar roles para diferentes funções
- Conceder privilégios às roles
- Atribuir roles a usuários
- Revogar roles de usuários

### Solução
```sql
-- Criar roles para diferentes funções
CREATE ROLE vendas_role;
CREATE ROLE estoque_role;
CREATE ROLE relatorios_role;

-- Conceder privilégios às roles
GRANT SELECT, INSERT, UPDATE ON aplicacao.pedidos TO vendas_role;
GRANT SELECT, UPDATE ON aplicacao.produtos TO estoque_role;
GRANT SELECT ON ALL TABLES IN SCHEMA relatorios TO relatorios_role;

-- Atribuir roles a usuários
GRANT vendas_role TO app_user;
GRANT estoque_role TO app_user;
GRANT relatorios_role TO relatorio_user;

-- Revogar roles de usuários
REVOKE estoque_role FROM app_user;
```

## Exercício 5: Privilégios em Nível de Coluna

### Descrição
Pratique a concessão de privilégios em nível de coluna.

### Requisitos
- Conceder SELECT em colunas específicas
- Conceder UPDATE em colunas específicas
- Revogar privilégios em nível de coluna
- Verificar privilégios em nível de coluna

### Solução
```sql
-- Conceder SELECT em colunas específicas
GRANT SELECT (nome, email) ON aplicacao.clientes TO app_user;

-- Conceder UPDATE em colunas específicas
GRANT UPDATE (estoque) ON aplicacao.produtos TO app_user;

-- Revogar privilégios em nível de coluna
REVOKE UPDATE (preco) ON aplicacao.produtos FROM app_user;

-- Verificar privilégios em nível de coluna
SELECT grantee, table_schema, table_name, column_name, privilege_type
FROM information_schema.column_privileges
WHERE grantee = 'app_user';
```

## Exercício 6: Privilégios de Execução

### Descrição
Pratique a concessão de privilégios para execução de funções e procedimentos.

### Requisitos
- Criar função/procedimento
- Conceder privilégio de execução
- Revogar privilégio de execução
- Verificar privilégios de execução

### Solução
```sql
-- Criar função
CREATE OR REPLACE FUNCTION aplicacao.calcular_total_pedido(pedido_id INT)
RETURNS DECIMAL AS $$
DECLARE
    total DECIMAL(10,2);
BEGIN
    SELECT SUM(quantidade * preco_unitario) INTO total
    FROM aplicacao.itens_pedido
    WHERE pedido_id = pedido_id;
    RETURN total;
END;
$$ LANGUAGE plpgsql;

-- Conceder privilégio de execução
GRANT EXECUTE ON FUNCTION aplicacao.calcular_total_pedido(INT) TO app_user;

-- Revogar privilégio de execução
REVOKE EXECUTE ON FUNCTION aplicacao.calcular_total_pedido(INT) FROM relatorio_user;

-- Verificar privilégios de execução
SELECT grantee, routine_schema, routine_name, privilege_type
FROM information_schema.routine_privileges
WHERE grantee = 'app_user';
```

## Exercício 7: Privilégios de Conexão

### Descrição
Pratique a concessão de privilégios de conexão a bancos de dados.

### Requisitos
- Conceder privilégio de conexão
- Revogar privilégio de conexão
- Limitar conexões por usuário
- Verificar privilégios de conexão

### Solução
```sql
-- Conceder privilégio de conexão
GRANT CONNECT ON DATABASE seguranca_db TO app_user;

-- Revogar privilégio de conexão
REVOKE CONNECT ON DATABASE seguranca_db FROM dev_user;

-- Limitar conexões por usuário (PostgreSQL)
ALTER USER app_user CONNECTION LIMIT 5;

-- Verificar privilégios de conexão
SELECT * FROM pg_database d
JOIN pg_shdescription s ON d.oid = s.objoid
WHERE s.objsubid = 0 AND d.datname = 'seguranca_db';
```

## Exercício 8: Privilégios Temporários

### Descrição
Pratique a concessão de privilégios temporários para tarefas específicas.

### Requisitos
- Conceder privilégios temporários
- Implementar expiração de privilégios
- Revogar privilégios após uso
- Registrar concessões temporárias

### Solução
```sql
-- Conceder privilégios temporários (simulação)
GRANT UPDATE ON aplicacao.produtos TO dev_user;

-- Registrar concessão temporária
INSERT INTO administracao.privilegios_temporarios
(username, privilegio, objeto, data_expiracao)
VALUES ('dev_user', 'UPDATE', 'aplicacao.produtos', CURRENT_DATE + INTERVAL '1 day');

-- Verificar e revogar privilégios expirados
DO $$
BEGIN
    FOR r IN SELECT username, privilegio, objeto
             FROM administracao.privilegios_temporarios
             WHERE data_expiracao < CURRENT_DATE
    LOOP
        EXECUTE 'REVOKE ' || r.privilegio || ' ON ' || r.objeto || ' FROM ' || r.username;
        
        -- Registrar revogação
        INSERT INTO administracao.log_acessos
        (username, tabela_acessada, tipo_operacao)
        VALUES (r.username, r.objeto, 'REVOKE_EXPIRADO');
    END LOOP;
END $$;
```

## Critérios de Avaliação

- Sintaxe correta dos comandos DCL
- Aplicação do princípio do menor privilégio
- Verificação adequada após concessão/revogação
- Organização lógica de permissões
- Documentação das decisões de segurança