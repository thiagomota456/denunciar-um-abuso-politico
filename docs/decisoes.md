# Decisões de arquitetura

Registro curto das decisões já tomadas. Para mudar uma, adicione uma nova entrada que a substitua, com o motivo.

| # | Decisão | Motivo | Alternativa descartada |
|---|---|---|---|
| D1 | Monólito modular, um único deploy | Briefing pede simplicidade e baixo custo; separar depois continua possível porque os módulos só se falam por `Contratos/` | Microsserviços |
| D2 | C# / ASP.NET Core (.NET 10 LTS), Angular, PostgreSQL 17 | Escolha do mantenedor; .NET 10 é a versão LTS atual | — |
| D3 | Um schema e um `DbContext` por módulo | Mantém as fronteiras dos módulos visíveis no banco | Um `DbContext` único |
| D4 | Autenticação por cookie com ASP.NET Core Identity | Frontend e API no mesmo domínio; cookie `HttpOnly` é mais seguro e simples que JWT no navegador | JWT em `localStorage` |
| D5 | Campos sensíveis cifrados na aplicação com Data Protection; CPF buscável por HMAC | Quem acessar o banco ou um backup não lê os dados | Criptografia só do disco |
| D6 | Provas cifradas em disco no VPS, entregues pelo próprio endpoint autorizado | Custo zero no MVP e sem links temporários para gerenciar; trocar por armazenamento S3 depois é uma implementação nova de `IArmazenamentoProvas` | Bucket S3 com links assinados |
| D7 | Hangfire com storage no PostgreSQL | Fila, agendamento e painel sem serviço extra | Redis, RabbitMQ |
| D8 | ClamAV no mesmo VPS | Gratuito; provas podem vir com arquivos maliciosos | Serviço de varredura pago |
| D9 | Verificação de OAB manual pelo admin no MVP | Não há API pública oficial do CNA; não se deve contornar o captcha do site | Raspagem do site do CNA |
| D10 | Sem login gov.br e sem validação de CPF na Receita no MVP | Adesão ao gov.br é um processo à parte e as APIs de CPF são pagas; o briefing deixa a validação definitiva com o profissional | Integração desde o início |
| D11 | MFA por app autenticador (TOTP), sem SMS | Custo zero e mais seguro que SMS | SMS |
| D12 | Minimal APIs, sem MediatR e sem AutoMapper | Menos camadas e menos dependências | Controllers + MediatR |
| D13 | Angular sem biblioteca de componentes | App leve em conexão lenta; o design é simples | Angular Material |
| D14 | Hospedagem: 1 VPS Hostinger KVM 2 em São Paulo, Docker Compose, Caddy | Dados no Brasil (LGPD), baixa latência, 8 GB para o ClamAV; ver [infraestrutura](infraestrutura.md) | Hetzner CX23, nuvem gerenciada |
| D15 | Distribuição de notificações por rodízio, sem pagamento por prioridade | Ética da OAB proíbe captação de clientela; ver [distribuição](arquitetura/03-distribuicao-notificacoes.md) | Leilão ou plano pago para advogados |
| D16 | Nomes do domínio em português no código e no banco | Os documentos e o negócio estão em português; evita tradução ambígua (ex.: "denúncia") | Inglês em todo o código |
| D17 | Recurso alheio responde 404 em vez de 403 | Não revela a existência de denúncias ou provas | 403 |
| D18 | 3 módulos no MVP (Identidade, Casos, Assistencia); Plataforma só na Fase 3 | As 12 áreas funcionais cabem em poucos projetos; menos projetos, menos fronteiras para manter | Um projeto por área funcional |
| D19 | No MVP, classificação manual pela moderação e casos encontrados pela busca | Rodízio e classificação automática só valem com volume; ficam para a Fase 2 | Notificações desde o MVP |
| D20 | Auditoria imutável por trigger no banco | Um usuário de banco só, sem gerenciar permissões separadas | Usuário de banco sem UPDATE/DELETE |
