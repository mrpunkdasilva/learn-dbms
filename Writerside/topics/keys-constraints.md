# Restrições de Chave

## Tipos de Chaves

### 1. Chave Primária (Primary Key - PK)

```mermaid
erDiagram
    ENTIDADE {
        string id PK "Identificador único"
        string nome
        string descricao
    }
```

#### Características
- Identifica unicamente cada tupla
- Não permite valores nulos
- Imutável
- Pode ser simples ou composta

#### Exemplos
```sql
CREATE TABLE Produto (
    codigo SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2)
);

-- Chave composta
CREATE TABLE ItemPedido (
    pedido_id INTEGER,
    produto_id INTEGER,
    quantidade INTEGER,
    PRIMARY KEY (pedido_id, produto_id)
);
```

### 2. Chave Estrangeira (Foreign Key - FK)

```mermaid
erDiagram
    DEPARTAMENTO ||--o{ FUNCIONARIO : possui
    DEPARTAMENTO {
        int id PK
        string nome
    }
    FUNCIONARIO {
        int id PK
        string nome
        int depto_id FK
    }
```

#### Características
- Referencia chave primária de outra tabela
- Mantém integridade referencial
- Pode ser nula (relacionamento opcional)
- Suporta ações referenciais (CASCADE, SET NULL, etc.)

#### Exemplos
```sql
CREATE TABLE Funcionario (
    id INTEGER PRIMARY KEY,
    nome VARCHAR(100),
    depto_id INTEGER,
    FOREIGN KEY (depto_id) 
        REFERENCES Departamento(id)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

### 3. Chave Única (Unique Key - UK)

```mermaid
erDiagram
    USUARIO {
        int id PK
        string email UK "Único por usuário"
        string username UK "Único no sistema"
        string senha
    }
```

#### Características
- Garante unicidade dos valores
- Permite valores nulos (diferente da PK)
- Múltiplas por tabela
- Pode ser composta

#### Exemplos
```sql
CREATE TABLE Usuario (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    username VARCHAR(50) UNIQUE,
    senha VARCHAR(255)
);
```

## Implementação em ORMs

### 1. JPA/Hibernate (Java)
```java
@Entity
public class Cliente {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String email;
    
    @ManyToOne
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;
}
```

### 2. Django ORM (Python)
```python
class Produto(models.Model):
    codigo = models.CharField(max_length=20, primary_key=True)
    sku = models.CharField(max_length=50, unique=True)
    categoria = models.ForeignKey(
        'Categoria',
        on_delete=models.CASCADE
    )
```

## Boas Práticas

### 1. Escolha de Chaves Primárias
- Use valores naturais quando apropriado
- Considere surrogate keys para flexibilidade
- Evite chaves compostas complexas
- Mantenha a imutabilidade

### 2. Gestão de Chaves Estrangeiras
- Defina ações referenciais apropriadas
- Considere o impacto na integridade
- Use índices para performance
- Documente relacionamentos

### 3. Unicidade
- Identifique campos que exigem unicidade
- Implemente validações em múltiplas camadas
- Considere unicidade combinada
- Trate conflitos adequadamente

## Padrões Comuns

### 1. Chaves Naturais vs Surrogate
```mermaid
erDiagram
    PRODUTO_NATURAL {
        string codigo PK "Chave natural"
        string nome
    }
    
    PRODUTO_SURROGATE {
        int id PK "Chave surrogate"
        string codigo UK "Chave natural"
        string nome
    }
```

### 2. Chaves Compostas
```mermaid
erDiagram
    MATRICULA {
        string aluno_id PK,FK
        string disciplina_id PK,FK
        string semestre PK
        float nota
    }
```

## Considerações de Performance

### 1. Indexação
- Índices automáticos em PKs
- Índices opcionais em FKs
- Índices únicos para UKs
- Impacto em inserções/atualizações

### 2. Joins
- Otimização de consultas
- Cardinalidade das relações
- Estratégias de indexação
- Planos de execução