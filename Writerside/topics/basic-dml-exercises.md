# Exercícios Básicos de DML

## Exercício 1: Consultas Simples

### Descrição
Pratique consultas básicas para recuperar dados das tabelas.

### Requisitos
- Selecionar todos os registros
- Selecionar colunas específicas
- Filtrar com WHERE
- Ordenar resultados

### Solução
```sql
-- Selecionar todos os registros
SELECT * FROM pratica.clientes;

-- Selecionar colunas específicas
SELECT nome, email FROM pratica.produtos;

-- Filtrar com WHERE
SELECT * FROM pratica.produtos
WHERE preco < 1000;

-- Ordenar resultados
SELECT * FROM pratica.clientes
ORDER BY nome ASC;
```

## Exercício 2: Inserção de Dados

### Descrição
Pratique a inserção de novos registros nas tabelas.

### Requisitos
- Inserir um único registro
- Inserir múltiplos registros
- Inserir com valores padrão
- Inserir com subconsulta

### Solução
```sql
-- Inserir um único registro
INSERT INTO pratica.clientes (nome, email)
VALUES ('Daniel Santos', 'daniel@email.com');

-- Inserir múltiplos registros
INSERT INTO pratica.produtos (nome, preco, estoque)
VALUES 
    ('Tablet', 899.99, 30),
    ('Mouse', 49.99, 100);

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
WHERE produto_id IN (
    SELECT id FROM pratica.produtos
    WHERE preco > 3000
);
```

## Exercício 5: Consultas com Funções

### Descrição
Pratique consultas utilizando funções SQL básicas.

### Requisitos
- Funções de agregação
- Funções de string
- Funções de data
- Funções matemáticas

### Solução
```sql
-- Funções de agregação
SELECT 
    COUNT(*) as total_clientes,
    MAX(data_cadastro) as cadastro_mais_recente
FROM pratica.clientes;

-- Funções de string
SELECT 
    UPPER(nome) as nome_maiusculo,
    LENGTH(email) as tamanho_email
FROM pratica.clientes;

-- Funções de data
SELECT 
    id,
    data_pedido,
    EXTRACT(MONTH FROM data_pedido) as mes,
    EXTRACT(YEAR FROM data_pedido) as ano
FROM pratica.pedidos;

-- Funções matemáticas
SELECT 
    nome,
    preco,
    ROUND(preco * 0.9, 2) as preco_com_desconto
FROM pratica.produtos;
```

## Exercício 6: Consultas com GROUP BY

### Descrição
Pratique consultas com agrupamento de dados.

### Requisitos
- Agrupar por uma coluna
- Agrupar por múltiplas colunas
- Filtrar grupos com HAVING
- Ordenar grupos

### Solução
```sql
-- Agrupar por uma coluna
SELECT 
    status,
    COUNT(*) as total_pedidos
FROM pratica.pedidos
GROUP BY status;

-- Agrupar por múltiplas colunas
SELECT 
    EXTRACT(YEAR FROM data_pedido) as ano,
    EXTRACT(MONTH FROM data_pedido) as mes,
    status,
    COUNT(*) as total_pedidos
FROM pratica.pedidos
GROUP BY ano, mes, status;

-- Filtrar grupos com HAVING
SELECT 
    cliente_id,
    COUNT(*) as total_pedidos
FROM pratica.pedidos
GROUP BY cliente_id
HAVING COUNT(*) > 3;

-- Ordenar grupos
SELECT 
    status,
    COUNT(*) as total_pedidos
FROM pratica.pedidos
GROUP BY status
ORDER BY total_pedidos DESC;
```

## Exercício 7: Consultas com JOIN

### Descrição
Pratique consultas que relacionam múltiplas tabelas.

### Requisitos
- INNER JOIN
- LEFT JOIN
- Múltiplos JOINs
- Filtros com JOIN

### Solução
```sql
-- INNER JOIN
SELECT 
    c.nome as cliente,
    p.id as pedido_id,
    p.data_pedido
FROM pratica.clientes c
INNER JOIN pratica.pedidos p ON c.id = p.cliente_id;

-- LEFT JOIN
SELECT 
    c.nome as cliente,
    COUNT(p.id) as total_pedidos
FROM pratica.clientes c
LEFT JOIN pratica.pedidos p ON c.id = p.cliente_id
GROUP BY c.nome;

-- Múltiplos JOINs
SELECT 
    c.nome as cliente,
    p.id as pedido_id,
    pr.nome as produto,
    ip.quantidade
FROM pratica.clientes c
JOIN pratica.pedidos p ON c.id = p.cliente_id
JOIN pratica.itens_pedido ip ON p.id = ip.pedido_id
JOIN pratica.produtos pr ON ip.produto_id = pr.id;

-- Filtros com JOIN
SELECT 
    c.nome as cliente,
    p.id as pedido_id
FROM pratica.clientes c
JOIN pratica.pedidos p ON c.id = p.cliente_id
WHERE p.status = 'Aprovado'
AND p.data_pedido > CURRENT_DATE - INTERVAL '7 days';
```

## Exercício 8: Subconsultas

### Descrição
Pratique o uso de subconsultas em operações DML.

### Requisitos
- Subconsulta no WHERE
- Subconsulta no FROM
- Subconsulta no SELECT
- Subconsulta com operadores de comparação

### Solução
```sql
-- Subconsulta no WHERE
SELECT * FROM pratica.clientes
WHERE id IN (
    SELECT DISTINCT cliente_id
    FROM pratica.pedidos
    WHERE status = 'Aprovado'
);

-- Subconsulta no FROM
SELECT 
    cliente,
    total_pedidos
FROM (
    SELECT 
        c.nome as cliente,
        COUNT(p.id) as total_pedidos
    FROM pratica.clientes c
    LEFT JOIN pratica.pedidos p ON c.id = p.cliente_id
    GROUP BY c.nome
) as resumo
WHERE total_pedidos > 0;

-- Subconsulta no SELECT
SELECT 
    p.id,
    p.nome,
    p.preco,
    (SELECT AVG(preco) FROM pratica.produtos) as preco_medio,
    p.preco - (SELECT AVG(preco) FROM pratica.produtos) as diferenca
FROM pratica.produtos p;

-- Subconsulta com operadores de comparação
SELECT * FROM pratica.produtos
WHERE preco > (
    SELECT AVG(preco) FROM pratica.produtos
);
```

## Exercício 9: Operadores de Conjunto

### Descrição
Pratique o uso de operadores de conjunto em consultas.

### Requisitos
- UNION
- INTERSECT
- EXCEPT
- Combinação de operadores

### Solução
```sql
-- UNION
SELECT nome, email, 'Cliente' as tipo
FROM pratica.clientes
UNION
SELECT nome, 'N/A', 'Produto' as tipo
FROM pratica.produtos;

-- INTERSECT
SELECT cliente_id
FROM pratica.pedidos
WHERE status = 'Aprovado'
INTERSECT
SELECT cliente_id
FROM pratica.pedidos
WHERE status = 'Entregue';

-- EXCEPT
SELECT id FROM pratica.clientes
EXCEPT
SELECT cliente_id FROM pratica.pedidos;

-- Combinação de operadores
(SELECT cliente_id FROM pratica.pedidos WHERE status = 'Aprovado')
UNION
(SELECT cliente_id FROM pratica.pedidos WHERE status = 'Entregue')
EXCEPT
(SELECT cliente_id FROM pratica.pedidos WHERE status = 'Cancelado');
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
SET estoque = 0;

-- Ops, não queremos zerar o estoque!
ROLLBACK TO novo_cliente;

-- Continuar com outras operações
INSERT INTO pratica.pedidos (cliente_id, status)
VALUES ((SELECT id FROM pratica.clientes WHERE nome = 'Gabriela Lima'), 'Novo');

COMMIT;
```

## Critérios de Avaliação

- Sintaxe correta
- Uso adequado de cláusulas e operadores
- Eficiência das consultas
- Integridade dos dados após manipulação
- Aplicação de boas práticas

## Dicas de Estudo

- Teste cada comando separadamente
- Verifique os resultados após cada operação
- Pratique com diferentes conjuntos de dados
- Experimente variações dos comandos
- Analise o plano de execução das consultas

---

> **Nota**: Estes exercícios são fundamentais para construir uma base sólida em manipulação de dados. Certifique-se de compreender completamente cada conceito antes de avançar para os exercícios intermediários.
