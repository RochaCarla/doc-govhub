# Tutorial: Primeiro Model no dbt

Guia curto para criar e testar um model dbt mínimo dentro da estrutura real do GovHub BR.

## Pré-requisitos

- Ambiente local configurado
- PostgreSQL acessível
- Dependências instaladas via `make setup`

## 1. Entrar no projeto dbt

Use o projeto `ipea` para o tutorial:

```bash
cd airflow_lappis/dags/dbt/ipea
dbt debug
```

## 2. Criar um model didático

Crie `models/tutorial_dbt/silver/exemplo_orgaos.sql`:

```sql
{{ config(materialized='table', schema='tutorial') }}

with source_data as (
    select 1 as id, 'Instituto de Pesquisa Econômica Aplicada' as nome, 'IPEA' as sigla
    union all
    select 2 as id, 'Universidade de Brasília' as nome, 'UNB' as sigla
)

select
    id,
    trim(nome) as nome,
    lower(sigla) as sigla,
    current_timestamp as dt_ingest
from source_data
where id is not null
```

Esse exemplo não depende de API ou tabela externa. Ele serve apenas para validar estrutura, execução e testes.

## 3. Adicionar testes

Crie `models/tutorial_dbt/silver/schema.yml`:

```yaml
version: 2

models:
  - name: exemplo_orgaos
    description: Model didático para validar execução local do dbt.
    columns:
      - name: id
        description: Identificador didático do órgão.
        tests:
          - not_null
          - unique
      - name: nome
        description: Nome do órgão.
        tests:
          - not_null
      - name: dt_ingest
        description: Data e hora de geração do model didático.
```

## 4. Executar

```bash
dbt run --select exemplo_orgaos
dbt test --select exemplo_orgaos
dbt show --select exemplo_orgaos --limit 5
```

## 5. Limpar o tutorial

Depois do teste, remova `models/tutorial_dbt/` antes de abrir PR, a menos que a mudança seja exatamente criar documentação ou fixture didática combinada com o time.

## Próximos passos

- Estudar os domínios reais em `models/*_dbt/`
- Ler [dbt](../pipeline/dbt.md)
- Ler [Qualidade de Dados](../pipeline/qualidade.md)
- Seguir [Padrões de Engenharia](../pipeline/padroes-engenharia.md)
