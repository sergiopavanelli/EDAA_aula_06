# EDAA — Aula 06: TADs Lineares

**Aluno:** Sérgio Pinton Pavanelli
**RA:** 123220202
**Unidade Curricular:** Estruturas de Dados e Análise de Algoritmos (0006963)
**UniBH — 2026/1**

---

## Sobre este repositório

Resoluções dos exercícios da **Aula 06**, que aborda os principais Tipos Abstratos de Dados lineares implementados em **Rust**:

| Grupo | TAD | Exercícios |
|-------|-----|-----------|
| 1 | Vec e operações básicas | 1 – 4 |
| 2 | Pilha (Stack) | 5 – 9 |
| 3 | Fila (Queue) | 10 – 13 |
| 4 | Deque (Double-Ended Queue) | 14 – 16 |
| 5 | Reflexão e análise | 17 – 20 |

## Arquivos

| Arquivo | Descrição |
|---------|-----------|
| `resolucoes.md` | Código Rust e análise de complexidade de todos os 20 exercícios |
| `exercicios061_20260330173622.pdf` | Enunciado original fornecido pelo professor |

## Tópicos abordados

- Inversão, contagem, remoção e mescla com `Vec`
- Calculadora RPN, histórico de navegação, undo/redo, balanceamento de delimitadores e `StackMin`
- Simulador de banco, impressora, buffer circular e fila de prioridade manual
- Palíndromo, janela deslizante máxima e fila com prioridade de frente com `VecDeque`
- Benchmark comparativo (`Vec` ingênua vs `VecDeque` vs fila circular)
- Escalonamento Round Robin

Cada implementação inclui a **complexidade de tempo e espaço** documentada.
