# Protocolo de aprovação de Pull Request

Este documento define o fluxo obrigatório para abertura, revisão e aprovação de Pull Requests (PRs) no repositório. O objetivo é garantir qualidade, rastreabilidade e consistência nas contribuições de código e documentação.

---

## Escopo

Este protocolo se aplica a qualquer PR que envolva:

- Código — novas DAGs, alterações em DAGs existentes, modelos dbt, plugins, helpers
- Documentação — criação ou edição de arquivos `.md`, `schema.yml`, `CONTRIBUTING.md`, etc.

---

## Fluxo geral

```
branch de trabalho → commits padronizados → PR aberto → revisão → aprovação → merge na main
```

Nenhum merge deve ser feito diretamente na `main` sem passar pelo fluxo de PR.

---

## 1. Mensagens de commit

O projeto adota o padrão **Conventional Commits**. Toda mensagem de commit deve seguir a estrutura:

```
<tipo>[escopo opcional]: <descrição>

[corpo opcional]

[rodapé(s) opcional(is)]
```

| Tipo | Quando usar |
|---|---|
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

**Exemplos:**

```
feat(siafi): adiciona dag de ingestao de notas de credito

fix(dbt): corrige deduplicacao no modelo slv_contratos_empenhos
Closes: #42

docs: adiciona guia de padroes de engenharia
```

Regras:
- Descrição em letras minúsculas, sem ponto final
- Corpo separado da descrição por uma linha em branco
- Para fechar uma issue automaticamente: `Closes: #<numero>` no rodapé

---

## 2. Nomenclatura de branches

O repositório usa dois formatos aceitos:

**Formato padrão:**
```
<tipo>/<descricao-curta>
```

**Formato com issue vinculada:**
```
<numero-da-issue>-<tipo>-<descricao-curta>
```

**Exemplos reais do repositório:**
```
feat/siafi-nota-credito-ingestao
fix/fechamento-conn-postgres
docs/protocolo-mr
149-feat-ingestao-sisbolsas
24-fix-dag-nota-de-credito
```

---

## 3. Antes de abrir o PR

- Certifique-se de que sua branch está atualizada em relação à `main`:
  ```bash
  git fetch origin
  git rebase origin/main
  ```
- Para PRs de código: execute os testes e o lint localmente antes de abrir:
  ```bash
  make lint
  make test
  ```
- Para PRs de DAGs: rode a DAG localmente e confirme que não há erros de importação.
- Para PRs de modelos dbt: rode `dbt run` e `dbt test` localmente.
- Para PRs de documentação: revise ortografia, links e formatação antes de abrir.

---

## 4. Preenchimento do PR

**Título:** curto e descritivo, seguindo o mesmo padrão dos commits.

**Descrição deve conter:**

- **O que foi feito:** resumo claro das mudanças
- **Por que foi feito:** contexto ou issue relacionada
- **Como testar:** passos para o revisor validar as mudanças
- **Checklist:** itens do `CONTRIBUTING.md` verificados

**Exemplo:**

```
## O que foi feito
Adicionada DAG de ingestão de notas de crédito do SIAFI.

## Por que
Demanda mapeada na issue #42.

## Como testar
1. Subir o ambiente local
2. Triggerar manualmente a DAG `siafi_nota_credito_ingestao`
3. Verificar registros inseridos no schema `siafi`

## Checklist
- [x] DAG segue padrão de nomenclatura
- [x] Commits seguem Conventional Commits
- [x] `make lint` passou
- [x] `make test` passou
- [x] Documentação atualizada se necessário
```

---

## 5. Revisores

### Quem deve revisar

| Tipo de PR | Revisores |
|---|---|
| DAGs de ingestão | A definir |
| Modelos dbt | A definir |
| Plugins e helpers | A definir |
| Documentação | A definir |

> **A definir com a equipe:** indicar os responsáveis por domínio.

### Número mínimo de aprovações

- PRs de código: **a definir**
- PRs de documentação: **a definir**

---


## 5.1 Durante a revisão

O revisor deve:

- Aprovar o PR se estiver tudo certo
- Solicitar mudanças com comentários claros e objetivos — explicar o que está errado e, sempre que possível, sugerir como corrigir
- Bloquear o PR (request changes) apenas em casos de problema real: bug, credencial exposta, violação de padrão crítico, dado sensível

Comentários de estilo ou preferência pessoal que não violam nenhum padrão documentado **não devem bloquear** o merge — podem ser deixados como sugestão opcional.

Prazo esperado de resposta do revisor: **a definir com a equipe**.

---

## 5.2 Após pedido de mudança

O autor deve:

- Responder a **todos** os comentários antes de pedir re-revisão — seja aplicando a mudança ou justificando por que não faz sentido
- Marcar cada comentário como resolvido após endereçá-lo
- Avisar o revisor quando as mudanças estiverem prontas para uma nova rodada

Se houver discordância sobre um comentário, discutir na própria thread do PR. Se não houver consenso, escalar para o time no canal de comunicação — não deixar o PR parado indefinidamente.

---

## 6. Critérios de aprovação

### Para código

- [ ] A DAG ou modelo segue os padrões definidos em `docs/padroes-engenharia.md`
- [ ] Não há `SELECT *` em modelos finais dbt
- [ ] Não há credenciais ou dados sensíveis commitados
- [ ] Testes passam (`make test`, `dbt test`)
- [ ] Lint passa (`make lint`)
- [ ] Commits seguem Conventional Commits
- [ ] A lógica está correta e o código é legível
- [ ] Plugins e helpers existentes foram reaproveitados

### Para documentação

- [ ] O conteúdo é preciso e reflete o estado real do repositório
- [ ] Formatação markdown está correta
- [ ] Links estão funcionando
- [ ] Não contradiz outras documentações existentes
- [ ] Commits seguem Conventional Commits

---

## 7. Merge

- O merge só pode ser feito após todas as aprovações necessárias.
- Usar **merge commit** (padrão do repositório) — preserva o histórico completo dos PRs.
- Deletar a branch após o merge.

---

## 8. PRs urgentes

Em casos excepcionais que exijam merge imediato (incidente em produção, correção crítica):

- Notificar o time no canal de comunicação antes de mergear.
- Mínimo de **1 aprovação** de pessoa a definir.
- Abrir issue de acompanhamento para revisão posterior se necessário.
