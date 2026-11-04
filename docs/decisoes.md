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
| D21 | Módulo `Encaminhamento` (schema `encaminhamento`), criado na Fase 1B. Substitui D18 na contagem: 4 módulos a partir dessa fase | Catálogo, recomendação, dossiê e acompanhamento formam uma fronteira própria e não dependem do fluxo com advogados | Colocar tudo em Assistencia |
| D22 | Recomendação de destinatários por regras de competência guardadas como dados (CSV + tabela), consultadas em SQL | Explicável ("por que este órgão"), auditável, sem custo; corrigir uma regra não exige deploy | IA classificando o destinatário |
| D23 | Envio automático só por e-mail. Formulários oficiais usam o modo assistido (link + campos prontos para copiar) | Os formulários exigem login gov.br da pessoa ou têm captcha, e a API do Fala.BR é só para órgãos; robô quebra a cada mudança de tela e não se deve contornar captcha | Robô de formulário com Playwright |
| D24 | Dossiê gerado por modelo a partir dos campos do caso, com o relato literal. Sem IA na primeira versão | Não inventar fatos; opinião política é dado sensível (LGPD art. 11) e mandar o relato a um provedor de IA seria um compartilhamento a mais; custo zero. Reabrir só com revisão obrigatória do texto pelo denunciante e consentimento específico | Reescrita do relato por IA |
| D25 | Dossiê em HTML com CSS de impressão; PDF pelo navegador; e-mail em texto puro | Nenhuma biblioteca de PDF para manter | QuestPDF ou outra biblioteca de PDF |
| D26 | Catálogo inicial versionado em `dados/catalogo/` (CSV com fonte e data de conferência por linha) e sindicatos importados da planilha oficial do CNES | Fontes oficiais, revisão por pull request, histórico no git | Cadastro só pela tela ou raspagem de sites |
| D27 | MailKit para SMTP e IMAP; resposta por endereço único por encaminhamento (`resposta+<token>@dominio`) | Captura as respostas sem serviço pago; o `SmtpClient` do .NET não é recomendado para código novo e não lê IMAP | Respostas registradas só à mão |
| D28 | Envio automático exige caso revisado pela moderação (status `publicada` ou `aprovada`); o modo assistido não exige | Protege as entidades de spam e a plataforma de uso abusivo; no modo assistido quem envia é o próprio denunciante | Envio sem revisão |
