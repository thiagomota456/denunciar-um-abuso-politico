# Glossário

| Termo | Significado | No código |
|---|---|---|
| Denunciante | Pessoa que registra uma denúncia. Pode ficar anônima no resumo público, mas sempre tem conta identificada. | usuário do Identity com a role `denunciante` + `perfil_denunciante` |
| Profissional | Advogado ou advogada com conta na plataforma. | `profissional` |
| Profissional verificado | Profissional cuja inscrição na OAB foi conferida pelo admin. Pode aparecer na busca e manifestar interesse. | `profissional.verificado_em` não nulo |
| Profissional autorizado | Profissional verificado que recebeu uma autorização ativa do denunciante para um caso específico. | `autorizacao` com `revogada_em` nulo |
| Escritório | Pessoa jurídica à qual profissionais podem estar ligados. | `escritorio` |
| Denúncia | O caso registrado. Tem uma parte pública e uma privada. | `denuncia` |
| Resumo público | Título, resumo, categoria, região aproximada, período e status. Só aparece depois da moderação. | colunas públicas de `denuncia` |
| Dados privados | Relato, envolvidos, testemunhas, contatos, capacidade financeira e provas. Sempre cifrados. | `denuncia_privada`, `capacidade_financeira`, `prova` |
| Prova | Arquivo enviado pelo denunciante: documento, foto, áudio ou vídeo. | `prova` |
| Capacidade financeira | O que o denunciante pode pagar, se precisa de atendimento gratuito e se pede apoio do fundo. Confidencial. | `capacidade_financeira` |
| Moderação | Revisão humana que classifica o caso, ajusta o resumo público e decide se publica. | `moderacao` |
| Classificação | Categoria, área jurídica e urgência do caso. | colunas de `denuncia` |
| Interesse | Pedido de um profissional para ajudar num caso. Não dá acesso a nada privado. | `interesse` |
| Autorização | Permissão dada pelo denunciante a um profissional, com escopo. Pode ser revogada. | `autorizacao` |
| Escopo | Quais partes dos dados privados a autorização libera: relato, provas, testemunhas, contato. | `autorizacao.escopo` |
| Atendimento | Relação entre profissional e caso depois da autorização, até a formalização ou o encerramento. | `atendimento` |
| Movimentação | Registro de andamento dentro do painel privado do caso. | `movimentacao` |
| Rodízio | Regra que distribui as notificações de casos entre profissionais elegíveis, priorizando quem foi menos notificado. Fase 2. | módulo Assistencia, tabela `notificacao` |
| Lote | Grupo de N profissionais notificados de uma vez para um caso. | `notificacao.lote` |
| Auditoria | Registro imutável de quem fez o quê, quando e sobre qual recurso. | `registro_auditoria` |
| Fundo de apoio | Recursos excedentes da plataforma destinados a pagar a assistência de quem não pode. Fase 3. | `apoio_fundo`, `lancamento` |
