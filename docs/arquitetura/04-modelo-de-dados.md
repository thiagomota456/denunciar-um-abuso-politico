# 4 · Modelo de dados (PostgreSQL + EF Core)

Um schema por módulo e um DbContext por schema: `identidade`, `casos`, `assistencia`, `plataforma`. Campos marcados como "cifrado" são criptografados na aplicação e ficam ilegíveis mesmo para quem acessar o banco.

```mermaid
erDiagram
    usuario ||--o| perfil_denunciante : tem
    usuario ||--o| profissional : tem
    escritorio |o--o{ profissional : agrupa
    usuario ||--o{ denuncia : registra
    denuncia ||--|| denuncia_privada : tem
    denuncia ||--o{ prova : tem
    denuncia ||--o| capacidade_financeira : tem
    denuncia ||--o{ interesse : recebe
    profissional ||--o{ interesse : manifesta
    interesse ||--o| autorizacao : gera
    autorizacao ||--o| atendimento : abre
    atendimento ||--o{ movimentacao : tem
    atendimento ||--o| avaliacao : recebe
    profissional ||--o{ notificacao : recebe
    denuncia ||--o{ notificacao : origina
    denuncia ||--o{ moderacao : passa_por
    usuario ||--o{ registro_auditoria : gera
    denuncia ||--o{ apoio_fundo : recebe

    usuario {
        uuid id PK
        string nome
        string cpf_cifrado "cifrado"
        string cpf_hash UK "para busca"
        string email UK
        string telefone_cifrado "cifrado"
        string papel "denunciante, profissional, moderador, admin"
        string senha_hash
        bool mfa_ativo
        timestamp identidade_verificada_em
    }
    perfil_denunciante {
        uuid usuario_id PK, FK
        string municipio
        string uf
        string contato_seguranca_cifrado "cifrado"
    }
    profissional {
        uuid usuario_id PK, FK
        uuid escritorio_id FK
        string oab_numero
        string oab_uf
        string oab_situacao
        timestamp verificado_em
        uuid verificado_por
        string_array areas
        jsonb abrangencia "UFs e municípios"
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
        timestamp publicada_em
    }
    denuncia_privada {
        uuid denuncia_id PK, FK
        text relato_cifrado "cifrado"
        text envolvidos_cifrado "cifrado"
        text testemunhas_cifrado "cifrado"
    }
    prova {
        uuid id PK
        uuid denuncia_id FK
        string tipo "documento, foto, audio, video"
        string caminho_arquivo "cifrado em disco"
        string sha256 "integridade"
        bigint tamanho_bytes
        string antivirus "pendente, limpo, bloqueado"
    }
    capacidade_financeira {
        uuid denuncia_id PK, FK
        string pode_pagar_cifrado "cifrado"
        string faixa_valor_cifrado "cifrado"
        string precisa_gratuito_cifrado "cifrado"
        string pede_apoio_fundo_cifrado "cifrado"
    }
    interesse {
        uuid id PK
        uuid denuncia_id FK
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
        uuid autor_id FK
        string tipo
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
        string criterio
        timestamp enviada_em
        timestamp aberta_em
    }
    registro_auditoria {
        bigint id PK
        uuid ator_id FK
        string acao
        string recurso
        string recurso_id
        string ip_hash
        timestamp em "só INSERT"
    }
    moderacao {
        uuid id PK
        uuid denuncia_id FK
        uuid moderador_id FK
        string decisao
        string motivo
        timestamp em
    }
    doacao {
        uuid id PK
        decimal valor
        string meio "pix, cartao"
        string id_gateway
        timestamp em
    }
    lancamento {
        uuid id PK
        string tipo "receita, despesa, fundo"
        string categoria
        decimal valor
        date data
        string comprovante
    }
    apoio_fundo {
        uuid id PK
        uuid denuncia_id FK
        decimal valor_aprovado
        string criterio
        timestamp decidido_em
    }
```

`doacao`, `lancamento` e `apoio_fundo` entram na Fase 3. A tabela `registro_auditoria` só aceita INSERT: o usuário de banco da aplicação não tem permissão de UPDATE nem DELETE nela.
