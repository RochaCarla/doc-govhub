# Política de segurança

## Escopo

Esta política cobre o repositório `airflow_lappis` e os sistemas que dele dependem, incluindo pipelines de ingestão de dados, modelos dbt, configurações de infraestrutura e qualquer credencial ou segredo associado ao projeto.

---

## Reportando vulnerabilidades

**Não abra issues públicas para relatar vulnerabilidades de segurança.**

Se você encontrou uma vulnerabilidade, brecha de segurança ou exposição de dados sensíveis, reporte de forma privada:

1. Envie um e-mail para a equipe responsável descrevendo o problema.
2. Inclua no reporte:
   - Descrição clara da vulnerabilidade
   - Passos para reproduzir (se aplicável)
   - Impacto potencial estimado
   - Versão ou commit afetado

A equipe se compromete a:

- Confirmar o recebimento do reporte em até **3 dias úteis**
- Fornecer uma avaliação inicial em até **7 dias úteis**
- Manter o reportador informado sobre o andamento da correção

---

## Boas práticas para contribuidores

### Credenciais e segredos

- **Nunca commitar credenciais, tokens, senhas ou chaves de API** no repositório — nem mesmo em branches de desenvolvimento ou PRs temporários.
- Variáveis de ambiente sensíveis devem ser declaradas no `local.env` (que está no `.gitignore`) e nunca hardcodadas no código.
- Ao adicionar uma nova conexão ou credencial ao Airflow, usar as **Connections** ou **Variables** do Airflow, não strings literais nas DAGs.
- Em caso de commit acidental de credencial, notificar a equipe imediatamente e revogar/rotacionar a credencial afetada.

### Dados sensíveis

O repositório processa dados que incluem informações de servidores públicos (SIAPE), dados financeiros (SIAFI, Tesouro Gerencial) e informações pessoais. Ao desenvolver:

- Nunca incluir dados reais em seeds, fixtures de teste ou exemplos de código.
- Usar dados sintéticos ou anonimizados em ambientes de desenvolvimento.
- Campos com CPF, nome completo, dados bancários ou outros dados pessoais devem ser tratados com atenção — preferir hash ou mascaramento nas camadas silver e gold quando não houver necessidade analítica de expô-los.

### Dependências

- Ao adicionar novas dependências em `requirements.txt` ou `pyproject.toml`, verificar se há vulnerabilidades conhecidas (`pip audit` ou equivalente).
- Evitar dependências sem manutenção ativa ou com histórico de problemas de segurança.

### Acesso ao ambiente

- Não compartilhar credenciais de acesso ao ambiente de produção ou staging entre pessoas.
- Acessos devem ser individuais e revogados quando um contribuidor sai do projeto.

---

## Versões suportadas

Correções de segurança são aplicadas apenas na versão principal ativa do repositório (branch `main`/`master`). Branches antigas não recebem backport de correções de segurança.

---

## Divulgação responsável

Após a correção de uma vulnerabilidade reportada, a equipe pode divulgar publicamente os detalhes do problema e da correção, dando crédito ao reportador (com consentimento). Vulnerabilidades críticas podem ser divulgadas antes da correção se houver risco imediato e público, mediante aviso prévio ao reportador.
