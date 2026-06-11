# Protocolo de Aprovação de Pull Requests

Este documento define o fluxo obrigatório para abertura, revisão e aprovação de Pull Requests (PRs) no GovHub BR. O objetivo é garantir qualidade, rastreabilidade e consistência nas contribuições de código, dados, infraestrutura e documentação.

> Nos repositórios hospedados no GitHub, o fluxo operacional usa Pull Requests (PRs). Em repositórios hospedados no GitLab, leia Merge Request (MR) como o mesmo protocolo de revisão.

## Escopo

Este protocolo se aplica a qualquer PR que envolva:

- código: novas DAGs, alterações em DAGs existentes, modelos dbt, plugins e helpers;
- documentação: criação ou edição de arquivos `.md`, `schema.yml`, `CONTRIBUTING.md`, entre outros;
- infraestrutura e CI/CD: workflows, configurações, dependências e automações.

## Fluxo geral

```text
branch de trabalho -> commits padronizados -> PR aberto -> revisão -> aprovação -> merge na main
```

Nenhum merge deve ser feito diretamente na `main` sem passar pelo fluxo de PR.

## 1. Mensagens de commit

O projeto adota o padrão **Conventional Commits**:

```text
<tipo>[escopo opcional]: <descrição>
```

| Tipo | Quando usar |
| --- | --- |
| `feat` | Nova DAG, novo modelo, nova funcionalidade |
| `fix` | Correção de bug ou comportamento incorreto |
| `docs` | Criação ou edição de documentação |
| `refactor` | Refatoração sem mudança de comportamento |
| `perf` | Melhoria de desempenho |
| `test` | Adição ou correção de testes |
| `build` | Mudanças em dependências ou sistema de build |
| `ci` | Mudanças em configurações de CI |
| `chore` | Ajustes que não afetam código-fonte ou testes |
| `style` | Formatação que não afeta lógica |

Exemplos:

- `feat(siafi): adiciona dag de ingestao de notas de credito`
- `fix(dbt): corrige deduplicacao no modelo slv_contratos_empenhos`
- `docs: adiciona guia de padroes de engenharia`

Regras:

- a descrição deve estar em letras minúsculas e sem ponto final;
- o corpo do commit, quando existir, deve ser separado da descrição por uma linha em branco;
- para fechar uma issue automaticamente, use `Closes: #<numero>` no rodapé.

## 2. Nomenclatura de branches

O repositório usa dois formatos aceitos.

Formato padrão:

```text
<tipo>/<descricao-curta>
```

Formato com issue vinculada:

```text
<numero-da-issue>-<tipo>-<descricao-curta>
```

Exemplos:

- `feat/siafi-nota-credito-ingestao`
- `fix/fechamento-conn-postgres`
- `docs/protocolo-pr`
- `149-feat-ingestao-sisbolsas`
- `24-fix-dag-nota-de-credito`

## 3. Antes de abrir o PR

Atualize sua branch em relação à `main` do repositório principal:

```bash
git fetch upstream
git rebase upstream/main
```

Se o clone usa apenas `origin` apontando para `GovHub-br/data-application-gov-hub`, use:

```bash
git fetch origin
git rebase origin/main
```

Para PRs de código, execute testes e lint localmente:

```bash
make lint
make test
```

Para PRs de DAGs, rode a DAG localmente e confirme que não há erro de importação:

```bash
airflow dags test <nome_da_dag> <data_execucao>
```

Para PRs de modelos dbt, execute os comandos dentro do projeto dbt alterado:

```bash
cd airflow_lappis/dags/dbt/<projeto>
dbt run --select <modelo>
dbt test --select <modelo>
```

Para PRs de documentação, revise ortografia, links, navegação e formatação.

## 4. Preenchimento do PR

O título do PR deve ser curto, descritivo e seguir o mesmo padrão dos commits.

A descrição deve conter:

- **Descrição:** o que foi feito e por que;
- **Issues relacionadas:** issue vinculada, quando existir;
- **Como testar / validar:** comandos e passos para o revisor;
- **Evidências:** logs, prints, resultados de testes, consultas ou links relevantes;
- **Checklist:** itens obrigatórios verificados antes da revisão.

Exemplo:

```md
## Descrição

Adicionada DAG de ingestão de notas de crédito do SIAFI.

## Issues relacionadas

Closes #42

## Como testar / validar

1. Subir o ambiente local
2. Executar `airflow dags test nota_credito_siafi_ingest_dag 2025-01-01`
3. Verificar registros inseridos no schema esperado

## Evidências

- `make lint` executado com sucesso
- `make test` executado com sucesso

## Checklist

- [x] Título do PR segue Conventional Commits
- [x] Issue relacionada foi referenciada
- [x] Testes/lint foram executados ou a ausência foi justificada
- [x] Documentação atualizada, se aplicável
```

## 5. Revisores

| Tipo de PR | Revisores |
| --- | --- |
| DAGs de ingestão | `@GovHub-br/developers` |
| Modelos dbt | `@GovHub-br/developers` |
| Plugins e helpers | `@GovHub-br/infra` |
| Documentação de configuração, infraestrutura ou deploy | `@GovHub-br/infra` |
| Documentação de DAGs, modelos dbt ou fluxo de dados | `@GovHub-br/developers` |

Caso um PR altere mais de um domínio, solicite revisão de todos os times responsáveis pelas áreas impactadas.

### Número mínimo de aprovações

- PRs de código: mínimo de **1 aprovação** de uma pessoa revisora do domínio alterado.
- PRs de documentação: mínimo de **1 aprovação** de uma pessoa responsável pelo tipo de documentação.
- PRs críticos, sensíveis ou com impacto em produção: recomendado exigir **2 aprovações**.

### CODEOWNERS e proteção da main

O repositório da aplicação mantém um `CODEOWNERS` com responsáveis por domínio técnico. Para que essas regras bloqueiem merges automaticamente, a branch `main` deve exigir revisão e revisão dos code owners, conforme a plataforma usada pelo repositório.

Enquanto essas proteções não estiverem ativas, autores, revisores e mantenedores devem aplicar este protocolo manualmente.

## 6. Durante a revisão

O revisor deve:

- aprovar o PR se estiver tudo certo;
- solicitar mudanças com comentários claros e objetivos;
- explicar o problema e, quando possível, sugerir como corrigir;
- bloquear o PR com `request changes` apenas em casos reais: bug, credencial exposta, violação de padrão crítico ou dado sensível.

Comentários de estilo ou preferência pessoal que não violam nenhum padrão documentado não devem bloquear o merge. Eles podem ser deixados como sugestão opcional.

Recomenda-se que a primeira resposta de revisão aconteça em até **2 dias úteis**, salvo indisponibilidade do time.

## 7. Após pedido de mudança

O autor deve:

- responder a todos os comentários antes de pedir nova revisão;
- aplicar a mudança solicitada ou justificar por que ela não faz sentido;
- marcar cada comentário como resolvido após endereçá-lo;
- avisar o revisor quando as mudanças estiverem prontas para nova rodada.

Se houver discordância, a discussão deve acontecer na própria thread do PR. Se não houver consenso, escale para o time no canal de comunicação para evitar que o PR fique parado indefinidamente.

## 8. Critérios de aprovação

### Para código

- [ ] A DAG ou modelo segue os [padrões de engenharia](padroes-engenharia.md)
- [ ] Não há `SELECT *` em modelos finais dbt
- [ ] Não há credenciais ou dados sensíveis commitados
- [ ] URLs, tokens, credenciais e certificados foram classificados corretamente: endpoint público não deve ser mascarado se o código depende dele; segredo real deve ser movido para variável, connection ou secret manager
- [ ] Testes passam (`make test`, `dbt test`)
- [ ] Lint passa (`make lint`)
- [ ] Commits seguem Conventional Commits
- [ ] A lógica está correta e o código é legível
- [ ] Plugins e helpers existentes foram reaproveitados quando aplicável

### Para documentação

- [ ] O conteúdo é preciso e reflete o estado real do repositório
- [ ] A formatação Markdown está correta
- [ ] Links estão funcionando
- [ ] Não contradiz outras documentações existentes
- [ ] A página nova foi adicionada ao `nav` do `mkdocs.yml`, quando aplicável

## 9. Merge

- O merge só pode ser feito após todas as aprovações necessárias.
- Usar **merge commit** como padrão do repositório, preservando o histórico completo dos PRs.
- Deletar a branch após o merge.

## 10. PRs urgentes

Em casos excepcionais que exijam merge imediato, como incidente em produção ou correção crítica:

- notifique o time no canal de comunicação antes de mergear;
- obtenha no mínimo **1 aprovação** de pessoa do time responsável;
- abra issue de acompanhamento para revisão posterior, se necessário.

## 11. Incidentes de segurança

PRs relacionados a credenciais expostas, dados sensíveis, autenticação, autorização ou limpeza de histórico devem ser tratados como críticos.

Regras mínimas:

- não publicar valores de secrets em issues, PRs, chats ou documentação;
- rotacionar ou revogar credenciais expostas antes de tratar limpeza de histórico como concluída;
- não substituir endpoints públicos por `***REMOVED***` quando isso quebra execução do cliente;
- coordenar reescrita de histórico com mantenedores e contribuidores;
- orientar reclone do repositório quando houver force-push após limpeza de histórico.

## Referências

- [Padrões de engenharia](padroes-engenharia.md)
- [Guia de contribuição](../CONTRIBUTING.md)
- [Segurança](../governanca/seguranca.md)
