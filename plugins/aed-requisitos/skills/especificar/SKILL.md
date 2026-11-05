---
name: especificar
description: Conduz elicitação, análise e escrita de um documento de requisitos no modelo AED, com revisão independente, pronto para uma LLM gerar o código.
disable-model-invocation: true
argument-hint: "[ideia do sistema ou caminho de um rascunho]"
---

# Especificar

Transforme a necessidade em `$ARGUMENTS` num documento de requisitos que baste
sozinho para outra LLM, sem acesso a esta conversa, implementar o sistema. O
documento é a única ponte: o que ficar só na conversa se perde na geração.

O modelo do documento está no fim deste arquivo. Se
`$ARGUMENTS` for um documento já aprovado, vá direto para "Alterar um documento
aprovado".

## 1. Elicitação

Entreviste o usuário sobre objetivo, atores, funcionalidades, o que fica de fora,
stack e como o sistema é executado. Faça 3 perguntas por vez, começando pelas que
mudam o desenho do sistema.

Pronto quando: objetivo em uma frase, atores, necessidades, escopo excluído e
stack confirmados pelo usuário.

## 2. Análise

Derive entidades, estados e transições, regras de negócio e a lista de RFs.
Mostre como lista curta e resolva conflitos e lacunas com o usuário.

Pronto quando: toda necessidade virou RF/RN ou foi excluída com motivo, e o
usuário aprovou a lista.

## 3. Especificação

Preencha o modelo em `aed/<sistema>/requisitos.md`:

- Cada CA em Dado/Quando/Então com saída literal: status ou código de saída e
  corpo ou mensagem exatos. Ex.: "então retorna `404` com
  `{"erro": "Livro não encontrado"}`".
- Todo RF tem um CA de sucesso e um de falha (ou de limite, quando não houver falha).
- Os mesmos nomes técnicos, em crases, em todo o documento.
- Seção sem aplicação: "Não aplicável" com o motivo.
- "Deve" para obrigação. A lista de CAs fica fechada aqui: é a base da medição
  feita pelo `/aed-requisitos:avaliar`.
- Ponto que o usuário ainda não decidiu: `[A ESCLARECER: <pergunta>]` no lugar,
  em vez de escolher por ele.

Pronto quando: nenhum placeholder `[...]` restante; marcadores `[A ESCLARECER]`
são resolvidos na revisão.

## 4. Revisão independente

1. Dispare o subagente `aed-requisitos:revisor` com a tarefa
   `Revise o documento em <caminho absoluto do requisitos.md>.` e nada mais: ele
   não vê esta conversa e julga só o que está escrito.
2. Para cada item reprovado ou ambiguidade: corrija direto o que for de redação;
   leve ao usuário, 3 perguntas por vez, o que for decisão. Resolva também os
   `[A ESCLARECER]`. As "Sugestões de CA" não bloqueiam: mostre-as ao usuário,
   que decide se entram.
3. Repita a partir do passo 1 até o veredito "aprovado". Depois de 3 rodadas sem
   aprovação, mostre ao usuário o que resta e pergunte como seguir.
4. Confira você mesmo o item que o revisor não pode ver: toda decisão de
   implementação tomada nesta conversa está no documento.

Pronto quando: veredito "aprovado", a conferência do passo 4 feita e o usuário
aprovou o documento. Marque a situação como "Aprovado", informe o total de CAs e
indique `/aed-requisitos:gerar`. Deixe claro que a revisão foi feita por outra
instância do modelo, não por outra pessoa.

## Alterar um documento aprovado

Toda mudança depois de "Aprovado" sobe a versão (1.0 → 1.1), ganha uma linha na
tabela de versões com a situação "Rascunho" e passa de novo pela revisão
independente. Sem isso, gerações de documentos diferentes ficam com a mesma versão,
e o `/aed-requisitos:gerar` recusa o documento.

## Modelo do documento

````markdown
# Requisitos — [Sistema]

| Versão | Data | Situação | Mudança |
|---|---|---|---|
| 1.0 | [AAAA-MM-DD] | [Rascunho / Aprovado] | Versão inicial |

## 1. Objetivo e atores

O sistema permite que [ator] [ação] para [benefício].

| Ator | Necessidade |
|---|---|
| [ator] | [necessidade] |

## 2. Escopo

- **Incluído:** [capacidades]
- **Excluído:** [capacidade — motivo]

## 3. Glossário

| Termo | Definição no domínio |
|---|---|
| [termo] | [significado] |

## 4. Tecnologia e execução

| Item | Valor |
|---|---|
| Linguagem e versão | [ex.: Node 22] |
| Framework | [nome e versão/faixa] |
| Bibliotecas permitidas | [lista fechada] |
| Persistência | [ex.: em memória, descartada ao encerrar] |
| Comando de inicialização | `[comando exato]`, executado em `[pasta]` |
| Forma de acesso | [ex.: HTTP em 127.0.0.1:8000 / CLI `app <args>`] |

Arquivos esperados ([obrigatórios / ilustrativos]):

```text
[árvore de pastas e arquivos]
```

## 5. Modelo de dados

### 5.1 [Entidade]

| Campo | Tipo | Obrigatório | Restrições | Exemplo |
|---|---|---|---|---|
| `[campo]` | [tipo] | [sim/não] | [limites, unicidade, geração] | [valor] |

- **Identificador:** [quem gera, tipo]
- **Estados:** [valores, estado inicial, transições válidas]
- **Relacionamentos:** [referências e efeito da exclusão]

## 6. Regras de negócio

| ID | Regra (condição → consequência) | RFs |
|---|---|---|
| RN-01 | [condição → consequência] | RF-01 |

## 7. Erros globais

- **Formato:** [exemplo literal]
- **Validação:** [campo ausente, tipo inválido, fora do limite → resposta]
- **Precedência:** [ordem quando há mais de um erro]
- **Estado:** [efeito de uma rejeição sobre os dados]

## 8. Requisitos funcionais

### RF-01 — [Título]

- **Descrição:** O sistema deve [comportamento].
- **Interface:** [método e rota / comando e argumentos / ação de tela]
- **Entrada:** [exemplo literal válido, ou "sem entrada"]
- **Sucesso:** [status/código + saída literal]
- **Erros:**

  | Condição | Status/código | Resposta literal |
  |---|---|---|
  | [condição] | [código] | [resposta] |

- **Regras:** [RN-xx ou não aplicável]
- **Critérios de aceitação:**
  - **CA-01.1** Dado [estado concreto], quando [ação], então [saída literal e estado final].
  - **CA-01.2** Dado [erro ou limite], quando [ação], então [saída literal].

## 9. Requisitos não funcionais

| ID | Requisito | Como verificar e limite de aprovação |
|---|---|---|
| RNF-01 | [propriedade] | [método, carga, limite] |

## 10. Restrições de geração

1. Entregar arquivos completos, sem trechos omitidos.
2. Implementar exatamente o escopo incluído.
3. Preservar nomes, contratos e mensagens literais deste documento.
4. Usar apenas a stack e as bibliotecas da seção 4.
5. Produzir apenas os arquivos da seção 4.
````
