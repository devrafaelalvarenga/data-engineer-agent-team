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
│   └── agents/                # os 9 subagentes do time
│       ├── nexus.md           # Gestor — único ponto de entrada de demandas
│       ├── axis.md            # Arquiteto — gate obrigatório de pré-distribuição
│       ├── pyxel.md           # Python / pipelines
│       ├── schema.md          # SQL / modelagem
│       ├── forge.md           # Infraestrutura / cloud
│       ├── sentinel.md        # Qualidade / governança
│       ├── flow.md            # Orquestração
│       ├── vera.md            # QA / gate de saída (corretude técnica)
│       └── codex.md           # Documentação / catalogação (gate de saída)
└── projects/                  # TODO código real desenvolvido pelo time vive aqui
    ├── <nome-do-projeto>/      # um subdiretório por projeto/artefato
    └── ...
```

## Onde criar um projeto novo

- Todo projeto novo (pipeline Python, modelo dbt, módulo de infraestrutura, DAG de orquestração etc.) é criado em **`/projects/<nome-do-projeto>/`** — nunca na raiz do repositório.
- A estrutura interna de cada subdiretório segue o **template definido pelo Axis** para aquele tipo de artefato (ver `ARCHITECTURE.md`, seção sobre scaffolding). Não invente um layout novo sem que o Axis tenha validado.
- `<nome-do-projeto>` também segue a convenção de nomenclatura do time: **em inglês**, minúsculo, com hífen (ex.: `orders-etl-pipeline`, `customer-360-dbt`, não `pipeline_pedidos` ou `PipelineDeClientes`).

## Fluxo de trabalho obrigatório (resumo — ver ARCHITECTURE.md para detalhes)

1. **Toda demanda começa pelo `nexus`** — não delegue direto a um especialista sem passar por ele primeiro, mesmo em conversas informais neste chat.
2. **Toda tarefa passa pelo gate do `axis`** antes de ser distribuída — rotineira ou não. É ele quem confirma/prepara a estrutura em `/projects/<nome>/` e a convenção de nomenclatura antes de qualquer especialista começar a trabalhar.
3. **Especialistas** (`pyxel`, `schema`, `forge`, `sentinel`, `flow`) implementam dentro do subdiretório de projeto já validado pelo Axis.
4. **Toda entrega passa por `vera`** (corretude técnica) e depois por `codex` (documentação/catalogação) antes de ser considerada concluída — sem exceção para tarefas simples.
5. Só depois da dupla aprovação (Vera + Codex) a tarefa volta ao **Nexus** como de fato finalizada.

## Convenção de nomenclatura e idioma (aplica-se a todo código em `/projects/`)

- **Identificadores de código** (variáveis, funções, classes, tabelas, colunas, arquivos, nomes de recursos de infraestrutura, DAGs, nomes de projeto/pasta) → **inglês**.
- **Comentários, docstrings e toda documentação** (READMEs, catálogo, glossário) → **português**.
- Definida e fiscalizada pelo `axis` (na entrada) e pela `vera` (na saída). Ver exemplo completo em `axis.md`.

## Comandos úteis

Este repositório não tem build/lint/test próprio — os 9 arquivos em `.claude/agents/` são prompts (Markdown + YAML), não código executável. Comandos relevantes ao trabalhar aqui:

- `/agents` — confirma que os 9 subagentes foram carregados corretamente numa sessão nova.
- `Shift+Tab` — alterna a sessão para fora do Plan Mode (`acceptEdits`) quando necessário.
- Uma vez que um projeto real exista em `/projects/<nome>/`, seus próprios comandos de build/lint/test são definidos pelo stack escolhido pelo especialista responsável e documentados pelo `codex` dentro do próprio subdiretório — não aqui.

## Plan Mode

Este projeto usa `"defaultMode": "plan"` em `.claude/settings.json`. Toda sessão nova começa em modo de leitura/planejamento — nenhuma escrita em `/projects/` acontece sem um plano aprovado primeiro. Isso reforça, a nível de produto, a mesma lógica de "não assumir, perguntar antes" que já está nos prompts do Nexus e do Axis.

## Otimização de tokens (decisão de arquitetura)

- **`omitClaudeMd: true`** está definido em `pyxel`, `schema`, `forge`, `sentinel`, `flow`, `vera` e `codex`. Esses agentes recebem o caminho exato do projeto e o contexto necessário já dentro da própria tarefa delegada pelo Nexus/Axis — não precisam recarregar este `CLAUDE.md` inteiro a cada invocação. **`nexus` e `axis` mantêm o carregamento normal**, pois orquestram o fluxo e usam a estrutura do repositório diretamente.
- **`vera` e `codex` rodam em `model: haiku`** — testar contra critérios já definidos e catalogar um artefato já aprovado são tarefas mais mecânicas/estruturadas, que não exigem o raciocínio mais caro de um modelo maior. Os demais especialistas seguem em `sonnet`, e o `axis` em `opus` (decisões estruturais se beneficiam de mais capacidade de raciocínio).
- Ao editar os arquivos de subagente, mantenha as `description` enxutas — o Claude Code carrega a descrição de **todos** os subagentes no início de toda sessão (soma acima de 15.000 tokens gera aviso); detalhe extra deve ir no corpo do prompt, que só carrega quando o agente é de fato invocado.

## Ao trabalhar fora dos subagentes (nesta conversa principal)

Se o pedido do usuário for sobre os **agentes/arquitetura do time** (criar, ajustar, revisar um papel), trabalhe nos arquivos de `.claude/agents/` e em `ARCHITECTURE.md` diretamente.

Se o pedido for sobre **um projeto de dados específico** (ex.: "crie um pipeline que lê do S3"), o fluxo correto é: confirmar com o usuário se ele quer que você simule o fluxo do time (nexus → axis → especialista → vera → codex) ou implemente diretamente; por padrão, para trabalho real dentro de `/projects/`, prefira invocar os subagentes na ordem descrita acima em vez de implementar por conta própria fora desse fluxo.

🧠 Como a IA deve se comunicar
Sou AI Data Engineering em formação— não preciso de explicação de conceitos básicos de programação, SQL, Python ou pipelines de dados tradicionais (ETL, orquestração, warehousing).
Explicar em detalhe apenas conceitos específicos do universo de IA/LLM ainda não dominados: embeddings, estratégias de chunking, vector DBs, RAG, LLMOps, arquitetura multi-agente.
Ir direto ao ponto: priorizar objetividade e justificativa técnica das decisões em vez de explicações longas.
Quando sugerir uma abordagem técnica, expor brevemente o trade-off (por que essa opção e não outra).
Responder sempre em Português do Brasil.
