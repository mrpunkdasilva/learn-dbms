# Laboratório: Criação de Database

## Objetivo
Praticar a criação e configuração de databases PostgreSQL, aplicando boas práticas e explorando diferentes opções de configuração.

## Pré-requisitos
- PostgreSQL 12 ou superior instalado
- Acesso administrativo ao servidor
- psql ou ferramenta similar de acesso

## Exercícios

### 1. Database Básico

```sql
-- Criar database simples
CREATE DATABASE lab_basic;

-- Verificar criação
SELECT datname, encoding, datcollate, datctype 
FROM pg_database 
WHERE datname = 'lab_basic';
```

### 2. Database com Configurações Específicas

```sql
-- Criar database com configurações customizadas
CREATE DATABASE lab_advanced
    WITH 
    ENCODING = 'UTF8'
    LC_COLLATE = 'pt_BR.UTF-8'
    LC_CTYPE = 'pt_BR.UTF-8'
    TEMPLATE = template0
    OWNER = lab_user
    CONNECTION LIMIT = 100;
```

### 3. Database com Tablespaces

```sql
-- Criar tablespace customizado
CREATE TABLESPACE lab_space
    LOCATION '/path/to/data/lab_tablespace';

-- Criar database usando tablespace
CREATE DATABASE lab_distributed
    TABLESPACE lab_space;
```

## Tarefas Práticas

### Tarefa 1: Setup Inicial
1. Criar um novo database para ambiente de desenvolvimento
2. Configurar encoding e locale apropriados
3. Estabelecer limites de conexão
4. Verificar configurações

### Tarefa 2: Configuração Avançada
1. Implementar tablespaces separados para dados e índices
2. Configurar parâmetros de performance
3. Estabelecer políticas de backup

### Tarefa 3: Migração
1. Criar database template
2. Clonar estrutura para novo database
3. Verificar integridade

## Verificações

```sql
-- Script de verificação de configurações
CREATE OR REPLACE FUNCTION check_database_config()
RETURNS TABLE (
    parameter_name VARCHAR,
    current_value VARCHAR,
    recommended_value VARCHAR
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        'max_connections'::VARCHAR,
        current_setting('max_connections')::VARCHAR,
        '100'::VARCHAR
    UNION ALL
    SELECT 
        'default_transaction_isolation',
        current_setting('default_transaction_isolation'),
        'read committed';
END;
$$ LANGUAGE plpgsql;
```

## Boas Práticas

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE CRIAÇÃO:                    ║
║  ✓ Encoding adequado                      ║
║  ✓ Locale correto                         ║
║  ✓ Permissões apropriadas                 ║
║  ✓ Limites de conexão                     ║
║  ✓ Tablespaces configurados               ║
╚════════════════════════════════════════════╝
```

## Troubleshooting

### Problemas Comuns

1. **Erro de Locale**
   ```sql
   -- Solução: Usar template0
   CREATE DATABASE lab_fix
       TEMPLATE template0
       LC_COLLATE 'pt_BR.UTF-8'
       LC_CTYPE 'pt_BR.UTF-8';
   ```

2. **Limite de Conexões**
   ```sql
   -- Ajustar limite
   ALTER DATABASE lab_fix
       CONNECTION LIMIT 150;
   ```

3. **Permissões**
   ```sql
   -- Corrigir ownership
   ALTER DATABASE lab_fix
       OWNER TO lab_admin;
   ```

## Scripts de Suporte

### Script de Validação

```sql
CREATE OR REPLACE PROCEDURE validate_database_creation(
    db_name VARCHAR
) AS $$
DECLARE
    v_exists BOOLEAN;
BEGIN
    SELECT EXISTS (
        SELECT 1 
        FROM pg_database 
        WHERE datname = db_name
    ) INTO v_exists;

    IF NOT v_exists THEN
        RAISE EXCEPTION 'Database % não existe', db_name;
    END IF;

    -- Verificar configurações básicas
    PERFORM check_database_config();
END;
$$ LANGUAGE plpgsql;
```

## Exercícios Avançados

1. **Replicação**
   - Configurar database para replicação
   - Implementar streaming replication
   - Testar failover

2. **Performance**
   - Otimizar configurações de memória
   - Ajustar parâmetros de vacuum
   - Monitorar desempenho

3. **Segurança**
   - Implementar criptografia
   - Configurar SSL
   - Estabelecer políticas de acesso

## Critérios de Conclusão

1. **Funcionalidade**
   - Database criado corretamente
   - Configurações aplicadas
   - Conexões funcionando

2. **Performance**
   - Tempos de resposta aceitáveis
   - Uso de recursos otimizado
   - Monitoramento estabelecido

3. **Segurança**
   - Permissões corretas
   - Acessos controlados
   - Logs configurados

## Próximos Passos

1. Explorar configurações avançadas
2. Implementar monitoramento
3. Documentar procedimentos
4. Automatizar processos

## Recursos Adicionais

- [PostgreSQL Database Creation](https://www.postgresql.org/docs/current/sql-createdatabase.html)
- [Database Configuration](https://www.postgresql.org/docs/current/runtime-config.html)
- [Tablespace Management](https://www.postgresql.org/docs/current/manage-ag-tablespaces.html)

---

> **Nota**: Mantenha este documento como referência para futuras criações de databases e configurações avançadas.