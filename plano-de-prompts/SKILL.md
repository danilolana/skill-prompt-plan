---
name: plano-de-prompts
description: Transforma uma alteração ou criação em um sistema (reunião, transcrição, resumo, briefing, mapa mental, lista de mudanças) em tópicos de trabalho, lê as skills instaladas e monta um plano de prompts prontos para Claude Code ou Codex (um por tópico, com etapas marcadas por modelo), registrando decisões e pendências no cofre do Obsidian quando houver. Use sempre que o usuário falar em "plano de prompts", "prompts para o Claude Code/Codex", "separe as alterações por tópicos", "analise essa reunião e monte as tarefas", "o sistema vai sofrer alterações", "atualize o cofre com o que foi decidido", ou colar o relatório de um prompt que já rodou e perguntar se pode seguir. Also use to turn meeting notes or change requests into an ordered, dependency-aware prompt plan with haiku/sonnet/opus routing, even if the user never says "skill" or "plano".
---

# Plano de prompts por tópico

Este fluxo leva uma mudança em um sistema, do material bruto (reunião, briefing) até prompts executáveis e um cofre do Obsidian atualizado. Ele nasceu de um projeto real e funciona porque separa quatro coisas que costumam se misturar: o que foi decidido, o que é suposição, o que está em aberto e o que já foi executado.

## Princípios

- **Decisão é do usuário.** Registre como decidido só o que ele disse. Tudo o que você inferiu vira "assumi" e entra na lista de confirmação. Registrar uma suposição como decisão contamina os prompts seguintes.
- **Leia antes de perguntar.** O cofre, a memória e o repositório respondem metade das dúvidas. Pergunte só o que muda o que você vai construir.
- **O prompt começa sem memória.** O agente de código que vai executá-lo não viu esta conversa. Cada prompt precisa levar contexto, regras, estado atual, decisões e critério de entrega.
- **Registre a cada decisão**, na mesma rodada em que ela é dita. Se o usuário pediu "sempre atualize o cofre", isso é regra da sessão inteira.
- **Seja honesto sobre o que não foi verificado.** "Testes passaram" e "não abri no navegador" valem o mesmo peso no relatório.
- **Fale como o usuário fala.** Português do Brasil por padrão, direto, sem emojis, listas só quando há comparação.

## Fase 0: contexto e acesso

1. Descubra o projeto-alvo e se existe cofre do Obsidian: pasta conectada na sessão, memória do usuário, nota-mapa (MOC) do projeto. Se a pasta não estiver conectada e o computador estiver ligado à sessão, peça acesso à menor pasta necessária, uma vez.
2. Leia as notas do projeto: mapa, arquitetura, decisões, pendências, regras de negócio. Notas pequenas: copie todas de uma vez para a área de trabalho e leia juntas.
3. Sem cofre: peça o contexto mínimo, trabalhe com o repositório se houver acesso e registre num arquivo markdown ou no próprio chat. Pergunte uma vez se o usuário quer criar um cofre ou arquivo de registro.
4. Se o estado atual vem só das notas (sem ver o código), diga isso e marque os pontos como "a confirmar no código".

## Fase 1: entender a mudança

1. Leia a **transcrição inteira**, não só o resumo. Resumos automáticos fundem divergências, inventam consenso e omitem hesitações. Quando resumo e transcrição discordam, registre a divergência em vez de escolher em silêncio. Trecho ilegível não vira fato.
2. Compare com o estado atual e produza **tópicos T1..Tn**. Cada tópico tem: título curto, **Decidido**, **Impacto** (módulos, telas, tabelas conhecidos e riscos) e **Em aberto**.
3. Cheque o que costuma estar errado em reuniões:
   - contas e números (preço por unidade × quantidade; "20 por 25" e "R$ 0,80 cada" não fecham);
   - nomes e identificadores (renomear nome de exibição é barato; renomear slug quebra referências);
   - prazos comparados com a data de hoje (o prazo pode ser hoje);
   - dependências entre tópicos e ordem de deploy;
   - riscos irreversíveis: migração de dados, remoção de registros, cobrança.
4. Separe o que é código do que não é (campanha, jurídico, marketing). O que não é código fica registrado como histórico, sem gerar prompt. Se o usuário disser "não vamos decidir X", retire X do escopo e registre isso.
5. Salve a nota de tópicos no cofre antes de perguntar (ver "Cofre").
6. Mostre ao usuário: tópicos em lista curta, o que muda no sistema e os pontos em aberto numerados, com sua recomendação em uma frase quando você tiver uma.

## Fase 2: fechar as pendências

- Pergunte em lote, numerado, respondível em uma linha por item. Inclua a opção padrão que você recomendaria.
- Ao receber respostas: grave no cofre (tópicos, decisões, pendências, nota da sessão) e responda com o que ficou registrado, o que continua em aberto e as suposições que você fez. Repita até zerar.
- Se uma resposta contradiz uma conta ou outra decisão, aponte na hora, com os números.
- Pergunte a política de git e deploy se ainda não souber. Padrão conservador: trabalhar na branch que o usuário indicar, commits locais, sem push, sem merge na branch de produção, sem deploy, migrações apenas preparadas e testadas em banco local.

## Fase 3: skills instaladas

- Liste as skills do ambiente atual. Lembre que o agente que vai executar os prompts (Claude Code, Codex) pode ter outras: peça os nomes ou, com autorização, leia só os nomes e descrições nas pastas de skills do computador (por exemplo `.claude/skills`, `.agents`, `.codex`, `.impeccable`, no projeto ou no diretório do usuário).
- Mapeie skill → tópico ou etapa (design, testes, banco, revisão). Use skills só onde elas agregam.
- Se o usuário disser que coloca as skills depois, deixe a linha `SKILLS:` em branco em cada prompt. Se disser "segura essa parte", registre e não pergunte de novo.

## Fase 4: o plano de prompts

Regras:
- **Um prompt por tópico que mexe em código.** Tópicos sem prompt dizem por quê (já funciona, fora de escopo).
- **Ordem por dependência**, em tabela: prompt, tópico, depende de, observação. Cada prompt checa o pré-requisito e manda parar e avisar se não estiver cumprido.
- **Autocontido**: use o modelo abaixo.
- **Roteamento de modelos** por etapa, marcado no texto como [HAIKU], [SONNET] ou [OPUS]:

| Etapa | Modelo |
| --- | --- |
| Inventário somente leitura, grep de ocorrências, atualização de textos e docs, rodar testes e relatar | haiku |
| Implementação de telas, rotas, serviços, seeds, testes | sonnet |
| Migração de dados irreversível, concorrência e transações, cobrança e webhook, desenho de modelo novo, revisão final de segurança e consistência | opus (use o mais forte disponível, e só nessas etapas) |

O prompt manda o orquestrador criar subagentes com o parâmetro de modelo explícito e rebaixar uma etapa [OPUS] que se mostrar simples, explicando por quê no relatório. A sessão principal deve ser aberta num modelo intermediário; o prompt não consegue forçar isso, então avise o usuário.

### Modelo de prompt

```text
OBJETIVO
<uma ou duas frases do que entregar e do que NÃO mexer>

SKILLS: <deixe em branco se o usuário vai preencher>

PRÉ-REQUISITO
<o que precisa já estar no repositório; se não estiver, pare e avise>

REGRAS GERAIS
- Projeto: <stack e pastas>. Leia <docs do repositório> antes de mudar algo.
- Git: <política decidida: branch, commits locais, sem push, sem merge na produção, sem deploy>. Antes de começar confirme a branch atual e a árvore limpa.
- Banco: mudança de schema ou dados só por migração nova e idempotente, testada em banco local descartável. Não rode nada em produção; entregue um roteiro.
- Segredos: não leia, imprima nem copie valores de .env, .secrets ou variáveis de ambiente.
- <regras do projeto: preços vêm da API, idioma da interface, convenções>
- Diagnostique e reporte primeiro; só depois altere. Nada fora do escopo.

MODELOS
Você é o orquestrador. Delegue cada etapa a um subagente com o parâmetro de modelo explícito: haiku nas [HAIKU], sonnet nas [SONNET], opus somente nas [OPUS]. Se uma etapa [OPUS] se mostrar simples, rebaixe e justifique.

ESTADO ATUAL (confirme no código, pode ter mudado)
<o que o cofre diz hoje>

REGRAS DE NEGÓCIO (decididas)
<só o que o usuário decidiu, numa lista; o que foi suposição fica de fora ou marcado>

ETAPAS
1. [HAIKU] Diagnóstico somente leitura: ...
2. [OPUS] <desenho do que é arriscado> ...
3. [SONNET] Backend ...
4. [SONNET] Frontend ...
5. [SONNET] Testes (liste os casos, inclusive concorrência e idempotência quando houver) e rode tudo; reporte resultados reais.
6. [OPUS] Revisão final do diff: sobras, riscos, segredos, valores alterados sem querer.

ENTREGA
Relatório curto: arquivos alterados, migrações, o que foi e o que NÃO foi testado, roteiro de produção (backup, migrar, validar) e pendências. Não faça push nem deploy.
```

### Armadilhas para escrever dentro dos prompts

Estas vieram de problemas reais; cada uma custa pouco prevenir e muito consertar depois.
- **Seed idempotente que recria o que você apagou**: se o seed insere por identificador a cada inicialização, renomear ou remover registros exige atualizar o seed junto.
- **Unicidade e ordem na migração**: renomear para um identificador que já existe falha; defina a ordem (mover referências, apagar, renomear, inserir) e torne a migração idempotente.
- **Ordem de deploy**: se o app inicializa dados ao subir, a API nova antes da migração corrompe o estado. Peça um guard que recuse subir sem a migração registrada, ou a confirmação do script de deploy.
- **Concorrência**: limites e saldos checados por contagem precisam de travamento. Em MySQL com REPEATABLE READ, encerre a transação antes de travar a linha, senão o snapshot deixa duas requisições passarem. Peça teste com concorrência real num banco descartável.
- **Idempotência de webhook** de pagamento, e crédito só com pagamento confirmado.
- **Trava de ambiente**: uma regra como "só sandbox" bloqueia a venda real. Se o objetivo é vender, o prompt deve pedir a trava como flag explícita e dizer isso ao usuário.
- **Painel admin pode desfazer invariantes** (remover uma flag, apagar uma chave de limite): peça validação.
- **Referências ao que será removido** (chaves estrangeiras, histórico): inventário antes, backup, migração dos registros existentes.
- **Quebra de linha CRLF** polui o diff em Windows; peça para preservar a quebra de linha original.
- **Verificação visual**: se o front não for aberto no navegador, o relatório deve dizer.

Entrega desta fase: salve a nota do plano no cofre, cole os prompts no chat em blocos de código (o usuário copia de lá) e termine com a ordem de execução e o que ainda está em aberto.

## Cofre do Obsidian

Notas por projeto, em português, com frontmatter (`title`, `tags`, `tipo`, `criado`, `atualizado`) e links `[[nota]]`:
- **Nota de tópicos** (ex.: "Reunião AAAA-MM-DD - Alterações por tópico"): origem, prazos, T1..Tn com Decidido / Impacto / Em aberto, escopo do plano.
- **Plano de prompts**: ordem e dependências, regra de modelos, regras de git, os prompts em blocos de código, instrução de trazer o relatório de volta.
- **Decisões**: seção datada com só o que foi decidido.
- **Pendências**: caixas de seleção; `[x]` só onde o código foi feito, com "falta deploy" quando for o caso; sobras dos relatórios entram aqui.
- **Nota de sessão**: pedido, feito, respostas por rodada, próximo passo, limites.
- **Mapa do projeto (MOC)**: link para cada nota nova.
- Nunca grave segredos nem valores de `.env`.

Escrita segura pela ponte com o computador:
1. Copie para a área de trabalho as notas que vai alterar (isso devolve o `mtimeMs`). Faça isso de novo antes de cada rodada de edição: o agente de código e o próprio usuário também escrevem no cofre.
2. Edite as cópias com substituições exatas (falhe se o trecho não existir) e acrescente seções em vez de reescrever.
3. Grave de volta com a proteção de data de modificação (`expectedMtimeMs`). Se for rejeitado, copie de novo e refaça.
4. Se o agente de código criou notas de sessão no cofre, leia e referencie com link; não duplique.
5. Se o shell do computador ainda estiver preparando o ambiente, use apenas copiar e gravar.

## Depois da execução

Quando o usuário colar o relatório de um prompt (do Claude Code, do Codex ou de outra ferramenta):
1. Leia com ceticismo: o que foi **testado de verdade**, o que **não** foi, desvios do prompt, riscos que o próprio agente listou, tamanho do diff (remoções grandes podem ser ruído de quebra de linha).
2. Atualize o cofre: marque pendências concluídas, acrescente as sobras, registre quem executou e em qual branch.
3. **Ajuste os prompts seguintes** com o que foi descoberto: uma técnica que resolveu concorrência, uma corrida que o agente achou fora do escopo, uma trava que atrapalha o objetivo, conflitos de branch. Diga que o prompt no cofre é a versão original se o usuário o alterou por conta própria.
4. Responda se pode rodar o próximo, o que precisa de decisão antes de publicar e o que é só limpeza.
5. Se sobrarem itens, ofereça um prompt curto de ajustes (Sonnet, com uma revisão Opus apenas onde há risco de deploy ou dados).
6. Antes de qualquer deploy, consolide um roteiro único: backup, ordem das migrações, variáveis de ambiente a conferir, validações e como desfazer.
