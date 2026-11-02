# Infraestrutura, custos e ambiente de desenvolvimento

Objetivo: o mais barato e simples possível no MVP, sem fechar portas para crescer.

> Preços pesquisados em novembro de 2026. Mudam com frequência (a Hetzner reajustou duas vezes em 2026 e a Hostinger cobra bem mais na renovação). Confira nas páginas oficiais antes de contratar.

## 1. Estimativa de custos mensais (MVP)

| Item | Opção barata | Custo/mês |
|---|---|---|
| VPS | Hostinger KVM 2 (São Paulo, 8 GB RAM) **ou** Hetzner CX23 (Alemanha/Finlândia, 4 GB RAM) | R$ 44 a R$ 78 (Hostinger, promoção → renovação) · ≈ €4 (Hetzner) |
| Domínio `.org.br` ou `.com.br` | Registro.br, R$ 40/ano | ≈ R$ 3,35 |
| DNS, proteção contra DDoS e cache | Cloudflare, plano gratuito | R$ 0 |
| HTTPS | Caddy + Let's Encrypt | R$ 0 |
| Antivírus de arquivos | ClamAV no próprio VPS | R$ 0 |
| E-mail transacional | Plano gratuito de um provedor SMTP (algumas centenas de e-mails/dia) | R$ 0 no início |
| MFA | App autenticador (TOTP), sem SMS | R$ 0 |
| Backup externo | Bucket compatível com S3 ou storage box, poucos GB | ≈ R$ 5 a R$ 20 |
| CI | GitHub Actions (gratuito em repositório público) | R$ 0 |
| **Total** | | **≈ R$ 35 a R$ 100/mês** |

Custos que ficam **de fora** do MVP de propósito:

- **SMS:** cobrado por mensagem. Use e-mail + app autenticador.
- **Validação de CPF na Receita** (ex.: APIs pagas do Serpro): cobrada por consulta. No MVP, valide os dígitos do CPF, confirme e-mail e telefone e deixe a validação definitiva para o profissional, como o briefing já prevê.
- **Login gov.br:** a integração com o Login Único passa por um processo de adesão e pode não estar disponível para uma entidade privada. Trate como Fase 2, se aprovado.
- **Consulta à OAB (CNA):** não há API pública oficial. No MVP, o admin verifica manualmente no site do CNA e anexa o comprovante ao registro do profissional. Não automatize contornando o captcha do site.

## 2. Qual VPS escolher

O Docker Compose vai rodar PostgreSQL + API .NET + Caddy + ClamAV. O ClamAV sozinho consome cerca de 1 a 1,5 GB de RAM para carregar as assinaturas, então **4 GB é o mínimo e 8 GB é o confortável**.

| | Hostinger KVM 2 | Hetzner CX23 |
|---|---|---|
| Local | São Paulo | Alemanha ou Finlândia |
| Latência para usuários no Brasil | Baixa | ≈ 200 ms |
| RAM | 8 GB | 4 GB |
| Preço | Promoção boa em contrato de 24 meses, renovação bem mais cara | Barato e estável, cobrança por hora |
| Dados pessoais sensíveis (LGPD) | Ficam no Brasil | Transferência internacional: possível, mas exige cuidado e declaração na política de privacidade |
| Painel, snapshots, API | Básico | Muito bom |

**Recomendação: Hostinger KVM 2 em São Paulo.** Os dados ficam no Brasil, o que simplifica a LGPD num projeto com dados sensíveis, a latência é baixa e os 8 GB acomodam o ClamAV com folga. Contrate por 24 meses para travar o preço promocional e anote a data de renovação.

Se a prioridade for só o menor preço, a Hetzner CX23 resolve; nesse caso, rode o ClamAV com limite de memória e ative swap de 2 GB.

## 3. Domínio

Verifiquei pelo serviço público de consulta (RDAP) do Registro.br, em 02/11/2026, e estes nomes **não estavam registrados**:

| Domínio | Observação |
|---|---|
| `denuncieabuso.org.br` | curto, imperativo, fácil de falar |
| `denuncieumabuso.org.br` | mais natural na fala |
| `denunciarumabuso.org.br` | igual ao nome do repositório |
| `abusopolitico.org.br` | curto, mas soa partidário: evite |
| `amparojuridico.org.br` | neutro, foca na assistência e não na denúncia |
| `vozedireito.org.br` | neutro, marca mais "institucional" |

As mesmas variações em `.com.br` também estavam livres.

**Recomendação:** `denuncieumabuso.org.br` como principal, mais o `.com.br` do mesmo nome apontando para ele (R$ 80/ano no total).

- **`.org.br` exige o CNPJ de uma entidade sem fins lucrativos.** Enquanto a associação não tiver CNPJ, registre o `.com.br` com o CPF de um responsável e migre depois.
- A consulta mostra só o que estava livre naquele momento. Registre logo no [registro.br](https://registro.br) e ative a renovação automática: o domínio é liberado 60 dias após expirar.

## 4. Organização do repositório

```
/
├── src/
│   ├── backend/
│   │   ├── Denuncias.sln
│   │   ├── Api/                     # host ASP.NET Core: Program.cs, registro dos módulos
│   │   ├── Comum/                   # criptografia de campos, auditoria, autorização
│   │   ├── Modulos/
│   │   │   ├── Identidade/          # cada módulo: Contratos, Dominio, Aplicacao, Infra, Endpoints
│   │   │   ├── Casos/
│   │   │   ├── Assistencia/
│   │   │   └── Plataforma/
│   │   └── tests/
│   │       ├── Unitarios/
│   │       ├── Integracao/          # Testcontainers + WebApplicationFactory
│   │       └── Arquitetura/         # fronteiras entre módulos
│   └── frontend/                    # Angular
├── deploy/
│   ├── Caddyfile
│   └── backup.sh
├── compose.yaml                     # serviços comuns
├── compose.dev.yaml                 # desenvolvimento
├── compose.prod.yaml                # produção
├── .env.example                     # versionado, sem segredos
├── .env                             # NÃO versionado
├── Makefile
└── docs/
```

## 5. Docker

Um `compose.yaml` base e dois arquivos que o complementam. Mesmo conjunto de serviços nos dois ambientes; muda só o que precisa mudar.

| Serviço | Imagem | Dev | Produção |
|---|---|---|---|
| `db` | `postgres:17-alpine` | porta 5432 exposta | sem porta exposta, volume persistente |
| `api` | build de `src/backend` | `dotnet watch` com código montado | imagem compilada, usuário sem privilégios |
| `web` | build de `src/frontend` | `ng serve` com hot reload | não existe: o build do Angular vai para dentro do Caddy |
| `caddy` | `caddy:2-alpine` | opcional | HTTPS automático, serve o Angular, `/api` → `api` |
| `clamav` | `clamav/clamav` | opcional (perfil `antivirus`) | sim, com limite de memória |
| `mailpit` | `axllent/mailpit` | caixa de e-mail falsa em `localhost:8025` | não existe |

`compose.yaml` (base):

```yaml
services:
  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 5s
      retries: 10

  api:
    build: ./src/backend
    env_file: .env
    environment:
      ConnectionStrings__Default: Host=db;Database=${DB_NAME};Username=${DB_USER};Password=${DB_PASSWORD}
    volumes:
      - provas:/data/provas
      - chaves:/data/chaves
    depends_on:
      db:
        condition: service_healthy

volumes:
  pgdata:
  provas:
  chaves:
```

`compose.dev.yaml`:

```yaml
services:
  db:
    ports: ["5432:5432"]

  api:
    build:
      target: dev                       # estágio do Dockerfile com o SDK
    command: dotnet watch --project Api run --urls http://0.0.0.0:8080
    volumes:
      - ./src/backend:/src
    ports: ["8080:8080"]
    environment:
      ASPNETCORE_ENVIRONMENT: Development
      Smtp__Host: mailpit
      Smtp__Port: 1025

  web:
    build:
      context: ./src/frontend
      target: dev
    command: npx ng serve --host 0.0.0.0 --proxy-config proxy.conf.json
    volumes:
      - ./src/frontend:/app
      - /app/node_modules
    ports: ["4200:4200"]

  mailpit:
    image: axllent/mailpit
    ports: ["8025:8025"]

  clamav:
    image: clamav/clamav
    profiles: ["antivirus"]
```

`compose.prod.yaml`:

```yaml
services:
  db:
    restart: unless-stopped

  api:
    build:
      target: final                     # runtime aspnet, sem SDK
    restart: unless-stopped
    environment:
      ASPNETCORE_ENVIRONMENT: Production

  caddy:
    build:
      context: .
      dockerfile: deploy/Caddy.Dockerfile   # compila o Angular e copia para o Caddy
    restart: unless-stopped
    ports: ["80:80", "443:443"]
    volumes:
      - ./deploy/Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
    depends_on: [api]

  clamav:
    image: clamav/clamav
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 1600m

volumes:
  caddy_data:
```

`deploy/Caddyfile`:

```
{$DOMINIO} {
    encode zstd gzip
    handle /api/* {
        reverse_proxy api:8080
    }
    handle {
        root * /srv
        try_files {path} /index.html
        file_server
    }
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
        X-Content-Type-Options nosniff
        Referrer-Policy no-referrer
    }
}
```

## 6. Variáveis de ambiente (`.env`)

- `.env.example` vai para o Git, com todas as chaves e valores de exemplo.
- `.env` fica fora do Git (`.gitignore`). No servidor: `chmod 600 .env`, dono `root`.
- O ASP.NET Core lê variáveis com `__` como seções (`Smtp__Host` → `Smtp:Host`), então não é preciso um `appsettings` por ambiente para segredos.
- O Angular **não** recebe segredos. A URL da API é relativa (`/api`), então a mesma build serve para todos os ambientes.

`.env.example`:

```dotenv
# Geral
DOMINIO=localhost
ASPNETCORE_ENVIRONMENT=Development

# Banco
DB_NAME=denuncias
DB_USER=denuncias
DB_PASSWORD=troque-esta-senha

# Criptografia de campos (Data Protection). Em produção, aponte para um volume com backup separado.
DataProtection__KeysPath=/data/chaves

# Arquivos de provas
Provas__Caminho=/data/provas
Provas__TamanhoMaximoMb=50

# E-mail (em dev, o Mailpit aceita qualquer coisa)
Smtp__Host=mailpit
Smtp__Port=1025
Smtp__Usuario=
Smtp__Senha=
Smtp__Remetente=nao-responda@localhost

# Antivírus
ClamAv__Host=clamav
ClamAv__Port=3310
ClamAv__Habilitado=false

# Hash de CPF para busca (HMAC-SHA256). Nunca troque depois de ter dados: os hashes antigos deixam de bater.
Cifra__ChaveHmac=gere-com-openssl-rand-base64-32

# Primeiro admin, criado pelo seed se não existir
Admin__Email=admin@localhost
Admin__SenhaInicial=troque-no-primeiro-login
```

**As chaves de criptografia são o item mais crítico do backup.** Sem elas, os campos cifrados do banco ficam ilegíveis para sempre. Faça backup do volume `chaves` e do valor de `Cifra__ChaveHmac` separados do backup do banco.

A autenticação usa cookie (ver [decisões](decisoes.md), D4), por isso não há chave de JWT.

## 7. Testes

| Camada | Ferramenta | O que cobre |
|---|---|---|
| Unitários (.NET) | xUnit | regras de domínio: rodízio, transições de status, escopo de autorização |
| Integração (.NET) | xUnit + Testcontainers (PostgreSQL real) + `WebApplicationFactory` | endpoints, EF Core, migrations, policies |
| **Permissões** | teste parametrizado de integração | **uma linha por célula da tabela de [permissões](arquitetura/05-permissoes.md)**: para cada papel × dado, a API devolve 200 ou 403/404 |
| Frontend | testes de unidade do Angular | componentes de formulário e guards de rota |
| Ponta a ponta | Playwright | o fluxo principal: denunciar → moderar → interesse → autorizar → acessar prova |

Regras práticas:

- Teste de integração contra **PostgreSQL de verdade** (Testcontainers), não SQLite nem banco em memória: o comportamento muda.
- O teste de permissões é o mais importante do projeto. Se alguém mudar uma policy, ele quebra.
- CI no GitHub Actions: `dotnet test` + testes do Angular em todo push; Playwright só nos PRs para `main`.

## 8. Rodar em desenvolvimento

Pré-requisitos: Docker e Git. Não é preciso instalar .NET, Node nem PostgreSQL na máquina.

```bash
git clone https://github.com/thiagomota456/denunciar-um-abuso-politico.git
```

```bash
cd denunciar-um-abuso-politico && cp .env.example .env
```

```bash
make dev
```

Depois disso:

| Endereço | O quê |
|---|---|
| http://localhost:4200 | Angular |
| http://localhost:8080/swagger | API |
| http://localhost:8025 | e-mails enviados (Mailpit) |
| `localhost:5432` | PostgreSQL |

`Makefile`:

```makefile
DEV  = docker compose -f compose.yaml -f compose.dev.yaml
PROD = docker compose -f compose.yaml -f compose.prod.yaml

dev:            ## sobe tudo com hot reload
	$(DEV) up --build

dev-av:         ## dev com antivírus
	$(DEV) --profile antivirus up --build

seed:           ## cria usuários e casos de exemplo, um por papel
	$(DEV) exec api dotnet run --project Api -- seed

migrate:        ## aplica as migrations de todos os módulos
	$(DEV) exec api dotnet run --project Api -- migrate

test:           ## testes do backend e do frontend
	cd src/backend && dotnet test
	cd src/frontend && npx ng test --watch=false

reset:          ## apaga o banco local e recomeça
	$(DEV) down -v

deploy:         ## no servidor: atualiza e reinicia
	git pull && $(PROD) up -d --build
```

- O `seed` cria um usuário de teste para cada papel (denunciante, profissional verificado, profissional não verificado, moderador, admin) e alguns casos, para testar o fluxo inteiro sem cadastrar nada à mão.
- No Windows sem `make`, os mesmos comandos funcionam direto com `docker compose -f compose.yaml -f compose.dev.yaml up --build`.

## 9. Produção no primeiro dia

1. Contratar o VPS (Ubuntu LTS), criar um usuário com chave SSH e desativar login por senha e de root.
2. Firewall liberando só as portas 22, 80 e 443. Ativar atualizações automáticas de segurança.
3. Instalar o Docker, clonar o repositório e criar o `.env` de produção.
4. Apontar o DNS do domínio para o IP do VPS (via Cloudflare).
5. `make deploy`. O Caddy emite o certificado HTTPS sozinho.
6. Agendar `deploy/backup.sh` no cron: `pg_dump` cifrado + volumes `provas` e `chaves` → armazenamento externo, todas as noites, guardando 30 dias.
7. Testar a restauração do backup uma vez antes de abrir para o público.

## Fontes

- [Hetzner Cloud: preços de 2026 (Cloudtally)](https://cloudtally.eu/hetzner-cloud-prices)
- [Hetzner Cloud Review 2026 (Better Stack)](https://betterstack.com/community/guides/web-servers/hetzner-cloud-review/)
- [Quanto custa o VPS da Hostinger no Brasil em 2026](https://kildaryoliver.com.br/quanto-custa-vps-hostinger-brasil/)
- [Hostinger VPS vale a pena em 2026? Preços de renovação](https://leiturasingular.com.br/hostinger-vps-vale-a-pena/)
- [Servidor VPS Hostinger: review 2026](https://tudosobrehospedagemdesites.com.br/hostinger/servidor-vps/)
- [Quanto custa um domínio (TecMundo)](https://www.tecmundo.com.br/internet/405860-descubra-quanto-custa-um-dominio-de-site-e-como-registrar-o-seu.htm)
- [Como registrar domínio no Brasil](https://guiaferramentasdigitais.com.br/guias/como-registrar-dominio-brasil/)
- Disponibilidade dos domínios: `https://rdap.registro.br/domain/<nome>`, consultado em 02/11/2026
