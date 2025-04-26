# SELECT: Recuperando Dados

```ascii
╔══════════════════════════════════════════════════════════════╗
║  QUERY_MASTER >> Arte da Consulta de Dados                 ║
║  INSTRUTOR: SELECT_SPECIALIST                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Fundamentos do SELECT

SELECT_SPECIALIST explica: "O comando SELECT é a base para recuperação de dados em SQL."

### Sintaxe Básica
```sql
SELECT coluna1, coluna2
FROM tabela
WHERE condição
GROUP BY coluna
HAVING condição_grupo
ORDER BY coluna;
```

## Cláusulas Principais

### 1. FROM
DATA_SOURCE_EXPERT demonstra: "Define a fonte dos dados"

```sql
-- Tabela única
FROM produtos

-- Múltiplas tabelas
FROM produtos p, categorias c

-- Subquery
FROM (SELECT * FROM vendas WHERE ano = 2023) v
```

### 2. WHERE
FILTER_MASTER apresenta: "Filtra os registros"

```sql
-- Comparações básicas
WHERE preco > 100
AND categoria = 'Eletrônicos'

-- Operadores IN/BETWEEN
WHERE status IN ('Ativo', 'Pendente')
AND data_criacao BETWEEN '2023-01-01' AND '2023-12-31'

-- Pattern matching
WHERE nome LIKE 'A%'
AND descricao ILIKE '%premium%'
```

### 3. JOIN
JOIN_SPECIALIST explica: "Combina dados de múltiplas tabelas"

```sql
-- INNER JOIN
SELECT p.nome, c.categoria
FROM produtos p
INNER JOIN categorias c ON p.categoria_id = c.id

-- LEFT JOIN
SELECT c.nome, COUNT(p.id) as total_pedidos
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.nome

-- Multiple JOINs
SELECT p.nome, c.categoria, f.nome as fornecedor
FROM produtos p
JOIN categorias c ON p.categoria_id = c.id
JOIN fornecedores f ON p.fornecedor_id = f.id
```

### 4. GROUP BY
AGGREGATION_MASTER demonstra: "Agrupa resultados"

```sql
-- Agrupamento simples
SELECT categoria, COUNT(*) as total
FROM produtos
GROUP BY categoria

-- Múltiplas colunas
SELECT categoria, status, SUM(valor) as total
FROM vendas
GROUP BY categoria, status

-- Com HAVING
SELECT cliente_id, COUNT(*) as total_pedidos
FROM pedidos
GROUP BY cliente_id
HAVING COUNT(*) > 5
```

### 5. ORDER BY
SORT_EXPERT apresenta: "Ordena resultados"

```sql
-- Ordenação simples
ORDER BY data_criacao DESC

-- Múltiplas colunas
ORDER BY categoria ASC, preco DESC

-- Com expressões
ORDER BY (preco * quantidade) DESC
```

## Funções e Expressões

### 1. Agregação
```sql
SELECT 
    COUNT(*) as total,
    SUM(valor) as valor_total,
    AVG(preco) as preco_medio,
    MAX(data) as data_mais_recente,
    MIN(data) as data_mais_antiga
FROM vendas;
```

### 2. String
```sql
SELECT 
    UPPER(nome) as nome_maiusculo,
    LOWER(email) as email_minusculo,
    SUBSTRING(descricao, 1, 100) as descricao_curta,
    CONCAT(nome, ' - ', categoria) as nome_completo
FROM produtos;
```

### 3. Data/Hora
```sql
SELECT 
    DATE_TRUNC('month', data_criacao) as mes,
    EXTRACT(YEAR FROM data_criacao) as ano,
    data_criacao + INTERVAL '7 days' as data_vencimento
FROM pedidos;
```

## Subconsultas

SUBQUERY_MASTER explica: "Consultas dentro de consultas"

```sql
-- Subconsulta no WHERE
SELECT nome
FROM produtos
WHERE categoria_id IN (
    SELECT id 
    FROM categorias 
    WHERE ativo = true
);

-- Subconsulta no FROM
SELECT dept_nome, total_funcionarios
FROM (
    SELECT departamento_id, COUNT(*) as total
    FROM funcionarios
    GROUP BY departamento_id
) f
JOIN departamentos d ON f.departamento_id = d.id;

-- Subconsulta correlacionada
SELECT p.nome
FROM produtos p
WHERE preco > (
    SELECT AVG(preco)
    FROM produtos
    WHERE categoria_id = p.categoria_id
);
```

## Otimização

PERFORMANCE_GURU compartilha dicas essenciais:

### 1. Índices
```sql
-- Use índices apropriados
CREATE INDEX idx_produtos_categoria 
ON produtos(categoria_id);

-- Índice composto para consultas frequentes
CREATE INDEX idx_pedidos_cliente_data 
ON pedidos(cliente_id, data_criacao);
```

### 2. Análise de Execução
```sql
-- Analise o plano de execução
EXPLAIN ANALYZE
SELECT c.nome, COUNT(p.id) as total_pedidos
FROM clientes c
JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.nome;
```

## Boas Práticas

BEST_PRACTICE_MASTER recomenda:

1. **Performance**
   - Evite SELECT *
   - Use JOINs apropriados
   - Limite resultados grandes
   - Utilize índices corretamente

2. **Legibilidade**
   - Use aliases descritivos
   - Formate SQL adequadamente
   - Comente consultas complexas
   - Mantenha consistência

3. **Manutenção**
   - Documente consultas importantes
   - Use views para consultas comuns
   - Implemente paginação
   - Monitore performance

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST SELECT:                        ║
║  □ Colunas necessárias apenas?           ║
║  □ Índices apropriados?                  ║
║  □ JOINs otimizados?                     ║
║  □ WHERE eficiente?                      ║
║  □ Resultados limitados?                 ║
╚════════════════════════════════════════════╝
```

## Conclusão

SELECT_SPECIALIST conclui: "O domínio do SELECT é fundamental para qualquer desenvolvedor SQL. Pratique diferentes tipos de consultas e sempre considere performance e manutenibilidade."

> **Dica Final**: Teste suas consultas com volumes de dados realistas para garantir performance em produção.