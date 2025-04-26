# INSERT: Inserindo Dados

```ascii
╔══════════════════════════════════════════════════════════════╗
║  DATA_MASTER >> Inserção de Dados                          ║
║  INSTRUTOR: INSERT_SPECIALIST                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Fundamentos do INSERT

INSERT_SPECIALIST explica: "O comando INSERT é fundamental para adicionar dados em suas tabelas."

### Sintaxe Básica

```sql
-- Inserção simples
INSERT INTO tabela (coluna1, coluna2) 
VALUES (valor1, valor2);

-- Múltiplos registros
INSERT INTO tabela (coluna1, coluna2) 
VALUES 
    (valor1, valor2),
    (valor3, valor4),
    (valor5, valor6);
```

## Formas de Inserção

### 1. Valores Explícitos

```sql
-- Com todas as colunas
INSERT INTO produtos (
    nome,
    preco,
    categoria,
    estoque
) VALUES (
    'Smartphone X',
    999.99,
    'Eletrônicos',
    100
);

-- Com colunas específicas
INSERT INTO usuarios (
    email,
    nome
) VALUES (
    'user@email.com',
    'João Silva'
);
```

### 2. Inserção via SELECT

DATA_TRANSFER_EXPERT demonstra: "Copie dados de outras tabelas!"

```sql
-- Inserção básica via SELECT
INSERT INTO produtos_backup
SELECT * FROM produtos
WHERE categoria = 'Eletrônicos';

-- Inserção seletiva
INSERT INTO resumo_vendas (produto, total_vendido)
SELECT 
    p.nome,
    SUM(v.quantidade)
FROM vendas v
JOIN produtos p ON v.produto_id = p.id
GROUP BY p.nome;
```

### 3. Valores Default e Expressões

```sql
-- Usando DEFAULT
INSERT INTO logs (
    evento,
    data_registro
) VALUES (
    'Login usuário',
    DEFAULT  -- Usa CURRENT_TIMESTAMP
);

-- Com expressões
INSERT INTO pedidos (
    produto_id,
    quantidade,
    valor_total
) VALUES (
    101,
    5,
    (SELECT preco FROM produtos WHERE id = 101) * 5
);
```

## Tratamento de Erros

ERROR_HANDLER apresenta: "Lidando com conflitos e erros"

```sql
-- Ignorar duplicados
INSERT INTO logs (id, mensagem)
ON CONFLICT (id) DO NOTHING;

-- Atualizar em caso de conflito
INSERT INTO produtos (codigo, nome, estoque)
VALUES ('P123', 'Novo Produto', 100)
ON CONFLICT (codigo) 
DO UPDATE SET 
    estoque = produtos.estoque + EXCLUDED.estoque;
```

## Inserção em Massa

BULK_INSERT_MASTER compartilha: "Otimize inserções grandes!"

```sql
-- Preparando statement
PREPARE insert_produto (text, decimal, int) AS
INSERT INTO produtos (nome, preco, estoque)
VALUES ($1, $2, $3);

-- Executando
EXECUTE insert_produto('Produto A', 99.99, 50);
EXECUTE insert_produto('Produto B', 149.99, 30);

-- Limpando
DEALLOCATE insert_produto;
```

## Boas Práticas

BEST_PRACTICES_SAGE recomenda:

### 1. Performance
- Use inserção em lote para múltiplos registros
- Considere desativar índices para cargas grandes
- Utilize transações apropriadamente
- Monitore o tamanho dos lotes

### 2. Integridade
- Valide dados antes da inserção
- Use constraints apropriadas
- Mantenha consistência referencial
- Trate valores nulos adequadamente

### 3. Manutenção
- Documente processos de carga
- Mantenha logs de inserção
- Implemente rollback strategy
- Monitore espaço em disco

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST INSERT:                        ║
║  □ Dados validados?                      ║
║  □ Constraints verificadas?              ║
║  □ Transação necessária?                 ║
║  □ Índices considerados?                 ║
║  □ Backup realizado?                     ║
╚════════════════════════════════════════════╝
```

## Exemplos Práticos

### Sistema de E-commerce

```sql
-- Inserindo novo produto
INSERT INTO produtos (
    codigo,
    nome,
    descricao,
    preco,
    estoque,
    categoria_id,
    criado_em,
    status
) VALUES (
    'PROD-123',
    'Smartphone Ultimate',
    'Smartphone de última geração',
    1299.99,
    50,
    (SELECT id FROM categorias WHERE nome = 'Eletrônicos'),
    CURRENT_TIMESTAMP,
    'ativo'
);

-- Registrando pedido
INSERT INTO pedidos (
    cliente_id,
    valor_total,
    status,
    data_pedido
) VALUES (
    1001,
    1299.99,
    'pendente',
    CURRENT_TIMESTAMP
);

-- Inserindo itens do pedido
INSERT INTO itens_pedido (
    pedido_id,
    produto_id,
    quantidade,
    preco_unitario
) VALUES (
    CURRVAL('pedidos_id_seq'),
    (SELECT id FROM produtos WHERE codigo = 'PROD-123'),
    1,
    1299.99
);
```

## Troubleshooting Comum

PROBLEM_SOLVER apresenta soluções para problemas frequentes:

1. **Violação de Chave Única**
```sql
-- Verificando existência
SELECT COUNT(*) 
FROM produtos 
WHERE codigo = 'PROD-123';

-- Inserção segura
INSERT INTO produtos (codigo, nome)
SELECT 'PROD-123', 'Novo Produto'
WHERE NOT EXISTS (
    SELECT 1 
    FROM produtos 
    WHERE codigo = 'PROD-123'
);
```

2. **Violação de Chave Estrangeira**
```sql
-- Verificando referência
SELECT id 
FROM categorias 
WHERE id = 5;

-- Inserção com verificação
INSERT INTO produtos (categoria_id, nome)
SELECT 5, 'Novo Produto'
WHERE EXISTS (
    SELECT 1 
    FROM categorias 
    WHERE id = 5
);
```

## Conclusão

INSERT_SPECIALIST conclui: "O domínio do INSERT vai além da sintaxe básica. Considere sempre performance, integridade e manutenibilidade em suas operações de inserção."

> **Dica Final**: Sempre teste suas inserções em ambiente de desenvolvimento antes de aplicar em produção.