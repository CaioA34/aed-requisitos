---
name: gerar
description: Gera o código a partir de um documento de requisitos AED num subagente isolado, que recebe apenas o documento, e monta o registro e o roteiro de testes que o /aed-requisitos:avaliar executa.
disable-model-invocation: true
argument-hint: "[caminho do requisitos.md]"
---

# Gerar

Teste se o documento em `$ARGUMENTS` basta sozinho: a implementação acontece no
subagente `aed-requisitos:gerador`, que recebe apenas o prompt fixo abaixo, com o
documento colado. Ele só tem as ferramentas Write e Edit (não lê arquivos nem roda
comandos), usa um modelo fixo e começa sem os CLAUDE.md do usuário.

1. Leia o documento e confirme a situação "Aprovado"; se for outra, indique
   `/aed-requisitos:especificar` e encerre. Anote o sistema e a versão e calcule o
   SHA-256 do arquivo.
2. Procure registros anteriores da mesma versão em
   `aed/<sistema>/geracao-<versão>*-registro.json`. Se algum tiver outro SHA-256,
   o documento mudou sem subir a versão: indique `/aed-requisitos:especificar`
   para registrar a nova versão e encerre.
3. Defina a pasta de destino `aed/<sistema>/geracao-<versão>/`, com sufixo `-2`,
   `-3`... se já existir. Use o caminho absoluto.
4. Anote a data/hora UTC de início e dispare o subagente `aed-requisitos:gerador`
   com o prompt fixo, trocando `{DIRETORIO}` pela pasta e `{ESPECIFICACAO}` pelo
   conteúdo integral do documento. O resto vai palavra por palavra: o prompt é
   fixo para que diferenças de resultado venham do documento.
5. Grave o registro em `aed/<sistema>/<pasta da geração>-registro.json`, fora da
   pasta do código, que fica exatamente como o subagente entregou:

   ```json
   {
     "sistema": "<sistema>",
     "documento": "<caminho do requisitos.md>",
     "versao_documento": "<versão>",
     "sha256_documento": "<hash>",
     "pasta": "<pasta da geração>",
     "agente": "aed-requisitos:gerador",
     "modelo": "claude-sonnet-5-5",
     "versao_prompt": "1",
     "versao_claude_code": "<saída de claude --version>",
     "inicio_utc": "<AAAA-MM-DDTHH:MM:SSZ>",
     "fim_utc": "<AAAA-MM-DDTHH:MM:SSZ>",
     "arquivos": [{"caminho": "<relativo à pasta>", "sha256": "<hash>"}]
   }
   ```

6. Monte o roteiro de testes no modelo do fim deste arquivo, em
   `aed/<sistema>/<pasta da geração>-roteiro.md`, fora da pasta do código.
   - Só depois que o subagente terminar, para que ele nunca veja o roteiro.
   - Só a partir do documento: não abra o código gerado. O roteiro testa o que o
     documento promete, não o que o código faz.
   - Um item por CA, com o mesmo ID, a mesma ordem e o texto copiado do
     documento. "Esperado" é a saída literal do "Então", sem paráfrase.
   - Preparação e Ação são passos executáveis derivados da interface, da entrada
     e do "Dado" do documento, no shell ou na forma de acesso da seção 4. Quando o
     documento não basta para escrever um passo exato, escreva o passo em palavras
     e registre o que faltou em "Pendências do documento".
   - Nenhum passo grava arquivo na pasta da geração; arquivos auxiliares, como
     saídas capturadas, vão para `$env:TEMP` (ou `/tmp` fora do Windows).
   - Um item por RNF, com o método e o limite do documento.
   - Confira que o número de itens de CA é igual ao total de CAs do documento.
7. Informe ao usuário os arquivos criados, o modelo e os caminhos do registro e do
   roteiro, e indique `/aed-requisitos:avaliar <pasta da geração>`.

Pronto quando: a pasta existe com a saída intacta do subagente, o registro está
gravado e o roteiro tem todos os CAs e RNFs do documento.

## Prompt fixo

Versão 1. O prompt, o texto e o `model` de `agents/gerador.md` e os valores
`modelo` e `versao_prompt` do registro andam juntos: mudar um exige subir a versão
e atualizar o registro acima.

```text
Implemente o sistema de software descrito na especificação abaixo.

Condições da tarefa:
1. Escreva todos os arquivos do sistema dentro do diretório {DIRETORIO}. Não crie
   nem modifique nenhum arquivo fora desse diretório.
2. A especificação abaixo é a única fonte de informação sobre o sistema. Não leia
   nenhum outro arquivo.
3. Entregue arquivos completos e funcionais, sem trechos omitidos e sem
   marcadores de continuação.
4. Não escreva testes automatizados.
5. Não faça perguntas: implemente a partir do que está escrito abaixo.
6. Ao terminar, responda apenas com a lista dos arquivos que você criou.

ESPECIFICAÇÃO:

{ESPECIFICACAO}
```

## Modelo do roteiro

````markdown
# Roteiro de testes de conformidade — [Sistema]

| Documento | Versão | Geração | Gerado em | Modelo |
|---|---|---|---|---|
| `[caminho do requisitos.md]` | [versão] | `[pasta da geração]` | [AAAA-MM-DD HH:MM] | [modelo] |

Executado pelo `/aed-requisitos:avaliar`, que grava o resultado em
`[pasta da geração]-avaliacao.md`.

## Como usar

1. Teste o código como o subagente entregou, sem corrigir nada.
2. Siga a ordem do roteiro. Antes de cada CA, deixe o sistema no estado do
   "Preparação"; se a persistência for em memória, reiniciar o sistema zera os dados.
3. O CA é atendido só quando o obtido for igual ao esperado: mesmo status ou código
   de saída e mesmo corpo ou mensagem. Ordem de chaves JSON e espaços não contam.
4. Se o sistema não iniciar, nenhum CA é atendido.

## 1. Preparação (não entra na conta)

- Instalar: `[comando de instalação das dependências]`
- Iniciar: `[comando de inicialização]`, em `[pasta]`
- Sistema acessível em [forma de acesso]

## 2. Critérios de aceitação

### RF-01 — [Título]

- **CA-01.1** [texto do CA, copiado do documento]
  - Preparação: [passos ou comandos que criam o estado do "Dado"]
  - Ação: `[comando exato do "Quando"]`
  - Esperado: [saída literal do "Então"]

## 3. Requisitos não funcionais (contados à parte)

- **RNF-01** [requisito]
  - Como verificar: [método e carga do documento]
  - Aprovado se: [limite do documento]

## 4. Pendências do documento

[Passos que o documento não permitiu escrever de forma exata, com o CA e o que
faltou. Ou "Nenhuma".]

## 5. Conta

Total: [N] CAs e [M] RNFs. Conformidade = CAs atendidos ÷ [N] × 100. Meta: ≥ 85%.
````
