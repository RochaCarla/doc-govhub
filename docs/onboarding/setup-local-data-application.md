# Setup local

Este guia detalha como configurar o ambiente de desenvolvimento local do projeto. O ambiente inclui Apache Airflow, PostgreSQL, Apache Superset e Jupyter, todos orquestrados via Docker Compose.

---

## Pré-requisitos

| Ferramenta | Versão mínima | Como verificar |
|---|---|---|
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

> **Atenção:** o projeto usa commits assinados com GPG. Configure a assinatura antes de fazer qualquer commit (ver seção [Assinatura GPG](#assinatura-gpg)).

---

## 2. Configurar variáveis de ambiente

Crie o arquivo `.env` na raiz do projeto. Este arquivo **nunca deve ser commitado** — ele já está no `.gitignore`.

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

> Os valores acima são os padrões para ambiente local. Em produção ou staging todas as credenciais devem ser substituídas por valores seguros — nunca commitar credenciais reais.

### Airflow Variables

Além das variáveis de infraestrutura, o Airflow depende de três **Airflow Variables** configuradas via interface ou CLI após subir o ambiente:

| Variable | Tipo | Descrição |
|---|---|---|
| `airflow_orgao` | string | Código do órgão alvo para ingestão |
| `airflow_variables` | YAML | Configurações por órgão (ex: `codigos_ug`) |
| `dynamic_schedules` | JSON | Schedules por `dag_id` |

Para configurá-las: `http://localhost:8080` → Admin → Variables.

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

Só é necessário rodar uma vez. Rode novamente se houver mudanças nas dependências.

---

## 4. Subir os serviços

```bash
docker compose up -d
```

Na primeira execução o Docker baixará as imagens necessárias, o que pode levar alguns minutos. Para acompanhar os logs:

```bash
docker compose logs -f
```

O Airflow sobe em modo `standalone` localmente — um único container em vez de múltiplos (webserver, scheduler, worker separados). Isso acelera a inicialização no ambiente de desenvolvimento.

---

## 5. Acessar os serviços

| Serviço | URL | Credenciais padrão |
|---|---|---|
| Apache Airflow | http://localhost:8080 | `airflow` / `airflow` |
| Apache Superset | http://localhost:8088 | `admin` / `admin` |
| Jupyter Lab | http://localhost:8888 | sem autenticação |
| PostgreSQL | `localhost:5432` | `postgres` / `postgres` |

> As credenciais da tabela são apenas para o ambiente local criado pelo Docker Compose. Ambientes compartilhados devem usar credenciais próprias e mecanismo de secrets.

> **Nota sobre o PostgreSQL:** um único servidor Postgres serve todos os bancos do ambiente — Airflow (`postgres`), Superset (`superset`) e Data Warehouse. O script `docker/postgres/init.sh` cria automaticamente três bancos na primeira inicialização: `airflow`, `superset` e `data_warehouse`.

---

## 6. Rodar o dbt localmente

O projeto tem dois projetos dbt: `dags/dbt/ipea/` e `dags/dbt/mir/`. O `profiles.yml` de cada projeto é montado automaticamente em `/opt/airflow/.dbt/profiles.yml` dentro do container. Para rodar localmente fora do container, verifique se as credenciais no `profiles.yml` apontam para `localhost:5432`.

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

> **Nota sobre `AIRFLOW_REPO_BASE`:** dentro do container, essa variável aponta para `AIRFLOW_HOME` (`/opt/airflow/`). Ao rodar o dbt fora do container, os caminhos precisam ser ajustados manualmente no `profiles.yml`.

---

## 7. Comandos úteis do Makefile

| Comando | O que faz |
|---|---|
| `make setup` | Configuração inicial completa |
| `make lint` | Executa verificações de lint no código |
| `make test` | Executa a suíte de testes automatizados |
| `make build` | Reconstrói as imagens Docker |
| `make clean` | Remove arquivos gerados e caches |

---

## Assinatura GPG

O projeto exige que todos os commits sejam assinados com GPG.

**1. Gerar uma chave GPG** (pule se já tiver uma):

```bash
gpg --full-generate-key
# Escolha: RSA, 4096 bits, mesmo e-mail da conta usada no GitHub ou GitLab
```

**2. Obter o ID da chave:**

```bash
gpg --list-secret-keys --keyid-format=long
# O ID aparece após "sec rsa4096/"
```

**3. Configurar o Git:**

```bash
git config --global user.signingkey SUA_KEY_ID
git config --global commit.gpgsign true
```

**4. Exportar e adicionar à plataforma do repositório:**

```bash
gpg --armor --export SUA_KEY_ID
# Copie a saída e cadastre a chave GPG no GitHub ou GitLab usado pelo repositório
```

---

## Troubleshooting

**Porta já em uso**

Verifique se algum serviço local já ocupa as portas 8080, 8088, 8888 ou 5432:

```bash
lsof -i :8080
lsof -i :5432
```

Pare o serviço conflitante ou ajuste as portas no `docker-compose.yml`.

---

**Airflow com erro de permissão nos logs**

Em sistemas Linux:

```bash
mkdir -p ./logs ./plugins
chmod -R 777 ./logs
```

---

**`make setup` falha na instalação de dependências**

Confirme que a versão do Python é exatamente 3.11.x:

```bash
python --version
```

Se tiver múltiplas versões instaladas, use `pyenv` para garantir o Python 3.11.

---

**dbt debug retorna erro de conexão**

Verifique se:
1. O container do PostgreSQL está rodando: `docker compose ps`
2. As credenciais no `profiles.yml` batem com `POSTGRES_USER` e `POSTGRES_PASSWORD` do `.env`
3. Está apontando para `localhost:5432`

---

**Airflow não encontra as DAGs**

O volume `./airflow_lappis/dags` é montado em `${AIRFLOW_HOME}/dags/` dentro do container. Confirme que o `docker-compose.yml` não foi modificado e que os arquivos existem localmente.

---

**Airflow Variables não definidas**

As DAGs de ingestão dependem de `airflow_orgao`, `airflow_variables` e `dynamic_schedules` para funcionar. Configure-as em:
`http://localhost:8080` → Admin → Variables

---

## Estrutura do ambiente local

```
data-application-gov-hub/
├── airflow_lappis/
│   ├── dags/               # DAGs de ingestão e transformação
│   │   ├── data_ingest/    # DAGs por origem de dados
│   │   ├── dbt/            # Projetos dbt (ipea, mir)
│   │   └── dashboards/     # DAGs de atualização de dashboards
│   ├── plugins/            # Clientes e integrações (montado no container)
│   ├── helpers/            # Funções auxiliares (montado no container)
│   └── templates/          # Templates XML para APIs SOAP (apenas local)
├── docker/
│   └── postgres/
│       └── init.sh         # Criação dos bancos na primeira inicialização
├── docker-compose.yml      # Definição de todos os serviços
├── Dockerfile              # Imagem customizada do Airflow
├── Dockerfile.superset     # Imagem customizada do Superset
├── .env                    # Variáveis de ambiente locais (não commitar)
├── Makefile                # Automação de tarefas
├── requirements.txt        # Dependências Python
└── pyproject.toml          # Configuração do projeto e ferramentas
```
