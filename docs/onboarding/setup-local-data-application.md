# Setup Local — data-application-gov-hub

Guia de configuração do ambiente de desenvolvimento local do `data-application-gov-hub`. O ambiente inclui Apache Airflow, PostgreSQL, Apache Superset e Jupyter, orquestrados via Docker Compose.

!!! note "Outro repositório"
    Este guia é específico para o repositório `data-application-gov-hub`. Para o ambiente de documentação (`doc-govhub`), consulte o [setup geral](setup-local.md).

---

## Pré-requisitos

| Ferramenta | Versão mínima | Como verificar |
|------------|---------------|----------------|
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.x | `docker compose version` |
| Make | qualquer | `make --version` |
| Python | 3.11.x | `python --version` |
| Git | qualquer | `git --version` |

---

## 1. Clonar o repositório

```bash
git clone git@github.com:GovHub-br/data-application-gov-hub.git
cd data-application-gov-hub
```

!!! warning "Commits assinados com GPG"
    O projeto exige commits assinados. Configure a assinatura GPG antes de fazer qualquer commit — veja a seção [Assinatura GPG](#assinatura-gpg) abaixo ou o [Git Workflow](git-workflow.md).

---

## 2. Configurar variáveis de ambiente

Crie o arquivo `.env` na raiz do projeto. Este arquivo **nunca deve ser commitado** — já está no `.gitignore`.

```bash
# Airflow
AIRFLOW_IMAGE_NAME=apache/airflow:latest
AIRFLOW__CORE__FERNET_KEY='lmnHJcz5u4D8SPEeD4qwAf1TUk_yLXXnQfwlvQ8MXsU='
AIRFLOW_HOME=/opt/airflow/
_AIRFLOW_WWW_USER_USERNAME=airflow
_AIRFLOW_WWW_USER_PASSWORD=airflow
AIRFLOW_UID=50000

# PostgreSQL
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=postgres
```

!!! warning
    Os valores acima são padrões para ambiente local. Em produção, todas as credenciais devem ser substituídas por valores seguros — nunca commitar credenciais reais.

### Airflow Variables

Após subir o ambiente, configure as três Airflow Variables em `http://localhost:8080` → Admin → Variables:

| Variable | Tipo | Descrição |
|----------|------|-----------|
| `airflow_orgao` | string | Código do órgão alvo para ingestão |
| `airflow_variables` | YAML | Configurações por órgão (ex: `codigos_ug`) |
| `dynamic_schedules` | JSON | Schedules por `dag_id` |

As DAGs de ingestão dependem dessas variáveis para funcionar. Sem elas, falharão ao iniciar.

---

## 3. Executar o setup inicial

```bash
make setup
```

Este comando realiza automaticamente:

- Criação dos ambientes virtuais Python
- Instalação de todas as dependências (`requirements.txt`, `pyproject.toml`)
- Configuração dos hooks de pre-commit
- Preparação do ambiente de desenvolvimento

Rode novamente se houver mudanças nas dependências.

---

## 4. Subir os serviços

```bash
docker-compose up -d
```

Na primeira execução o Docker baixará as imagens necessárias — pode levar alguns minutos. Para acompanhar:

```bash
docker-compose logs -f
```

!!! note "Modo standalone"
    O Airflow sobe em modo `standalone` localmente — um único container em vez de webserver, scheduler e worker separados. Isso acelera a inicialização no ambiente de desenvolvimento.

---

## 5. Acessar os serviços

| Serviço | URL | Credenciais padrão |
|---------|-----|--------------------|
| Apache Airflow | http://localhost:8080 | `airflow` / `airflow` |
| Apache Superset | http://localhost:8088 | `admin` / `admin` |
| Jupyter Lab | http://localhost:8888 | sem autenticação |
| PostgreSQL | `localhost:5432` | `postgres` / `postgres` |

!!! note "PostgreSQL único"
    Um único servidor Postgres serve todos os bancos do ambiente — Airflow, Superset e Data Warehouse. Os bancos são criados automaticamente por `docker/postgres/init.sh` na primeira inicialização.

---

## 6. Rodar o dbt localmente

O projeto tem dois projetos dbt: `dags/dbt/ipea/` e `dags/dbt/mir/`. Para rodar fora do container, verifique se o `profiles.yml` aponta para `localhost:5432`.

```bash
# Entrar no diretório do projeto dbt desejado
cd airflow_lappis/dags/dbt/ipea

# Verificar conexão
dbt debug

# Rodar todos os modelos
dbt run

# Rodar apenas um domínio
dbt run --select pessoas_dbt

# Rodar apenas uma camada
dbt run --select pessoas_dbt.silver

# Executar os testes
dbt test

# Gerar e servir a documentação
dbt docs generate
dbt docs serve
```

!!! note "AIRFLOW_REPO_BASE"
    Dentro do container, `AIRFLOW_REPO_BASE` aponta para `AIRFLOW_HOME` (`/opt/airflow/`). Ao rodar o dbt fora do container, os caminhos precisam ser ajustados manualmente no `profiles.yml`.

---

## 7. Comandos do Makefile

| Comando | O que faz |
|---------|-----------|
| `make setup` | Configuração inicial completa |
| `make lint` | Verificações de lint no código |
| `make test` | Suíte de testes automatizados |
| `make build` | Reconstrói as imagens Docker |
| `make clean` | Remove arquivos gerados e caches |

---

## Assinatura GPG

```bash
# 1. Gerar chave (pule se já tiver uma)
gpg --full-generate-key
# Escolha: RSA, 4096 bits, mesmo e-mail da conta GitHub

# 2. Obter o ID da chave
gpg --list-secret-keys --keyid-format=long

# 3. Configurar o Git
git config --global user.signingkey SUA_KEY_ID
git config --global commit.gpgsign true

# 4. Exportar e adicionar ao GitHub
gpg --armor --export SUA_KEY_ID
# Cole a saída em: GitHub → Settings → SSH and GPG keys → New GPG key
```

---

## Estrutura do ambiente local

```
data-application-gov-hub/
├── airflow_lappis/
│   ├── dags/               # DAGs de ingestão e transformação
│   │   ├── data_ingest/    # DAGs por origem de dados
│   │   ├── dbt/            # Projetos dbt (ipea, mir)
│   │   └── dashboards/     # DAGs de atualização de dashboards
│   ├── plugins/            # Clientes e integrações
│   ├── helpers/            # Funções auxiliares
│   └── templates/          # Templates XML para APIs SOAP (apenas local)
├── docker/
│   └── postgres/
│       └── init.sh         # Criação dos bancos na primeira inicialização
├── docker-compose.yml
├── Dockerfile
├── Dockerfile.superset
├── .env                    # Variáveis de ambiente locais (não commitar)
├── Makefile
├── requirements.txt
└── pyproject.toml
```

---

## Troubleshooting

**Porta já em uso**

```bash
lsof -i :8080
lsof -i :5432
```

Pare o serviço conflitante ou ajuste as portas no `docker-compose.yml`.

**Airflow com erro de permissão nos logs** (Linux)

```bash
mkdir -p ./logs ./plugins
chmod -R 777 ./logs
```

**`make setup` falha na instalação de dependências**

Confirme que a versão do Python é exatamente 3.11.x:

```bash
python --version
```

Se tiver múltiplas versões, use `pyenv` para garantir o Python 3.11.

**`dbt debug` retorna erro de conexão**

1. Verifique se o container do PostgreSQL está rodando: `docker-compose ps`
2. Confirme que as credenciais no `profiles.yml` batem com `POSTGRES_USER` e `POSTGRES_PASSWORD` do `.env`
3. Verifique que está apontando para `localhost:5432`

**Airflow não encontra as DAGs**

O volume `./airflow_lappis/dags` é montado em `${AIRFLOW_HOME}/dags/` dentro do container. Confirme que o `docker-compose.yml` não foi modificado e que os arquivos existem localmente.

**Airflow Variables não definidas**

Configure em `http://localhost:8080` → Admin → Variables as três variáveis: `airflow_orgao`, `airflow_variables` e `dynamic_schedules`.
