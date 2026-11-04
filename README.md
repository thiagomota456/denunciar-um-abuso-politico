# denunciar-um-abuso-politico

Plataforma web, apartidária e sem fins lucrativos, para cidadãos denunciarem abusos políticos e violações de direitos, guardarem provas com segurança, encaminharem o caso aos órgãos e entidades competentes e encontrarem advogados verificados dispostos a ajudar.

## Stack

- **Backend:** C# / ASP.NET Core (.NET 10 LTS) + EF Core, monólito modular
- **Frontend:** Angular
- **Banco:** PostgreSQL
- **Hospedagem mínima:** 1 VPS com Docker Compose (Caddy + API + PostgreSQL + ClamAV)

## Para quem vai construir (pessoas ou IA)

Comece pelo [AGENTS.md](AGENTS.md): ordem de leitura, regras de segurança e como pegar a próxima tarefa em [docs/tarefas.md](docs/tarefas.md).

- [Requisitos](docs/requisitos.md) · [Convenções](docs/convencoes.md) · [Decisões](docs/decisoes.md) · [Glossário](docs/glossario.md) · [Perguntas frequentes](docs/faq.md)

Sugestões, reclamações e pedidos para incluir uma entidade, profissão ou tipo de abuso: veja [como fazer](docs/faq.md#sugestões-reclamações-e-pedidos). No GitHub, use os modelos de issue e nunca publique dados pessoais ou detalhes de um caso.

## Arquitetura

Diagramas em Mermaid (o GitHub renderiza direto):

1. [Visão geral: módulos e integrações](docs/arquitetura/01-visao-geral.md)
2. [Fluxo do caso entre os atores](docs/arquitetura/02-fluxo-do-caso.md)
3. [Distribuição de notificações (rodízio)](docs/arquitetura/03-distribuicao-notificacoes.md)
4. [Modelo de dados](docs/arquitetura/04-modelo-de-dados.md)
5. [Privacidade e permissões](docs/arquitetura/05-permissoes.md)
6. [Encaminhamento a órgãos e entidades](docs/arquitetura/06-encaminhamento.md)

Catálogo inicial de entidades, canais e regras de competência: [dados/catalogo/](dados/catalogo/).

## Infraestrutura e desenvolvimento

Custos, escolha de VPS e domínio, Docker, variáveis de ambiente, testes e como rodar localmente: [docs/infraestrutura.md](docs/infraestrutura.md).
