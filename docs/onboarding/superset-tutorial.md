# Tutorial: Primeiro Dashboard no Superset

Guia passo a passo para criar seu primeiro dashboard no GovHub BR.

## Pré-requisitos

- Ambiente local rodando (`docker compose up -d`)
- Superset acessível em http://localhost:8088
- Dados na camada Gold do PostgreSQL (rodar `dbt run` antes)

## 1. Acessar Superset

1. Abra http://localhost:8088
2. Login: `admin` / `admin`

## 2. Configurar Database

1. Menu → Data → Databases → + Database
2. Preencher:
   - Database Name: `GovHub`
   - SQLAlchemy URI: `postgresql://postgres_dw:postgres_dw@postgres:5432/data_warehouse`
3. Test Connection → Save

## 3. Criar Dataset

1. Menu → Data → Datasets → + Dataset
2. Selecionar:
   - Database: `GovHub`
   - Schema: `gold`
   - Table: `fato_transferencias`
3. Save

## 4. Criar Chart

1. Menu → Charts → + Chart
2. Selecionar dataset: `fato_transferencias`
3. Escolher tipo de chart (ex: Bar Chart)
4. Configurar:
   - Metric: `SUM(valor_total)`
   - Group by: `orgao_concedente`
   - Filters: últimos 12 meses
5. Run Query → Save

## 5. Criar Dashboard

1. Menu → Dashboards → + Dashboard
2. Nomear: "Meu Primeiro Dashboard"
3. Arrastar o chart criado
4. Adicionar filtros (date range, órgão)
5. Save

## 6. Próximos passos

- Criar charts com diferentes visualizações (line, pie, map)
- Configurar filtros cross-dashboard
- Explorar SQL Lab para queries ad-hoc
- Estudar dashboards existentes

## Referências

- [Superset Creating Dashboards](https://superset.apache.org/docs/creating-charts-dashboards/creating-your-first-dashboard)
- [Superset Explore](https://superset.apache.org/docs/creating-charts-dashboards/exploring-data)
