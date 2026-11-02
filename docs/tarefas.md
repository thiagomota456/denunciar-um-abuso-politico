# Tarefas

Backlog em ordem. Faça uma tarefa por vez, de cima para baixo: cada uma depende das anteriores. Ao terminar, troque `[ ]` por `[x]`.

Cada tarefa diz o que **ler** antes, o que **entregar** e como saber que está **aceita**. Toda tarefa também precisa cumprir o "Pronto significa" do [AGENTS.md](../AGENTS.md).

---

## Fase 0 — Fundação

### [ ] T0.1 Esqueleto do repositório

- **Ler:** [infraestrutura.md](infraestrutura.md) (seções 4 a 8), [convencoes.md](convencoes.md).
- **Entregar:**
  - `src/backend/Denuncias.sln` com os projetos `Api`, `Comum`, `Modulos/Identidade`, `Modulos/Casos`, `Modulos/Assistencia`, `Modulos/Plataforma` e os três projetos de teste.
  - `src/frontend` com Angular (standalone, roteamento, as quatro áreas lazy vazias e `styles/tokens.css`).
  - Dockerfiles com estágios `dev` e `final`; `compose.yaml`, `compose.dev.yaml`, `compose.prod.yaml`; `deploy/Caddyfile`; `.env.example`; `Makefile`.
  - Endpoint anônimo `GET /api/v1/saude` que responde `{ "status": "ok" }` e checa a conexão com o banco.
  - GitHub Actions rodando `dotnet test` e os testes do Angular em todo push.
- **Aceite:**
  - `cp .env.example .env && make dev` sobe tudo numa máquina só com Docker.
  - `http://localhost:4200` abre a página inicial e `http://localhost:8080/api/v1/saude` responde 200.
  - `make test` roda pelo menos um teste em cada lado e passa. CI verde.

### [ ] T0.2 Camada comum de segurança

- **Ler:** regras 1 a 7 do [AGENTS.md](../AGENTS.md), [permissões](arquitetura/05-permissoes.md), [convencoes.md](convencoes.md) (Banco, API, Autenticação).
- **Entregar:**
  - `FallbackPolicy` exigindo usuário autenticado.
  - Atributo `[Cifrado]` + value converter do EF Core com Data Protection; chaves persistidas em `DataProtection__KeysPath`.
  - Serviço de hash HMAC para CPF.
  - `IAuditoria` + tabela `registro_auditoria` só com INSERT (o usuário de banco da aplicação sem UPDATE/DELETE nela).
  - Tratamento de erros em `ProblemDetails`, sem detalhes internos em produção.
  - Logs estruturados com filtro que impede gravar campos `[Cifrado]`, CPF, e-mail e telefone.
  - Rate limiter nomeado para login, cadastro e upload.
  - Teste de arquitetura: um módulo não referencia o interior de outro.
- **Aceite:**
  - Teste prova que um campo `[Cifrado]` gravado não aparece em texto puro numa consulta SQL direta.
  - Teste prova que uma rota sem `[AllowAnonymous]` responde 401 para anônimo.
  - Teste prova que a aplicação não consegue fazer UPDATE nem DELETE em `registro_auditoria`.

### [ ] T0.3 Seed de desenvolvimento

- **Ler:** regra 10 do [AGENTS.md](../AGENTS.md), [glossario.md](glossario.md).
- **Entregar:** comando `make seed` que cria um usuário por papel (denunciante, profissional verificado, profissional não verificado, moderador, admin) com senha conhecida **só em Development**, e algumas denúncias em cada status.
- **Aceite:** o seed não roda com `ASPNETCORE_ENVIRONMENT=Production`. Textos neutros, sem partidos, políticos ou posições reais.

---

## Fase 1 — MVP

### [ ] T1.1 Cadastro e login

- **Ler:** [requisitos.md](requisitos.md) §2, [modelo de dados](arquitetura/04-modelo-de-dados.md) (`usuario`, `perfil_denunciante`), [convencoes.md](convencoes.md) (Autenticação).
- **Entregar:** cadastro de denunciante (nome, CPF, e-mail, telefone, município e UF), validação dos dígitos do CPF, CPF único via `cpf_hash`, confirmação de e-mail, login e logout com cookie, recuperação de senha, MFA por app autenticador (opcional para denunciante). Telas no Angular.
- **Aceite:** CPF inválido ou repetido é recusado com mensagem clara. Login bloqueado temporariamente após tentativas seguidas. CPF e telefone gravados cifrados.

### [ ] T1.2 Cadastro de profissional

- **Ler:** [requisitos.md](requisitos.md) §4, modelo de dados (`profissional`, `escritorio`).
- **Entregar:** cadastro com número e UF da OAB, áreas, abrangência, disponibilidade, aceita gratuito, site e redes; escritório opcional com CNPJ. Status inicial: não verificado. MFA obrigatório.
- **Aceite:** profissional não verificado não aparece em busca nenhuma e recebe 404 em qualquer dado privado (teste).

### [ ] T1.3 Verificação de profissionais pelo admin

- **Ler:** [decisoes.md](decisoes.md) D9.
- **Entregar:** fila de profissionais pendentes no painel admin; aprovar (com anexo do comprovante da consulta ao CNA) ou recusar com motivo; e-mail ao profissional.
- **Aceite:** só admin acessa. Aprovação grava `verificado_em`, `verificado_por` e auditoria.

### [ ] T1.4 Registro de denúncia em etapas

- **Ler:** [requisitos.md](requisitos.md) §3 e §10, modelo de dados (`denuncia`, `denuncia_privada`, `capacidade_financeira`).
- **Entregar:** formulário em etapas: categoria → parte pública → parte privada → capacidade financeira → anonimato → revisão. Rascunho salvo a cada etapa. Envio muda o status para `em_revisao`. Denunciante lista e edita os próprios casos.
- **Aceite:** funciona em 360 px e por teclado. Linguagem simples, com exemplo em cada campo. Campos privados gravados cifrados. Outro usuário recebe 404.

### [ ] T1.5 Upload de provas

- **Ler:** regra 5 do [AGENTS.md](../AGENTS.md), [decisoes.md](decisoes.md) D6 e D8, modelo de dados (`prova`).
- **Entregar:** upload com limite de tamanho e tipos permitidos (PDF, imagens, áudio, vídeo, verificados pelo conteúdo e não só pela extensão), SHA-256, gravação cifrada em disco atrás de `IArmazenamentoProvas`, job do Hangfire que passa no ClamAV, listagem e remoção pelo dono.
- **Aceite:** arquivo infectado fica `bloqueado` e nunca pode ser baixado. Não existe URL estática para nenhuma prova.

### [ ] T1.6 Moderação

- **Ler:** regra 8 do [AGENTS.md](../AGENTS.md), [fluxo do caso](arquitetura/02-fluxo-do-caso.md) passos 2 e 3.
- **Entregar:** fila de denúncias `em_revisao`; o moderador lê o relato para classificar, edita o resumo público, define categoria, área e urgência, e publica ou devolve com motivo.
- **Aceite:** só a moderação muda o status para `publicada`. Toda decisão grava `moderacao` e auditoria. O moderador não vê provas nem contato (teste).

### [ ] T1.7 Consulta pública

- **Ler:** [requisitos.md](requisitos.md) §12 (módulo público).
- **Entregar:** lista paginada e busca de denúncias publicadas, filtros por categoria, UF e área, página de detalhe. Denúncia anônima mostra "Denunciante anônimo".
- **Aceite:** a resposta da API não contém nenhum campo além dos do resumo público (teste que compara as chaves do JSON). Página leve: sem imagens pesadas, primeira carga rápida em 3G.

### [ ] T1.8 Painel de busca e manifestação de interesse

- **Ler:** [requisitos.md](requisitos.md) §5 (painel de busca), modelo de dados (`interesse`).
- **Entregar:** busca para profissional verificado com os filtros públicos + área jurídica; botão para manifestar interesse com mensagem; lista "meus interesses".
- **Aceite:** um interesse por profissional por caso. Manifestar interesse não libera nada privado (teste).

### [ ] T1.9 Autorização pelo denunciante

- **Ler:** [fluxo do caso](arquitetura/02-fluxo-do-caso.md) passo 6, [permissões](arquitetura/05-permissoes.md), modelo de dados (`autorizacao`).
- **Entregar:** o denunciante vê os interesses no seu caso com o perfil público de cada profissional; autoriza escolhendo o escopo (relato, provas, testemunhas, contato) ou recusa; pode revogar a qualquer momento.
- **Aceite:** revogação corta o acesso na próxima requisição (teste). Autorizar, recusar e revogar gravam auditoria.

### [ ] T1.10 Acesso do profissional autorizado e atendimento

- **Ler:** regras 2, 5 e 6 do [AGENTS.md](../AGENTS.md), modelo de dados (`atendimento`).
- **Entregar:** tela do caso com só os dados do escopo; download de prova por link de 5 minutos; abertura de atendimento (`avaliando` → `formalizado` ou `encerrado`).
- **Aceite:** campo fora do escopo nunca vem na resposta. Link expirado responde 404. Cada leitura e download vai para a auditoria.

### [ ] T1.11 Painel administrativo básico

- **Ler:** [requisitos.md](requisitos.md) §12 (módulo administrativo).
- **Entregar:** lista de usuários com bloqueio e desbloqueio; consulta da auditoria com filtros; botão "reportar conteúdo abusivo" nas páginas públicas e fila desses reportes para a moderação.
- **Aceite:** acesso do admin a identidade ou contato exige um motivo, que fica gravado na auditoria.

### [ ] T1.12 Direitos do titular (LGPD)

- **Ler:** [requisitos.md](requisitos.md) §9.
- **Entregar:** "baixar meus dados" (JSON com tudo do titular); pedido de exclusão de conta que anonimiza o que pode ser apagado e mantém o que a lei obriga a guardar; página de política de privacidade com `[PREENCHER: texto jurídico]`; o denunciante vê quem acessou seus dados.
- **Aceite:** depois da exclusão, nenhum dado pessoal do titular é legível; a auditoria continua íntegra, com o ator anonimizado.

### [ ] T1.13 Testes de ponta a ponta e matriz de permissões completa

- **Ler:** [permissões](arquitetura/05-permissoes.md), [convencoes.md](convencoes.md) (Testes).
- **Entregar:** teste parametrizado com todas as células da tabela de permissões; Playwright no fluxo principal: cadastrar → denunciar → moderar → publicar → manifestar interesse → autorizar → baixar prova → revogar.
- **Aceite:** todos passam no CI. Quebrar qualquer policy faz pelo menos um teste falhar.

### [ ] T1.14 Deploy

- **Ler:** [infraestrutura.md](infraestrutura.md) seções 5 e 9.
- **Entregar:** imagem de produção do Caddy com o build do Angular; `deploy/backup.sh` (pg_dump cifrado + volumes `provas` e `chaves`, guardando 30 dias); `deploy/restaurar.sh`; passo a passo de servidor novo em `docs/deploy.md`.
- **Aceite:** um servidor limpo fica no ar seguindo o passo a passo. Uma restauração de backup num banco vazio foi testada e documentada.

---

## Fase 2 — Operação e expansão

- [ ] **T2.1 Distribuição de notificações em rodízio**, conforme [distribuição](arquitetura/03-distribuicao-notificacoes.md). Job do Hangfire; N e X configuráveis no `.env`; teste que simula 100 casos e mostra a distribuição equilibrada.
- [ ] **T2.2 Painel privado do caso** com movimentações entre denunciante e profissional.
- [ ] **T2.3 Avaliações** após o encerramento do atendimento: uma por atendimento, moderadas, sem nota por vitória em processo.
- [ ] **T2.4 Busca avançada**: texto livre no resumo público (busca de texto do PostgreSQL), ordenação por urgência e data.
- [ ] **T2.5 Classificação sugerida automaticamente** a partir da categoria e da parte pública; a moderação confirma.
- [ ] **T2.6 Integrações externas**: login gov.br e verificação da OAB, se houver adesão aprovada ou API oficial.

## Fase 3 — Sustentabilidade e impacto

- [ ] **T3.1 Doações** por PIX e cartão via gateway, com webhook idempotente.
- [ ] **T3.2 Lançamentos financeiros** (receitas, despesas, fundo) pelo admin, com comprovante.
- [ ] **T3.3 Página pública de transparência** com receitas, despesas e destinação, sem dados de beneficiários.
- [ ] **T3.4 Fundo de apoio**: pedido pelo denunciante, decisão por critérios públicos, registro em `apoio_fundo`.
- [ ] **T3.5 Publicidade** sem rastrear o usuário, nunca nas páginas de denúncia privada.
- [ ] **T3.6 Indicadores** operacionais e de impacto para o admin.

---

## Achados

Coisas encontradas durante as tarefas que ficaram fora do escopo. Adicione aqui em vez de fazer na hora.

- (vazio)
