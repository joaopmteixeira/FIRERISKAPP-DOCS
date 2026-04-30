# FireRiskApp

**Avaliação de risco de incêndio em edifícios históricos**

FireRiskApp é uma aplicação web que implementa o método **CHICHORRO** (FEUP) para calcular o Risco de Incêndio (RI) em edifícios existentes com valor patrimonial.

O método combina quatro fatores independentes:

```
RI = f(POI, CTI, DPI, ESCI)
```

| Fator | Descrição |
|-------|-----------|
| **POI** | Potencial Ocorrência de Incêndio |
| **CTI** | Consequências do Incêndio para os Utilizadores |
| **DPI** | Desenvolvimento e Propagação do Incêndio |
| **ESCI** | Eficácia de Socorro e Combate ao Incêndio |

O resultado é classificado numa escala de 12 classes, de **A++** (risco muito baixo) a **F** (risco muito elevado).

---

## Documentação rápida

- [Visão Geral do Projeto](PROJECT_OVERVIEW.md) — objetivos, stack, estado das fases
- [Arquitetura](ARCHITECTURE.md) — estrutura de ficheiros, endpoints, sessionStorage
- [Próximos Passos](NEXT_STEPS.md) — tarefas em curso e concluídas
- [Alterações UX/UI](FRONTEND_UX_MODIFICATIONS.md) — histórico de melhorias de interface

---

## Versão atual

**v3.1** — implementação completa do modelo CHICHORRO 3.1 (dissertação Rui Sobral, 2019).

- Backend: Python Flask → Render
- Frontend: React + TypeScript + Vite + Tailwind CSS → Cloudflare Pages

---

## Autores

- **João Teixeira** — CHICHORRO v3.0, arquitetura da aplicação web
- **Rui Sobral** — CHICHORRO v3.1 (dissertação FEUP, 2019)
- **Ricardo Ferreira** — CHICHORRO v2.0 (base do método simplificado, Fase 3)
