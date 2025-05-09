# Laboratório: Hierarquia de Roles

## Objetivo
Implementar uma estrutura hierárquica de roles em um banco de dados PostgreSQL, aplicando o princípio de separação de responsabilidades e herança de permissões.

## Cenário
Você é um arquiteto de segurança em uma empresa com múltiplos departamentos e níveis hierárquicos. Sua tarefa é implementar uma estrutura de roles que reflita a organização da empresa e facilite o gerenciamento de permissões.

## Setup Inicial

```sql
-- Conectar ao banco de dados como superusuário
-- psql -U postgres -d lab_dcl

-- Verificar roles existentes
SELECT rolname, rolsuper, rolinherit, rolcanlogin
FROM pg_roles
WHERE rolname NOT LIKE 'pg_%';

-- Registrar início do laboratório
INSERT INTO lab_workspace.lab_progress (lab_name)
VALUES ('Hierarquia de Roles');
```

## Parte 1: Criação de Estrutura Básica de Roles

### 1.1 Roles Departamentais

```sql
-- Criar roles para departamentos
CREATE ROLE dept_vendas;
CREATE ROLE dept_marketing;
CREATE ROLE dept_financeiro;
CREATE ROLE dept_ti;

-- Verificar roles criadas
SELECT rolname FROM pg_roles WHERE rolname LIKE 'dept_%';
```

### 1.2 Roles Hierárquicas

```sql
-- Criar roles para níveis hierárquicos
CREATE ROLE nivel_operacional;
CREATE ROLE nivel_tatico;
CREATE ROLE nivel_estrategico;

-- Criar roles para funções específicas
CREATE ROLE analista;
CREATE ROLE supervisor;
CREATE ROLE gerente;
CREATE ROLE diretor;

-- Estabelecer hierarquia
GRANT analista TO nivel_operacional;
GRANT supervisor TO nivel_tatico;
GRANT gerente TO nivel_tatico;
GRANT diretor TO nivel_estrategico;
```

## Parte 2: Implementação de Herança de Permissões

### 2.1 Permissões Departamentais

```sql
-- Conceder permissões aos departamentos
-- Vendas
GRANT USAGE ON SCHEMA lab_workspace TO dept_vendas;
GRANT SELECT, INSERT ON lab_workspace.clientes TO dept_vendas;
GRANT SELECT, INSERT, UPDATE ON lab_workspace.pedidos TO dept_vendas;
GRANT SELECT, INSERT, UPDATE ON lab_workspace.itens_pedido TO dept_vendas;
GRANT SELECT ON lab_workspace.produtos TO dept_vendas;

-- Marketing
GRANT USAGE ON SCHEMA lab_workspace TO dept_marketing;
GRANT SELECT ON lab_workspace.clientes TO dept_marketing;
GRANT SELECT ON lab_workspace.pedidos TO dept_marketing;
GRANT SELECT ON lab_workspace.produtos TO dept_marketing;

-- Financeiro
GRANT USAGE ON SCHEMA lab_workspace TO dept_financeiro;
GRANT SELECT ON lab_workspace.clientes TO dept_financeiro;
GRANT SELECT ON lab_workspace.pedidos TO dept_financeiro;
GRANT SELECT ON lab_workspace.itens_pedido TO dept_financeiro;
GRANT UPDATE (limite_credito) ON lab_workspace.clientes TO dept_financeiro;

-- TI
GRANT USAGE ON SCHEMA lab_workspace TO dept_ti;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA lab_workspace TO dept_ti;
```

### 2.2 Permissões por Nível Hierárquico

```sql
-- Nível Operacional
GRANT SELECT ON ALL TABLES IN SCHEMA lab_workspace TO nivel_operacional;

-- Nível Tático
GRANT nivel_operacional TO nivel_tatico;
GRANT INSERT, UPDATE ON ALL TABLES IN SCHEMA lab_workspace TO nivel_tatico;

-- Nível Estratégico
GRANT nivel_tatico TO nivel_estrategico;
GRANT DELETE ON ALL TABLES IN SCHEMA lab_workspace TO nivel_estrategico;
GRANT USAGE, CREATE ON SCHEMA lab_workspace TO nivel_estrategico;
```

## Parte 3: Criação de Usuários com Múltiplas Roles

### 3.1 Usuários Departamentais

```sql
-- Criar usuários com roles departamentais e hierárquicas
CREATE USER vendedor01 WITH PASSWORD 'v123' IN ROLE dept_vendas, nivel_operacional;
CREATE USER supervisor_vendas WITH PASSWORD 's123' IN ROLE dept_vendas, nivel_tatico;
CREATE USER gerente_vendas WITH PASSWORD 'g123' IN ROLE dept_vendas, nivel_tatico;
CREATE USER diretor_vendas WITH PASSWORD 'd123' IN ROLE dept_vendas, nivel_estrategico;

CREATE USER analista_marketing WITH PASSWORD 'a123' IN ROLE dept_marketing, nivel_operacional;
CREATE USER gerente_marketing WITH PASSWORD 'g123' IN ROLE dept_marketing, nivel_tatico;

CREATE USER analista_financeiro WITH PASSWORD 'a123' IN ROLE dept_financeiro, nivel_operacional;
CREATE USER controller WITH PASSWORD 'c123' IN ROLE dept_financeiro, nivel_tatico;

CREATE USER admin_ti WITH PASSWORD 'a123' IN ROLE dept_ti, nivel_estrategico;
```

### 3.2 Verificação de Membros de Roles

```sql
-- Verificar membros de cada role
SELECT r.rolname, u.usename
FROM pg_roles r
JOIN pg_auth_members m ON r.oid = m.roleid
JOIN pg_roles u ON m.member = u.oid
WHERE r.rolname LIKE 'dept_%' OR r.rolname LIKE 'nivel_%'
ORDER BY r.rolname, u.usename;
```

## Parte 4: Implementação de Roles Funcionais

### 4.1 Roles para Funções Específicas

```sql
-- Criar roles para funções específicas
CREATE ROLE atendimento_cliente;
GRANT SELECT, INSERT, UPDATE ON lab_workspace.clientes TO atendimento_cliente;
GRANT SELECT ON lab_workspace.pedidos TO atendimento_cliente;

CREATE ROLE gestor_estoque;
GRANT SELECT, UPDATE ON lab_workspace.produtos TO gestor_estoque;

CREATE ROLE processador_pedidos;
GRANT SELECT, UPDATE ON lab_workspace.pedidos TO processador_pedidos;
GRANT SELECT, INSERT, UPDATE ON lab_workspace.itens_pedido TO processador_pedidos;
GRANT SELECT ON lab_workspace.produtos TO processador_pedidos;
GRANT SELECT ON lab_workspace.clientes TO processador_pedidos;

-- Atribuir roles funcionais aos usuários
GRANT atendimento_cliente TO vendedor01;
GRANT gestor_estoque TO supervisor_vendas;
GRANT processador_pedidos TO vendedor01, supervisor_vendas;
```

### 4.2 Roles Temporárias

```sql
-- Criar role para acesso temporário
CREATE ROLE acesso_auditoria;
GRANT SELECT ON ALL TABLES IN SCHEMA lab_workspace TO acesso_auditoria;

-- Criar role com restrição de tempo
CREATE ROLE acesso_noturno;
GRANT SELECT ON ALL TABLES IN SCHEMA lab_workspace TO acesso_noturno;

-- Implementar controle de acesso baseado em tempo
CREATE OR REPLACE FUNCTION lab_workspace.check_time_access()
RETURNS boolean AS $$
DECLARE
    current_hour integer;
BEGIN
    current_hour := EXTRACT(HOUR FROM CURRENT_TIME);
    RETURN current_hour BETWEEN 18 AND 6;
END;
$$ LANGUAGE plpgsql;

-- Nota: Em um ambiente real, você usaria Row-Level Security para implementar isso
-- Este é apenas um exemplo conceitual
```

## Parte 5: Análise e Auditoria de Roles

### 5.1 Visualização da Hierarquia de Roles

```sql
-- Criar função para visualizar hierarquia de roles
CREATE OR REPLACE FUNCTION lab_workspace.role_hierarchy(role_name text)
RETURNS TABLE (role text, level int) AS $$
WITH RECURSIVE role_tree AS (
    SELECT r.rolname AS role, 1 AS level
    FROM pg_roles r
    WHERE r.rolname = role_name
    
    UNION ALL
    
    SELECT r.rolname, rt.level + 1
    FROM role_tree rt
    JOIN pg_auth_members m ON rt.role = (SELECT rolname FROM pg_roles WHERE oid = m.roleid)
    JOIN pg_roles r ON r.oid = m.member
)
SELECT role, level FROM role_tree ORDER BY level, role;
$$ LANGUAGE sql;

-- Visualizar hierarquia para um usuário específico
SELECT * FROM lab_workspace.role_hierarchy('diretor_vendas');
```

### 5.2 Relatório de Permissões Efetivas

```sql
-- Criar visão para relatório de permissões efetivas
CREATE OR REPLACE VIEW lab_workspace.effective_permissions AS
WITH role_members AS (
    SELECT r.rolname AS role, u.usename AS username
    FROM pg_roles r
    JOIN pg_auth_members m ON r.oid = m.roleid
    JOIN pg_roles u ON m.member = u.oid
    WHERE u.rolcanlogin
    
    UNION
    
    SELECT r.rolname AS role, r.rolname AS username
    FROM pg_roles r
    WHERE r.rolcanlogin
)
SELECT 
    rm.username,
    t.table_schema,
    t.table_name,
    t.privilege_type
FROM role_members rm
JOIN information_schema.role_table_grants t ON rm.role = t.grantee
WHERE t.table_schema = 'lab_workspace'
ORDER BY rm.username, t.table_schema, t.table_name, t.privilege_type;

-- Consultar permissões efetivas
SELECT * FROM lab_workspace.effective_permissions WHERE username = 'vendedor01';
```

## Verificações e Testes

1. Conecte-se como diferentes usuários e verifique suas permissões:
   ```
   psql -U vendedor01 -d lab_dcl
   psql -U gerente_marketing -d lab_dcl
   psql -U admin_ti -d lab_dcl
   ```

2. Teste operações permitidas e não permitidas:
   ```sql
   -- Como vendedor01
   SELECT * FROM lab_workspace.clientes;
   INSERT INTO lab_workspace.clientes (nome, email) VALUES ('Cliente Teste', 'teste@email.com');
   DELETE FROM lab_workspace.clientes WHERE id = 1; -- Deve falhar
   
   -- Como gerente_vendas
   SELECT * FROM lab_workspace.clientes;
   INSERT INTO lab_workspace.clientes (nome, email) VALUES ('Cliente Gerente', 'gerente@email.com');
   UPDATE lab_workspace.clientes SET status = 'Inativo' WHERE id = 1;
   
   -- Como admin_ti
   CREATE TABLE lab_workspace.teste_admin (id serial primary key, descricao text);
   DROP TABLE lab_workspace.teste_admin;
   ```

3. Verifique a hierarquia de roles:
   ```sql
   SELECT * FROM lab_workspace.role_hierarchy('nivel_estrategico');
   SELECT * FROM lab_workspace.role_hierarchy('gerente_vendas');
   ```

## Conclusão

Neste laboratório, você implementou:

- Uma estrutura hierárquica de roles que reflete a organização da empresa
- Herança de permissões entre níveis hierárquicos
- Separação de responsabilidades por departamento
- Roles funcionais para tarefas específicas
- Ferramentas para análise e auditoria de permissões

Esta abordagem estruturada para gerenciamento de permissões oferece vários benefícios:

1. **Simplicidade de administração**: Alterações em permissões podem ser feitas em nível de role, afetando automaticamente todos os usuários associados
2. **Consistência**: Usuários com funções similares recebem permissões consistentes
3. **Segurança**: O princípio de privilégio mínimo é aplicado em cada nível
4. **Flexibilidade**: Usuários podem pertencer a múltiplas roles, combinando permissões conforme necessário

## Próximos Passos

1. Implemente Row-Level Security (RLS) para controle de acesso mais granular
2. Desenvolva um sistema automatizado para gerenciamento de roles
3. Integre com sistemas de identidade externos (LDAP, Active Directory)
4. Implemente rotação automática de credenciais

---

> **Nota**: Registre a conclusão do laboratório:
> ```sql
> UPDATE lab_workspace.lab_progress 
> SET completion_time = CURRENT_TIMESTAMP, status = 'COMPLETED' 
> WHERE lab_name = 'Hierarquia de Roles';
> ```