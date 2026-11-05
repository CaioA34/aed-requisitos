# aed-requisitos (MVP)

Aplica o processo do trabalho AED: escrever um documento de requisitos que baste
sozinho para uma LLM gerar o sistema, gerar o código a partir dele e medir a
conformidade do código com o documento.

## Fluxo

1. `/aed-requisitos:especificar <ideia do sistema>` → `aed/<sistema>/requisitos.md`,
   revisado pelo subagente `revisor`, que lê só o documento
2. `/aed-requisitos:gerar aed/<sistema>/requisitos.md` → código em
   `aed/<sistema>/geracao-<versão>/`, escrito pelo subagente `gerador`, e ao lado
   `geracao-<versão>-registro.json` (hashes, modelo, horários) e
   `geracao-<versão>-roteiro.md` (montado só a partir do documento)
3. `/aed-requisitos:avaliar aed/<sistema>/geracao-<versão>` →
   `geracao-<versão>-avaliacao.md`, escrito a partir do relatório do subagente
   `avaliador`, com a conformidade (CAs atendidos ÷ total) e a meta de 85%

Os três subagentes usam modelo fixo (`claude-sonnet-5-5`), começam sem os
CLAUDE.md do usuário e têm só as ferramentas de que precisam: o `revisor` só lê,
o `gerador` só escreve e o `avaliador` lê e executa comandos. O modelo, os
prompts fixos e o texto dos agentes são variáveis controladas: mudar qualquer um
deles muda o que se está medindo.

O `avaliador` primeiro lê o código e prevê, para cada CA, se ele será atendido.
Depois executa o roteiro como está escrito. O resultado de cada CA vem da
execução; a leitura aponta a causa das falhas, e as divergências entre previsão e
execução ficam no relatório. A skill confere os hashes do código antes e depois:
se o avaliador alterar o código, a avaliação sai marcada como inválida.

O `avaliador` roda o código gerado na sua máquina. Fora do modo automático, o
Claude Code pede permissão para cada comando.

## Fora do MVP, de propósito

- Estatística entre gerações (taxa de sucesso, teste de hipótese) e o relatório
  do experimento.
- Comparação documento estruturado × texto livre.
- Revisão por uma pessoa diferente do autor (a revisão é de outra instância do
  modelo, sem acesso à conversa).

## Ideias para expandir

- Gerar e avaliar N vezes a mesma versão e consolidar as avaliações.
- Um braço de controle com texto livre e o mesmo contrato de execução.
