# Tabelas Fato

## Tipos de Fatos

### 1. Fatos Transacionais
```sql
CREATE TABLE fato_vendas (
    sk_data INT,
    sk_produto INT,
    sk_cliente INT,
    sk_loja INT,
    quantidade INT,
    valor_venda DECIMAL(10,2),
    custo DECIMAL(10,2),
    PRIMARY KEY (sk_data, sk_produto, sk_cliente, sk_loja)
);
```

### 2. Fatos Periódicos
```sql
CREATE TABLE fato_estoque_mensal (
    sk_data INT,
    sk_produto INT,
    sk_loja INT,
    quantidade_media INT,
    valor_estoque DECIMAL(10,2),
    giro_estoque DECIMAL(5,2)
);
```

### 3. Fatos Acumulativos
```sql
CREATE TABLE fato_pedidos (
    sk_pedido INT,
    sk_cliente INT,
    sk_data_pedido INT,
    sk_data_aprovacao INT,
    sk_data_envio INT,
    sk_data_entrega INT,
    valor_total DECIMAL(10,2)
);
```

## Granularidade

### 1. Níveis Comuns
- Transação individual
- Diário
- Semanal
- Mensal

### 2. Agregações
```sql
-- Exemplo de agregação
CREATE TABLE fato_vendas_diarias AS
SELECT 
    sk_data,
    sk_produto,
    sk_loja,
    SUM(quantidade) as qtd_total,
    SUM(valor_venda) as valor_total
FROM fato_vendas
GROUP BY sk_data, sk_produto, sk_loja;
```

## Métricas

### 1. Tipos de Métricas
- Aditivas
- Semi-aditivas
- Não-aditivas

### 2. Cálculos Comuns
```sql
-- Exemplo de métricas calculadas
SELECT 
    d.mes,
    SUM(f.valor_venda) as receita_total,
    AVG(f.valor_venda) as ticket_medio,
    SUM(f.valor_venda - f.custo) as margem_bruta
FROM fato_vendas f
JOIN dim_tempo d ON f.sk_data = d.sk_data
GROUP BY d.mes;
```