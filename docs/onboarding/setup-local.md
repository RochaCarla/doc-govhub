# Setup Local

Configuração do ambiente de desenvolvimento local do GovHub BR.

## Pré-requisitos

- Docker com o plugin Docker Compose
- Make
- Python 3.11+
- Git (com GPG configurado — ver [Git Workflow](git-workflow.md))

## Instalação

### 1. Clonar o repositório

```bash
git clone git@github.com:GovHub-br/data-application-gov-hub.git
cd data-application-gov-hub
```

### 2. Setup automático

```bash
make setup
```

Isso irá:

- Criar os ambientes virtuais necessários
- Instalar as dependências
- Configurar os hooks de pre-commit
- Preparar o ambiente de desenvolvimento

### 3. Variáveis de ambiente

Configurar conforme [guia de instalação](https://gov-hub.io/govhub/documentacao/instalacao/).

### 4. Subir os serviços

```bash
docker compose up -d
```

## Serviços Locais

| Serviço | URL | Credenciais padrão |
|---------|-----|-------------------|
| Airflow | http://localhost:8080 | airflow / airflow |
| Jupyter | http://localhost:8888 | Sem autenticação local |
| Superset | http://localhost:8088 | admin / admin |

!!! warning "Apenas local"
    Essas credenciais são padrões de desenvolvimento. Não use esses valores em staging, produção ou ambientes compartilhados.

## Comandos do Makefile

| Comando | Descrição |
|---------|-----------|
| `make setup` | Configuração inicial |
| `make build` | Constrói imagens Docker |
| `make lint` | Verifica qualidade de código |
| `make test` | Executa testes |
| `make clean` | Remove arquivos gerados |

## Verificação

Após `docker compose up -d`, verifique:

```bash
# Todos os containers rodando
docker compose ps

# Airflow saudável
curl http://localhost:8080/health

# PostgreSQL saudável
docker compose ps postgres
```

## Troubleshooting

Veja [Troubleshooting](troubleshooting.md) para problemas comuns.
