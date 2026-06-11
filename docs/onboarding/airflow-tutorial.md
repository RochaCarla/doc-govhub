# Tutorial: Primeira DAG no Airflow

Guia curto para criar uma DAG didática no padrão usado pelo GovHub BR.

!!! note "DAG de tutorial"
    Use este exemplo para aprender e testar localmente. Para PR real, crie a DAG na pasta da origem correta em `airflow_lappis/dags/data_ingest/<origem>/` e siga os [padrões de engenharia](../pipeline/padroes-engenharia.md).

## Pré-requisitos

- Ambiente local rodando (`docker compose up -d`)
- Airflow acessível em http://localhost:8080

## 1. Criar o arquivo da DAG

Crie `airflow_lappis/dags/data_ingest/tutorial/minha_primeira_dag.py`:

```python
import logging
from datetime import datetime, timedelta

from airflow.decorators import dag, task


@dag(
    schedule_interval=None,
    start_date=datetime(2025, 1, 1),
    catchup=False,
    default_args={
        "owner": "tutorial",
        "retries": 1,
        "retry_delay": timedelta(minutes=2),
    },
    tags=["tutorial"],
)
def minha_primeira_dag() -> None:
    @task
    def extract() -> dict[str, int | str]:
        logging.info("[minha_primeira_dag.py] Iniciando extracao didatica")
        return {"registros": 42, "fonte": "exemplo"}

    @task
    def load(data: dict[str, int | str]) -> None:
        logging.info(
            "[minha_primeira_dag.py] Simulando carga de %s registros da fonte %s",
            data["registros"],
            data["fonte"],
        )

    load(extract())


dag_instance = minha_primeira_dag()
```

## 2. Verificar na UI

1. Acesse http://localhost:8080
2. Procure `minha_primeira_dag` na lista
3. Ative a DAG
4. Clique em **Trigger DAG**

## 3. Testar pela linha de comando

```bash
docker compose exec airflow airflow dags list
docker compose exec airflow airflow dags test minha_primeira_dag 2025-01-01
```

## 4. Próximos passos

- Trocar a simulação por um cliente real em `airflow_lappis/plugins/`
- Persistir dados com `ClientPostgresDB` e `get_postgres_conn()`
- Usar `get_dynamic_schedule()` em DAGs reais de ingestão
- Adicionar logs com volume processado e destino da carga

## Referências

- [Apache Airflow](../pipeline/airflow.md)
- [Padrões de engenharia](../pipeline/padroes-engenharia.md)
