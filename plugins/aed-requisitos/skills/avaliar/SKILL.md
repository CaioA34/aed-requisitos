---
name: avaliar
description: Mede a conformidade de uma geração AED com o documento de requisitos num subagente isolado, que lê o código gerado e depois executa o roteiro de testes.
disable-model-invocation: true
argument-hint: "[pasta da geração, ex.: aed/notas/geracao-1.0]"
---

# Avaliar

Meça quanto a geração em `$ARGUMENTS` cumpre o documento que a originou. A
avaliação acontece no subagente `aed-requisitos:avaliador`. Ele lê o código, depois
executa o roteiro, e quem decide cada CA é a execução. O subagente usa um modelo
fixo, começa sem os CLAUDE.md do usuário e não vê esta conversa.

1. Localize, ao lado da pasta, `<pasta>-registro.json` e `<pasta>-roteiro.md`. Se
   faltar algum, indique `/aed-requisitos:gerar` e encerre. Leia o registro para
   saber o documento, o sistema e a versão.
2. Confira a integridade antes de avaliar:
   - O SHA-256 do documento é igual a `sha256_documento`.
   - Todo arquivo de `arquivos` existe com o mesmo SHA-256, e a pasta não tem
     arquivo fora dessa lista.

   Se algo não bater, informe o que mudou e encerre: o código avaliado precisa ser
   o que o gerador entregou.
3. Defina o arquivo de saída `<pasta>-avaliacao.md`, ao lado da pasta; se já
   existir, use `<pasta>-avaliacao-2.md`, `-3`... Avaliar de novo a mesma geração
   mede a variação do avaliador.
4. Anote a data/hora UTC de início e dispare o subagente `aed-requisitos:avaliador`
   com o prompt fixo, trocando os campos pelos caminhos absolutos. O resto vai
   palavra por palavra.
5. Confira a integridade de novo, como no passo 2. Arquivo alterado ou apagado
   invalida a avaliação: grave o relatório assim mesmo e marque "inválida" no
   cabeçalho. Arquivo novo criado pela execução não invalida, mas entra no
   cabeçalho. Depois apague os arquivos novos, para a pasta voltar a ser só a
   saída do gerador.
6. Confira o relatório:
   - A tabela de CAs tem uma linha por CA do roteiro, com os mesmos IDs.
   - A linha "Conformidade" bate com a contagem de `atendido` na tabela.

   Se a contagem não bater, corrija a linha pela tabela e anote a correção no
   cabeçalho. Se faltar CA, conte-o como não atendido e anote também.
7. Grave `<pasta>-avaliacao.md` com o cabeçalho abaixo seguido do relatório do
   subagente, sem editar o relatório além do passo 6.
8. Informe ao usuário a conformidade, se a meta de 85% foi atingida, os CAs que
   falharam e o caminho do arquivo.

Pronto quando: a avaliação está gravada, a integridade foi conferida antes e
depois e a conformidade bate com a tabela.

## Prompt fixo

Versão 1. O prompt, o texto e o `model` de `agents/avaliador.md` e os valores do
cabeçalho andam juntos: mudar um exige subir a versão.

```text
Avalie a geração.
Documento: {DOCUMENTO}
Roteiro: {ROTEIRO}
Pasta da geração: {PASTA}
```

## Cabeçalho da avaliação

```markdown
# Avaliação de conformidade — [Sistema]

| Documento | Versão | Geração | Avaliador | Prompt | Início (UTC) | Fim (UTC) |
|---|---|---|---|---|---|---|
| `[caminho]` | [versão] | `[pasta]` | `aed-requisitos:avaliador` (claude-sonnet-5-5) | 1 | [AAAA-MM-DDTHH:MM:SSZ] | [AAAA-MM-DDTHH:MM:SSZ] |

- **Integridade:** [código idêntico ao registro antes e depois | inválida: <o que mudou>]
- **Arquivos criados pela execução (removidos):** [lista | nenhum]
- **Correções da conferência:** [lista | nenhuma]
- **Meta (≥ 85% dos CAs):** [atingida | não atingida]
```
