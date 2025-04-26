# CREATE Statements: Construindo Estruturas de Dados

```ascii
╔══════════════════════════════════════════════════════════════╗
║  NEURAL.MATRIX >> SQL.CREATE                                ║
║  INSTRUTOR: DATA_ARCHITECT_SAGE                            ║
╚══════════════════════════════════════════════════════════════╝
```

## Introdução ao CREATE

O comando CREATE é a base para construção de estruturas em bancos de dados. Ele permite criar diversos objetos como databases, tabelas, índices, views e outros elementos essenciais.

## CREATE DATABASE

O DATA_ARCHITECT_SAGE explica: "Começamos sempre pelo início - a criação do banco de dados. É como construir a fundação de uma casa."

```sql
CREATE DATABASE ecommerce
    WITH 
    ENCODING = 'UTF8'
    LC_COLLATE = 'pt_BR.UTF-8'
    LC_CTYPE = 'pt_BR.UTF-8'
    TEMPLATE = template0;
```

**Por que estes parâmetros?**
- `ENCODING`: Garante suporte a caracteres especiais
- `LC_COLLATE` e `LC_CTYPE`: Configurações para português brasileiro
- `TEMPLATE`: Base limpa para novo banco

## CREATE TABLE

SCHEMA_MASTER diz: "As tabelas são o coração do seu banco de dados. Vamos criar uma estrutura robusta para um e-commerce."

### Exemplo Básico
```sql
CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) NOT NULL CHECK (preco >= 0),
    descricao TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Exemplo Avançado com Relacionamentos
```sql
CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER NOT NULL,
    status VARCHAR(20) NOT NULL,
    valor_total DECIMAL(10,2) NOT NULL,
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT fk_cliente 
        FOREIGN KEY (cliente_id) 
        REFERENCES clientes(id)
        ON DELETE RESTRICT,
        
    CONSTRAINT chk_status 
        CHECK (status IN ('pendente', 'aprovado', 'cancelado')),
        
    CONSTRAINT chk_valor 
        CHECK (valor_total > 0)
);
```

SECURITY_GUARDIAN acrescenta: "Observe as constraints - são fundamentais para integridade dos dados!"

## CREATE INDEX

INDEX_MASTER compartilha: "Índices são como o índice de um livro - essenciais para encontrar informações rapidamente."

### Tipos de Índices

```sql
-- Índice básico
CREATE INDEX idx_produtos_nome ON produtos(nome);

-- Índice único
CREATE UNIQUE INDEX idx_usuarios_email ON usuarios(email);

-- Índice composto
CREATE INDEX idx_pedidos_cliente_data ON pedidos(cliente_id, data_pedido);

-- Índice parcial
CREATE INDEX idx_produtos_ativos ON produtos(nome) 
WHERE status = 'ativo';
```

## CREATE VIEW

QUERY_MASTER explica: "Views são consultas pré-definidas que simplificam o acesso aos dados."

```sql
CREATE VIEW vw_pedidos_detalhados AS
SELECT 
    p.id as pedido_id,
    c.nome as cliente_nome,
    p.valor_total,
    p.status,
    COUNT(i.id) as total_itens
FROM pedidos p
JOIN clientes c ON p.cliente_id = c.id
JOIN itens_pedido i ON p.id = i.pedido_id
GROUP BY p.id, c.nome, p.valor_total, p.status;
```

## CREATE SEQUENCE

DB_WIZARD diz: "Sequences são úteis para gerar números únicos de forma automática."

```sql
CREATE SEQUENCE seq_codigo_produto
    INCREMENT BY 1
    START WITH 1000
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```

## Exercícios Práticos

TRAINING_MASTER propõe: "Vamos praticar com um cenário real!"

### Sistema de Blog

```sql
-- Criando tabela de autores
CREATE TABLE autores (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    bio TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Criando tabela de posts
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    titulo VARCHAR(200) NOT NULL,
    conteudo TEXT NOT NULL,
    autor_id INTEGER NOT NULL,
    status VARCHAR(20) DEFAULT 'rascunho',
    publicado_em TIMESTAMP,
    
    CONSTRAINT fk_autor
        FOREIGN KEY (autor_id)
        REFERENCES autores(id),
        
    CONSTRAINT chk_status
        CHECK (status IN ('rascunho', 'publicado', 'arquivado'))
);

-- Criando índices estratégicos
CREATE INDEX idx_posts_autor ON posts(autor_id);
CREATE INDEX idx_posts_status ON posts(status);
CREATE INDEX idx_posts_publicacao ON posts(publicado_em DESC);

-- Criando view para posts publicados
CREATE VIEW vw_posts_publicados AS
SELECT 
    p.id,
    p.titulo,
    a.nome as autor_nome,
    p.publicado_em
FROM posts p
JOIN autores a ON p.autor_id = a.id
WHERE p.status = 'publicado'
ORDER BY p.publicado_em DESC;
```

## Boas Práticas

BEST_PRACTICES_GURU compartilha dicas essenciais:

1. **Nomenclatura**
   - Use nomes descritivos
   - Siga um padrão consistente
   - Prefira minúsculas para objetos
   - Use underscores para separar palavras

2. **Constraints**
   - Nomeie todas as constraints importantes
   - Use CHECK para validações de domínio
   - Implemente FKs com cuidado
   - Considere ON DELETE/UPDATE actions

3. **Índices**
   - Crie índices com propósito
   - Evite índices redundantes
   - Nomeie índices de forma descritiva
   - Considere o impacto na escrita

```ascii
╔════════════════════════════════════╗
║  LEMBRE-SE:                       ║
║  - Teste antes em desenvolvimento ║
║  - Mantenha documentação         ║
║  - Faça backup antes de criar    ║
║  - Planeje crescimento           ║
╚════════════════════════════════════╝
```