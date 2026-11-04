# AGENTS.md

Instruções para agentes de IA (e pessoas) que vão construir este projeto. Leia este arquivo inteiro antes de escrever código.

## O que é

Plataforma web, apartidária e sem fins lucrativos, para cidadãos denunciarem abusos políticos e violações de direitos, guardarem provas com segurança, encaminharem o caso aos órgãos e entidades competentes e encontrarem advogados verificados. Os dados do denunciante são privados por padrão e só são compartilhados com um profissional quando o próprio denunciante autoriza.

## Ordem de leitura

1. [docs/requisitos.md](docs/requisitos.md): o que construir (briefing original e o adendo de encaminhamento, §15).
2. [docs/arquitetura/](docs/arquitetura/): como o sistema funciona, em diagramas Mermaid.
3. [docs/convencoes.md](docs/convencoes.md): nomes, estrutura de pastas, padrões de código.
4. [docs/decisoes.md](docs/decisoes.md): decisões já tomadas e o motivo. Não reabra sem motivo novo.
5. [docs/tarefas.md](docs/tarefas.md): backlog em ordem. **Trabalhe uma tarefa por vez, na ordem.**
6. [docs/infraestrutura.md](docs/infraestrutura.md): Docker, `.env`, testes, deploy.
7. [docs/glossario.md](docs/glossario.md): termos do domínio.
8. [docs/faq.md](docs/faq.md): texto da página de perguntas frequentes. Ele descreve o comportamento prometido ao usuário: o código precisa bater com ele.

Se dois documentos se contradizem, a ordem de prioridade é: `requisitos.md` > `decisoes.md` > `arquitetura/` > o resto. Aponte a contradição em vez de escolher em silêncio.

## Stack (fixa)

| Camada | Tecnologia |
|---|---|
| Backend | C# / ASP.NET Core (.NET 10 LTS), EF Core, monólito modular |
| Frontend | Angular (a versão estável no momento da T0.1, registrada em `docs/decisoes.md`), standalone components, signals |
| Banco | PostgreSQL 17 |
| Jobs | Hangfire com storage no PostgreSQL |
| Testes | xUnit, Testcontainers, `WebApplicationFactory`, Playwright |
| Infra | Docker Compose, Caddy, ClamAV, 1 VPS |

Não adicione Redis, RabbitMQ, Kafka, Elasticsearch, Kubernetes, microsserviços, MediatR, AutoMapper nem outro banco. Se achar que uma dependência nova é necessária, registre a justificativa em `docs/decisoes.md` antes.

## Regras que nunca podem ser quebradas

Estas regras protegem pessoas reais que podem sofrer retaliação. Elas valem mais que qualquer prazo.

1. **Autorização sempre no backend.** Toda rota exige autenticação por padrão (`FallbackPolicy`). Rotas públicas são marcadas com `[AllowAnonymous]` explicitamente. Esconder um botão no Angular não é controle de acesso.
2. **Acesso por caso, não só por papel.** Um profissional só lê dados privados de uma denúncia se existir uma `autorizacao` ativa (não revogada) para ele, e só os campos dentro do `escopo`. Siga a tabela de [permissões](docs/arquitetura/05-permissoes.md) célula por célula.
3. **Profissional não verificado nunca acessa nada privado.** Nem com autorização.
4. **Campos sensíveis são cifrados na aplicação** antes de ir para o banco. A lista está no [modelo de dados](docs/arquitetura/04-modelo-de-dados.md) (campos marcados com "cifrado"). Campo cifrado não entra em `WHERE` nem `ORDER BY`.
5. **Provas nunca são servidas como arquivo estático.** Só pelo endpoint da API que checa a autorização, devolve o arquivo e grava o acesso na auditoria.
6. **Toda leitura de dado privado vai para `registro_auditoria`.** Toda autorização, revogação, edição e mudança de status também. A tabela é só INSERT.
7. **Nunca registre dados pessoais em logs**, mensagens de erro, URLs ou query strings. Use IDs.
8. **O resumo público só aparece depois da moderação.** Só o caso de uso de publicação, restrito ao papel moderador, muda o status para `publicada`.
9. **Sem captação de clientela.** Nenhum recurso que permita a um profissional pagar por prioridade, destaque ou acesso a casos.
10. **Neutralidade política.** Seeds, textos de exemplo, testes e interface não podem citar partidos, políticos ou posições políticas reais.
11. **Nada sai para terceiros sem autorização explícita por destinatário.** O denunciante vê o dossiê exatamente como será enviado, escolhe a identificação permitida pelo canal e confirma com a senha. O que foi enviado fica guardado cifrado com o SHA-256.
12. **O dossiê não inventa nada.** Usa só os campos e o relato do denunciante, com o relato literal. Se faltar algo que o canal exige, pergunte ao denunciante.
13. **Sem robô em sistema de terceiros.** Não automatize login gov.br, não contorne captcha e não envie a entidade que a recomendação não indicou para o caso. Formulário oficial que não aceita integração usa o modo assistido.

## Como trabalhar

- Pegue a primeira tarefa não concluída de [docs/tarefas.md](docs/tarefas.md). Leia os documentos que ela cita.
- Faça só o que a tarefa pede. Se encontrar outra coisa a fazer, anote no fim de `docs/tarefas.md` em "Achados".
- Escreva os testes da tarefa junto com o código, não depois.
- Ao terminar, marque a tarefa com `[x]` e atualize os documentos de arquitetura se o código divergiu deles.
- Mudou o modelo de dados? Crie uma migration do EF Core e atualize `docs/arquitetura/04-modelo-de-dados.md` no mesmo commit.

## Comandos

Disponíveis depois da tarefa T0.1 (ver `Makefile`). `make dev` só precisa de Docker; `make test` também precisa do SDK do .NET e do Node instalados.

```bash
make dev        # sobe tudo com hot reload
make test       # testes do backend e do frontend
make seed       # usuários de exemplo, um por papel
make migrate    # aplica migrations
make reset      # apaga o banco local
```

## Pronto significa

- [ ] Compila sem warnings novos.
- [ ] `make test` passa.
- [ ] Endpoints novos têm teste de autorização: o papel certo recebe 200, os outros recebem 403 ou 404.
- [ ] Nenhum dado pessoal em log.
- [ ] Textos da interface em português do Brasil, claros para quem não conhece Direito.
- [ ] Tela nova funciona em 360 px de largura e por teclado.
- [ ] Documentação atualizada se o comportamento mudou.

## Idioma

- Documentação, mensagens de commit e textos da interface: português do Brasil.
- Código: nomes do domínio em português (`Denuncia`, `Prova`, `Autorizacao`), sem acento. Termos técnicos do framework ficam como são (`Controller`, `DbContext`, `Service`). Detalhes em [docs/convencoes.md](docs/convencoes.md).
