# denunciar-um-abuso-politico

Plataforma web, apartidária e sem fins lucrativos, para cidadãos denunciarem abusos políticos e violações de direitos, guardarem provas com segurança e encontrarem advogados verificados dispostos a ajudar.

## Stack

- **Backend:** C# / ASP.NET Core 8 + EF Core, monólito modular
- **Frontend:** Angular
- **Banco:** PostgreSQL
- **Hospedagem mínima:** 1 VPS com Docker Compose (Caddy + API + PostgreSQL + ClamAV)

## Arquitetura

Diagramas em Mermaid (o GitHub renderiza direto):

1. [Visão geral: módulos e integrações](docs/arquitetura/01-visao-geral.md)
2. [Fluxo do caso entre os atores](docs/arquitetura/02-fluxo-do-caso.md)
3. [Distribuição de notificações (rodízio)](docs/arquitetura/03-distribuicao-notificacoes.md)
4. [Modelo de dados](docs/arquitetura/04-modelo-de-dados.md)
5. [Privacidade e permissões](docs/arquitetura/05-permissoes.md)

## Infraestrutura e desenvolvimento

Custos, escolha de VPS e domínio, Docker, variáveis de ambiente, testes e como rodar localmente: [docs/infraestrutura.md](docs/infraestrutura.md).
