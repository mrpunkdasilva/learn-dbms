# Conceitos Fundamentais do Modelo ER

Os conceitos fundamentais do Modelo Entidade-Relacionamento (ER) formam a base para a modelagem conceitual de dados. Este capítulo explora os elementos essenciais que compõem um diagrama ER.

## Visão Geral dos Conceitos

```mermaid
mindmap
    root((Conceitos ER))
        Entidades
            Tipos
            Ocorrências
            Identificadores
        Relacionamentos
            Binários
            Ternários
            Recursivos
        Atributos
            Simples
            Compostos
            Multivalorados
        Cardinalidade
            Um-para-Um
            Um-para-Muitos
            Muitos-para-Muitos
```

## Elementos Básicos

### 1. Entidades

Uma entidade representa um objeto ou conceito do mundo real que pode ser distintamente identificado.

#### Tipos de Entidades
- **Entidades Fortes**: Existem independentemente de outras entidades
- **Entidades Fracas**: Dependem de outras entidades para existir
- **Entidades Associativas**: Resultam da associação entre outras entidades

- **Exemplo**:
  - EMPLOYEE (entidade forte)
  - DEPENDENT (entidade fraca)
  - DEPARTAMENT (entidade associativa)

![Entities Type](Entities Type)



### 2. Relacionamentos

Representam associações entre entidades, descrevendo como elas interagem entre si.

> Nunca crie relacionamentos entre atributos
>
> Sempre crie relacionamentos entre entidades! 
> {style="warning"}

#### Características dos Relacionamentos
- **Grau**: Número de entidades participantes
- **Papel**: Função de cada entidade no relacionamento

```mermaid
erDiagram
    FUNCIONARIO ||--o{ DEPENDENTE : "possui"
    FUNCIONARIO }|--|| DEPARTAMENTO : "trabalha em"
    DEPARTAMENTO ||--o{ PROJETO : "gerencia"
```

### 3. Atributos

Descrevem propriedades ou características das entidades e relacionamentos.

#### Classificação dos Atributos

1. **Quanto à Estrutura**
   - Simples (atômicos)
   - Compostos
   - Multivalorados

2. **Quanto à Função**
   - Descritivos
   - Identificadores (chaves)
   - Referenciais

```mermaid
erDiagram
    PESSOA {
        string cpf PK "Identificador"
        string nome "Simples"
        string endereco "Composto"
        string[] telefones "Multivalorado"
    }
```

### 4. Cardinalidade

Define o número de instâncias de uma entidade que podem estar relacionadas com instâncias de outra entidade.

#### Tipos de Cardinalidade

```mermaid
erDiagram
    CLIENTE ||--|| CONTA : "1:1"
    CLIENTE ||--o{ PEDIDO : "1:N"
    PRODUTO }o--o{ CATEGORIA : "N:M"
```

- **Um-para-Um (1:1)**
  - Cada instância se relaciona com no máximo uma instância
  - Exemplo: Pessoa ↔ CPF

- **Um-para-Muitos (1:N)**
  - Uma instância se relaciona com várias instâncias
  - Exemplo: Departamento ↔ Funcionários

- **Muitos-para-Muitos (N:M)**
  - Várias instâncias se relacionam com várias instâncias
  - Exemplo: Alunos ↔ Disciplinas

## Regras e Restrições

### 1. Integridade Referencial
- Garante consistência entre relacionamentos
- Previne referências inválidas
- Mantém a coerência dos dados

### 2. Participação
- **Total**: Todas as instâncias participam do relacionamento
- **Parcial**: Algumas instâncias podem não participar

### 3. Exclusividade
- Define se uma instância pode participar em múltiplos relacionamentos
- Estabelece restrições de unicidade

## Exemplos Práticos

### Sistema Acadêmico

```mermaid
erDiagram
    ALUNO ||--o{ MATRICULA : realiza
    ALUNO {
        string matricula PK
        string nome
        string email
    }
    DISCIPLINA ||--o{ MATRICULA : possui
    DISCIPLINA {
        string codigo PK
        string nome
        int creditos
    }
    MATRICULA {
        string aluno_id FK
        string disciplina_id FK
        float nota
        string status
    }
    PROFESSOR ||--o{ DISCIPLINA : ministra
    PROFESSOR {
        string id PK
        string nome
        string titulacao
    }
```

### Sistema de E-commerce

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : faz
    CLIENTE {
        string id PK
        string nome
        string email
    }
    PEDIDO ||--|{ ITEM_PEDIDO : contem
    PEDIDO {
        string numero PK
        date data
        float total
    }
    PRODUTO }|--|| ITEM_PEDIDO : compoe
    PRODUTO {
        string codigo PK
        string nome
        float preco
    }
```

## Considerações de Design

1. **Normalização vs. Desnormalização**
   - Equilíbrio entre integridade e desempenho
   - Decisões baseadas em requisitos

2. **Granularidade**
   - Nível adequado de detalhamento
   - Decomposição de entidades complexas

3. **Manutenibilidade**
   - Facilidade de evolução
   - Clareza na representação

## Conclusão

Os conceitos fundamentais do Modelo ER são essenciais para:
- Compreensão da estrutura de dados
- Comunicação entre stakeholders
- Base para implementação física
- Documentação do sistema

A aplicação correta destes conceitos resulta em modelos:
- Claros e compreensíveis
- Precisos e consistentes
- Flexíveis e extensíveis
- Implementáveis e manuteníveis