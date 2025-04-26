# Padrões de Auditoria

## Log de Alterações

```mermaid
erDiagram
    ENTIDADE ||--o{ LOG_ALTERACAO : registra
    
    ENTIDADE {
        int id PK
        string nome
        string status
    }
    
    LOG_ALTERACAO {
        int id PK
        string tabela
        int registro_id
        string campo
        string valor_anterior
        string valor_novo
        timestamp data_alteracao
        string usuario
    }
```

### Implementação
```sql
CREATE TABLE log_alteracao (
    id INT PRIMARY KEY,
    tabela VARCHAR(50),
    registro_id INT,
    campo VARCHAR(50),
    valor_anterior TEXT,
    valor_novo TEXT,
    data_alteracao TIMESTAMP,
    usuario VARCHAR(50)
);

-- Trigger de exemplo
CREATE TRIGGER tr_audit_entidade
AFTER UPDATE ON entidade
FOR EACH ROW
INSERT INTO log_alteracao (
    tabela, registro_id, campo, 
    valor_anterior, valor_novo, 
    data_alteracao, usuario
) VALUES (
    'entidade', NEW.id, 'status',
    OLD.status, NEW.status,
    CURRENT_TIMESTAMP, CURRENT_USER
);
```

## Trilha de Auditoria

```mermaid
erDiagram
    PEDIDO ||--o{ EVENTO_PEDIDO : registra
    
    PEDIDO {
        int id PK
        float valor
        string status
    }
    
    EVENTO_PEDIDO {
        int id PK
        int pedido_id FK
        string tipo_evento
        string descricao
        timestamp data_evento
        string usuario
    }
```

### Implementação
```sql
CREATE TABLE evento_pedido (
    id INT PRIMARY KEY,
    pedido_id INT,
    tipo_evento VARCHAR(50),
    descricao TEXT,
    data_evento TIMESTAMP,
    usuario VARCHAR(50),
    FOREIGN KEY (pedido_id) REFERENCES pedido(id)
);
```

## Snapshot

```mermaid
erDiagram
    CONTA ||--o{ SNAPSHOT_CONTA : registra
    
    CONTA {
        int id PK
        float saldo_atual
        string status
    }
    
    SNAPSHOT_CONTA {
        int id PK
        int conta_id FK
        float saldo
        string status
        date data_snapshot
    }
```

### Implementação
```sql
CREATE TABLE snapshot_conta (
    id INT PRIMARY KEY,
    conta_id INT,
    saldo DECIMAL(10,2),
    status VARCHAR(20),
    data_snapshot DATE,
    FOREIGN KEY (conta_id) REFERENCES conta(id)
);

-- Procedure para criar snapshot
CREATE PROCEDURE criar_snapshot_diario()
BEGIN
    INSERT INTO snapshot_conta (
        conta_id, saldo, status, data_snapshot
    )
    SELECT 
        id, saldo_atual, status, CURRENT_DATE
    FROM conta;
END;
```