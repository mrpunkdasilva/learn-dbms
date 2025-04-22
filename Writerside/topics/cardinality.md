# Cardinalidade

A cardinalidade é um conceito fundamental na modelagem ER que define o número de instâncias de uma entidade que podem se relacionar com instâncias de outra entidade.

## Tipos de Cardinalidade

### 1. Um-para-Um (1:1)

- Cada instância de uma entidade está associada a no máximo uma instância da outra entidade
- Representada como: `||--||`

```mermaid
erDiagram
    PESSOA ||--|| CNH : "possui"
    PESSOA {
        string cpf PK
        string nome
    }
    CNH {
        string numero PK
        string cpf_pessoa FK
        date validade
    }
```

#### Exemplos Práticos (1:1)
- Pessoa ↔ CPF
- Funcionário ↔ Matrícula
- País ↔ Capital

### 2. Um-para-Muitos (1:N)

- Uma instância de uma entidade pode estar associada a várias instâncias da outra entidade
- Representada como: `||--o{` ou `||--|{`

```mermaid
erDiagram
    DEPARTAMENTO ||--o{ FUNCIONARIO : "emprega"
    DEPARTAMENTO {
        string codigo PK
        string nome
    }
    FUNCIONARIO {
        string matricula PK
        string nome
        string dept_codigo FK
    }
```

#### Exemplos Práticos (1:N)
- Departamento → Funcionários
- Cliente → Pedidos
- Professor → Turmas

### 3. Muitos-para-Muitos (N:M)

- Várias instâncias de uma entidade podem estar associadas a várias instâncias da outra entidade
- Representada como: `}o--o{`

```mermaid
erDiagram
    ALUNO }o--o{ DISCIPLINA : "matriculado"
    ALUNO {
        string matricula PK
        string nome
    }
    DISCIPLINA {
        string codigo PK
        string nome
    }
    MATRICULA {
        string aluno_matricula FK
        string disciplina_codigo FK
        float nota
    }
```

#### Exemplos Práticos (N:M)
- Alunos ↔ Disciplinas
- Produtos ↔ Fornecedores
- Autores ↔ Livros

## Participação

### Participação Total
- Todas as instâncias da entidade participam do relacionamento
- Representada por linha dupla: `||`

```mermaid
erDiagram
    PEDIDO ||--|{ ITEM_PEDIDO : "contém"
```

### Participação Parcial
- Algumas instâncias podem não participar do relacionamento
- Representada por linha com círculo: `o{`

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
```

## Restrições de Cardinalidade

### Mínima
- Número mínimo de instâncias relacionadas
- Exemplo: zero (opcional) ou um (obrigatório)

### Máxima
- Número máximo de instâncias relacionadas
- Exemplo: um ou muitos (n)

## Notações Comuns

### 1. Notação Crow's Foot
```mermaid
erDiagram
    ENTIDADE_A ||--|| ENTIDADE_B : "1:1"
    ENTIDADE_C ||--o{ ENTIDADE_D : "1:N"
    ENTIDADE_E }o--o{ ENTIDADE_F : "N:M"
```

### 2. Notação Chen
- (1,1) : Exatamente um
- (0,1) : Zero ou um
- (1,N) : Um ou mais
- (0,N) : Zero ou mais

## Exemplos Complexos

### Sistema de Biblioteca

```mermaid
erDiagram
    LIVRO }o--|| CATEGORIA : "pertence"
    LIVRO }o--o{ AUTOR : "escrito"
    LIVRO ||--o{ EMPRESTIMO : "emprestado"
    USUARIO ||--o{ EMPRESTIMO : "realiza"
    
    LIVRO {
        string isbn PK
        string titulo
        int ano
    }
    
    EMPRESTIMO {
        string id PK
        date data_emp
        date data_dev
        string livro_isbn FK
        string usuario_id FK
    }
```

### Sistema de E-commerce

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
    PEDIDO ||--|{ ITEM_PEDIDO : "contém"
    PRODUTO }|--|| ITEM_PEDIDO : "incluído"
    PRODUTO }o--o{ CATEGORIA : "classificado"
    
    PEDIDO {
        string numero PK
        date data
        float total
        string cliente_id FK
    }
    
    ITEM_PEDIDO {
        string pedido_num FK
        string produto_cod FK
        int quantidade
        float preco_unit
    }
```

## Boas Práticas

### 1. Análise de Requisitos
- Entenda regras de negócio
- Identifique restrições
- Valide com stakeholders

### 2. Modelagem
- Escolha cardinalidade apropriada
- Considere participação
- Documente decisões

### 3. Implementação
- Planeje chaves estrangeiras
- Defina índices adequados
- Implemente restrições

## Considerações Importantes

### 1. Performance
- Impacto em consultas
- Estratégias de indexação
- Otimização de joins

### 2. Manutenibilidade
- Flexibilidade para mudanças
- Documentação clara
- Padrões consistentes

### 3. Integridade
- Regras de negócio
- Consistência dos dados
- Validações necessárias

## Conclusão

A cardinalidade é essencial para:
- Estruturação correta dos dados
- Integridade do modelo
- Performance do sistema
- Manutenibilidade do código

Uma modelagem adequada de cardinalidade:
- Reflete regras de negócio
- Facilita implementação
- Previne problemas futuros
- Melhora qualidade dos dados