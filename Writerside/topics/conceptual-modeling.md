# Modelagem Conceitual

## Diagramas Entidade-Relacionamento (ER)

Os diagramas Entidade-Relacionamento (ER) são uma ferramenta fundamental para modelagem conceitual de dados. Eles descrevem as relações entre diferentes entidades em um domínio específico de conhecimento.

### Componentes Básicos

#### 1. Entidades
Uma entidade representa um objeto ou conceito do mundo real. Por convenção, os nomes das entidades são escritos em maiúsculas e no singular.

```mermaid
erDiagram
    CLIENTE {
        string nome
        string cpf PK "Identificador único"
        string email UK
        string telefone
    }
```

#### 2. Relacionamentos
Os relacionamentos descrevem como as entidades se conectam entre si. A cardinalidade indica quantas instâncias de uma entidade podem se relacionar com outra.

##### Tipos de Cardinalidade:
- `||--||` : Exatamente um para um
- `||--o{` : Um para muitos (zero ou mais)
- `||--|{` : Um para muitos (pelo menos um)
- `}o--o{` : Muitos para muitos (zero ou mais)

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
    PEDIDO ||--|{ ITEM_PEDIDO : "contém"
    PRODUTO }|--|| ITEM_PEDIDO : "está em"
```

#### 3. Atributos
Os atributos são características que descrevem uma entidade.

Tipos de Atributos:
- `PK` : Chave Primária
- `FK` : Chave Estrangeira
- `UK` : Chave Única

```mermaid
erDiagram
    PRODUTO {
        string codigo PK "Código único do produto"
        string nome
        float preco
        int estoque
        string categoria FK "Referência à categoria"
    }
    CATEGORIA {
        string id PK
        string nome UK
        string descricao
    }
    PRODUTO }|--|| CATEGORIA : "pertence"
```

### Exemplo Completo de Modelagem

Vamos modelar um sistema de e-commerce:

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
    CLIENTE {
        string cpf PK
        string nome
        string email UK
        string endereco
    }
    PEDIDO ||--|{ ITEM_PEDIDO : "contém"
    PEDIDO {
        string numero PK
        date data
        float valorTotal
        string status
        string cpfCliente FK
    }
    PRODUTO }|--|| ITEM_PEDIDO : "compõe"
    PRODUTO {
        string codigo PK
        string nome
        float preco
        int estoque
        string idCategoria FK
    }
    ITEM_PEDIDO {
        string numeroPedido PK,FK
        string codigoProduto PK,FK
        int quantidade
        float precoUnitario
    }
    CATEGORIA ||--o{ PRODUTO : "classifica"
    CATEGORIA {
        string id PK
        string nome UK
        string descricao
    }
```

### Boas Práticas

1. **Nomenclatura**
   - Use nomes significativos
   - Mantenha consistência
   - Evite abreviações ambíguas

2. **Cardinalidade**
   - Defina claramente as restrições
   - Considere casos extremos
   - Documente as regras de negócio

3. **Atributos**
   - Identifique chaves primárias
   - Estabeleça chaves estrangeiras
   - Defina atributos obrigatórios

### Direções do Diagrama

Os diagramas ER podem ser orientados em diferentes direções:

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
    PEDIDO ||--|{ ITEM : "contém"
```

### Estilização

É possível personalizar a aparência dos diagramas:

```mermaid
erDiagram
    CLIENTE ||--o{ PEDIDO : "realiza"
    PEDIDO ||--|{ ITEM : "contém"
```

## Conclusão

A modelagem ER é uma técnica poderosa para:
- Visualizar estruturas de dados
- Documentar requisitos
- Comunicar com stakeholders
- Planejar implementações

O uso de diagramas Mermaid torna a criação e manutenção desses modelos mais eficiente e integrada com documentação moderna.
