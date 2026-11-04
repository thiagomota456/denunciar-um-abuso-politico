# Catálogo de entidades, canais e regras de competência

Dados de partida do módulo Encaminhamento ([arquitetura](../../docs/arquitetura/06-encaminhamento.md)). `make importar-catalogo` lê estes arquivos e faz upsert no schema `encaminhamento` (tarefa T1B.1). Correções entram por pull request, com a fonte.

| Arquivo | Conteúdo |
|---|---|
| `entidades.csv` | Quem recebe denúncias. Chave: `id` |
| `canais.csv` | Como cada entidade recebe. Chave: `entidade_id` + `endereco` |
| `regras-competencia.csv` | Qual grupo de entidades é competente para cada tipo de caso. Chave: `id` |
| `categorias-profissionais.csv` | Profissões e o conselho que as fiscaliza. Chave: `codigo` |

## Formato

- UTF-8, separador vírgula, primeira linha com os nomes das colunas.
- Listas dentro de uma célula usam `|` (ex.: `PA|AP`). Célula vazia em lista = qualquer valor.
- Booleanos: `true`, `false` ou vazio. **Vazio quer dizer "não sabemos"**, não "não".
- `aceita_anonimo` e `aceita_sigilo`: `sim`, `nao` ou `desconhecido`. O modo anônimo só é oferecido com `sim`.
- `situacao` do canal: `ok` (conferido na fonte) ou `revisar` (só o site principal foi conferido). Canal `revisar` aparece com o aviso "confira no site oficial".
- Toda linha tem `fonte_url` e `conferido_em`. Sem elas, a importação recusa a linha.

## Grupos usados nas regras

`mte`, `mpt`, `mpf`, `mp_eleitoral`, `mp_estadual`, `defensoria_estadual`, `dpu`, `disque100`, `falabr`, `conselho_profissional`, `sindicato`, `conselho_tutelar`.

O grupo vira entidade assim: `mpt`, `mp_estadual` e `defensoria_estadual` pela UF do caso; `conselho_profissional` pela profissão (`categorias_profissionais`); `sindicato` pela base territorial e pela busca de texto na categoria do CNES; os demais são nacionais.

## O que já foi conferido (03/11/2026)

- **Canais nacionais conferidos na fonte oficial:** Inspeção do Trabalho do MTE (denúncia geral, desigualdade salarial, Ipê trabalho escravo, Ipê trabalho infantil, assédio eleitoral), MPF Serviços e carta para denúncia anônima, MP Eleitoral via MPF Serviços, Disque 100 (telefone e WhatsApp), Fala.BR.
- **MPT:** os 24 formulários regionais (`peticionamento.prtN.mpt.mp.br/denuncia`) responderam. Anonimato, anexos e protocolo de cada um ainda precisam ser conferidos.
- **Sites principais que responderam**, com canal marcado `revisar`: 25 Ministérios Públicos estaduais, 25 Defensorias estaduais, a DPU e 26 conselhos federais.

**Nenhum canal está em modo `email` ainda.** Nenhum dos canais conferidos aceita denúncia por e-mail. O modo `email` entra conforme a moderação confirmar canais de sindicatos, conselhos regionais e ouvidorias que aceitam e-mail.

## Pendências

- **MPES, MPSE, Defensoria do DF e Defensoria do Piauí:** os sites não responderam ao teste automático. Conferir os endereços à mão.
- **Conselho Federal de Química:** o site não respondeu. Fora do catálogo até ser conferido (a profissão `quimica` não está em `categorias-profissionais.csv`).
- **Conselhos regionais** (cerca de 550): catalogar a partir das listas publicadas por cada conselho federal.
- **Superintendências Regionais do Trabalho** e **ouvidorias estaduais e municipais**.
- **Conselhos Tutelares** (um ou mais por município): o grupo existe nas regras, mas ainda não há entidades. Até lá, a recomendação orienta procurar a prefeitura.
- **Sindicatos:** importar pela T1B.2 a planilha do CNES, que está na [página de entidades sindicais registradas do MTE](https://www.gov.br/trabalho-e-emprego/pt-br/servicos/sindicatos/cadastro-de-entidades/entidade-sindical-registrada). A última planilha encontrada foi `sindicatosbrasil2024.xlsx`; confira se há uma mais nova e registre as colunas reais aqui.
- **DPU nas regras:** hoje as regras apontam só para a Defensoria estadual. Avaliar a DPU para casos contra órgãos federais.

## Como corrigir ou acrescentar

1. Confira a informação no site oficial da entidade.
2. Edite o CSV, com `fonte_url` apontando para a página exata e `conferido_em` com a data.
3. Abra um pull request explicando a mudança. Regras novas precisam de um caso de teste na T1B.4.
