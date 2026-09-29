# Time de Agentes de Engenharia de Dados — Pacote Claude Code

## Como instalar

1. Copie a pasta `.claude/` inteira para a raiz do seu projeto (ou mescle
   com uma `.claude/` já existente — atenção especial ao `settings.json`,
   caso você já tenha um: mescle as chaves em vez de sobrescrever).
2. Copie `ARCHITECTURE.md` e `CLAUDE.md` para a raiz do projeto.
3. Copie a pasta `projects/` (ou crie uma vazia) — é onde o time cria
   cada projeto de dados novo, um subdiretório por projeto.
4. Reinicie o Claude Code (necessário na primeira vez que a pasta
   .claude/agents/ é criada) ou abra uma nova sessão no projeto.
5. Rode `/agents` para confirmar que os 9 subagentes foram carregados.

## Agentes incluídos

nexus · axis · pyxel · schema · forge · sentinel · flow · vera · codex

Veja ARCHITECTURE.md para o papel de cada um e o fluxo de trabalho completo.
Veja CLAUDE.md para a estrutura do repositório e as regras práticas
(onde criar projetos, convenção de nomenclatura, fluxo obrigatório).

## Plan Mode como padrão (.claude/settings.json)

O pacote inclui `.claude/settings.json` com `"defaultMode": "plan"`. Isso
faz toda sessão iniciar em **Plan Mode**: o Claude Code só explora e
propõe um plano (sem editar arquivos ou rodar comandos de escrita) até
você aprovar. Reforça, a nível de produto, o princípio de "perguntar
antes de assumir" já presente nos prompts do Nexus e do Axis.

- Para sair do Plan Mode numa sessão específica: `Shift+Tab` alterna
  para `acceptEdits`.
- Para desativar esse padrão no projeto: remova o arquivo ou apague a
  chave `defaultMode`.

## Uso

- Automático: descreva a tarefa e o Claude Code delega com base na
  `description` de cada agente.
- Explícito: "@pyxel crie um pipeline que..." ou "use o agente axis para..."
- Fluxo completo (fiel à arquitetura): comece sempre pedindo ao nexus,
  que indica a quem delegar; ao final de uma tarefa técnica, peça
  explicitamente "use a vera para validar essa entrega" e depois
  "use o codex para documentar", replicando o duplo gate antes de
  considerar algo concluído.
# data-engineer-agent-team
