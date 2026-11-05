# aed-requisitos

Repositório da AED "Processo e Artefato" de Sistemas Distribuídos: um processo de
análise de requisitos, um modelo de documento em Markdown e um experimento que
mede se uma LLM gera código conforme ao documento (meta: 85% dos critérios de
aceitação).

O processo está empacotado como o plugin do Claude Code
[`aed-requisitos`](plugins/aed-requisitos/README.md), com três skills:

| Skill | O que faz |
|---|---|
| `/aed-requisitos:especificar` | Elicitação, análise e escrita do documento, com revisão independente |
| `/aed-requisitos:gerar` | Gera o código num subagente que recebe só o documento |
| `/aed-requisitos:avaliar` | Lê o código gerado, executa o roteiro e calcula a conformidade |

## Instalar

```bash
claude plugin marketplace add https://github.com/CaioA34/aed-requisitos.git
claude plugin install aed-requisitos@aed-requisitos
```

Atualizar depois de mudanças no repositório:

```bash
claude plugin marketplace update aed-requisitos
claude plugin update aed-requisitos@aed-requisitos
```
