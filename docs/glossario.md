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
| Moderação | Revisão humana que classifica o caso, ajusta o resumo público e decide se publica, aprova sem publicar (só para encaminhamento, Fase 1B) ou devolve. | `moderacao` |
| Classificação | Categoria, área jurídica e urgência do caso. | colunas de `denuncia` |
| Interesse | Pedido de um profissional para ajudar num caso. Não dá acesso a nada privado. | `interesse` |
| Autorização | Permissão dada pelo denunciante a um profissional, com escopo. Pode ser revogada. | `autorizacao` |
| Escopo | Quais partes dos dados privados a autorização libera: relato, provas, testemunhas, contato e, na Fase 1B, encaminhamentos. | `autorizacao.escopo` |
| Atendimento | Relação entre profissional e caso depois da autorização, até a formalização ou o encerramento. | `atendimento` |
| Movimentação | Registro de andamento dentro do painel privado do caso. | `movimentacao` |
| Rodízio | Regra que distribui as notificações de casos entre profissionais elegíveis, priorizando quem foi menos notificado. Fase 2. | módulo Assistencia, tabela `notificacao` |
| Lote | Grupo de N profissionais notificados de uma vez para um caso. | `notificacao.lote` |
| Manifestação | Mensagem enviada pelo fale conosco: dúvida, sugestão, reclamação, pedido de inclusão ou correção no catálogo, ou reporte de conteúdo abusivo. Não é uma denúncia. | `manifestacao` |
| Auditoria | Registro imutável de quem fez o quê, quando e sobre qual recurso. | `registro_auditoria` |
| Fundo de apoio | Recursos excedentes da plataforma destinados a pagar a assistência de quem não pode. Fase 3. | `apoio_fundo`, `lancamento` |
| Entidade | Quem pode receber uma denúncia: órgão público, Ministério Público, conselho profissional, sindicato, ouvidoria, canal de direitos humanos. Fase 1B. | `entidade` |
| Natureza | Papel da entidade: fiscalizadora, investigativa ou judicial, representação, acolhimento, assistência jurídica. | `entidade.natureza` |
| Canal | Meio oficial pelo qual a entidade recebe denúncias, com o que ele aceita (anônimo, sigilo, anexos, login, captcha). | `canal` |
| Modo de envio | Como a plataforma usa um canal: `email` (automático), `assistido` (o denunciante envia com os campos prontos), `orientacao` (roteiro e dossiê impresso). | `canal.modo_envio` |
| Regra de competência | Linha que diz qual entidade é competente para um tipo de abuso, vínculo, setor e profissão. | `regra_competencia` |
| Recomendação | Lista de entidades competentes para um caso, com o motivo de cada uma. Calculada, não gravada. | caso de uso `RecomendarDestinatarios` |
| Dossiê | Documento gerado para um destinatário a partir do caso, com o relato literal e a lista de provas. | `encaminhamento.dossie_cifrado` |
| Encaminhamento | Envio de um caso a uma entidade, com status, protocolo, comprovante e respostas. | `encaminhamento` |
| Identificação | O que o dossiê revela do denunciante: `identificado`, `sigiloso` ou `anonimo`, conforme o canal aceita. | `encaminhamento.identificacao` |
| Protocolo | Número que a entidade dá ao registrar a denúncia. Cifrado, porque às vezes permite consultar o processo. | `encaminhamento.protocolo_cifrado` |
| Comprovante | Arquivo que prova o envio (tela ou PDF do site oficial). Guardado como as provas. | `comprovante` |
| CNES | Cadastro Nacional de Entidades Sindicais, do Ministério do Trabalho. Fonte dos sindicatos do catálogo. | `make importar-cnes` |
