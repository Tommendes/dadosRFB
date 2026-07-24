---
name: criar-x-prompt
description: Cria um arquivo de prompt em docs/prompts/ a partir de um enunciado e, em seguida, executa o prompt — exceto quando o usuário pedir só para criar. Use quando o usuário invocar criar-x-prompt, pedir para criar um prompt a partir de um enunciado, ou disser "só criar" / "só pra criar" o prompt.
disable-model-invocation: true
---

# Criar X Prompt

## Fluxo

1. Receber o **enunciado** (objetivo, escopo, restrições, arquivos/referências).
2. Escolher pasta e nome do arquivo sob `docs/prompts/`.
3. **Escrever** o `.md` do prompt.
4. **Executar** o prompt na mesma conversa — **exceto** se o usuário disser que é só para criar.

## Só criar (não executar)

Não implementar nada se o enunciado (ou mensagem) indicar explicitamente, por exemplo:

- "só criar"
- "só pra criar"
- "apenas criar o prompt"
- "não executar"

Nesse caso: gravar o `.md`, confirmar o caminho, e parar.

## Destino do arquivo

Base: `docs/prompts/<camada>/<NOME>.md`

**Organização por camada da aplicação** — uma pasta por camada, não por domínio funcional. Ex.: `docs/prompts/api/`, `docs/prompts/dashboard/`, `docs/prompts/shared/`.

Pastas sob `docs/prompts/` (preferir reutilizar as existentes; criar nova se a camada ainda não existir):

| Pasta | Quando usar |
|-------|-------------|
| `api` | Backend, endpoints, services, jobs, ingestão |
| `dashboard` | Frontend, telas, forms, grids, componentes UI |
| `shared` | Código/tipos/padrões compartilhados entre camadas |
| `docs` | Documentação, runbooks, glossários (fora de código) |

Se o enunciado cruzar camadas: preferir a camada principal da mudança; se for contrato compartilhado, usar `shared`. Se a camada não couber nas acima: criar pasta nova em `snake_case` alinhada à estrutura do repo (ex.: `workers`, `cli`).

Nome do arquivo: `SCREAMING_SNAKE_CASE.md`, curto e descritivo (ex.: `PARSER_CNPJ.md`, `CRUD_API_PATTERN.md`).

Se pasta/nome não estiverem claros no enunciado, **inferir** pela camada afetada; só perguntar se houver ambiguidade real.

## Como escrever o prompt

Espelhar o estilo dos prompts acionáveis do repo (quando existirem sob `docs/prompts/`):

1. Título `# PROMPT — …` ou `# Padrão para …`
2. Cabeçalho curto: status (se fizer sentido), escopo, referências a docs/código existentes
3. Objetivo em 1–2 parágrafos
4. Fonte de verdade (arquivos, funções, tabelas)
5. Passos numerados / seções de implementação
6. Restrições (“não fazer X”, “não alterar Y”)
7. Checklist final verificável

Regras:

- Basear-se no **enunciado** — não inventar requisitos.
- Preferir caminhos e símbolos reais do repo.
- Se o enunciado citar docs externos ou outro workspace: indicar extração para `docs/prompts/` (não depender de paths externos como canônicos).
- Português; conciso; sem fluff.
- Não duplicar um prompt existente: se já houver arquivo equivalente, atualizar ou perguntar.

## Execução (após criar)

Salvo “só criar”:

1. Ler o `.md` acabado de gravar.
2. Seguir o prompt como especificação da tarefa.
3. Implementar no código/docs conforme o prompt.
4. Ao terminar: resposta breve (o que mudou / path do prompt).

## Checklist da skill

```
- [ ] Enunciado entendido (objetivo + restrições)
- [ ] Pasta e nome definidos sob docs/prompts/
- [ ] Arquivo .md criado
- [ ] Se NÃO for “só criar”: prompt lido e executado
- [ ] Se for “só criar”: parou após gravar o arquivo
```
