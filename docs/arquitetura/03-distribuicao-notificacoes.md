# 3 · Distribuição de notificações em rodízio

Implementado como um job recorrente do Hangfire e uma consulta SQL com filtro e ordenação. Sem serviço de fila nem motor de regras à parte.

```mermaid
flowchart LR
    A["Caso publicado<br/>moderação aprovou resumo e classificação"] --> B
    B["Filtrar elegíveis<br/>OAB verificada e regular · área jurídica<br/>abrangência geográfica · aceita novos casos"] --> C
    C["Ordenar por adequação<br/>especialidade · proximidade<br/>aceita gratuito, se o caso pedir"] --> D
    D["Aplicar rodízio<br/>primeiro quem foi menos notificado em 30 dias<br/>teto semanal por profissional"] --> E
    E["Notificar lote de N<br/>e-mail + aviso no painel<br/>grava cada envio em notificacao"] --> F
    F{"Interesse em até X dias?"}
    F -->|sim| G["Denunciante decide<br/>compara perfis e autoriza ou recusa"]
    F -.->|não: amplia a abrangência e chama o próximo lote| B
```

> N e X ainda precisam ser definidos.

## Regras

- **Sem leilão nem pagamento por caso.** Ninguém compra prioridade, o que evita captação irregular de clientela (Código de Ética da OAB).
- **A notificação não abre dados privados.** O profissional vê só o resumo público até o denunciante autorizar.
- **Tudo é registrado.** Quem foi notificado, quando e por qual critério, para auditar se o rodízio está equilibrado.
