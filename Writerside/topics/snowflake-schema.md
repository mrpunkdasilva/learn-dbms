# Schema Floco de Neve

## Estrutura Básica

```mermaid
graph TD
    A[Fato Vendas] --> B[Dim Produto]
    A --> C[Dim Cliente]
    B --> D[Dim Categoria]
    D --> E[Dim Departamento]
    C --> F[Dim Cidade]
    F --> G[Dim Estado]
    G --> H[Dim País]
    
    style A fill:#f9f,stroke:#333
    style B fill:#bbf,stroke:#333
    style C fill:#bbf,stroke:#333
    style D fill:#bbf,stroke:#333
    style E fill:#bbf,stroke:#333
    style F fill:#bbf,stroke:#333
    style G fill:#bbf,stroke:#333
    style H fill:#bbf,stroke:#333
```

## Características

### 1. Normalização de Dimensões
```sql
-- Hierarquia de localização normalizada
CREATE TABLE dim_cidade (
    sk_cidade INT PRIMARY KEY,
    nome_cidade VARCHAR(50),
    sk_estado INT,
    FOREIGN KEY (sk_estado) REFERENCES dim_estado(sk_estado)
);

CREATE TABLE dim_estado (
    sk_estado INT PRIMARY KEY,
    nome_estado VARCHAR(50),
    sk_pais INT,
    FOREIGN KEY (sk_pais) REFERENCES dim_pais(sk_pais)
);
```

### 2. Hierarquias Explícitas
- Níveis separados
- Integridade referencial
- Economia de espaço
- Manutenção facilitada

## Comparação com Schema Estrela

### 1. Vantagens
- Menor redundância
- Consistência de dados
- Atualizações eficientes
- Normalização completa

### 2. Desvantagens
- Performance reduzida
- Mais joins necessários
- Complexidade maior
- Queries mais complexas