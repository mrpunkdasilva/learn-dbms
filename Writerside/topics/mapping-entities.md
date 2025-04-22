# Mapeamento de Entidades

O processo de mapeamento de entidades é fundamental na transformação do modelo conceitual para o modelo relacional. Este processo requer compreensão profunda de teoria dos conjuntos, álgebra relacional e dependências funcionais.

## Fundamentos Teóricos

### 1. Definições Básicas

Seja E uma entidade com atributos A = {A₁, A₂, ..., Aₙ}:
- Domínio: dom(Aᵢ) é o conjunto de valores possíveis para Aᵢ
- Tupla: t ∈ dom(A₁) × dom(A₂) × ... × dom(Aₙ)
- Relação: R ⊆ dom(A₁) × dom(A₂) × ... × dom(Aₙ)

### 2. Propriedades Fundamentais

1. Unicidade:
```
∀t₁,t₂ ∈ R : t₁ ≠ t₂ → t₁[K] ≠ t₂[K]
onde K é chave primária
```

2. Integridade Referencial:
```
∀t₁ ∈ R₁ : t₁[FK] ≠ null → ∃t₂ ∈ R₂ : t₁[FK] = t₂[PK]
```

3. Dependência Funcional:
```
X → Y ⟺ ∀t₁,t₂ ∈ R : t₁[X] = t₂[X] → t₁[Y] = t₂[Y]
```

## Regras de Mapeamento Detalhadas

### 1. Entidades Fortes

#### 1.1 Definição Formal
Para uma entidade forte E(K, A₁, ..., Aₙ):
```
R = {t | t ∈ dom(K) × dom(A₁) × ... × dom(Aₙ)}
```

#### 1.2 Propriedades
- Chave Primária: K → {A₁, ..., Aₙ}
- Unicidade: ∀t₁,t₂ ∈ R : t₁[K] = t₂[K] → t₁ = t₂
- Não-nulidade: ∀t ∈ R : t[K] ≠ null

#### 1.3 Exemplo Matemático
```
PRODUTO = {
    (k,a₁,...,aₙ) | 
    k ∈ dom(código),
    a₁ ∈ dom(nome),
    ...,
    aₙ ∈ dom(atributoₙ)
}
```

### 2. Entidades Fracas

#### 2.1 Definição Formal
Para uma entidade fraca W dependente de E:
```
W = {(k₁,fk,a₁,...,aₙ) | 
    k₁ ∈ dom(discriminador),
    fk ∈ dom(E.K),
    ∃e ∈ E : e.K = fk
}
```

#### 2.2 Dependências
- Chave Parcial: k₁ →ᵩ {a₁,...,aₙ}
- Dependência Existencial: W[fk] ⊆ E[K]

#### 2.3 Exemplo Detalhado
```
DEPENDENTE = {
    (seq, func_id, nome, data_nasc) |
    (seq, func_id) é único,
    func_id ∈ FUNCIONARIO[id]
}
```

### 3. Atributos Complexos

#### 3.1 Atributos Compostos
Seja C = {c₁,...,cₙ} um atributo composto:

Método 1 - Decomposição:
```
R(K, c₁,...,cₙ, outros_atributos)
```

Método 2 - Nova Relação:
```
R(K, outros_atributos)
C(K, c₁,...,cₙ)
K → {c₁,...,cₙ}
```

#### 3.2 Atributos Multivalorados
Para atributo multivalorado M:
```
E(K, A₁,...,Aₙ)
M(K, valor, metadata)
onde:
- K referencia E.K
- (K, valor) é único
```

## Padrões de Mapeamento

### 1. Hierarquia de Generalização

#### 1.1 Single Table
```
R(K, A₁,...,Aₙ, B₁,...,Bₘ, C₁,...,Cₖ, tipo)
onde:
- A são atributos comuns
- B,C são atributos específicos
- tipo discrimina a subclasse
```

#### 1.2 Table Per Class
```
R₁(K, A₁,...,Aₙ)
R₂(K, A₁,...,Aₙ, B₁,...,Bₘ)
R₃(K, A₁,...,Aₙ, C₁,...,Cₖ)
```

### 2. Agregação

Para agregação de E₁ e E₂ em AG:
```
AG(K, FK₁, FK₂, A₁,...,Aₙ)
onde:
- K é identificador próprio
- FK₁ referencia E₁
- FK₂ referencia E₂
```

## Casos Especiais

### 1. Auto-relacionamento
```
E(K, A₁,...,Aₙ)
R(K₁, K₂, tipo_relação)
onde:
- K₁,K₂ referenciam E.K
- (K₁,K₂) é único
```

### 2. Relacionamentos n-ários
Para relacionamento entre E₁,...,Eₙ:
```
R(K₁,...,Kₙ, A₁,...,Aₘ)
onde:
- Kᵢ referencia Eᵢ.K
- (K₁,...,Kₙ) é chave primária
```

## Otimizações e Considerações

### 1. Análise de Dependências

#### 1.1 Dependências Funcionais
```
F⁺ = {X → Y | X → Y é derivável de F}
```

#### 1.2 Dependências Multivaloradas
```
X →→ Y significa que Y depende multivalorado de X
```

### 2. Normalização

#### 2.1 Primeira Forma Normal (1NF)
- Atomicidade dos atributos
- Não permite grupos repetitivos

#### 2.2 Segunda Forma Normal (2NF)
- Satisfaz 1NF
- Não há dependências parciais

#### 2.3 Terceira Forma Normal (3NF)
- Satisfaz 2NF
- Não há dependências transitivas

## Exemplos Avançados

### 1. Sistema de Recursos Humanos
```
FUNCIONARIO(id, nome, data_admissao)
CARGO(código, nome, nível)
HISTÓRICO_CARGO(func_id, cargo_id, data_início, data_fim)
HABILIDADE(func_id, tipo, nível, certificação)
```

### 2. Sistema Acadêmico Completo
```
ALUNO(matrícula, nome, curso)
DISCIPLINA(código, nome, créditos)
PROFESSOR(id, nome, departamento)
TURMA(id, disciplina_id, professor_id, semestre)
MATRÍCULA(aluno_id, turma_id, nota, frequência)
```

## Validação do Mapeamento

### 1. Critérios de Qualidade

1. Preservação de Informação:
```
∀e ∈ E, ∃t ∈ R : representa(t,e)
```

2. Preservação de Dependências:
```
∀d ∈ D, mapeamento(d) ∈ D'
onde D,D' são conjuntos de dependências
```

3. Minimização de Redundância:
```
∄t₁,t₂ ∈ R : duplica_info(t₁,t₂)
```

### 2. Testes de Integridade

1. Teste de Chaves:
```
∀R, ∃K : K é minimal e K → R
```

2. Teste de Referências:
```
∀FK ∈ R₁, ∃PK ∈ R₂ : FK ⊆ PK
```

## Conclusão

O mapeamento efetivo requer:
1. Compreensão profunda da teoria
2. Aplicação consistente das regras
3. Consideração dos requisitos específicos
4. Validação rigorosa do resultado

Pontos críticos:
- Preservação semântica
- Integridade referencial
- Normalização adequada
- Eficiência operacional
