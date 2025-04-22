# Relacionamentos

Um relacionamento representa uma associação entre duas ou mais entidades, descrevendo como elas interagem entre si no contexto do domínio.

## Características dos Relacionamentos

### 1. Grau do Relacionamento

#### Relacionamento Binário
- Envolve duas entidades
- Tipo mais comum
- Exemplo: Cliente realiza Pedido

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
```

#### Relacionamento Ternário
- Envolve três entidades
- Usado em casos específicos
- Exemplo: Professor leciona Disciplina para Turma

```mermaid
erDiagram
    PROFESSOR ||--o{ ALOCACAO : "participa"
    DISCIPLINA ||--o{ ALOCACAO : "participa"
    TURMA ||--o{ ALOCACAO : "participa"
    ALOCACAO {
        string professor_id FK
        string disciplina_id FK
        string turma_id FK
        int carga_horaria
    }
```

#### Relacionamento Recursivo
- Uma entidade se relaciona consigo mesma
- Exemplo: Funcionário supervisiona Funcionário

```mermaid
erDiagram
    FUNCIONARIO ||--o{ FUNCIONARIO : "supervisiona"
```

### 2. Cardinalidade

#### Um-para-Um (1:1)
- Cada instância se relaciona com no máximo uma outra
- Exemplo: Pessoa tem um CPF

```mermaid
erDiagram
    PESSOA ||--|| CPF : "possui"
```

#### Um-para-Muitos (1:N)
- Uma instância se relaciona com várias outras
- Exemplo: Departamento tem vários Funcionários

```mermaid
erDiagram
    DEPARTAMENTO ||--o{ FUNCIONARIO : "possui"
```

#### Muitos-para-Muitos (N:M)
- Várias instâncias se relacionam com várias outras
- Exemplo: Aluno cursa várias Disciplinas

```mermaid
erDiagram
    ALUNO }o--o{ DISCIPLINA : "cursa"
```

### 3. Participação

#### Participação Total
- Todas as instâncias da entidade participam do relacionamento
- Representada por linha dupla

```mermaid
erDiagram
    PEDIDO ||--|{ ITEM_PEDIDO : "contém"
```

#### Participação Parcial
- Algumas instâncias podem não participar
- Representada por linha simples

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
```

## Atributos em Relacionamentos

Relacionamentos podem ter seus próprios atributos.

```mermaid
erDiagram
    FUNCIONARIO }|--|| PROJETO : "trabalha em"
    TRABALHO {
        string func_id FK
        string proj_id FK
        int horas
        date data_inicio
        string funcao
    }
```

## Papéis em Relacionamentos

Cada entidade desempenha um papel específico no relacionamento.

```mermaid
erDiagram
    PESSOA ||--o{ PESSOA : "é pai de"
```

## Exemplos Práticos

### Sistema Acadêmico

```mermaid
erDiagram
    PROFESSOR ||--o{ TURMA : "leciona"
    ALUNO }o--o{ TURMA : "matriculado"
    DISCIPLINA ||--o{ TURMA : "oferecida"
    TURMA {
        string codigo PK
        string semestre
        int vagas
        string horario
    }
```

### Sistema de Vendas

```mermaid
erDiagram
    VENDEDOR ||--o{ VENDA : "realiza"
    CLIENTE ||--o{ VENDA : "efetua"
    PRODUTO }o--o{ VENDA : "incluído"
    VENDA {
        string numero PK
        date data
        float valor_total
        string status
    }
```

## Boas Práticas

### 1. Nomenclatura
- Use verbos para nomes de relacionamentos
- Seja claro e específico
- Mantenha consistência

### 2. Design
- Evite relacionamentos redundantes
- Minimize relacionamentos complexos
- Documente restrições importantes

### 3. Implementação
- Considere o impacto na performance
- Planeje índices adequados
- Mantenha a integridade referencial

## Considerações de Modelagem

### 1. Normalização
- Balance normalização com performance
- Considere requisitos de consulta
- Avalie impacto nas operações

### 2. Restrições
- Defina regras de negócio
- Implemente validações
- Mantenha consistência

### 3. Evolução
- Planeje para mudanças
- Documente decisões
- Mantenha flexibilidade

## Padrões Comuns

### 1. Associação Simples
```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
```

### 2. Agregação
```mermaid
erDiagram
    DEPARTAMENTO ||--o{ FUNCIONARIO : "contém"
    FUNCIONARIO }|--|| PROJETO : "trabalha"
```

### 3. Composição
```mermaid
erDiagram
    PEDIDO ||--|{ ITEM_PEDIDO : "composto"
```

## Conclusão

Relacionamentos são essenciais para:
- Estruturação dos dados
- Integridade do modelo
- Representação do negócio
- Implementação eficiente

A modelagem adequada de relacionamentos:
- Facilita a manutenção
- Melhora a performance
- Garante consistência
- Suporta evolução