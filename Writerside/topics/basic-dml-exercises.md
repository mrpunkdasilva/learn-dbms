# Exercícios Básicos de DML

## Exercício 1: Consultas Simples

### Descrição
Pratique consultas básicas para recuperar dados das tabelas.

### Requisitos
- Selecionar todos os clientes
- Filtrar produtos por preço
- Ordenar resultados
- Limitar número de registros

### Solução
```sql
-- Selecionar todos os clientes
SELECT * FROM pratica.clientes;

-- Filtrar produtos por preço
SELECT nome, preco 
FROM pratica.produtos 
WHERE preco < 1000;

-- Ordenar resultados
SELECT nome, preco 
FROM pratica.produtos 
ORDER BY preco DESC;

-- Limitar número de registros
SELECT nome, email 
FROM pratica.clientes 
LIMIT 5;
```

## Exercício 2: Inserção de Dados

### Descrição
Pratique a inserção de novos registros nas tabelas.

### Requisitos
- Inserir um único cliente
- Inserir múltiplos produtos
- Inserir com valores padrão
- Inserir com subconsulta

### Solução
```sql
-- Inserir um único cliente
INSERT INTO pratica.clientes (nome, email)
VALUES ('Daniel Santos', 'daniel@email.com');

-- Inserir múltiplos produtos
INSERT INTO pratica.produtos (nome, preco, estoque)
VALUES 
    ('Mouse', 89.90, 30),
    ('Teclado', 149.90, 25),
    ('Monitor', 899.90, 10);

-- Inserir com valores padrão
INSERT INTO pratica.clientes (nome)
VALUES ('Eduardo Lima');

-- Inserir com subconsulta
INSERT INTO pratica.pedidos (cliente_id, status)
SELECT id, 'Novo' 
FROM pratica.clientes 
WHERE nome = 'Ana Silva';
```

## Exercício 3: Atualização de Dados

### Descrição
Pratique a atualização de registros existentes.

### Requisitos
- Atualizar um único registro
- Atualizar múltiplos registros
- Atualizar com condição
- Atualizar com expressão

### Solução
```sql
-- Atualizar um único registro
UPDATE pratica.clientes
SET email = 'ana.silva@email.com'
WHERE id = 1;

-- Atualizar múltiplos registros
UPDATE pratica.produtos
SET estoque = estoque + 10
WHERE estoque < 30;

-- Atualizar com condição
UPDATE pratica.pedidos
SET status = 'Aprovado'
WHERE cliente_id IN (
    SELECT id FROM pratica.clientes
    WHERE nome LIKE 'Ana%'
);

-- Atualizar com expressão
UPDATE pratica.produtos
SET preco = preco * 1.1
WHERE nome = 'Smartphone';
```

## Exercício 4: Exclusão de Dados

### Descrição
Pratique a exclusão de registros das tabelas.

### Requisitos
- Excluir um único registro
- Excluir múltiplos registros
- Excluir com condição
- Excluir com subconsulta

### Solução
```sql
-- Excluir um único registro
DELETE FROM pratica.clientes
WHERE id = 5;

-- Excluir múltiplos registros
DELETE FROM pratica.produtos
WHERE estoque = 0;

-- Excluir com condição
DELETE FROM pratica.pedidos
WHERE status = 'Cancelado'
AND data_pedido < CURRENT_DATE - INTERVAL '30 days';

-- Excluir com subconsulta
DELETE FROM pratica.itens_pedido
WHERE pedido_id IN (
    SELECT id FROM pratica.pedidos
    WHERE status = 'Cancelado'
);
```

## Exercício 5: Consultas com Junções

### Descrição
Pratique consultas que relacionam dados de múltiplas tabelas.

### Requisitos
- Junção interna (INNER JOIN)
- Junção externa (LEFT JOIN)
- Múltiplas junções
- Filtros em junções

### Solução
```sql
-- Junção interna
SELECT c.nome, p.id as pedido_id
FROM pratica.clientes c
INNER JOIN pratica.pedidos p ON c.id = p.cliente_id;

-- Junção externa
SELECT c.nome, p.id as pedido_id
FROM pratica.clientes c
LEFT JOIN pratica.pedidos p ON c.id = p.cliente_id;

-- Múltiplas junções
SELECT c.nome, p.id as pedido_id, pr.nome as produto
FROM pratica.clientes c
JOIN pratica.pedidos p ON c.id = p.cliente_id
JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
JOIN pratica.produtos pr ON ip.produto_id = pr.id;

-- Filtros em junções
SELECT c.nome, p.id as pedido_id
FROM pratica.clientes c
JOIN pratica.pedidos p ON c.id = p.cliente_id
WHERE p.status = 'Aprovado'
AND c.data_cadastro > '2023-01-01';
```

## Exercício 6: Funções Agregadas

### Descrição
Pratique o uso de funções agregadas para análise de dados.

### Requisitos
- Contagem de registros
- Soma de valores
- Média, mínimo e máximo
- Agrupamento de resultados

### Solução
```sql
-- Contagem de registros
SELECT COUNT(*) as total_clientes
FROM pratica.clientes;

-- Soma de valores
SELECT SUM(estoque) as estoque_total
FROM pratica.produtos;

-- Média, mínimo e máximo
SELECT 
    AVG(preco) as preco_medio,
    MIN(preco) as preco_minimo,
    MAX(preco) as preco_maximo
FROM pratica.produtos;

-- Agrupamento de resultados
SELECT status, COUNT(*) as quantidade
FROM pratica.pedidos
GROUP BY status
ORDER BY quantidade DESC;
```

## Exercício 7: Subconsultas

### Descrição
Pratique o uso de subconsultas para operações mais complexas.

### Requisitos
- Subconsulta no WHERE
- Subconsulta no FROM
- Subconsulta no SELECT
- Subconsulta com EXISTS

### Solução
```sql
-- Subconsulta no WHERE
SELECT nome
FROM pratica.clientes
WHERE id IN (
    SELECT cliente_id
    FROM pratica.pedidos
    WHERE status = 'Aprovado'
);

-- Subconsulta no FROM
SELECT temp.status, temp.total
FROM (
    SELECT status, COUNT(*) as total
    FROM pratica.pedidos
    GROUP BY status
) as temp
WHERE temp.total > 5;

-- Subconsulta no SELECT
SELECT 
    nome,
    (SELECT COUNT(*) FROM pratica.pedidos WHERE cliente_id = c.id) as total_pedidos
FROM pratica.clientes c;

-- Subconsulta com EXISTS
SELECT nome
FROM pratica.produtos p
WHERE EXISTS (
    SELECT 1 
    FROM pratica.itens_pedido ip
    WHERE ip.produto_id = p.id
    AND ip.quantidade > 5
);
```

## Exercício 8: Operações com Datas

### Descrição
Pratique operações e manipulações com campos de data.

### Requisitos
- Filtrar por data
- Calcular diferença entre datas
- Extrair componentes de data
- Agrupar por período

### Solução
```sql
-- Filtrar por data
SELECT *
FROM pratica.pedidos
WHERE data_pedido >= '2023-01-01'
AND data_pedido < '2023-02-01';

-- Calcular diferença entre datas
SELECT 
    id,
    data_pedido,
    CURRENT_DATE as hoje,
    CURRENT_DATE - data_pedido::date as dias_decorridos
FROM pratica.pedidos;

-- Extrair componentes de data
SELECT 
    id,
    data_pedido,
    EXTRACT(YEAR FROM data_pedido) as ano,
    EXTRACT(MONTH FROM data_pedido) as mes,
    EXTRACT(DAY FROM data_pedido) as dia
FROM pratica.pedidos;

-- Agrupar por período
SELECT 
    EXTRACT(MONTH FROM data_pedido) as mes,
    COUNT(*) as total_pedidos
FROM pratica.pedidos
GROUP BY mes
ORDER BY mes;
```

## Exercício 9: Operações com Strings

### Descrição
Pratique operações e manipulações com campos de texto.

### Requisitos
- Concatenação de strings
- Conversão de maiúsculas/minúsculas
- Extração de substrings
- Busca por padrões

### Solução
```sql
-- Concatenação de strings
SELECT 
    nome,
    email,
    nome || ' <' || email || '>' as contato
FROM pratica.clientes;

-- Conversão de maiúsculas/minúsculas
SELECT 
    nome,
    UPPER(nome) as nome_maiusculo,
    LOWER(email) as email_minusculo
FROM pratica.clientes;

-- Extração de substrings
SELECT 
    nome,
    SUBSTRING(nome FROM 1 FOR 3) as iniciais,
    SUBSTRING(email FROM POSITION('@' IN email) + 1) as dominio
FROM pratica.clientes;

-- Busca por padrões
SELECT nome, email
FROM pratica.clientes
WHERE nome LIKE 'A%'
OR email LIKE '%@gmail.com';
```

## Exercício 10: Transações

### Descrição
Pratique o uso de transações para garantir a integridade dos dados.

### Requisitos
- Iniciar transação
- Confirmar alterações
- Reverter alterações
- Pontos de salvamento

### Solução
```sql
-- Iniciar transação
BEGIN;

-- Operações dentro da transação
INSERT INTO pratica.clientes (nome, email)
VALUES ('Fernando Gomes', 'fernando@email.com');

UPDATE pratica.produtos
SET estoque = estoque - 5
WHERE nome = 'Smartphone';

-- Confirmar alterações
COMMIT;

-- Exemplo com rollback
BEGIN;

UPDATE pratica.produtos
SET preco = preco * 0.5;

-- Ops, desconto muito alto!
ROLLBACK;

-- Exemplo com savepoint
BEGIN;

INSERT INTO pratica.clientes (nome, email)
VALUES ('Gabriela Lima', 'gabriela@email.com');

SAVEPOINT novo_cliente;

UPDATE pratica.produtos
SET estoque = -10;

-- Erro no estoque, voltar ao savepoint
ROLLBACK TO novo_cliente;

INSERT INTO pratica.pedidos (cliente_id, status)
VALUES ((SELECT id FROM pratica.clientes WHERE nome = 'Gabriela Lima'), 'Novo');

COMMIT;
```

## Critérios de Avaliação

- Sintaxe SQL correta
- Uso apropriado dos comandos DML
- Eficiência das consultas
- Tratamento adequado de erros
- Uso de transações quando necessário

## Dicas de Estudo

- Teste cada comando separadamente
- Verifique os resultados após cada operação
- Pratique com diferentes conjuntos de dados
- Experimente variações dos comandos
- Analise o plano de execução das consultas

---

> **Nota**: Estes exercícios são fundamentais para construir uma base sólida em manipulação de dados. Certifique-se de compreender completamente cada conceito antes de avançar para os exercícios intermediários.