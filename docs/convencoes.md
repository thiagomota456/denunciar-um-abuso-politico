# Convenções

Padrões para manter o código previsível. Na dúvida, copie o padrão de um módulo que já existe.

## Estrutura do backend

```
src/backend/
├── Denuncias.sln
├── Api/                              # host: Program.cs, registro dos módulos, middlewares
├── Comum/                            # usado por todos os módulos
│   ├── Cifra/                        # atributo [Cifrado], value converter, hash HMAC
│   ├── Auditoria/                    # IAuditoria, AuditoriaDbContext (schema auditoria)
│   ├── Autorizacao/                  # policies e handlers por caso
│   └── Web/                          # ProblemDetails, paginação
├── Modulos/
│   ├── Identidade/                   # cadastro, login, profissionais e verificação
│   ├── Casos/                        # denúncias, provas, classificação, moderação
│   └── Assistencia/                  # interesse, autorização, atendimento; notificações e avaliações na F2
│                                     # Plataforma/ (doações, fundo) só é criado na Fase 3
└── tests/
    ├── Unitarios/
    ├── Integracao/
    └── Arquitetura/                  # garante que um módulo não usa o interior de outro
```

Cada módulo é **um projeto de class library** com esta forma:

```
Modulos/Casos/
├── CasosModulo.cs                    # AddCasos(services) e MapCasos(endpoints)
├── Contratos/                        # ÚNICA parte pública: interfaces e DTOs para outros módulos
├── Dominio/                          # entidades e regras, sem EF nem HTTP
├── Aplicacao/                        # casos de uso: uma classe por ação (ex.: PublicarDenuncia)
├── Infra/                            # CasosDbContext, configurações EF, migrations
└── Endpoints/                        # Minimal APIs agrupadas por recurso
```

- Tudo fora de `Contratos/` é `internal`.
- Um módulo chama outro **só** por uma interface de `Contratos/`. Nunca acessa o `DbContext` de outro módulo.
- Sem MediatR: o endpoint chama a classe de caso de uso diretamente.
- Sem AutoMapper: mapeamento manual, explícito.

## API

- Minimal APIs com `MapGroup`, prefixo `/api/v1/<recurso>`.
- JSON em camelCase. Datas em UTC, ISO 8601. IDs `uuid` (versão 7, ordenável).
- Erros sempre em `ProblemDetails`. Mensagens para o usuário em português; sem detalhes internos.
- **Recurso de outra pessoa responde 404**, não 403, para não revelar que existe.
- Paginação: `?pagina=1&tamanho=20`, máximo 100. Resposta `{ itens, total, pagina, tamanho }`.
- Validação com a validação nativa do ASP.NET Core para Minimal APIs (atributos nos DTOs de entrada).
- Endpoints de dados privados recebem o ID do caso na rota, e a policy checa a autorização daquele caso.
- Documento OpenAPI nativo em `/openapi/v1.json`, só em Development.

## Banco

- Tabelas e colunas em `snake_case`, nomes em português sem acento (ver [modelo de dados](arquitetura/04-modelo-de-dados.md)).
- Um schema e um `DbContext` por módulo. Migrations ficam no `Infra/Migrations` do módulo. A API aplica todas ao subir em Development; em produção, pelo comando `migrate`.
- Referência a tabela de outro módulo é só o ID, sem chave estrangeira no banco.
- Campos sensíveis: propriedade com `[Cifrado]`. A coluna recebe o texto cifrado; o nome da coluna termina em `_cifrado`. Não dá para filtrar por ela: se precisar filtrar, a informação vai numa coluna comum e deixa de ser sensível (ex.: `denuncia.precisa_gratuito`).
- Busca por CPF: coluna `cpf_hash` com HMAC-SHA256 (chave em `Cifra__ChaveHmac`), nunca o CPF puro.
- Nada de exclusão física de denúncia, prova ou auditoria pela aplicação. Exclusão a pedido do titular segue a política LGPD (tarefa T1.12).

## Nomes em C#

| O quê | Padrão | Exemplo |
|---|---|---|
| Entidade | substantivo singular | `Denuncia`, `Prova`, `Autorizacao` |
| Caso de uso | verbo + objeto | `PublicarDenuncia`, `RevogarAutorizacao` |
| DTO de entrada | `<CasoDeUso>Requisicao` | `PublicarDenunciaRequisicao` |
| DTO de saída | `<Recurso>Resposta` | `DenunciaPublicaResposta` |
| Enum | singular | `StatusDenuncia.EmRevisao` |
| Teste | `Metodo_Cenario_Resultado` | `Publicar_SemModeracao_Falha` |

Sem acento em identificadores. Termos do framework ficam em inglês (`DbContext`, `Controller`, `Handler`).

## Estrutura do frontend

```
src/frontend/src/app/
├── core/               # autenticação, interceptors, guards, cliente da API
├── shared/             # componentes visuais reutilizáveis, pipes
└── areas/              # uma rota lazy por área
    ├── publico/
    ├── denunciante/
    ├── profissional/
    └── admin/
```

- Standalone components, signals para estado, `inject()` em vez de construtor.
- Formulários reativos tipados. Formulários longos viram etapas (um componente por etapa) e salvam rascunho a cada etapa.
- **Sem biblioteca de componentes** no MVP: CSS próprio com variáveis (tokens) em `styles/tokens.css`. Mantém o app leve em 3G.
- Toda chamada passa por `core/api`. Nenhum componente usa `HttpClient` direto.
- Guards de rota existem só para a experiência; a segurança é do backend.
- Acessibilidade: `<label>` em todo campo, foco visível, contraste AA, funciona por teclado, alvo de toque ≥ 44 px.

## Autenticação

- ASP.NET Core Identity com **cookie** `HttpOnly`, `Secure`, `SameSite=Lax`. O Angular e a API ficam no mesmo domínio (o Caddy faz o proxy de `/api`), então não há JWT. E-mail, senha, MFA e bloqueio usam as colunas do próprio Identity; os papéis usam as roles do Identity.
- Proteção contra CSRF: antiforgery do ASP.NET Core com cookie `XSRF-TOKEN` e header `X-XSRF-TOKEN`, que o `HttpClient` do Angular envia sozinho.
- MFA com app autenticador (TOTP): opcional para denunciante, obrigatório para profissional, moderador e admin.
- Rate limit no login, no cadastro e no upload.

## Testes

- Cada caso de uso tem teste de unidade das regras.
- Cada endpoint tem teste de integração com PostgreSQL real (Testcontainers), incluindo **um teste por papel que não deveria ter acesso**.
- O teste de matriz de permissões (`tests/Integracao/Permissoes/`) espelha a tabela de [permissões](arquitetura/05-permissoes.md). Ao mudar a tabela, mude o teste no mesmo commit.
- Dados de teste neutros: nada de partidos, políticos ou posições reais.

## Commits

- Em português, no imperativo: `Adiciona upload de provas`, `Corrige revogação de autorização`.
- Um commit por tarefa ou subtarefa, citando o ID: `T1.5: Adiciona varredura de provas com ClamAV`.
