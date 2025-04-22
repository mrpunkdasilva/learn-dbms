# Atributos

Atributos são propriedades ou características que descrevem entidades e relacionamentos em um modelo ER.

## Tipos de Atributos

### 1. Quanto à Estrutura

#### Atributos Simples
- Valores atômicos, indivisíveis
- Exemplo: CPF, idade, email

```mermaid
erDiagram
    CLIENTE {
        string cpf PK
        int idade
        string email
    }
```

#### Atributos Compostos
- Podem ser divididos em partes menores
- Exemplo: endereço (rua, número, cidade)

```mermaid
erDiagram
    FUNCIONARIO {
        string id PK
        string nome
        string end_rua
        string end_numero
        string end_cidade
        string end_estado
    }
```

#### Atributos Multivalorados
- Podem ter múltiplos valores
- Exemplo: telefones, emails

```mermaid
erDiagram
    PESSOA {
        string id PK
        string nome
        string[] telefones "Múltiplos valores"
        string[] emails "Múltiplos valores"
    }
```

### 2. Quanto à Função

#### Atributos Identificadores (Chaves)
- **Chave Primária (PK)**
  - Identifica unicamente cada instância
  - Não pode ser nulo ou duplicado

```mermaid
erDiagram
    PRODUTO {
        string codigo PK "Identificador único"
        string nome
        float preco
    }
```

- **Chave Estrangeira (FK)**
  - Referencia chave primária de outra entidade
  - Estabelece relacionamentos

```mermaid
erDiagram
    PEDIDO {
        string numero PK
        string cliente_id FK "Referência ao cliente"
        date data
        float valor
    }
```

- **Chave Única (UK)**
  - Valor único mas não primário
  - Exemplo: email, matrícula

```mermaid
erDiagram
    USUARIO {
        string id PK
        string email UK "Único mas não primário"
        string nome
    }
```

#### Atributos Descritivos
- Descrevem características
- Não são identificadores
- Exemplo: nome, descrição, data

```mermaid
erDiagram
    LIVRO {
        string isbn PK
        string titulo "Descritivo"
        string autor "Descritivo"
        int paginas "Descritivo"
    }
```

### 3. Quanto ao Valor

#### Atributos Obrigatórios
- Não podem ser nulos
- Essenciais para a entidade

```mermaid
erDiagram
    FUNCIONARIO {
        string matricula PK "Obrigatório"
        string nome "Obrigatório"
        string cargo "Obrigatório"
        string telefone "Opcional"
    }
```

#### Atributos Opcionais
- Podem ser nulos
- Não essenciais

#### Atributos Derivados
- Calculados a partir de outros
- Exemplo: idade (calculada da data de nascimento)

```mermaid
erDiagram
    PESSOA {
        string id PK
        string nome
        date data_nascimento
        int idade "Derivado"
    }
```

## Boas Práticas

### 1. Nomenclatura
- Use nomes significativos
- Mantenha padrão consistente
- Evite abreviações ambíguas

### 2. Tipos de Dados
- Escolha tipos apropriados
- Defina tamanhos adequados
- Considere restrições

### 3. Normalização
- Evite redundância
- Mantenha atomicidade
- Considere dependências

## Exemplos Práticos

### Sistema de Vendas

```mermaid
erDiagram
    PRODUTO {
        string codigo PK
        string nome
        string descricao
        float preco
        int estoque
        string categoria FK
        date data_cadastro
        boolean ativo
    }
```

### Sistema de RH

```mermaid
erDiagram
    FUNCIONARIO {
        string matricula PK
        string nome
        date data_admissao
        float salario_base
        float bonus "Derivado"
        string departamento FK
        string[] habilidades
        string end_rua
        string end_numero
        string end_cidade
    }
```

## Considerações Importantes

### 1. Integridade
- Defina restrições adequadas
- Valide valores permitidos
- Mantenha consistência

### 2. Performance
- Otimize tipos de dados
- Planeje índices
- Considere volume

### 3. Manutenibilidade
- Documente decisões
- Facilite evolução
- Mantenha simplicidade

## Padrões Comuns

### 1. Atributos de Auditoria
```mermaid
erDiagram
    ENTIDADE {
        string id PK
        string nome
        date criado_em
        string criado_por
        date alterado_em
        string alterado_por
    }
```

### 2. Atributos de Status
```mermaid
erDiagram
    PEDIDO {
        string numero PK
        string status
        boolean ativo
        date data_status
    }
```

## Conclusão

Atributos são fundamentais para:
- Descrição de dados
- Integridade do modelo
- Funcionalidade do sistema
- Qualidade da informação

A modelagem adequada de atributos:
- Facilita manutenção
- Melhora performance
- Garante consistência
- Suporta evolução