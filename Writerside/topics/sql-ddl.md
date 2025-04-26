# DDL - Linguagem de Definição de Dados

A Linguagem de Definição de Dados (DDL) é um componente fundamental do SQL usado para definir e modificar a estrutura do banco de dados. Com ela, podemos criar, alterar e excluir objetos como tabelas, índices, views e outros elementos estruturais.

## Principais Comandos DDL

O DDL possui quatro comandos principais que formam a base para gerenciamento de estruturas de banco de dados:

### CREATE
O comando CREATE é usado para criar novos objetos no banco de dados. É o comando mais básico e essencial do DDL.

**Exemplos práticos:**
```sql
-- Criação de banco de dados
CREATE DATABASE loja;

-- Criação de tabela com diferentes tipos de dados e constraints
CREATE TABLE produtos (
    id INT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2),
    categoria_id INT,
    FOREIGN KEY (categoria_id) REFERENCES categorias(id)
);
```

### ALTER
O ALTER permite modificar a estrutura de objetos existentes. É crucial para evolução do schema do banco de dados.

**Exemplos comuns:**
```sql
-- Adicionando uma nova coluna
ALTER TABLE produtos
ADD COLUMN descricao TEXT;

-- Modificando o tipo de uma coluna
ALTER TABLE produtos
ALTER COLUMN preco TYPE NUMERIC(12,2);
```

### DROP
Usado para remover objetos do banco de dados. Deve ser usado com extrema cautela pois é irreversível.

**Exemplo de uso:**
```sql
-- Removendo uma tabela
DROP TABLE IF EXISTS produtos_antigos;
```

### TRUNCATE
Remove todos os registros de uma tabela, mas mantém sua estrutura. É mais eficiente que DELETE para limpeza completa.

```sql
TRUNCATE TABLE logs_temporarios;
```

## Boas Práticas de DDL

### 1. Segurança em Primeiro Lugar
- Sempre faça backup antes de operações DDL
- Teste comandos em ambiente de desenvolvimento
- Mantenha scripts de rollback preparados

### 2. Planejamento
- Documente todas as alterações estruturais
- Avalie impactos em aplicações existentes
- Considere volumes de dados nas operações

### 3. Versionamento
- Mantenha controle de versão dos schemas
- Use migrations para alterações estruturais
- Documente a evolução do banco de dados

## Considerações de Performance

### Impacto das Operações
- Operações DDL podem bloquear tabelas
- Alterações em tabelas grandes requerem planejamento
- Considere janelas de manutenção para operações críticas

### Otimizações
- Use índices adequadamente
- Planeje particionamento quando necessário
- Considere clustering quando apropriado

## Exercício Prático

Vamos criar uma estrutura básica para um sistema de vendas:

```sql
-- Schema do sistema
CREATE SCHEMA vendas;

-- Tabela de produtos
CREATE TABLE vendas.produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) CHECK (preco > 0),
    estoque INT DEFAULT 0
);

-- Adicionando índice para busca por nome
CREATE INDEX idx_produtos_nome 
ON vendas.produtos(nome);
```

## Conclusão

O DDL é fundamental para o gerenciamento de bancos de dados, permitindo criar e manter estruturas de dados de forma eficiente. O domínio desses comandos, junto com boas práticas de uso, é essencial para qualquer desenvolvedor ou DBA.

> **Dica de Segurança**: Sempre mantenha backups atualizados e teste suas operações DDL em ambiente de desenvolvimento antes de aplicar em produção.
