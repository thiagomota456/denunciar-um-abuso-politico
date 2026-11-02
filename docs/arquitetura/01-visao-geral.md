# 1 · Visão geral: módulos e integrações

Monólito modular com um único deploy. Cada módulo tem suas próprias tabelas e serviços e só conversa com os outros por interfaces internas, o que permite separá-los no futuro se for preciso.

```mermaid
flowchart LR
    subgraph Usuarios["Quem usa"]
        U1["Denunciante<br/>registra casos e provas"]
        U2["Advogado ou escritório<br/>busca casos, presta assistência"]
        U3["Moderador e admin<br/>revisa, verifica, audita"]
        U4["Visitante e doador<br/>consulta, doa, vê transparência"]
    end

    FE["Angular<br/>um app, carregamento sob demanda por área<br/>servido como estático pelo Caddy"]

    subgraph API["API ASP.NET Core (C#) · monólito modular"]
        direction TB
        M1["Identidade e acesso<br/>login, MFA, CPF via gov.br · F1"]
        M2["Denúncias<br/>registro em etapas, status · F1"]
        M3["Provas e arquivos<br/>upload cifrado, hash, antivírus · F1"]
        M4["Consentimento<br/>autorização por escopo, revogação · F1"]
        M5["Profissionais<br/>perfil, OAB, verificação · F1"]
        M6["Classificação<br/>categoria, área, urgência · F1 manual, F2 auto"]
        M7["Notificações<br/>rodízio, e-mail, painel · F2"]
        M8["Moderação<br/>revisão do resumo, abuso · F1"]
        M9["Atendimentos e avaliações · F2"]
        M10["Doações e fundo<br/>transparência · F3"]
        M11["Publicidade · F3"]
        M12["Auditoria e LGPD · F1"]
        COMUM["Camada comum: autorização por papel e por caso · criptografia de campos · jobs · rate limit e antispam · logs sem dados pessoais"]
    end

    subgraph Dados["Dados e infraestrutura (mesmo VPS)"]
        PG[("PostgreSQL<br/>EF Core, backup diário cifrado")]
        FS[("Arquivos de provas<br/>disco cifrado; depois S3 barato")]
        JOBS["Hangfire no Postgres<br/>e-mails, antivírus, rodízio"]
        KEYS["Data Protection do .NET<br/>chave fora do banco"]
    end

    subgraph Externos["Serviços externos"]
        E1["gov.br<br/>validação de identidade"]
        E2["CNA / OAB<br/>situação da inscrição"]
        E3["E-mail e SMS"]
        E4["Pagamentos PIX e cartão · F3"]
        E5["ClamAV<br/>antivírus no mesmo VPS"]
    end

    U1 & U2 & U3 & U4 --> FE
    FE -->|HTTPS / JSON| API
    API --> PG & FS & JOBS & KEYS
    API -.-> E1 & E2 & E3 & E4 & E5
```

**Legenda:** F1, F2, F3 = fase do roadmap (MVP, operação, sustentabilidade). Linha tracejada = integração externa, opcional e substituível.

## Implantação mínima

```mermaid
flowchart TB
    Internet((Internet)) --> Caddy
    subgraph VPS["1 VPS de entrada · Docker Compose"]
        Caddy["Caddy<br/>HTTPS grátis (Let's Encrypt)<br/>serve o Angular"] -->|/api| Api["API ASP.NET Core<br/>+ Hangfire"]
        Api --> Pg[("PostgreSQL")]
        Api --> Disco[("Volume de provas<br/>cifrado")]
        Api --> Clam["ClamAV"]
    end
    Pg -. backup noturno cifrado .-> Backup[("Armazenamento externo barato")]
    Disco -. backup noturno .-> Backup
    Api -. SMTP .-> Email["E-mail transacional<br/>plano gratuito"]
```
