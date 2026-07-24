# Prompt para Cursor IDE — Transformação do ETL CNPJ (RFB) em Microserviço

Você está atuando dentro do repositório clonado do projeto "Dados Públicos CNPJ" (ETL Python
que baixa, extrai e carrega os dados abertos de CNPJ da Receita Federal em um banco relacional).

NÃO IMPLEMENTE NADA AINDA NESTA PRIMEIRA RODADA. Sua primeira tarefa é PLANEJAR.

## Objetivo final do projeto

Transformar este ETL Python standalone em um microserviço completo, composto por:

### ORQUESTRADOR PYTHON em code/coletar_dados_e_gravar.py

1. Uma API em Node.js + TypeScript que:
   - Orquestra o ETL Python existente como worker/child process (não reescrever o parser Python
     em JS; reaproveitar o código Python já validado, invocando-o via child_process a partir de jobs).
   - Persiste TODO o estado operacional em MongoDB (jobs de download/extração/import, progresso,
     percentual, logs, última versão de dados importada com sucesso, filas pendentes), de forma que
     um refresh do navegador ou um restart do processo Node NUNCA reinicie um job já em andamento —
     o job deve ser retomável/consultável a partir do estado salvo no Mongo.
   - Usa Knex.js como camada de acesso ao banco relacional de dados de CNPJ, de forma agnóstica de
     client (suportando tanto `mysql2` quanto `pg`), controlado por DB_CLIENT no .env. Eu tenho hoje
     MariaDB local, mas dado o volume (mais de 70 milhões de linhas de CNPJ, mais de 17GB de dados
     descompactados), quero uma recomendação técnica objetiva no ROADMAP sobre migrar para PostgreSQL
     (COPY, índices BRIN, paralelismo de query) versus manter MySQL/MariaDB, incluindo esforço de
     migração estimado. Implemente a camada de forma que essa troca seja apenas configuração +
     migrations, não reescrita de regra de negócio.
   - Expõe endpoints REST de CONSULTA aos dados de CNPJ (empresa, estabelecimento, sócios, simples,
     cnae, natureza jurídica, motivo de situação cadastral, país, município), com paginação, filtros
     (por CNPJ, UF, município, CNAE, situação cadastral, data) e autenticação, para que OUTRAS
     aplicações minhas consumam esses dados. Deve retornar em json.
   - Expõe endpoints administrativos/operacionais: iniciar checagem manual de nova versão, disparar
     import manual, ver progresso de job em andamento, listar histórico de jobs, e um endpoint para
     "limpar pasta de downloads" (ação explícita do usuário — o sistema NUNCA apaga arquivos baixados
     automaticamente após o import, apenas quando o usuário aciona essa limpeza manual pela UI).
   - Implementa um cron mensal (usar a variável ETL_CRON do .env, hoje "0 3 5 * *" = todo dia 5 às
     03h) que verifica no site da Receita Federal se existe uma versão de dados mais nova que a
     última versão registrada como importada com sucesso no MongoDB (ex.: última = "2026-07",
     encontrou "2026-08" disponível no Nextcloud/WebDAV da RFB). Se houver versão nova: dispara o
     job de download+extração+import automaticamente E envia notificação via WhatsApp (usando
     WHATS_HOST/WHATS_TOKEN) para o número em WHATSAPP_DEV, avisando que uma nova versão foi
     encontrada/está sendo processada. Se o job falhar, também notificar.
   - Implementa camada de usuários e controle de acesso (RBAC) cobrindo 100% das operações da UI —
     ou seja, toda ação exposta na interface precisa estar protegida por permissão, não só as rotas
     de dados. Autenticação via JWT (access token curto + refresh token, respeitando
     JWT_ACCESS_TTL/JWT_REFRESH_TTL do .env), usuário admin inicial criado a partir de
     AUTH_ADMIN_EMAIL/AUTH_ADMIN_PASSWORD do .env.
   - Toda variável presente no .env.example do projeto deve, na inicialização da aplicação, ser
     conferida contra o .env real; se uma chave existir no .env.example e não existir no .env, ela
     deve ser adicionada automaticamente ao .env com o valor padrão do .env.example (nunca sobrescrever
     valor já definido pelo usuário). Documente esse comportamento no ROADMAP.

2. Um frontend em Vue 3 (Vite + Pinia) que:
   - Tem tela de login/autenticação.
   - Tem um dashboard mostrando: versão de dados atualmente importada, status do último job,
     progresso em tempo real de um job em andamento (polling ou WebSocket/SSE — decida e justifique
     no roadmap), histórico de execuções.
   - Permite disparar manualmente: checagem de nova versão, download, extração, import.
   - Tem tela de consulta aos dados de CNPJ (busca por CNPJ, razão social, UF, CNAE etc.), paginada. A barra de pesquisa só deve ser habilitada se o usuário estiver logado e tiver recebido ao menos 3 dígitos para pesquisar.
   - Tem tela de administração de usuários e permissões (RBAC) — CRUD de usuários, atribuição de
     papéis/permissões, cobrindo toda ação disponível no sistema.
   - Tem o botão "Limpar pasta de arquivos baixados", com confirmação explícita, que chama o endpoint
     administrativo correspondente.
   - Tem tela de configuração/visualização das variáveis de ambiente relevantes (sem expor segredos
     em texto puro na tela).

3. Infraestrutura:
   - Dockerfile dedicado para a API (Node).
   - Dockerfile dedicado para o frontend (Vue, build estático servido por nginx ou similar).
   - docker-compose para desenvolvimento local no WSL Ubuntu, incluindo MongoDB e o banco relacional
     escolhido, replicando o mais próximo possível o que rodará depois no EasyPanel.
   - Documentação de deploy no EasyPanel (VPS Ubuntu), incluindo variáveis de ambiente necessárias,
     volumes persistentes (MongoDB, banco relacional, pasta de downloads do ETL) e ordem de deploy,
     considerando que o fluxo é: homologar localmente no WSL Ubuntu → só depois subir no VPS via
     EasyPanel.
   - Utilizar npm para gerenciar as dependências do projeto e também para executar o projeto.	

## O que eu quero que você entregue NESTA rodada (planejamento, não código):

1. Um arquivo `ROADMAP.md` na raiz do projeto, com:
   - Visão geral da arquitetura final (diagrama em texto/mermaid está ótimo).
   - Decisão recomendada MySQL/MariaDB vs PostgreSQL, com justificativa técnica e trade-offs,
     considerando volume de ~70M+ linhas e 17GB+.
   - Estrutura de pastas proposta para o monorepo (api/, frontend/, etl-python existente mantido
     onde está, docker/, docs/).
   - Modelagem do estado operacional no MongoDB (coleções: jobs, execuções, versões importadas,
     notificações enviadas, usuários — se não usar Mongo para usuários, justificar por que usar o
     banco relacional para isso).
   - Modelagem de RBAC (papéis, permissões, matriz de quais operações cada papel pode executar).
   - Lista das fases de implementação, numeradas (ex.: Fase 0 — setup do monorepo e docker-compose
     local; Fase 1 — camada Mongo de estado operacional; Fase 2 — worker que invoca o Python
     existente e reporta progresso; Fase 3 — cron + verificação de nova versão + WhatsApp; Fase 4 —
     endpoints de consulta pública/externa; Fase 5 — auth/RBAC; Fase 6 — frontend Vue; Fase 7 —
     Dockerfiles + docker-compose; Fase 8 — homologação local WSL; Fase 9 — deploy EasyPanel).
   - Para CADA fase, critérios de aceite claros e testáveis.

2. Para CADA fase listada no ROADMAP.md, crie um arquivo separado `docs/prompts/FASE_<n>_<slug>.md`
   contendo o PROMPT EXATO que eu deverei colar de volta no seu chat quando for a hora de implementar
   aquela fase especificamente — cada prompt deve ser autocontido (contexto da fase, arquivos
   esperados, contrato de API/dados esperado, critérios de aceite, e o que NÃO fazer ainda). Utilize .cursor/skills/criar-x-prompt/SKILL.md como base para criar o prompt.

3. NÃO escreva código de implementação nesta rodada. Apenas ROADMAP.md e os prompts de cada fase.
   Ao final, me pergunte se pode iniciar pela Fase 0.

4. Sempre que possível, crie waves de prompts e execute simultâneamente os prompts para acelerar o processo de implementação.

Considere as seguintes variáveis já definidas no meu .env atual como fonte de verdade sobre nomes
de variáveis a preservar (não invente nomes novos sem necessidade, reaproveite os existentes e só
proponha novas variáveis quando realmente necessário, deixando claro quais são novas):

NODE_ENV, PORT, LOG_LEVEL, AUTH_SECRET, AUTH_REFRESH_SECRET, JWT_ACCESS_TTL, JWT_REFRESH_TTL,
AUTH_ADMIN_EMAIL, AUTH_ADMIN_PASSWORD, DB_CLIENT, DB_HOST, DB_PORT, DB_USER, DB_PASSWORD,
DB_DATABASE, DB_PREFIX, DB_CHARSET, DB_COLLATION, MONGODB_URI, MONGODB_DATABASE, BASE_API_URL,
BASE_DASHBOARD_URL, SMTP_HOST, SMTP_PORT, SMTP_SECURE, SMTP_USER, SMTP_PASS, EMAIL_FROM,
LANDING_CONTACT_TO, WHATS_HOST, WHATS_TOKEN, WHATSAPP_DEV, RFB_BASE_URL, RFB_SHARE_TOKEN,
RFB_WEBDAV_URL, TEMP_FOLDER, DATA_DIR, ETL_BATCH_SIZE, ENABLE_JOB_WORKER, JOB_POLL_MS, ETL_CRON.
```