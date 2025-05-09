# Exercícios Avançados de DCL

## Exercício 1: Implementação de Políticas de Segurança

### Descrição
Implemente políticas de segurança abrangentes para um sistema empresarial.

### Requisitos
- Definir políticas por departamento
- Implementar controle de acesso granular
- Estabelecer hierarquia de permissões
- Documentar políticas implementadas

### Solução
```sql
-- Criar roles para departamentos
CREATE ROLE dept_vendas_role;
CREATE ROLE dept_financeiro_role;
CREATE ROLE dept_rh_role;
CREATE ROLE dept_ti_role;

-- Criar roles para níveis hierárquicos
CREATE ROLE nivel_operacional_role;
CREATE ROLE nivel_gerencial_role;
CREATE ROLE nivel_diretoria_role;

-- Estabelecer hierarquia
GRANT nivel_operacional_role TO nivel_gerencial_role;
GRANT nivel_gerencial_role TO nivel_diretoria_role;

-- Definir permissões por departamento - Vendas
GRANT USAGE ON SCHEMA aplicacao TO dept_vendas_role;
GRANT SELECT, INSERT ON aplicacao.clientes TO dept_vendas_role;
GRANT SELECT, INSERT ON aplicacao.pedidos TO dept_vendas_role;
GRANT SELECT ON aplicacao.produtos TO dept_vendas_role;

-- Definir permissões por departamento - Financeiro
GRANT USAGE ON SCHEMA aplicacao, relatorios TO dept_financeiro_role;
GRANT SELECT ON aplicacao.pedidos TO dept