# Exercícios Básicos de DDL

## Exercício 1: Criação de Tabela Simples

### Descrição
Crie uma tabela para armazenar informações básicas de produtos.

### Requisitos
- Nome do produto (máximo 100 caracteres, obrigatório)
- Preço (decimal com 2 casas decimais, obrigatório)
- Descrição (texto livre, opcional)
- Data de cadastro (data/hora automática)

### Solução
```sql
CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) NOT NULL,
    descricao TEXT,
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Exercício 2: Alteração de Estrutura

### Descrição
Modifique a tabela de produtos adicionando novas colunas.

### Requisitos
- Adicionar coluna para quantidade em estoque
- Adicionar coluna para código de barras (único)
- Adicionar coluna para status do produto

### Solução
```sql
ALTER TABLE produtos
    ADD COLUMN estoque INTEGER DEFAULT 0,
    ADD COLUMN codigo_barras VARCHAR(13) UNIQUE,
    ADD COLUMN status VARCHAR(20) DEFAULT 'ativo';
```

## Exercício 3: Constraints Básicas

### Descrição
Adicione restrições básicas à tabela de produtos.

### Requisitos
- Preço não pode ser negativo
- Estoque não pode ser negativo
- Status deve ser 'ativo' ou 'inativo'

### Solução
```sql
ALTER TABLE produtos
    ADD CONSTRAINT check_preco CHECK (preco >= 0),
    ADD CONSTRAINT check_estoque CHECK (estoque >= 0),
    ADD CONSTRAINT check_status CHECK (status IN ('ativo', 'inativo'));
```

## Exercício 4: Tabelas Relacionadas

### Descrição
Crie uma tabela de categorias e relacione com produtos.

### Requisitos
- Tabela de categorias com nome e descrição
- Relacionamento entre produtos e categorias
- Categoria é obrigatória para produtos

### Solução
```sql
CREATE TABLE categorias (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(50) NOT NULL UNIQUE,
    descricao TEXT
);

ALTER TABLE produtos
    ADD COLUMN categoria_id INTEGER NOT NULL,
    ADD CONSTRAINT fk_categoria 
    FOREIGN KEY (categoria_id) 
    REFERENCES categorias(id);
```

## Exercício 5: Índices Simples

### Descrição
Crie índices para melhorar a performance das consultas.

### Requisitos
- Índice para busca por nome do produto
- Índice para código de barras
- Índice para categoria

### Solução
```sql
CREATE INDEX idx_produtos_nome ON produtos(nome);
CREATE INDEX idx_produtos_codigo ON produtos(codigo_barras);
CREATE INDEX idx_produtos_categoria ON produtos(categoria_id);
```

## Exercício 6: Exclusão e Recriação

### Descrição
Pratique operações de remoção e recriação de objetos.

### Requisitos
- Remover índices criados
- Remover constraints
- Remover tabelas com dependências

### Solução
```sql
DROP INDEX IF EXISTS idx_produtos_nome;
DROP INDEX IF EXISTS idx_produtos_codigo;
DROP INDEX IF EXISTS idx_produtos_categoria;

ALTER TABLE produtos
    DROP CONSTRAINT IF EXISTS fk_categoria,
    DROP CONSTRAINT IF EXISTS check_preco,
    DROP CONSTRAINT IF EXISTS check_estoque,
    DROP CONSTRAINT IF EXISTS check_status;

DROP TABLE IF EXISTS produtos CASCADE;
DROP TABLE IF EXISTS categorias CASCADE;
```

## Exercício 7: Schema e Database

### Descrição
Crie um novo schema e database para isolamento.

### Requisitos
- Novo database para testes
- Schema específico para produtos
- Mover objetos entre schemas

### Solução
```sql
CREATE DATABASE loja_teste;

\c loja_teste

CREATE SCHEMA produtos_schema;

CREATE TABLE produtos_schema.produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);
```

## Exercício 8: Sequences

### Descrição
Trabalhe com sequences personalizadas.

### Requisitos
- Sequence para código do produto
- Sequence com valor inicial específico
- Alteração de sequence existente

### Solução
```sql
CREATE SEQUENCE seq_codigo_produto
    START WITH 1000
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;

ALTER TABLE produtos
    ALTER COLUMN id SET DEFAULT nextval('seq_codigo_produto');
```

## Exercício 9: Comentários e Documentação

### Descrição
Adicione comentários aos objetos do banco.

### Requisitos
- Comentários em tabelas
- Comentários em colunas
- Comentários em constraints

### Solução
```sql
COMMENT ON TABLE produtos IS 'Tabela de cadastro de produtos';
COMMENT ON COLUMN produtos.nome IS 'Nome do produto';
COMMENT ON CONSTRAINT check_preco ON produtos IS 'Garante preço não negativo';
```

## Exercício 10: Visão Consolidada

### Descrição
Crie uma estrutura completa integrando todos os conceitos.

### Requisitos
- Tabelas relacionadas
- Constraints apropriadas
- Índices necessários
- Documentação completa

### Solução
```sql
-- Schema dedicado
CREATE SCHEMA IF NOT EXISTS loja;

-- Tabela de categorias
CREATE TABLE loja.categorias (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(50) NOT NULL UNIQUE,
    descricao TEXT,
    ativa BOOLEAN DEFAULT true,
    data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de produtos
CREATE TABLE loja.produtos (
    id SERIAL PRIMARY KEY,
    codigo_barras VARCHAR(13) UNIQUE,
    nome VARCHAR(100) NOT NULL,
    descricao TEXT,
    preco DECIMAL(10,2) NOT NULL CHECK (preco >= 0),
    estoque INTEGER DEFAULT 0 CHECK (estoque >= 0),
    categoria_id INTEGER NOT NULL REFERENCES loja.categorias(id),
    status VARCHAR(20) DEFAULT 'ativo' CHECK (status IN ('ativo', 'inativo')),
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Índices
CREATE INDEX idx_produtos_nome ON loja.produtos(nome);
CREATE INDEX idx_produtos_categoria ON loja.produtos(categoria_id);

-- Comentários
COMMENT ON SCHEMA loja IS 'Schema para sistema de loja';
COMMENT ON TABLE loja.produtos IS 'Cadastro de produtos da loja';
COMMENT ON TABLE loja.categorias IS 'Categorias de produtos';
```

## Critérios de Avaliação

- Sintaxe SQL correta
- Uso apropriado de constraints
- Nomenclatura adequada
- Documentação clara
- Scripts de rollback

## Dicas de Estudo

- Teste cada comando separadamente
- Verifique a estrutura criada
- Documente as decisões tomadas
- Pratique os scripts de rollback