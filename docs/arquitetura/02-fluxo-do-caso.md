# 2 · Fluxo do caso entre os atores

Os 8 passos do processo de assistência. Entre parênteses, o módulo responsável.

No MVP, a classificação é feita pelo moderador junto com a revisão, e o profissional encontra o caso pela busca. A sugestão automática de classificação e as notificações em rodízio entram na Fase 2.

```mermaid
sequenceDiagram
    autonumber
    actor D as Denunciante
    participant P as Plataforma
    actor M as Moderação
    actor A as Profissional

    D->>P: Cadastra a denúncia e envia as provas (Casos)
    P->>M: Coloca na fila de revisão
    M->>P: Classifica, revisa o resumo e publica sem dados pessoais (Casos)
    P->>A: Caso aparece na busca. Na F2, também notifica em rodízio (Assistencia)
    A->>P: Manifesta interesse, vendo só o resumo público (Assistencia)
    P->>D: Mostra o perfil do profissional
    alt Autoriza
        D->>P: Autoriza e escolhe o escopo do que compartilhar (Assistencia)
        P->>A: Libera os dados do escopo autorizado
        A->>P: Avalia e formaliza a assistência (Assistencia)
        P-->>D: Painel privado do caso
        P-->>A: Painel privado do caso
    else Recusa
        D->>P: Recusa o pedido
        P-->>A: Avisa que o pedido foi recusado
    end

    Note over D,A: Em todos os passos, Auditoria registra quem fez, o quê, quando e sobre qual caso
```
