# 4 · Modelo de dados (PostgreSQL + EF Core)

Um schema por módulo, cada um com seu `DbContext`:

| Schema | Dono | Tabelas |
|---|---|---|
| `identidade` | módulo Identidade | tabelas do ASP.NET Core Identity, `perfil_denunciante`, `profissional`, `escritorio` |
| `casos` | módulo Casos | `denuncia`, `denuncia_privada`, `capacidade_financeira`, `prova`, `moderacao` |
| `assistencia` | módulo Assistencia | `interesse`, `autorizacao`, `atendimento`, `movimentacao`, `avaliacao`, `notificacao` |
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
        string categoria
        string area_juridica
        string municipio_aprox
        string uf
        date periodo_inicio
        date periodo_fim
        string status "rascunho, em_revisao, publicada, em_atendimento, encerrada"
        bool anonimo
        int urgencia "1 a 3"
        bool precisa_gratuito "não público; usado no filtro de profissionais"
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
        string decisao "publicar, devolver"
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
        string_array escopo "relato, provas, testemunhas, contato"
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

Tabelas fora do diagrama:

- **`auditoria.registro_auditoria`**: `id bigserial`, `ator_id uuid`, `acao`, `recurso`, `recurso_id`, `em`. Um trigger no banco recusa UPDATE e DELETE. Guarda só IDs, nunca dados pessoais.
- **Fase 3, schema `plataforma`**: `doacao` (valor, meio, id no gateway, data), `lancamento` (tipo receita/despesa/fundo, categoria, valor, data, comprovante) e `apoio_fundo` (denuncia_id, valor aprovado, critério, data da decisão).

Referências entre schemas (ex.: `denuncia.autor_id` → usuário) são só IDs, sem chave estrangeira no banco: cada módulo é dono das suas tabelas.
