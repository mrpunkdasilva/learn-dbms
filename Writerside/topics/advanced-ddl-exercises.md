# Exercícios Avançados de DDL

## Exercício 1: Sistema de Particionamento

### Descrição
Implemente um sistema de particionamento automático para uma tabela de logs.

### Requisitos
- Particionamento por data
- Manutenção automática
- Rotação de partições

### Solução
```sql
-- Criar tabela particionada
CREATE TABLE logs (
    id BIGSERIAL,
    timestamp TIMESTAMP NOT NULL,
    nivel VARCHAR(20),
    mensagem TEXT,
    dados JSONB
) PARTITION BY RANGE (timestamp);

-- Função para criar partições automaticamente
CREATE OR REPLACE FUNCTION criar_particao_logs(
    data_inicio DATE,
    data_fim DATE
) RETURNS VOID AS $$
BEGIN
    EXECUTE format(
        'CREATE TABLE IF NOT EXISTS logs_%s 
         PARTITION OF logs 
         FOR VALUES FROM (%L) TO (%L)',
        to_char(data_inicio, 'YYYYMM'),
        data_inicio,
        data_fim
    );
END;
$$ LANGUAGE plpgsql;

-- Criar partições para os próximos 12 meses
DO $$
DECLARE
    v_data DATE := date_trunc('month', CURRENT_DATE);
    v_fim DATE;
BEGIN
    FOR i IN 0..11 LOOP
        v_fim := v_data + interval '1 month';
        PERFORM criar_particao_logs(v_data, v_fim);
        v_data := v_fim;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

## Exercício 2: Gerenciamento de Extensões

### Descrição
Desenvolva um sistema de gerenciamento de extensões PostgreSQL.

### Requisitos
- Instalação controlada
- Versionamento
- Dependências

### Solução
```sql
-- Schema para gerenciamento de extensões
CREATE SCHEMA ext_manager;

-- Tabela de controle de extensões
CREATE TABLE ext_manager.extensions (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    versao VARCHAR(50) NOT NULL,
    dependencias JSONB,
    instalada_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ultima_atualizacao TIMESTAMP,
    status VARCHAR(20) DEFAULT 'INSTALADA',
    UNIQUE(nome, versao)
);

-- Função para instalar extensão
CREATE OR REPLACE FUNCTION ext_manager.instalar_extensao(
    p_nome VARCHAR,
    p_versao VARCHAR DEFAULT NULL,
    p_deps JSONB DEFAULT NULL
) RETURNS VOID AS $$
BEGIN
    -- Verificar se já está instalada
    IF EXISTS (
        SELECT 1 
        FROM ext_manager.extensions 
        WHERE nome = p_nome 
        AND status = 'INSTALADA'
    ) THEN
        RAISE EXCEPTION 'Extensão % já está instalada', p_nome;
    END IF;

    -- Instalar extensão
    IF p_versao IS NULL THEN
        EXECUTE format('CREATE EXTENSION IF NOT EXISTS %I', p_nome);
    ELSE
        EXECUTE format('CREATE EXTENSION IF NOT EXISTS %I VERSION %L',
            p_nome, p_versao);
    END IF;

    -- Registrar instalação
    INSERT INTO ext_manager.extensions (
        nome, versao, dependencias
    ) VALUES (
        p_nome, 
        COALESCE(p_versao, 
            (SELECT extversion FROM pg_extension WHERE extname = p_nome)),
        p_deps
    );
END;
$$ LANGUAGE plpgsql;

-- Função para atualizar extensão
CREATE OR REPLACE FUNCTION ext_manager.atualizar_extensao(
    p_nome VARCHAR,
    p_versao_nova VARCHAR
```

## Exercício 3: Sistema de Versionamento

### Descrição
Crie um sistema de versionamento de schema do banco de dados.

### Requisitos
- Controle de versões
- Migrations
- Rollback

### Solução
```sql
-- Schema para versionamento
CREATE SCHEMA IF NOT EXISTS versioning;

-- Tabela de controle de versões
CREATE TABLE versioning.schema_versions (
    version_id SERIAL PRIMARY KEY,
    version_number VARCHAR(50) NOT NULL,
    applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    description TEXT,
    script_name VARCHAR(200),
    checksum VARCHAR(64),
    applied_by VARCHAR(100) DEFAULT CURRENT_USER,
    status VARCHAR(20) DEFAULT 'SUCCESS',
    UNIQUE(version_number)
);

-- Função para aplicar versão
CREATE OR REPLACE FUNCTION versioning.apply_version(
    p_version VARCHAR,
    p_description TEXT,
    p_script TEXT
) RETURNS VOID AS $$
DECLARE
    v_checksum VARCHAR(64);
BEGIN
    -- Calcular checksum
    v_checksum := encode(sha256(p_script::bytea), 'hex');
    
    -- Registrar versão
    INSERT INTO versioning.schema_versions (
        version_number,
        description,
        script_name,
        checksum
    ) VALUES (
        p_version,
        p_description,
        p_script,
        v_checksum
    );
    
    -- Executar script
    EXECUTE p_script;
    
EXCEPTION WHEN OTHERS THEN
    -- Marcar como falha em caso de erro
    UPDATE versioning.schema_versions
    SET status = 'FAILED',
        description = description || E'\nError: ' || SQLERRM
    WHERE version_number = p_version;
    
    RAISE;
END;
$$ LANGUAGE plpgsql;
```

## Exercício 4: Auditoria Avançada

### Descrição
Implemente um sistema de auditoria completo com histórico de alterações.

### Requisitos
- Rastreamento de todas as alterações DDL
- Histórico de modificações de dados
- Sistema de recuperação

### Solução
```sql
-- Schema para auditoria
CREATE SCHEMA audit;

-- Tabela de eventos DDL
CREATE TABLE audit.ddl_events (
    id BIGSERIAL PRIMARY KEY,
    evento_timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    usuario_bd NAME DEFAULT CURRENT_USER,
    usuario_app VARCHAR(50),
    tipo_comando VARCHAR(20),
    objeto_schema VARCHAR(100),
    objeto_nome VARCHAR(100),
    comando_sql TEXT
);

-- Função para capturar eventos DDL
CREATE OR REPLACE FUNCTION audit.log_ddl_event() 
RETURNS event_trigger AS $$
DECLARE
    obj record;
    comando TEXT;
BEGIN
    SELECT * INTO obj FROM pg_event_trigger_ddl_commands() LIMIT 1;
    
    SELECT current_query() INTO comando;
    
    INSERT INTO audit.ddl_events (
        tipo_comando,
        objeto_schema,
        objeto_nome,
        comando_sql
    ) VALUES (
        obj.command_tag,
        obj.schema_name,
        obj.object_name,
        comando
    );
END;
$$ LANGUAGE plpgsql;

-- Event trigger para DDL
CREATE EVENT TRIGGER trg_audit_ddl 
ON ddl_command_end
EXECUTE FUNCTION audit.log_ddl_event();
```

## Critérios de Avaliação

1. **Robustez**
   - Tratamento de erros
   - Validações
   - Consistência dos dados

2. **Performance**
   - Eficiência das operações
   - Uso de recursos
   - Escalabilidade

3. **Manutenibilidade**
   - Clareza do código
   - Documentação
   - Modularidade

4. **Segurança**
   - Controle de acesso
   - Proteção dos dados
   - Auditoria

5. **Funcionalidade**
   - Atendimento aos requisitos
   - Completude da solução
   - Integração com sistemas existentes

## Recursos Adicionais

### Documentação
- [PostgreSQL Partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html)
- [PostgreSQL Replication](https://www.postgresql.org/docs/current/runtime-config-replication.html)
- [PostgreSQL Extensions](https://www.postgresql.org/docs/current/extend-extensions.html)

### Ferramentas Recomendadas
- pgAdmin 4
- DBeaver
- pg_partman
- pg_audit

### Boas Práticas

```ascii
╔════════════════════════════════════════════╗
║  CHECKLIST DE IMPLEMENTAÇÃO:              ║
║  ✓ Testes de performance                  ║
║  ✓ Scripts de rollback                    ║
║  ✓ Documentação técnica                   ║
║  ✓ Monitoramento                          ║
║  ✓ Backup e recuperação                   ║
╚════════════════════════════════════════════╝
```

## Próximos Passos

1. **Prática Avançada**
   - Implementar soluções em ambiente de teste
   - Realizar testes de carga
   - Simular cenários de falha

2. **Aprofundamento**
   - Estudar casos de uso específicos
   - Explorar recursos avançados
   - Participar de comunidades

3. **Projetos Práticos**
   - Desenvolver soluções completas
   - Documentar implementações
   - Compartilhar experiências

## Conclusão

Os exercícios avançados de DDL apresentados neste módulo cobrem aspectos críticos do desenvolvimento de bancos de dados em nível empresarial. A prática consistente destes conceitos é fundamental para o domínio das técnicas avançadas de gerenciamento de dados.

## Referências

1. PostgreSQL Documentation
2. Database Design Best Practices
3. Enterprise Database Architecture
4. Performance Tuning Guidelines

---

> **Nota**: Mantenha este documento como referência para implementações futuras e continue praticando os conceitos apresentados.