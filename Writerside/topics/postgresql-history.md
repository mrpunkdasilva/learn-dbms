# História do PostgreSQL

A história do PostgreSQL é fascinante e remonta a mais de 40 anos, com raízes acadêmicas que evoluíram para um dos sistemas de banco de dados mais avançados e confiáveis do mundo. Esta jornada ilustra como um projeto de pesquisa universitária se transformou em um sistema de banco de dados empresarial de classe mundial através do poder do desenvolvimento de código aberto.

## As Origens: Projeto Ingres (1977-1985)

A história do PostgreSQL começa na Universidade da Califórnia, Berkeley, com o projeto Ingres (Interactive Graphics and Retrieval System), liderado pelo Professor Michael Stonebraker.

- **1977**: O desenvolvimento do Ingres começa como um projeto de pesquisa para explorar os conceitos de bancos de dados relacionais, recentemente propostos por E.F. Codd da IBM.
- **1979-1982**: O Ingres é implementado e distribuído, tornando-se um dos primeiros sistemas de banco de dados relacionais disponíveis.
- **1982-1985**: O código do Ingres é refinado e comercializado, eventualmente formando a base para vários produtos comerciais, incluindo Microsoft SQL Server (via Sybase).

## Projeto Postgres (1986-1994)

Após o sucesso do Ingres, Stonebraker iniciou um novo projeto chamado "Postgres" (Post-Ingres) para abordar as limitações dos sistemas de banco de dados relacionais da época.

- **1986**: O desenvolvimento do Postgres começa, com o objetivo de adicionar suporte para tipos de dados complexos e melhorar a extensibilidade.
- **1989**: Postgres V1 é lançado para um pequeno número de usuários externos.
- **1990**: Postgres V2 é lançado com um novo sistema de regras.
- **1991**: Postgres V3 é lançado com suporte a múltiplos gerenciadores de armazenamento e um executor de consultas melhorado.
- **1993**: Postgres V4 é lançado, a última versão oficial do projeto de pesquisa Berkeley.
- **1994**: O projeto Postgres na Berkeley é oficialmente encerrado após a publicação de dezenas de artigos e a formação de várias empresas para comercializar a tecnologia.

## Postgres95 (1994-1996)

Após o fim do projeto Berkeley, dois estudantes de pós-graduação, Andrew Yu e Jolly Chen, adicionaram um interpretador SQL ao Postgres e lançaram o código como Postgres95.

- **1994-1995**: Yu e Chen substituem a linguagem de consulta QUEL do Postgres original por SQL.
- **Julho de 1996**: Postgres95 v0.03 é lançado na web.
- **Agosto de 1996**: Postgres95 v0.02 é lançado, com várias correções de bugs.
- **Setembro de 1996**: Postgres95 v0.01 é lançado, a última versão sob o nome Postgres95.

## PostgreSQL (1996-Presente)

Em 1996, tornou-se claro que o nome "Postgres95" não resistiria ao teste do tempo, e o projeto foi renomeado para "PostgreSQL" para refletir o suporte a SQL. Assim começou a era moderna do PostgreSQL.

- **8 de novembro de 1996**: PostgreSQL 6.0 é lançado, a primeira versão com o novo nome.
- **1997-1999**: As versões 6.1 a 6.5 são lançadas, adicionando recursos como transações, suporte a triggers, regras de reescrita de consultas e melhorias de desempenho.
- **2000**: PostgreSQL 7.0 é lançado, introduzindo Write-Ahead Logging para maior confiabilidade.
- **2001**: PostgreSQL 7.1 adiciona suporte a subconsultas.
- **2002**: PostgreSQL 7.2 melhora o suporte a padrões SQL.
- **2002**: PostgreSQL 7.3 adiciona schemas e melhora o tratamento de caracteres internacionais.
- **2003**: PostgreSQL 7.4 adiciona funções de janela (window functions) e autovacuum.

### A Era Moderna (2005-Presente)

- **2005**: PostgreSQL 8.0 introduz point-in-time recovery e tablespaces.
- **2005**: PostgreSQL 8.1 adiciona roles, particionamento por tabela e melhorias em desempenho.
- **2006**: PostgreSQL 8.2 melhora o desempenho de consultas e adiciona estatísticas de consulta.
- **2008**: PostgreSQL 8.3 adiciona suporte a XML e melhorias no planejador de consultas.
- **2009**: PostgreSQL 8.4 adiciona Common Table Expressions (CTEs), TRUNCATE recursivo e melhorias em funções de janela.
- **2010**: PostgreSQL 9.0 introduz replicação streaming e hot standby.
- **2011**: PostgreSQL 9.1 adiciona sincronização de replicação e extensões.
- **2012**: PostgreSQL 9.2 melhora o desempenho e adiciona JSON e índices de intervalo.
- **2013**: PostgreSQL 9.3 adiciona visualizações materializadas e suporte a JSON melhorado.
- **2014**: PostgreSQL 9.4 introduz JSONB, replicação lógica e slots de replicação.
- **2016**: PostgreSQL 9.5 adiciona UPSERT, índices BRIN e melhorias em operações em massa.
- **2016**: PostgreSQL 9.6 adiciona paralelismo, melhorias em consultas de agregação e sincronização de replicação múltipla.
- **2017**: PostgreSQL 10 introduz particionamento declarativo, replicação lógica nativa e melhorias em paralelismo.
- **2018**: PostgreSQL 11 melhora o particionamento, adiciona procedimentos armazenados e melhorias em paralelismo.
- **2019**: PostgreSQL 12 adiciona melhorias em índices, otimização de consultas e gerenciamento de JSON.
- **2020**: PostgreSQL 13 melhora o desempenho de índices, adiciona deduplicação de dados e melhorias em particionamento.
- **2021**: PostgreSQL 14 adiciona melhorias em pipeline de replicação lógica, melhor compressão e desempenho de consultas.
- **2022**: PostgreSQL 15 introduz melhorias em compressão, ordenação, expressões regulares e desempenho geral.
- **2023**: PostgreSQL 16 adiciona suporte a paralelismo em replicação lógica, melhorias em particionamento e desempenho.

## Marcos Importantes na Evolução do PostgreSQL

### Governança e Comunidade

- **1996**: Formação do PostgreSQL Global Development Group (PGDG) para supervisionar o desenvolvimento.
- **1997**: Criação das primeiras listas de discussão dedicadas ao PostgreSQL.
- **2001**: Criação da infraestrutura de build para testar automaticamente em múltiplas plataformas.
- **2003**: Primeira PostgreSQL Conference realizada nos EUA.
- **2005**: Criação da PostgreSQL License, aprovada pela OSI.
- **2007**: Formação da PostgreSQL Europe.
- **2009**: Criação do site postgresql.org como o centro oficial do projeto.

### Adoção Empresarial

- **2000s iniciais**: Empresas começam a oferecer suporte comercial para PostgreSQL.
- **2008**: EnterpriseDB (agora EDB) se torna uma força importante no ecossistema PostgreSQL.
- **2010**: Heroku adota PostgreSQL como seu banco de dados principal.
- **2012**: VMware anuncia o vFabric Postgres baseado em PostgreSQL.
- **2013**: Amazon lança o Amazon RDS para PostgreSQL.
- **2016**: Apple migra seus serviços de iCloud do Oracle para o PostgreSQL.
- **2017**: Google Cloud lança o Cloud SQL para PostgreSQL.
- **2018**: Microsoft lança o Azure Database for PostgreSQL.
- **2019**: Zalando, Uber, Netflix e outras grandes empresas de tecnologia relatam uso extensivo de PostgreSQL.

### Extensões Importantes

- **2001**: PostGIS é lançado, adicionando suporte geoespacial ao PostgreSQL.
- **2011**: Sistema de extensões formalmente adicionado ao PostgreSQL 9.1.
- **2012**: Introdução do Foreign Data Wrappers (FDW) no PostgreSQL 9.2.
- **2015**: CitusDB (agora Citus Data, adquirida pela Microsoft) lança extensão para escalabilidade horizontal.
- **2017**: TimescaleDB é lançado para suporte a séries temporais.
- **2021**: pgvector é lançado para suporte a pesquisa de similaridade vetorial.

## Legado e Impacto

O PostgreSQL evoluiu de um projeto de pesquisa acadêmica para um dos sistemas de banco de dados mais respeitados e utilizados no mundo. Seu impacto pode ser visto em vários aspectos:

- **Influência Técnica**: Muitos conceitos pioneiros no PostgreSQL foram posteriormente adotados por outros sistemas de banco de dados.
- **Modelo de Desenvolvimento**: O modelo de desenvolvimento comunitário do PostgreSQL demonstrou que software de missão crítica pode ser desenvolvido com sucesso como um projeto de código aberto.
- **Educação**: O PostgreSQL é amplamente utilizado em ambientes educacionais para ensinar conceitos de banco de dados.
- **Indústria**: Formou a base para numerosas startups e produtos comerciais.

## Conclusão

A história do PostgreSQL é um testemunho do poder da inovação acadêmica combinada com desenvolvimento comunitário de código aberto. De suas origens como um projeto de pesquisa universitária, o PostgreSQL cresceu para se tornar um sistema de banco de dados de classe mundial que compete - e frequentemente supera - alternativas comerciais caras.

O compromisso contínuo com a inovação, padrões abertos e qualidade de software garantiu que o PostgreSQL permaneça relevante e continue a crescer em popularidade, mesmo após mais de quatro décadas desde suas raízes conceituais.

---

Continue para [Características do PostgreSQL](postgresql-features.md) para explorar em detalhes os recursos que tornaram o PostgreSQL tão poderoso e versátil.