# Padrões Temporais

## Histórico de Mudanças

```mermaid
erDiagram
    PRODUTO ||--o{ PRODUTO_HISTORICO : registra
    
    PRODUTO {
        int id PK
        string nome
        float preco_atual
    }
    
    PRODUTO_HISTORICO {
        int id PK
        int produto_id FK
        float preco
        date data_inicio
        date data_fim
    }
```

### Implementação
```sql
CREATE TABLE produto (
    id INT PRIMARY KEY,
    nome VARCHAR(100),
    preco_atual DECIMAL(10,2)
);

CREATE TABLE produto_historico (
    id INT PRIMARY KEY,
    produto_id INT,
    preco DECIMAL(10,2),
    data_inicio DATE,
    data_fim DATE,
    FOREIGN KEY (produto_id) REFERENCES produto(id)
);
```

## Versionamento

```mermaid
erDiagram
    DOCUMENTO ||--o{ VERSAO : possui
    
    DOCUMENTO {
        int id PK
        string titulo
        int versao_atual
    }
    
    VERSAO {
        int id PK
        int documento_id FK
        int numero
        text conteudo
        timestamp data_criacao
    }
```

### Implementação
```sql
CREATE TABLE documento (
    id INT PRIMARY KEY,
    titulo VARCHAR(200),
    versao_atual INT
);

CREATE TABLE versao (
    id INT PRIMARY KEY,
    documento_id INT,
    numero INT,
    conteudo TEXT,
    data_criacao TIMESTAMP,
    FOREIGN KEY (documento_id) REFERENCES documento(id)
);
```

## Dados Temporais

```mermaid
erDiagram
    CONTRATO {
        int id PK
        date data_inicio
        date data_fim
        string status
        float valor
    }
```

### Implementação
```sql
CREATE TABLE contrato (
    id INT PRIMARY KEY,
    data_inicio DATE,
    data_fim DATE,
    status VARCHAR(20),
    valor DECIMAL(10,2)
);

-- Índices para consultas temporais
CREATE INDEX idx_contrato_periodo 
ON contrato(data_inicio, data_fim);
```