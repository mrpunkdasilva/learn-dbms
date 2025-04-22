# Restrições no Modelo ER

As restrições no modelo ER são regras que garantem a integridade e consistência dos dados. Elas definem limites e condições que os dados devem satisfazer.

## Tipos de Restrições

### 1. Restrições de Chave

```mermaid
erDiagram
    ENTIDADE {
        string id PK "Chave Primária"
        string codigo UK "Chave Única"
        string ref FK "Chave Estrangeira"
    }
```

#### Chave Primária (PK)
- Identifica unicamente cada instância
- Não pode ser nula
- Não pode ser duplicada

#### Chave Estrangeira (FK)
- Referencia chave primária de outra entidade
- Mantém integridade referencial
- Pode ser nula (dependendo da participação)

#### Chave Única (UK)
- Garante valores únicos
- Pode ser nula (diferente da PK)
- Permite múltiplas por entidade

### 2. Restrições de Participação

```mermaid
erDiagram
    PEDIDO ||--|{ ITEM : "contém"
    PRODUTO }|--|| ITEM : "presente"
```

#### Participação Total
- Todas as instâncias devem participar
- Representada por linha dupla
- Exemplo: Todo item deve ter um produto

#### Participação Parcial
- Participação opcional
- Representada por linha simples
- Exemplo: Cliente pode não ter pedidos

### 3. Restrições de Sobreposição

```mermaid
erDiagram
    PESSOA ||--o{ PESSOA_FISICA : "é"
    PESSOA ||--o{ PESSOA_JURIDICA : "é"
```

#### Disjunção (XOR)
- Entidade participa em apenas um relacionamento
- Mutuamente exclusivo
- Exemplo: Pessoa Física XOR Jurídica

#### Sobreposição
- Entidade pode participar em múltiplos relacionamentos
- Não exclusivo
- Exemplo: Funcionário pode ser Cliente

### 4. Restrições de Cardinalidade

```mermaid
erDiagram
    DEPARTAMENTO ||--o{ FUNCIONARIO : "emprega"
    PROJETO }o--o{ FUNCIONARIO : "trabalha"
```

#### Cardinalidade Mínima
- Número mínimo de participações
- Exemplo: Zero (opcional) ou Um (obrigatório)

#### Cardinalidade Máxima
- Número máximo de participações
- Exemplo: Um ou Muitos (N)

## Implementação de Restrições

### 1. Nível de Banco de Dados

```sql
CREATE TABLE Produto (
    codigo VARCHAR(10) PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) CHECK (preco > 0)
);
```

### 2. Nível de Aplicação

```java
public class Produto {
    @Id
    private String codigo;
    
    @NotNull
    private String nome;
    
    @Positive
    private BigDecimal preco;
}
```

## Exemplos Práticos

### Sistema de Vendas

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
    PEDIDO ||--|{ ITEM_PEDIDO : "contém"
    PRODUTO }|--|| ITEM_PEDIDO : "incluído"
    
    CLIENTE {
        string cpf PK
        string nome UK
        string email UK
    }
    
    PEDIDO {
        string numero PK
        string cliente_cpf FK
        date data
        float total
    }
    
    ITEM_PEDIDO {
        string pedido_num FK
        string produto_cod FK
        int quantidade
        float preco_unit
    }
```

### Sistema Acadêmico

```mermaid
erDiagram
    ALUNO }o--o{ DISCIPLINA : "matriculado"
    PROFESSOR ||--o{ DISCIPLINA : "ministra"
    DEPARTAMENTO ||--o{ PROFESSOR : "emprega"
    
    ALUNO {
        string matricula PK
        string nome
        string email UK
    }
    
    DISCIPLINA {
        string codigo PK
        string nome
        int creditos
        string prof_id FK
    }
```

## Boas Práticas

### 1. Definição de Restrições
- Identifique regras de negócio
- Documente claramente
- Valide com stakeholders

### 2. Implementação
- Use mecanismos apropriados
- Mantenha consistência
- Considere performance

### 3. Manutenção
- Monitore violações
- Atualize quando necessário
- Mantenha documentação

## Considerações Importantes

### 1. Performance
- Impacto das restrições
- Otimização de verificações
- Estratégias de indexação

### 2. Integridade
- Consistência dos dados
- Validações necessárias
- Tratamento de exceções

### 3. Flexibilidade
- Evolução do modelo
- Mudanças de requisitos
- Manutenibilidade

## Padrões Comuns

### 1. Restrições de Negócio

```mermaid
erDiagram
    CONTA {
        string numero PK
        float saldo "CHECK >= 0"
        string status "CHECK IN ('ativo','inativo')"
    }
```

### 2. Restrições Temporais

```mermaid
erDiagram
    CONTRATO {
        string id PK
        date inicio "CHECK < fim"
        date fim "CHECK > inicio"
    }
```

## Conclusão

Restrições são essenciais para:
- Integridade dos dados
- Regras de negócio
- Consistência do modelo
- Qualidade da informação

Uma modelagem adequada de restrições:
- Previne inconsistências
- Facilita manutenção
- Melhora confiabilidade
- Garante qualidade