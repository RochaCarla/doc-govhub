# Visão Geral da Arquitetura

O GovHub BR adota a **Arquitetura Medallion** (Bronze → Silver → Gold) para processar dados de sistemas estruturantes do governo federal.

## Diagrama de Arquitetura

```mermaid
graph TB
    subgraph "Fontes"
        TG[TransfereGov]
        SI[Siape]
        SF[Siafi]
        CG[ComprasGov]
        SO[Siorg]
    end

    subgraph "Ingestão & Orquestração"
        AF[Apache Airflow]
    end

    subgraph "Bronze (Raw)"
        MN[MinIO - Object Storage]
    end

    subgraph "Silver / Gold"
        PG[(PostgreSQL)]
    end

    subgraph "Transformação"
        DBT[dbt Models]
    end

    subgraph "Visualização & Análise"
        SS[Apache Superset]
        JH[JupyterHub]
    end

    subgraph "Governança & Acesso"
        OM[OpenMetadata]
        TR[Trino + Ranger]
    end

    subgraph "Infra (GitOps)"
        ARGO[Argo CD]
        K8S[Kubernetes]
    end

    TG --> AF
    SI --> AF
    SF --> AF
    CG --> AF
    SO --> AF
    AF -->|"1. extract"| MN
    AF -->|"2. load"| PG
    PG --> DBT
    DBT -->|"silver/gold"| PG
    PG --> SS
    PG -->|"dados sensíveis"| TR
    TR --> JH
    PG --> OM
    ARGO --> K8S
```

## Camadas Medallion

| Camada | Storage | Descrição |
|--------|---------|-----------|
| **Bronze** | MinIO | Dados brutos ingeridos (JSON, CSV, Parquet raw) |
| **Silver** | PostgreSQL | Dados limpos, deduplicados, normalizados |
| **Gold** | PostgreSQL | Dados agregados, métricas, prontos para BI |

## Stack Tecnológica

| Componente | Tecnologia | Papel |
|------------|-----------|-------|
| Orquestração | Apache Airflow | DAGs de ingestão (extract → load → trigger dbt) |
| Transformação | dbt | Models SQL, testes, documentação |
| Object Storage | MinIO | Camada Bronze (dados brutos) |
| Banco Analítico | PostgreSQL | Camadas Silver e Gold (schemas por fork) |
| BI / Dashboards | Apache Superset | Visualização (acesso direto ao PG) |
| Notebooks | JupyterHub | Análise interativa (via Trino para dados sensíveis) |
| Governança | OpenMetadata | Catálogo, linhagem, ownership |
| Acesso Governado | Trino + Ranger | Row-level security para dados sensíveis |
| GitOps | Argo CD | Deploy declarativo em K8s |
| Containers | Docker / Kubernetes | Runtime de todos os serviços |

## Repositórios

| Repositório | Descrição |
|-------------|-----------|
| `gov-hub` | Site oficial e documentação pública do GovHub BR |
| `data-application-gov-hub` | Pipeline principal (Airflow, dbt, Jupyter, Superset) |
| `continuous-deployment` | Infra GitOps (K8s manifests, Helm, Argo CD) |
| `data-application-cidades` | Fork temático para dados municipais |
| `data-application-minc` | Fork temático para o Ministério da Cultura |
| `dados-desestruturados` | Processamento e experimentos com dados não estruturados |
| `govhub-research` | Pesquisa: IA, OCR, parsers |
| `openmetadata-declarative-governance` | Governança declarativa |
| `data-governance-workshop` | Workshop Ranger + Trino |

!!! note "Repositórios internos"
    A organização também possui repositórios privados de apoio. Eles não são detalhados nesta documentação pública.

## Decisões Arquiteturais

- **Medallion Architecture**: Separação clara entre raw (Bronze), limpo (Silver) e agregado (Gold)
- **GitOps**: Infraestrutura 100% declarativa via Argo CD — nenhum `kubectl apply` manual em produção
- **App-of-Apps**: Padrão Argo CD para gerenciar múltiplos serviços com sync waves
- **Forks leves**: Mesmo cluster, isolamento por schemas PG (cidades, MinC)
- **Open source**: Todo código público, comunidade aberta a contribuições

## Ambientes

| Ambiente | Descrição | Overlay |
|----------|-----------|---------|
| Local | Docker Compose (`docker compose up -d`) | — |
| Pré-produção | Cluster K8s (validação) | `values.preprod.yaml` |
| Produção | Cluster K8s (GitOps) | `values.prod.yaml` |
