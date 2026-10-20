# prompt-skills-plan

Skill do Claude (`plano-de-prompts`) que transforma uma alteração ou criação em um sistema (reunião, transcrição, briefing, mapa mental) em tópicos de trabalho e em um plano de prompts prontos para Claude Code ou Codex, registrando decisões e pendências no cofre do Obsidian.

## O que a skill faz
1. Lê o contexto do projeto (cofre do Obsidian, memória, repositório).
2. Separa a mudança em tópicos: decidido, impacto, em aberto.
3. Fecha as pendências com perguntas numeradas e registra cada resposta.
4. Lê as skills instaladas e mapeia skill por tópico (ou deixa o campo em branco).
5. Gera um prompt autocontido por tópico de código, em ordem de dependência, com etapas marcadas por modelo (haiku, sonnet, opus) e regras de git, banco e segredos.
6. Depois da execução, avalia o relatório, atualiza o cofre e ajusta os prompts seguintes.

## Instalação
Copie a pasta `plano-de-prompts/` para o diretório de skills do seu agente (por exemplo `~/.claude/skills/`), ou importe o arquivo `.skill` pela interface do Claude.

## Estrutura
- `plano-de-prompts/SKILL.md`: a skill (autocontida).
