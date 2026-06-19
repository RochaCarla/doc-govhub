# Roteiro de Onboarding

Guia para novos contribuidores do GovHub BR.

## Primeiro Dia

1. **Ler** o [README do projeto](https://github.com/GovHub-br/data-application-gov-hub)
2. **Clonar** o repositório
3. **Configurar** o ambiente local ([Setup Local](setup-local.md))
4. **Subir** os serviços (`docker compose up -d`)
5. **Explorar** Airflow, Superset e Jupyter nos links locais

## Primeira Semana

1. **Entender** a [Arquitetura Medallion](../arquitetura/medallion.md)
2. **Seguir** o [Tutorial Airflow](airflow-tutorial.md) — criar uma DAG simples
3. **Seguir** o [Tutorial dbt](dbt-tutorial.md) — criar um model
4. **Configurar** Git com GPG ([Git Workflow](git-workflow.md))
5. **Abrir** seu primeiro PR com uma melhoria pequena

## Trilhas

| Trilha | Foco | Tutoriais |
|--------|------|-----------|
| **Pipeline** | Airflow + dbt | [Airflow](airflow-tutorial.md), [dbt](dbt-tutorial.md) |
| **Visualização** | Superset | [Superset](superset-tutorial.md) |
| **Infra** | K8s + Argo CD | [Kubernetes](../infraestrutura/kubernetes.md), [Argo CD](../infraestrutura/argocd.md) |
| **Pesquisa** | IA, OCR, parsers | [Pesquisa](../comunidade/pesquisa.md) |

## Links Úteis

- **Repo principal**: [data-application-gov-hub](https://github.com/GovHub-br/data-application-gov-hub)
- **Infra**: [continuous-deployment](https://github.com/GovHub-br/continuous-deployment)
- **Documentação oficial**: [gov-hub.io](https://gov-hub.io)
- **Issues**: [Abrir issue](https://github.com/GovHub-br/data-application-gov-hub/issues)

## Contato

- Email: lablivreunb@gmail.com
- Issues no GitHub para dúvidas técnicas
