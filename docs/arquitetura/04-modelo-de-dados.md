# 4 · Modelo de dados (PostgreSQL + EF Core)

Um schema por módulo, cada um com seu `DbContext`:

| Schema | Dono | Tabelas |
|---|---|---|
| `identidade` | módulo Identidade | tabelas do ASP.NET Core Identity, `perfil_denunciante`, `profissional`, `escritorio` |
| `casos` | módulo Casos | `denuncia`, `denuncia_privada`, `capacidade_financeira`, `prova`, `moderacao`, `manifestacao` |
| `assistencia` | módulo Assistencia | `interesse`, `autorizacao`, `atendimento`, `movimentacao`, `avaliacao`, `notificacao` |
| `encaminhamento` | módulo Encaminhamento (Fase 1B) | `entidade`, `canal`, `regra_competencia`, `categoria_profissional`, `modelo_documento`, `encaminhamento`, `evento_encaminhamento`, `comprovante` |
| `auditoria` | Comum | `registro_auditoria` |
| `plataforma` | módulo Plataforma (Fase 3) | `doacao`, `lancamento`, `apoio_fundo` |

Campos com "cifrado" são criptografados na aplicação e ficam ilegíveis para quem acessar o banco ou um backup. Por isso **não dá para filtrar nem ordenar por eles em SQL**: tudo que precisa entrar num `WHERE` fica em coluna comum.

```mermaid
erDiagram
    usuario ||--o| perfil_denunciante : tem
    usuario ||--o| profissional : tem
    escritorio |o--o{ profissional : agrupa
    usuario ||--o{ denuncia : registra
    denuncia ||--|| denuncia_privada : tem
    denuncia ||--o| capacidade_financeira : tem
    denuncia ||--o{ prova : tem
    denuncia ||--o{ moderacao : passa_por
    denuncia ||--o{ interesse : recebe
    profissional ||--o{ interesse : manifesta
    interesse ||--o| autorizacao : gera
    autorizacao ||--o| atendimento : abre
    atendimento ||--o{ movimentacao : tem
    atendimento ||--o| avaliacao : recebe
    profissional ||--o{ notificacao : recebe
    denuncia ||--o{ notificacao : origina

    usuario {
        uuid id PK "IdentityUser: e-mail, senha, MFA e bloqueio ficam nas colunas do Identity"
        string nome
        string cpf_cifrado "cifrado"
        string cpf_hash UK "HMAC, para checar duplicidade"
        string telefone_cifrado "cifrado; não usar PhoneNumber do Identity"
        timestamp criado_em
    }
    perfil_denunciante {
        uuid usuario_id PK, FK
        string municipio
        string uf
    }
    profissional {
        uuid usuario_id PK, FK
        uuid escritorio_id FK
        string oab_numero
        string oab_uf
        timestamp verificado_em "nulo = não verificado"
        uuid verificado_por
        string observacao_verificacao
        string_array areas
        string_array ufs_atendidas
        bool aceita_novos_casos
        bool aceita_gratuito
        jsonb site_redes
    }
    escritorio {
        uuid id PK
        string cnpj UK
        string razao_social
        string site
    }
    denuncia {
        uuid id PK
        uuid autor_id FK
        string titulo
        string resumo_publico
        string categoria "tipo de abuso; lista em 06-encaminhamento"
        string area_juridica
        string municipio_aprox
        string uf
        date periodo_inicio
        date periodo_fim
        string status "rascunho, em_revisao, aprovada, publicada, em_atendimento, encerrada"
        bool anonimo
        int urgencia "1 a 3"
        bool precisa_gratuito "não público; usado no filtro de profissionais"
        string vinculo "não público; F1B: clt, servidor_estatutario..."
        string setor "não público; F1B: privado, publico_federal..."
        string categoria_profissional "não público; F1B: código ou nulo"
        string municipio_ibge "não público; F1B"
        bool afeta_varias_pessoas "não público; F1B"
        string empregador_cifrado "cifrado; F1B: nome, CNPJ e endereço de quem é denunciado"
        timestamp publicada_em
    }
    denuncia_privada {
        uuid denuncia_id PK, FK
        text relato_cifrado "cifrado"
        text envolvidos_cifrado "cifrado"
        text testemunhas_cifrado "cifrado"
    }
    capacidade_financeira {
        uuid denuncia_id PK, FK
        string pode_pagar_cifrado "cifrado"
        string faixa_valor_cifrado "cifrado"
        string pede_apoio_fundo_cifrado "cifrado"
    }
    prova {
        uuid id PK
        uuid denuncia_id FK
        string tipo "documento, foto, audio, video"
        string nome_original_cifrado "cifrado"
        string caminho_arquivo "arquivo cifrado em disco"
        string sha256 "integridade"
        bigint tamanho_bytes
        string antivirus "pendente, limpo, bloqueado"
    }
    moderacao {
        uuid id PK
        uuid denuncia_id FK
        uuid moderador_id
        string decisao "publicar, aprovar_sem_publicar, devolver"
        string motivo
        timestamp em
    }
    interesse {
        uuid id PK
        uuid denuncia_id FK "único com profissional_id"
        uuid profissional_id FK
        text mensagem
        string status "aberto, aceito, recusado"
    }
    autorizacao {
        uuid id PK
        uuid interesse_id FK
        string_array escopo "relato, provas, testemunhas, contato, encaminhamentos"
        timestamp concedida_em
        timestamp revogada_em "nulo = ativa"
    }
    atendimento {
        uuid id PK
        uuid autorizacao_id FK
        string status "avaliando, formalizado, encerrado"
        timestamp formalizado_em
    }
    movimentacao {
        uuid id PK
        uuid atendimento_id FK
        uuid autor_id
        text texto_cifrado "cifrado"
        timestamp criado_em
    }
    avaliacao {
        uuid atendimento_id PK, FK
        int comunicacao "1 a 5"
        int clareza "1 a 5"
        int cumpriu_compromissos "1 a 5"
        text comentario
        bool moderada
    }
    notificacao {
        uuid id PK
        uuid profissional_id FK
        uuid denuncia_id FK
        int lote
        timestamp enviada_em
    }
```

`aprovada` = revisada pela moderação e liberada para encaminhamento automático, sem resumo público. Serve para quem quer encaminhar o caso a órgãos sem publicá-lo.

## Schema `encaminhamento` (Fase 1B)

Detalhes de uso em [encaminhamento](06-encaminhamento.md). O catálogo (`entidade`, `canal`, `regra_competencia`, `categoria_profissional`) é público e não tem dados pessoais. `denuncia_id` é referência a outro schema, sem chave estrangeira.

```mermaid
erDiagram
    entidade ||--o{ canal : tem
    categoria_profissional ||--o{ regra_competencia : filtra
    entidade ||--o{ encaminhamento : recebe
    canal ||--o{ encaminhamento : usado_em
    encaminhamento ||--o{ evento_encaminhamento : tem
    encaminhamento ||--o{ comprovante : tem

    entidade {
        uuid id PK
        string grupo "ex.: mpt, mp_estadual, sindicato; as regras apontam para o grupo"
        string nome
        string sigla
        string tipo "inspecao_trabalho, ministerio_publico, conselho_profissional, sindicato, federacao, confederacao, ouvidoria, defensoria, direitos_humanos, conselho_tutelar"
        string natureza "fiscalizadora, investigativa_judicial, representacao, acolhimento, assistencia_juridica"
        string esfera "federal, estadual, municipal, nao_estatal"
        string_array ufs "vazio = todo o Brasil"
        string_array municipios_ibge "vazio = toda a UF"
        string_array categorias_profissionais "vazio = qualquer"
        text categoria_texto "texto do CNES, para busca de texto"
        string cnpj
        string codigo_sindical UK "só sindicatos, chave da importação do CNES"
        string origem "catalogo, cnes, sugestao"
        string fonte_url
        date conferido_em
        bool ativa
    }
    canal {
        uuid id PK
        uuid entidade_id FK
        string tipo "formulario_web, email, telefone, whatsapp, presencial, correio"
        string endereco "URL, e-mail, telefone ou endereço postal"
        string modo_envio "email, assistido, orientacao, api"
        string aceita_anonimo "sim, nao, desconhecido"
        string aceita_sigilo "sim, nao, desconhecido"
        bool exige_login_govbr
        bool tem_captcha
        int max_anexos
        int max_anexos_mb
        bool gera_protocolo
        int prazo_resposta_dias
        jsonb campos "campos do formulário oficial e de onde vem o texto de cada um"
        text instrucoes
        string fonte_url
        date conferido_em
        string situacao "ok, url_falhou, email_devolvido, revisar"
    }
    regra_competencia {
        uuid id PK
        string categoria "tipo de abuso"
        string_array vinculos "vazio = qualquer"
        string_array setores "vazio = qualquer"
        string categoria_profissional FK "nulo = qualquer"
        bool so_se_afeta_varias_pessoas
        string entidade_grupo "destino: as entidades do grupo que atendem a UF, o município ou a profissão do caso"
        string canal_preferido "endereço do canal a usar, quando a entidade tem vários"
        int prioridade
        string motivo "texto mostrado ao denunciante"
        string fonte_url
    }
    categoria_profissional {
        string codigo PK
        string nome
        string conselho_sigla "ex.: COFEN; nulo se não regulamentada"
    }
    encaminhamento {
        uuid id PK
        uuid denuncia_id "único com entidade_id enquanto ativo"
        uuid entidade_id FK
        uuid canal_id FK
        string identificacao "identificado, sigiloso, anonimo"
        string status "rascunho, autorizado, enviado, falha, protocolado, respondido, encerrado, cancelado"
        text dossie_cifrado "cifrado; cópia exata do que foi autorizado"
        string dossie_sha256
        uuid_array provas_enviadas
        timestamp autorizado_em
        string versao_consentimento
        string token_resposta UK "endereço de resposta do e-mail"
        string protocolo_cifrado "cifrado"
        timestamp enviado_em
        timestamp lembrete_em
    }
    evento_encaminhamento {
        uuid id PK
        uuid encaminhamento_id FK
        string tipo "criado, autorizado, enviado, falha, protocolo, comprovante, resposta, lembrete, status, cancelado"
        text texto_cifrado "cifrado; ex.: corpo da resposta"
        uuid autor_id "nulo = sistema"
        timestamp em
    }
    comprovante {
        uuid id PK
        uuid encaminhamento_id FK
        string caminho_arquivo "arquivo cifrado em disco, como as provas"
        string sha256
        string antivirus "pendente, limpo, bloqueado"
    }
```

`modelo_documento` (fora do diagrama): `tipo_entidade`, `cabecalho`, `pedido`, `versao`. Um modelo por tipo de entidade, editável pela moderação.

Tabelas fora do diagrama:

- **`casos.manifestacao`** (fale conosco, pedidos e reportes): `id`, `tipo` (`duvida`, `sugestao`, `reclamacao`, `incluir_entidade`, `incluir_categoria`, `corrigir_catalogo`, `conteudo_abusivo`), `autor_id` (nulo = visitante), `denuncia_id` (nulo, ou o caso reportado, ou um caso do próprio autor), `entidade_id` (nulo, ou a entidade a corrigir), `dados_pedido jsonb` (nome, UF e site oficial da entidade pedida; nada pessoal), `texto_cifrado` (cifrado), `email_resposta_cifrado` (cifrado, opcional), `status` (`aberta`, `em_analise`, `respondida`, `aceita`, `recusada`), `resposta_cifrado` (cifrado), `tratada_por`, `criado_em`, `respondida_em`.
- **`auditoria.registro_auditoria`**: `id bigserial`, `ator_id uuid`, `acao`, `recurso`, `recurso_id`, `em`. Um trigger no banco recusa UPDATE e DELETE. Guarda só IDs, nunca dados pessoais.
- **Fase 3, schema `plataforma`**: `doacao` (valor, meio, id no gateway, data), `lancamento` (tipo receita/despesa/fundo, categoria, valor, data, comprovante) e `apoio_fundo` (denuncia_id, valor aprovado, critério, data da decisão).

Referências entre schemas (ex.: `denuncia.autor_id` → usuário) são só IDs, sem chave estrangeira no banco: cada módulo é dono das suas tabelas.
