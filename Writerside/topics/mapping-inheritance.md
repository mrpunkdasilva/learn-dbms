# Mapeamento de Herança

O mapeamento de hierarquias de herança do modelo ER para o modelo relacional requer estratégias específicas para preservar a semântica e garantir eficiência. Este capítulo apresenta as principais abordagens e suas implicações.

## Visão Geral das Estratégias

```mermaid
mindmap
    root((Estratégias))
        Single Table
            Tabela única
            Discriminador
            Campos nulos
        Table Per Class
            Tabela por classe
            Redundância
            Independência
        Joined Tables
            Tabela base
            Tabelas derivadas
            Chaves estrangeiras
```

## Estratégias de Mapeamento

### 1. Single Table (Tabela Única)

#### Exemplo: Sistema Acadêmico

```mermaid
erDiagram
    PESSOA {
        string id PK
        string nome
        string tipo
        string matricula "Aluno"
        string siape "Professor"
        string departamento "Professor"
        float cr "Aluno"
        string turma "Aluno"
        string titulacao "Professor"
        string sala "Professor"
    }
```

#### Exemplo: Sistema de Veículos

```mermaid
erDiagram
    VEICULO {
        string placa PK
        string modelo
        string tipo
        int num_portas "Carro"
        float capacidade_carga "Caminhao"
        int num_eixos "Caminhao"
        int cilindradas "Moto"
        string tipo_guidao "Moto"
    }
```

#### Características
- Todos os atributos em uma única tabela
- Campo discriminador para identificar subclasses
- Atributos específicos podem ser nulos

#### Vantagens
- Consultas simples
- Sem necessidade de junções
- Fácil manutenção

#### Desvantagens
- Desperdício de espaço
- Muitos campos nulos
- Menor integridade de dados

### 2. Table Per Class (Tabela por Classe)

#### Exemplo: Sistema Acadêmico

```mermaid
erDiagram
    PESSOA {
        string id PK
        string nome
        string email
    }
    
    ALUNO {
        string id PK
        string nome
        string email
        string matricula
        float cr
        string turma
    }
    
    PROFESSOR {
        string id PK
        string nome
        string email
        string siape
        string departamento
        string titulacao
        string sala
    }
```

#### Exemplo: Sistema de Produtos

```mermaid
erDiagram
    PRODUTO {
        string codigo PK
        string nome
        float preco
    }
    
    ELETRONICO {
        string codigo PK
        string nome
        float preco
        string voltagem
        string garantia
        float potencia
    }
    
    LIVRO {
        string codigo PK
        string nome
        float preco
        string isbn
        string autor
        int paginas
    }
    
    ALIMENTO {
        string codigo PK
        string nome
        float preco
        date validade
        float peso
        string nutricional
    }
```

#### Características
- Cada classe tem sua própria tabela
- Todos os atributos são replicados
- Chaves independentes

#### Vantagens
- Modelo mais flexível
- Melhor integridade de dados
- Consultas específicas eficientes

#### Desvantagens
- Redundância de dados
- Consultas polimórficas complexas
- Maior espaço de armazenamento

### 3. Joined Tables (Tabelas Unidas)

#### Exemplo: Sistema Bancário

```mermaid
erDiagram
    CONTA ||--o| CONTA_CORRENTE : extends
    CONTA ||--o| CONTA_POUPANCA : extends
    CONTA ||--o| CONTA_INVESTIMENTO : extends
    
    CONTA {
        string numero PK
        string titular
        float saldo
        date abertura
    }
    
    CONTA_CORRENTE {
        string numero PK,FK
        float limite
        float taxa_mensal
    }
    
    CONTA_POUPANCA {
        string numero PK,FK
        float taxa_juros
        int dia_aniversario
    }
    
    CONTA_INVESTIMENTO {
        string numero PK,FK
        string perfil_risco
        float taxa_admin
        string carteira
    }
```

#### Exemplo: Sistema de Funcionários

```mermaid
erDiagram
    FUNCIONARIO ||--o| VENDEDOR : extends
    FUNCIONARIO ||--o| GERENTE : extends
    FUNCIONARIO ||--o| TECNICO : extends
    
    FUNCIONARIO {
        string matricula PK
        string nome
        date admissao
        float salario_base
    }
    
    VENDEDOR {
        string matricula PK,FK
        float comissao
        float meta_mensal
        string regiao
    }
    
    GERENTE {
        string matricula PK,FK
        string departamento
        int num_subordinados
        float bonus_anual
    }
    
    TECNICO {
        string matricula PK,FK
        string especialidade
        string certificacoes
        int nivel
    }
```

#### Características
- Tabela base para superclasse
- Tabelas separadas para subclasses
- Chaves estrangeiras para relacionamento

#### Vantagens
- Normalização completa
- Sem campos nulos
- Integridade referencial

#### Desvantagens
- Necessidade de junções
- Performance reduzida
- Complexidade de manutenção

## Casos Especiais

### 1. Herança Múltipla

```mermaid
erDiagram
    PESSOA ||--o| ALUNO_MONITOR : extends
    FUNCIONARIO ||--o| ALUNO_MONITOR : extends
    
    PESSOA {
        string id PK
        string nome
        string email
    }
    
    FUNCIONARIO {
        string matricula PK
        string nome
        float salario
    }
    
    ALUNO_MONITOR {
        string pessoa_id PK,FK
        string func_matricula PK,FK
        string disciplina
        int carga_horaria
        float bolsa
    }
```

### 2. Herança Hierárquica

```mermaid
erDiagram
    COLABORADOR ||--o| FUNCIONARIO : extends
    FUNCIONARIO ||--o| GERENTE : extends
    GERENTE ||--o| DIRETOR : extends
    
    COLABORADOR {
        string id PK
        string nome
        string email
    }
    
    FUNCIONARIO {
        string id PK,FK
        float salario
        string departamento
    }
    
    GERENTE {
        string id PK,FK
        float bonus
        int num_subordinados
    }
    
    DIRETOR {
        string id PK,FK
        float participacao_lucros
        string area_responsavel
    }
```

## Considerações de Design

### 1. Escolha da Estratégia

```mermaid
mindmap
    root((Fatores))
        Dados
            Volume
            Distribuição
            Crescimento
        Consultas
            Frequência
            Complexidade
            Performance
        Manutenção
            Evolução
            Flexibilidade
            Complexidade
```

#### Fatores de Decisão
- Volume de dados
- Padrão de consultas
- Requisitos de integridade
- Flexibilidade necessária

#### Recomendações
1. **Single Table**
   - Hierarquias simples
   - Poucos atributos específicos
   - Consultas frequentes polimórficas

2. **Table Per Class**
   - Subclasses muito diferentes
   - Consultas específicas frequentes
   - Poucos dados compartilhados

3. **Joined Tables**
   - Alta normalização necessária
   - Dados compartilhados importantes
   - Evolução frequente do esquema

### 2. Impacto nas Operações

```mermaid
erDiagram
    OPERACAO ||--o{ CONSULTA : inclui
    OPERACAO ||--o{ INSERCAO : inclui
    OPERACAO ||--o{ ATUALIZACAO : inclui
    
    OPERACAO {
        string tipo
        string estrategia
        int complexidade
        float performance
    }
    
    CONSULTA {
        string tipo
        bool polimorfismo
        int joins_necessarios
    }
    
    INSERCAO {
        string tipo
        int tabelas_afetadas
        bool cascata
    }
    
    ATUALIZACAO {
        string tipo
        bool propagacao
        int impacto
    }
```

### 2. Otimizações

#### Índices
- Chaves primárias
- Campos discriminadores
- Chaves estrangeiras
- Campos de consulta frequente

#### Restrições
- Integridade referencial
- Validações de tipo
- Regras de negócio

## Padrões e Anti-padrões

### Padrões Recomendados

1. **Discriminador Explícito**
   - Campo tipo sempre presente
   - Validações consistentes
   - Documentação clara

2. **Nomenclatura Consistente**
   - Prefixos/sufixos padronizados
   - Relacionamentos claros
   - Convenções estabelecidas

### Anti-padrões

1. **Mistura de Estratégias**
   - Inconsistência no modelo
   - Complexidade desnecessária
   - Difícil manutenção

2. **Herança Profunda**
   - Muitos níveis hierárquicos
   - Performance degradada
   - Complexidade aumentada

## Conclusão

A escolha da estratégia de mapeamento de herança deve considerar:

```mermaid
mindmap
    root((Decisão))
        Requisitos
            Performance
            Flexibilidade
            Manutenibilidade
        Características
            Volume
            Distribuição
            Acesso
        Técnico
            Sistema
            Ferramentas
            Infraestrutura
```
