# 1 · Visão geral: módulos e integrações

Monólito modular com um único deploy. O backend tem **3 módulos no MVP** (um projeto C# e um schema do banco cada), um quarto na Fase 1B (Encaminhamento) e um quinto na Fase 3. Os módulos só conversam entre si por interfaces públicas (`Contratos/`), o que permite separá-los no futuro se for preciso.

```mermaid
flowchart LR
    subgraph Usuarios["Quem usa"]
        U1["Denunciante"]
        U2["Advogado ou escritório"]
        U3["Moderador e admin"]
        U4["Visitante"]
    end

    FE["Angular<br/>áreas: pública, denunciante,<br/>profissional, admin"]

    subgraph API["API ASP.NET Core · um único processo"]
        direction TB
        subgraph MI["Identidade"]
            I1["cadastro, login, MFA"]
            I2["profissionais e verificação da OAB"]
        end
        subgraph MC["Casos"]
            C1["denúncias e classificação"]
            C2["provas"]
            C3["moderação e fale conosco"]
        end
        subgraph MA["Assistencia"]
            A1["interesse e autorização"]
            A2["atendimentos"]
            A3["notificações em rodízio · F2"]
            A4["avaliações · F2"]
        end
        subgraph ME["Encaminhamento · F1B"]
            E4["catálogo de entidades e canais"]
            E5["recomendação por competência"]
            E6["dossiê, envio e acompanhamento"]
        end
        subgraph MP["Plataforma · F3"]
            P1["doações, fundo, transparência, publicidade"]
        end
        COMUM["Comum: autorização por caso · cifra de campos · auditoria · rate limit"]
    end

    subgraph Infra["No mesmo VPS"]
        PG[("PostgreSQL<br/>dados + fila do Hangfire")]
        FS[("Disco cifrado<br/>arquivos de provas")]
        AV["ClamAV<br/>antivírus"]
    end

    subgraph Externos["Serviços externos"]
        E1["SMTP<br/>e-mails"]
        E2["Gateway PIX e cartão · F3"]
        E3["gov.br · F2, se aprovado"]
        E7["Entidades com canal de e-mail<br/>conferido pela moderação · F1B"]
        E8["Caixa IMAP<br/>respostas das entidades · F1B"]
    end

    U1 & U2 & U3 & U4 --> FE
    FE -->|"HTTPS /api"| API
    API --> PG & FS & AV
    API -.-> E1 & E2 & E3 & E7 & E8
```

**Legenda:** sem marcação = MVP (Fase 1). F1B, F2 e F3 = fases seguintes. Linha tracejada = serviço externo.

A verificação da OAB é **manual** no MVP: o admin confere no site do CNA, que não tem API pública (ver [decisões](../decisoes.md), D9).

Formulários oficiais (MTE, MPT, MPF, Fala.BR) não aparecem como integração: o denunciante os preenche no site oficial, com os campos prontos pela plataforma (modo assistido, ver [encaminhamento](06-encaminhamento.md) e D23).

## Implantação

```mermaid
flowchart TB
    Internet((Internet)) --> Caddy
    subgraph VPS["1 VPS · Docker Compose"]
        Caddy["Caddy<br/>HTTPS automático<br/>serve o Angular"] -->|/api| Api["API ASP.NET Core<br/>+ Hangfire"]
        Api --> Pg[("PostgreSQL")]
        Api --> Disco[("Volume de provas")]
        Api --> Clam["ClamAV"]
    end
    Pg -. backup noturno cifrado .-> Backup[("Armazenamento externo")]
    Disco -. backup noturno .-> Backup
    Api -. SMTP .-> Email["Provedor de e-mail"]
```
