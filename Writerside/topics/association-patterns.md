# Padrões de Associação

## One-to-One

```mermaid
erDiagram
    USUARIO ||--|| PERFIL : possui
    
    USUARIO {
        int id PK
        string username
        string email
    }
    
    PERFIL {
        int usuario_id FK
        string bio
        string avatar
    }
```

### Implementação
```sql
CREATE TABLE usuario (
    id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100)
);

CREATE TABLE perfil (
    usuario_id INT PRIMARY KEY,
    bio TEXT,
    avatar VARCHAR(200),
    FOREIGN KEY (usuario_id) REFERENCES usuario(id)
);
```

## One-to-Many

```mermaid
erDiagram
    PEDIDO ||--|{ ITEM : contém
    
    PEDIDO {
        int id PK
        date data
        float total
    }
    
    ITEM {
        int id PK
        int pedido_id FK
        string produto
        int quantidade
    }
```

### Implementação
```sql
CREATE TABLE pedido (
    id INT PRIMARY KEY,
    data DATE,
    total DECIMAL(10,2)
);

CREATE TABLE item (
    id INT PRIMARY KEY,
    pedido_id INT,
    produto VARCHAR(100),
    quantidade INT,
    FOREIGN KEY (pedido_id) REFERENCES pedido(id)
);
```

## Many-to-Many

```mermaid
erDiagram
    PRODUTO }|--|{ CATEGORIA : pertence
    
    PRODUTO {
        int id PK
        string nome
        float preco
    }
    
    PRODUTO_CATEGORIA {
        int produto_id FK
        int categoria_id FK
    }
    
    CATEGORIA {
        int id PK
        string nome
        string descricao
    }
```

### Implementação
```sql
CREATE TABLE produto (
    id INT PRIMARY KEY,
    nome VARCHAR(100),
    preco DECIMAL(10,2)
);

CREATE TABLE categoria (
    id INT PRIMARY KEY,
    nome VARCHAR(50),
    descricao TEXT
);

CREATE TABLE produto_categoria (
    produto_id INT,
    categoria_id INT,
    PRIMARY KEY (produto_id, categoria_id),
    FOREIGN KEY (produto_id) REFERENCES produto(id),
    FOREIGN KEY (categoria_id) REFERENCES categoria(id)
);
```