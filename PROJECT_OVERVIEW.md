# FireRiskApp — Project Overview

## O que é

Software web de avaliação de risco de incêndio em edifícios históricos.
Método: **CHICHORRO** (FEUP) — sucessão de dissertações de mestrado na Faculdade de Engenharia da Universidade do Porto (FEUP).

**Autores e dissertações:**

| Autor | Ano | Versão | Dissertação |
|-------|-----|--------|-------------|
| Daniel Martins | 2015 | v1.0 | [Ver dissertação](https://hdl.handle.net/10216/89100) |
| Ricardo Ferreira | 2016 | v2.0 | [Ver dissertação](https://hdl.handle.net/10216/83332) |
| Bruno Silva | 2016 | v2.1 | [Ver dissertação](https://hdl.handle.net/10216/85695) |
| João Teixeira | 2018 | v3.0 | [Ver dissertação](https://hdl.handle.net/10216/113497) |
| Rui Sobral | 2019 | v3.1 | [Ver dissertação](https://hdl.handle.net/10216/122003) |

**Stack:**
- Frontend: React + TypeScript + Vite + Tailwind CSS → Cloudflare Pages
- Backend: Python Flask → Render
- Repo: `joaopmteixeira/chichorro-fire-risk-app`
- App live: `fireriskapp-demo.joaopmteixeira.net` · backend: `chichorro-fire-risk-app.onrender.com`

**Branch principal de trabalho:** `3.1-dev`

---

## Modelo matemático CHICHORRO

```
RI = f(POI, CTI, DPI, ESCI)
```

Cada fator global = média dos fatores parciais com valor > 0.
(`0.00` = "Não se aplica" — excluído da média)

**Escala de classificação 3.1 (12 classes):**

| Classe | RI       |
|--------|----------|
| A++    | ≤ 0.50   |
| A+     | ≤ 0.75   |
| A      | ≤ 1.00   |
| B+     | ≤ 1.10   |
| B      | ≤ 1.25   |
| B-     | ≤ 1.50   |
| C+     | ≤ 1.75   |
| C      | ≤ 2.00   |
| C-     | ≤ 2.25   |
| D      | ≤ 2.50   |
| E      | ≤ 3.00   |
| F      | > 3.00   |

**Limite aceitável (RI_RIA):** calculado a partir de `POI_CC_Idade` (período de construção do edifício).

---

## Estado atual por fase

| Fase | Descrição                                            | Estado                          |
|------|------------------------------------------------------|---------------------------------|
| 1    | CHICHORRO 3.0 — migração e consolidação React        | ✅ Concluído                    |
| 2    | CHICHORRO 3.1 — melhorias RS + módulo intervenções   | ✅ Implementação concluída (pendente teste e2e — C2)|
| 3    | CHICHORRO 4.0 — modelo simplificado RF+Bruno Silva   | 🔮 Futuro                       |

## Módulos implementados

| Módulo      | Subfatores                                              | Estado                                         |
|-------------|---------------------------------------------------------|------------------------------------------------|
| POI         | CC, IEE, IA, ICONFA, ICONSA, IVCA, ILGC, EF, EA, FA, PPP, ATIV | ✅                               |
| CTI         | CI, VHE, VVE                                            | ✅                                             |
| DPI         | REIC, EI, VDGF, PE, OGS                                 | ✅                                             |
| ESCI        | GP, SID, AE, HE, EXT, RIA, CPB                          | ✅                                             |
| RI          | Resultado final + escala 3.1 + aceitabilidade via POI_CC_Idade | ✅                                    |
| Intervenções| 34 ativas/passivas, conjuntos predefinidos, RI pós-intervenção | ✅                                    |

**Próximo passo:** C2 — teste end-to-end completo em produção e aprovação pelo João. Ver `docs/NEXT_STEPS.md`.

---

## Fases de desenvolvimento

### Fase 1 — CHICHORRO 3.0 ✅ Concluído

Reimplementação fiel da versão 3.0 (autor: João Pedro Teixeira) em React.
Base: código legacy `chichorro_old/` (RI_v4.html + POI/CTI/DPI/ESCI.js).

- Arquitetura declarativa (definitions files) para POI, DPI, ESCI
- CTI com lógica própria (inputs numéricos + física complexa)
- RI com aceitabilidade por período de construção
- Auth com cookie Flask, sessão em sessionStorage, export/import de sessão JSON
- Testes end-to-end em produção — aprovados

### Fase 2 — CHICHORRO 3.1 ✅ Quase concluído

Implementação das melhorias do modelo 3.0 → 3.1, desenvolvidas por **Rui Sobral [RS]** na dissertação FEUP 2019.

Alterações do modelo implementadas:
- `CTI`: VHE/VVE com `Dispositivos` separados por veia (B10 pendente no frontend)
- `DPI_OGS`: 7 → 4 campos, nova lógica de formação e regulamento
- `ESCI_GP`: novo campo `ESCI_GP_Auto` (tipo de detetor automático)
- `ESCI_EXT`: novo campo `ESCI_EXT_Formacao`
- `ESCI_RIA+CS`: novos campos `ESCI_RIA_Formacao` e `ESCI_RIA_CS`
- `RI`: escala 3.1 (A++…F, 12 classes); aceitabilidade via `POI_CC_Idade`
- **Intervenções**: 34 ativas/passivas, conjuntos predefinidos, `/RI/interv` endpoint

Referência de análise: `docs/migration/V3_1_MATCHUP_MATRIX.md`

### Fase 3 — CHICHORRO 4.0 🔮 Futuro

Integração do **modelo simplificado** (implementado na v2.0 por **Ricardo Ferreira [RF]**, melhorado por **Bruno Silva**) na base do CHICHORRO 3.1.

O modelo simplificado permite avaliação mais rápida com menos inputs, mantendo boa aproximação ao resultado completo.

Passos:
1. Localizar código v2.0 (RF + Bruno Silva) em `reference/chichorro-2.0-rf/`
2. Analisar como foi implementado o modelo simplificado
3. Definir quais inputs do modelo completo (3.1) são substituídos/simplificados
4. Implementar modelo simplificado no backend Python
5. Implementar UI simplificada no frontend React
6. Testes comparativos modelo completo vs simplificado
7. Deploy

---

## Referências de código

| Pasta                         | Propósito                                                                                |
|-------------------------------|------------------------------------------------------------------------------------------|
| `app/frontend/` + `app/backend/` | **Base ativa** — toda a implementação nova vai aqui                               |
| `reference/chichorro-3.0-jt/` | Versão 3.0 legacy completa (autor: João Teixeira) — referência histórica                |
| `reference/chichorro-3.1-rs/` | Versão 3.1 (autor: Rui Sobral) — referência funcional viva para matchup e migração      |
| `reference/chichorro-2.0-rf/` | Versão 2.0 (autor: Ricardo Ferreira + Bruno Silva) — referência para futura v4.0        |

**Modos de trabalho:**
- `active_build` → editar apenas `app/frontend/` e `app/backend/`
- `v3_0_legacy_compare` → consultar `reference/chichorro-3.0-jt/` como comparação histórica
- `v3_1_matchup` → usar `reference/chichorro-3.1-rs/` como referência funcional; implementar em `app/`
- `v4_0_research` → investigar `reference/chichorro-2.0-rf/`; não alterar base ativa sem decisão explícita

---

## Deploy

### Atual
- Frontend: Cloudflare Pages (auto-deploy do GitHub, branch `deploy`)
- Backend: Render (free tier — cold start ~50s após inatividade)

### Futuro (quando o produto estiver completo)
- VPS único (ex: Hetzner/DigitalOcean) com nginx
- nginx serve o React estático + proxy reverso para Flask/Gunicorn
- Elimina dependência do Render free tier e cold start

---

## Princípios de desenvolvimento

1. A lógica de cálculo permanece em Python (Flask). O React é só UI.
2. Não alterar contratos de API durante migrações.
3. Reproduzir regras de visibilidade condicional (`visibleWhen`) exatamente como no legacy.
4. Cada fase só avança quando a anterior está verificada e testada em produção.
5. Guardar contexto e progresso nos ficheiros de docs a cada sessão de trabalho.
