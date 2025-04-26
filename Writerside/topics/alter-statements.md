# ALTER Statements: Modificando Estruturas de Dados

```ascii
╔══════════════════════════════════════════════════════════════╗
║  SCHEMA_EVOLUTION_MASTER >> Modificações Estruturais        ║
║  INSTRUTOR: DATABASE_ARCHITECT                             ║
╚══════════════════════════════════════════════════════════════╝
```

## Fundamentos do ALTER

DATABASE_ARCHITECT explica: "O comando ALTER é sua ferramenta para evolução do banco de dados. Com ele, você pode modificar estruturas existentes sem perder dados."

## ALTER TABLE

SCHEMA_MASTER apresenta: "Vamos explorar as principais operações de alteração de tabelas."

### 1. Adicionando Colunas

```sql
-- Adicionando uma coluna simples
ALTER TABLE produtos
ADD COLUMN descricao TEXT;

-- Adicionando múltiplas colunas
ALTER TABLE usuarios
ADD COLUMN ultimo_acesso TIMESTAMP,
ADD COLUMN tentativas_login INTEGER DEFAULT 0,
ADD COLUMN bloqueado BOOLEAN DEFAULT FALSE;

-- Adicionando coluna com constraint
ALTER TABLE pedidos
ADD COLUMN valor_total DECIMAL(10,2) NOT NULL DEFAULT 0.0
CHECK (valor_total >= 0);
```

### 2. Modificando Colunas

MIGRATION_SPECIALIST adverte: "Cuidado ao modificar tipos de dados - certifique-se da compatibilidade!"

```sql
-- Alterando tipo de dados
ALTER TABLE produtos
ALTER COLUMN preco TYPE NUMERIC(12,2);

-- Modificando valor default
ALTER TABLE usuarios
ALTER COLUMN status SET DEFAULT 'ativo';

-- Removendo valor default
ALTER TABLE logs
ALTER COLUMN nivel DROP DEFAULT;

-- Tornando coluna NOT NULL
ALTER TABLE clientes
ALTER COLUMN email SET NOT NULL;
```

### 3. Constraints

INTEGRITY_GUARDIAN compartilha: "Constraints garantem a qualidade dos seus dados."

```sql
-- Adicionando Primary Key
ALTER TABLE produtos
ADD CONSTRAINT pk_produtos PRIMARY KEY (id);

-- Adicionando Foreign Key
ALTER TABLE pedidos
ADD CONSTRAINT fk_cliente 
FOREIGN KEY (cliente_id) 
REFERENCES clientes(id)
ON DELETE RESTRICT;

-- Adicionando Unique Constraint
ALTER TABLE usuarios
ADD CONSTRAINT uq_email UNIQUE (email);

-- Adicionando Check Constraint
ALTER TABLE produtos
ADD CONSTRAINT chk_preco 
CHECK (preco_venda > preco_custo);
```

### 4. Renomeando Objetos

REFACTORING_MASTER diz: "Às vezes precisamos reorganizar nossa estrutura."

```sql
-- Renomeando tabela
ALTER TABLE usuarios
RENAME TO users;

-- Renomeando coluna
ALTER TABLE produtos
RENAME COLUMN descricao TO detalhes;

-- Renomeando constraint
ALTER TABLE pedidos
RENAME CONSTRAINT fk_cliente TO fk_pedidos_cliente;
```

## ALTER INDEX

INDEX_WIZARD explica: "Índices também precisam de manutenção."

```sql
-- Renomeando índice
ALTER INDEX idx_old_name 
RENAME TO idx_new_name;

-- Modificando configurações do índice
ALTER INDEX idx_produtos 
SET (fillfactor = 90);
```

## ALTER SEQUENCE

SEQUENCE_MASTER compartilha: "Ajuste suas sequences conforme necessário."

```sql
-- Modificando sequence
ALTER SEQUENCE seq_pedidos
INCREMENT BY 10
MAXVALUE 999999
CYCLE;
```

## Cenários Práticos

PRACTICAL_GURU apresenta: "Vamos ver alguns cenários comuns do mundo real!"

### Evolução de Sistema de E-commerce

```sql
-- Adicionando suporte a múltiplos endereços
ALTER TABLE clientes
ADD COLUMN endereco_entrega JSONB,
ADD COLUMN endereco_cobranca JSONB;

-- Implementando soft delete
ALTER TABLE produtos
ADD COLUMN deleted_at TIMESTAMP,
ADD COLUMN active BOOLEAN DEFAULT TRUE;

-- Adicionando auditoria
ALTER TABLE pedidos
ADD COLUMN created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
ADD COLUMN updated_at TIMESTAMP,
ADD COLUMN created_by INTEGER,
ADD COLUMN updated_by INTEGER;

-- Adicionando constraints de auditoria
ALTER TABLE pedidos
ADD CONSTRAINT fk_created_by 
    FOREIGN KEY (created_by) REFERENCES usuarios(id),
ADD CONSTRAINT fk_updated_by 
    FOREIGN KEY (updated_by) REFERENCES usuarios(id);
```

## Boas Práticas

BEST_PRACTICES_SAGE compartilha diretrizes essenciais:

### 1. Segurança
- Faça backup antes de alterações
- Teste em ambiente de desenvolvimento
- Planeje janelas de manutenção
- Prepare scripts de rollback

### 2. Performance
- Considere o impacto em tabelas grandes
- Avalie bloqueios necessários
- Use transações apropriadamente
- Monitore uso de recursos

### 3. Manutenção
- Documente todas as alterações
- Mantenha scripts de migração
- Use controle de versão
- Comunique mudanças à equipe

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE ALTERAÇÕES:                 ║
║  □ Backup realizado?                      ║
║  □ Testado em desenvolvimento?            ║
║  □ Script de rollback preparado?          ║
║  □ Impacto analisado?                     ║
║  □ Equipe notificada?                     ║
║  □ Janela de manutenção agendada?        ║
╚════════════════════════════════════════════╝
```

## Troubleshooting Comum

ERROR_HANDLER apresenta soluções para problemas frequentes:

1. **Erro de Dependência**
```sql
-- Verificando dependências
SELECT * FROM pg_depend 
WHERE objid = 'sua_tabela'::regclass;

-- Removendo dependências com cautela
DROP VIEW IF EXISTS view_dependente CASCADE;
```

2. **Problemas de Bloqueio**
```sql
-- Verificando bloqueios
SELECT * FROM pg_locks 
WHERE relation = 'sua_tabela'::regclass;

-- Finalizando sessões bloqueantes (com cautela!)
SELECT pg_terminate_backend(pid);
```

## Conclusão

DATABASE_ARCHITECT conclui: "O ALTER é poderoso, mas requer responsabilidade. Sempre planeje suas alterações e siga as boas práticas para manter seu banco de dados saudável e evolutivo."

> **Dica Final**: Mantenha um histórico de todas as alterações estruturais em seu banco de dados. Isso será valioso para troubleshooting futuro e para entender a evolução do sistema.