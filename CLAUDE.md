# CLAUDE.md

Este arquivo orienta o Claude Code ao trabalhar neste repositório.

## O que é este projeto

Este repositório hospeda um **time de agentes de IA para engenharia de dados**, implementado como subagentes do Claude Code (`.claude/agents/`), e os **projetos de dados reais** que esse time desenvolve, isolados em `/projects/`.

A arquitetura completa do time (papéis, fluxo de decisão, gates obrigatórios) está documentada em **`ARCHITECTURE.md`** — leia-o antes de delegar ou executar qualquer tarefa relacionada aos agentes. Este `CLAUDE.md` cobre a estrutura do repositório e as regras práticas de onde/como trabalhar.

## Estrutura do repositório

```
/
├── ARCHITECTURE.md          # papéis, fluxo de decisão e gates do time — leia primeiro
├── CLAUDE.md                # este arquivo
├── README.md                # instruções de instalação do pacote de agentes
├── .claude/
│   ├── settings.json         # Plan Mode como padrão da sessão
│   └── agents/                # os 11 subagentes do time
│       ├── scope.md           # Analista de requisitos — porta de entrada de projetos novos
│       ├── nexus.md           # Gestor — entrada de backlog/status/incidentes
│       ├── axis.md            # Arquiteto — gate obrigatório de pré-distribuição
│       ├── pyxel.md           # Python / pipelines
│       ├── schema.md          # SQL / modelagem
│       ├── forge.md           # Infraestrutura / cloud
│       ├── sentinel.md        # Qualidade / governança
│       ├── flow.md            # Orquestração
│       ├── vera.md            # QA / gate de saída (corretude técnica)
│       ├── codex.md           # Documentação / catalogação (gate de saída)
│       └── deploy.md          # Implantação — após Vera + Codex; prod com aprovação do Axis + humana
├── docs/                      # criado pelo Axis sob demanda
│   ├── templates/             # templates de estrutura por tipo de artefato
│   └── adr/                   # ADRs transversais (ADRs de projeto ficam em projects/<nome>/docs/adr/)
└── projects/                  # TODO código real desenvolvido pelo time vive aqui
    ├── <nome-do-projeto>/      # um subdiretório por projeto/artefato
    └── ...
```

## Onde criar um projeto novo

- Todo projeto novo (pipeline Python, modelo dbt, módulo de infraestrutura, DAG de orquestração etc.) é criado em **`/projects/<nome-do-projeto>/`** — nunca na raiz do repositório.
- A estrutura interna de cada subdiretório segue o **template definido pelo Axis** para aquele tipo de artefato (ver `ARCHITECTURE.md`, seção sobre scaffolding). Não invente um layout novo sem que o Axis tenha validado.
- `<nome-do-projeto>` também segue a convenção de nomenclatura do time: **em inglês**, minúsculo, com hífen (ex.: `orders-etl-pipeline`, `customer-360-dbt`, não `pipeline_pedidos` ou `PipelineDeClientes`).

## Fluxo de trabalho obrigatório (resumo — ver ARCHITECTURE.md para detalhes)

0. **Projeto novo ou ideia vaga começa pelo `scope`** — ele devolve perguntas ou um brief funcional; a conversa principal media as perguntas ao stakeholder (subagentes não conversam com o usuário) **de forma interativa, com `AskUserQuestion`** (até 4 perguntas por chamada, em sequência; o Scope não tem limite de perguntas). O brief tem dois status: `PENDENTE` e `VALIDADO` (stakeholder confirmou e não há perguntas em aberto). `VALIDADO` vai ao `nexus`, que o analisa e faz a sua validação antes de planejar; depois o `axis` valida a viabilidade no gate. Cada um só valida quando não houver questionamento pendente; qualquer questionamento devolve o brief ao Scope como `PENDENTE`, que valida as respostas com o stakeholder e reenvia a quem devolveu.
1. **Toda demanda começa pelo `nexus`** (projetos novos: com o brief `VALIDADO` do `scope`) — não delegue direto a um especialista sem passar por ele primeiro, mesmo em conversas informais neste chat.
2. **Toda tarefa passa pelo gate do `axis`** antes de ser distribuída — rotineira ou não. É ele quem confirma/prepara a estrutura em `/projects/<nome>/` e a convenção de nomenclatura antes de qualquer especialista começar a trabalhar.
3. **Especialistas** (`pyxel`, `schema`, `forge`, `sentinel`, `flow`) implementam dentro do subdiretório de projeto já validado pelo Axis.
4. **Toda entrega passa por `vera`** (corretude técnica) e depois por `codex` (documentação/catalogação) antes de ser considerada concluída — sem exceção para tarefas simples.
5. Entregas que vão a um ambiente passam, depois da dupla aprovação (Vera + Codex), pelo **`deploy`**, que implanta a versão aprovada (somente staging e produção), em janela de manutenção, com blue/green para tabelas, smoke test pós-deploy e rollback se falhar; **produção exige duas aprovações a cada deploy: o `axis` aprova o projeto aplicado e o humano aprova as atividades de subida executadas pelo `deploy`**. O `forge` provisiona a infraestrutura; o `deploy` não a altera nem assume o papel de nenhum outro agente.
6. Só depois disso (Vera + Codex + Deploy, quando aplicável) a tarefa volta ao **Nexus** como de fato finalizada.

## Convenção de nomenclatura e idioma (aplica-se a todo código em `/projects/`)

- **Identificadores de código** (variáveis, funções, classes, tabelas, colunas, arquivos, nomes de recursos de infraestrutura, DAGs, nomes de projeto/pasta) → **inglês**.
- **Comentários, docstrings e toda documentação** (READMEs, catálogo, glossário) → **português**.
- Definida e fiscalizada pelo `axis` (na entrada) e pela `vera` (na saída). Ver exemplo completo em `axis.md`.

## Comandos úteis

Este repositório não tem build/lint/test próprio — os 11 arquivos em `.claude/agents/` são prompts (Markdown + YAML), não código executável. Comandos relevantes ao trabalhar aqui:

- `/agents` — confirma que os 11 subagentes foram carregados corretamente numa sessão nova.
- `Shift+Tab` — alterna a sessão para fora do Plan Mode (`acceptEdits`) quando necessário.
- Uma vez que um projeto real exista em `/projects/<nome>/`, seus próprios comandos de build/lint/test são definidos pelo stack escolhido pelo especialista responsável e documentados pelo `codex` dentro do próprio subdiretório — não aqui.

## Plan Mode

Este projeto usa `"defaultMode": "plan"` em `.claude/settings.json`. Toda sessão nova começa em modo de leitura/planejamento — nenhuma escrita em `/projects/` acontece sem um plano aprovado primeiro. Isso reforça, a nível de produto, a mesma lógica de "não assumir, perguntar antes" que já está nos prompts do Nexus e do Axis.

## Otimização de tokens (decisão de arquitetura)

- **`omitClaudeMd: true`** está definido em `scope`, `pyxel`, `schema`, `forge`, `sentinel`, `flow`, `vera`, `codex` e `deploy`. Esses agentes recebem o caminho exato do projeto e o contexto necessário já dentro da própria tarefa delegada pelo Nexus/Axis — não precisam recarregar este `CLAUDE.md` inteiro a cada invocação. **`nexus` e `axis` mantêm o carregamento normal**, pois orquestram o fluxo e usam a estrutura do repositório diretamente. O `axis` tem `Write`/`Edit` restritos por prompt a templates, esqueleto de projeto, `docs/brief.md` e ADRs — nunca código de produção.
- **`vera` e `codex` rodam em `model: haiku`** — testar contra critérios já definidos e catalogar um artefato já aprovado são tarefas mais mecânicas/estruturadas, que não exigem o raciocínio mais caro de um modelo maior. Os demais especialistas e o `scope` seguem em `sonnet`, e o `axis` em `opus` (decisões estruturais se beneficiam de mais capacidade de raciocínio).
- Ao editar os arquivos de subagente, mantenha as `description` enxutas — o Claude Code carrega a descrição de **todos** os subagentes no início de toda sessão (soma acima de 15.000 tokens gera aviso); detalhe extra deve ir no corpo do prompt, que só carrega quando o agente é de fato invocado.

## Repasse de estado entre invocações (conversa principal)

Cada invocação de subagente é **isolada**: ele não lembra das anteriores e só enxerga o que vem no prompt (e o que está gravado em disco). Por isso, a cada chamada a conversa principal deve repassar **todo o estado necessário**, sem resumir nem reconstruir de memória:

- **Scope**: ideia bruta + fatos e respostas de **todas** as rodadas anteriores + o **texto integral do brief anterior** (itens 1 a 11, com status) quando houver. Sem o brief, o Scope não consegue atualizá-lo e se recusa a reconstruí-lo (e deve se recusar).
- **Nexus, Axis, especialistas, Vera, Codex e Deploy**: o brief `VALIDADO` completo (ou o caminho de `projects/<nome>/docs/brief.md`, quando já gravado), o plano de tarefas do Nexus, as decisões do Axis (ADRs, estrutura de pastas, quem atua), o escopo exato da tarefa e as **aprovações já obtidas** (Vera, Codex, Axis, humana) com a versão a que se referem.
- Cole os textos integrais; não escreva "o brief da rodada anterior". O que já está em disco pode ser passado por caminho.

## Ao trabalhar fora dos subagentes (nesta conversa principal)

Se o pedido do usuário for sobre os **agentes/arquitetura do time** (criar, ajustar, revisar um papel), trabalhe nos arquivos de `.claude/agents/` e em `ARCHITECTURE.md` diretamente.

Se o pedido for sobre **um projeto de dados específico** (ex.: "crie um pipeline que lê do S3"), o fluxo correto é: confirmar com o usuário se ele quer que você simule o fluxo do time (nexus → axis → especialista → vera → codex → deploy) ou implemente diretamente; por padrão, para trabalho real dentro de `/projects/`, prefira invocar os subagentes na ordem descrita acima em vez de implementar por conta própria fora desse fluxo.

🧠 Como a IA deve se comunicar
Sou AI Data Engineering em formação— não preciso de explicação de conceitos básicos de programação, SQL, Python ou pipelines de dados tradicionais (ETL, orquestração, warehousing).
Explicar em detalhe apenas conceitos específicos do universo de IA/LLM ainda não dominados: embeddings, estratégias de chunking, vector DBs, RAG, LLMOps, arquitetura multi-agente.
Ir direto ao ponto: priorizar objetividade e justificativa técnica das decisões em vez de explicações longas.
Quando sugerir uma abordagem técnica, expor brevemente o trade-off (por que essa opção e não outra).
Responder sempre em Português do Brasil.
