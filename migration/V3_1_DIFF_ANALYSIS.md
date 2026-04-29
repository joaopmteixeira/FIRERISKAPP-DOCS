# V3.1 Diff Analysis — 3.0 → 3.1

Fonte: leitura direta de `reference/chichorro-3.1-rs/chichorro-html/`
Data da análise: 2026-04-20
Analista: Claude (arquiteto técnico)

---

## Resumo executivo

A migração 3.0 → 3.1 envolve:
- **4 módulos com alterações funcionais** (POI, ESCI, DPI, RI)
- **1 módulo sem alterações identificadas** (CTI — a confirmar)
- **1 feature nova de raiz** (Módulo de Intervenções)
- **Nova escala de classificação RI** (6 classes → 12 classes)

---

## 1. POI — `POI_CC`

### Alteração
Novo campo `POI_CC_Idade` adicionado à função `POI_CC()`.

### Valores
| Valor backend | Label |
|---|---|
| `>2008` | Posterior a 2008 |
| `1991-2008` | 1991–2008 |
| `1975-1990` | 1975–1990 |
| `1968-1974` | 1968–1974 |
| `1951-1967` | 1951–1967 |
| `<1951` | Anterior a 1951 |

### Comportamento
- `POI_CC_Idade` **não altera o score de POI_CC** — é apenas passado como parâmetro
- É consumido pela função `RI()` para calcular `RI_RIA` (limite de aceitabilidade)
- Na versão 3.0 ativa, este campo já existe em `RiPage.tsx` como seletor separado
- Em 3.1, migra conceptualmente para o subfator CC do POI

### Impacto no backend
- `Chichorro_POI.py`: assinatura `POI_CC(Comb, Infiltracoes, Revestimento, Idade)` — Idade aceite mas não usada no cálculo
- `Chichorro_RI.py`: `RI()` usa `POI_CC_Idade` para definir `RI_RIA`
- `Flask.py`: endpoint `/POI/CC` precisa de aceitar `POI_CC_Idade` no payload

### Impacto no frontend
- `poiDefinitions.ts`: adicionar campo `POI_CC_Idade` ao subfator `cc`
- `RiPage.tsx`: remover seletor de período (passa a vir de POI)

---

## 2. ESCI — `ESCI_GP`

### Alteração
Novo campo `ESCI_GP_Auto` — tipo de detetor automático.

### Valores
| Valor backend | Label | Score (ex: ≤10km, ≤10min) |
|---|---|---|
| `Termovelocimetrico` | Termovelocimétrico | 0.90 |
| `Otico` | Ótico | 0.80 |
| `Aspiracao` | Aspiração | 0.70 |

### Comportamento
- Só é relevante quando `ESCI_GP_DetAler == 'Automatico'`
- A deteção por aspiração é a melhor — reduz o score em ~0.20 vs ausência de deteção
- A termovelocimétrica é a menos eficaz entre os automáticos

### Impacto no backend
- `Chichorro_ESCI.py`: assinatura `ESCI_GP(Distancia, Tempo, DetAler, Auto)` — novo 4.º parâmetro
- `Flask.py`: endpoint `/ESCI/GP` aceita `ESCI_GP_Auto`

### Impacto no frontend
- `esciDefinitions.ts`: adicionar campo `ESCI_GP_Auto` com `visibleWhen: (v) => v.ESCI_GP_DetAler === 'Automatico'`

---

## 3. ESCI — `ESCI_EXT`

### Alterações
1. **Renomeação das opções de OGS** (atenção: string values mudam)
2. **Novo campo `ESCI_EXT_Formacao`**

### OGS — comparação de valores
| 3.0 | 3.1 |
|---|---|
| `Sem OGS - PP+F` | `Sem OGS - R+PP` |
| `Com OGS - PP+F` | `Com OGS - R+PP` |
| `Com OGS - PE+S` | `Com OGS - PE+S` (igual) |

### ESCI_EXT_Formacao
| Valor | Label |
|---|---|
| `Sem Formacao` | Sem formação |
| `Com Formacao` | Com formação |

### Assinatura backend
- **3.0:** `ESCI_EXT(Aplica, OGS, Extintores)`
- **3.1:** `ESCI_EXT(Aplica, OGS, Formacao, Extintores)`

### Impacto no score
A formação reduz o score em 0.05–0.10 por linha da tabela.

### Impacto no frontend
- `esciDefinitions.ts`: subfator `ext`
  - Atualizar opções de `ESCI_EXT_OGS`
  - Adicionar campo `ESCI_EXT_Formacao` (visível quando `Aplica`)
  - Rever `visibleWhen` de `ESCI_EXT_OGS` e `ESCI_EXT_Extintores`

---

## 4. ESCI — `ESCI_RIA`

### Alterações
1. **Renomeação das opções de OGS** (mesma lógica que EXT)
2. **Novo campo `ESCI_RIA_Formacao`**
3. **Novo campo `ESCI_RIA_CS`** (Coluna Seca)

### OGS — comparação de valores
| 3.0 | 3.1 |
|---|---|
| `Sem OGS - PP+F` | `Sem OGS - R+PP` |
| `Com OGS - PP+F` | `Com OGS - R+PP` |
| `Com OGS - PE+S` | `Com OGS - PE+S` (igual) |

### ESCI_RIA_CS
| Valor | Label | Efeito |
|---|---|---|
| `RIA` | Só RIA | base |
| `RIA+CS` | RIA + Coluna Seca | melhora score ~0.05 |

### Assinatura backend
- **3.0:** `ESCI_RIA(Aplica, OGS, RIA)`
- **3.1:** `ESCI_RIA(Aplica, OGS, Formacao, CS, RIA)`

### Impacto no frontend
- `esciDefinitions.ts`: subfator `ria`
  - Atualizar opções de `ESCI_RIA_OGS`
  - Adicionar `ESCI_RIA_Formacao`
  - Adicionar `ESCI_RIA_CS`
  - Rever ordem e `visibleWhen`

---

## 5. DPI — `DPI_OGS`

### Alteração completa — 7 campos → 4 campos

**3.0 (campos existentes):**
- `DPI_OGS_Aplica`
- `DPI_OGS_P_Emergencia_Exigencia`
- `DPI_OGS_P_Emergencia_Regul`
- `DPI_OGS_Formacao_Simulacro_Exigencia`
- `DPI_OGS_Formacao_Simulacro_Regul`
- `DPI_OGS_Equipa_Exigencia`
- `DPI_OGS_Equipa_Regul`

**3.1 (novos campos):**
| Campo | Opções |
|---|---|
| `DPI_OGS_Aplica` | `Nao Se Aplica` / `Aplica` |
| `DPI_OGS_OGS` | `Registos e Procedimento de Prevenção` / `Registo e Plano de Prevenção` / `Plano de Segurança e Simulacro` |
| `DPI_OGS_Formacao` | `Sem Formacao` / `Com Formacao` |
| `DPI_OGS_Regulamento` | `Existe` / `Respeita Totalmente` / `Nao Respeita` |

### Ajuste pós-cálculo (atenção — lógica não óbvia)
Após calcular o score base, o backend aplica:
- `Regulamento == 'Existe'` → score -= 0.1
- `Regulamento == 'Respeita Totalmente'` → score -= 0.2
- `Regulamento == 'Nao Respeita'` → score -= 0.0

### Assinatura backend
- **3.0:** `DPI_OGS(Aplica, P_Emergencia_Exigencia, P_Emergencia_Regul, ...)`
- **3.1:** `DPI_OGS(Aplica, OGS, Formacao, Regulamento)`

### Impacto no frontend
- `dpiDefinitions.ts`: substituição completa do subfator `ogs` — de 7 campos para 4

---

## 6. RI — Escala de classificação

### Nova escala (12 classes em vez de 6)

| 3.0 | 3.1 | Limiar |
|---|---|---|
| A1 | A++ | ≤ 0.90 |
| — | A+ | ≤ 0.95 |
| A2 | A | ≤ 1.00 |
| — | B+ | ≤ 1.05 |
| B | B | ≤ 1.10 |
| — | B- | ≤ 1.15 |
| — | C+ | ≤ 1.20 |
| C | C | ≤ 1.25 |
| — | C- | ≤ 1.30 |
| D | D | ≤ 1.50 |
| E | E | ≤ 1.70 |
| — | F | > 1.70 |

### Limites de aceitabilidade por `POI_CC_Idade`
Estes não mudaram em relação ao que estava em 3.0:
| Período | `RI_RIA` |
|---|---|
| `>2008` | 1.00 |
| `1991-2008` | 1.05 |
| `1975-1990` | 1.10 |
| `1968-1974` | 1.15 |
| `1951-1967` | 1.20 |
| `<1951` | 1.25 |

### Impacto no frontend
- `RiPage.tsx`: substituir lógica de escala e labels visuais

---

## 7. CTI — Estado da análise

A dissertação RS menciona melhorias em CPIVHE e CPIVVE.
**Não foi feita leitura direta de `reference/chichorro-3.1-rs/Chichorro_CTI.py`.**
Estado: **unknown — requer análise antes de implementação**.

---

## 8. Módulo de Intervenções — Feature nova

### Visão geral
- 34 intervenções booleanas (`RI_interv_01` a `RI_interv_34`)
- Organizadas em 6 conjuntos cumulativos (Conjunto1 a Conjunto6) por `POI_ATIV_TipoEdif`
- Cada intervenção modifica diretamente os parâmetros de entrada e recalcula o RI
- O backend devolve: `RI_interv`, `RI_Escala_interv`, `Custo_interv`

### Ficheiro de referência
`reference/chichorro-3.1-rs/chichorro-html/Chichorro_RI_inter.py`

### Mapa das 34 intervenções (resumido)
| # | Parâmetros afetados | Custo relativo |
|---|---|---|
| 01 | `ESCI_HE_Aplica='Aplica'`, `ESCI_HE_Distancia='<=30m'` | 50 |
| 02 | `ESCI_AE_Andar`, `ESCI_AE_Acesso` (lógica dependente de EF) | 50 |
| 03 | `ESCI_SID_Dispositivos='Sinal'` | 1 |
| 04 | `ESCI_SID_Dispositivos='Sinal+Ilum'`, CTI dispositivos | 5 |
| 05 | CTI `Dispositivos='Sinalizacao + Iluminacao'` | 3 |
| 06 | `ESCI_GP_DetAler='Automatico'`, `GP_Auto='Termovelocimetrico'` | 3 |
| 07 | `ESCI_GP_DetAler='Automatico'`, `GP_Auto='Otico'` | 3 |
| 08 | `ESCI_GP_DetAler='Automatico'`, `GP_Auto='Aspiracao'` | 3 |
| 09 | `ESCI_SID_Dispositivos='Sinal+Ilum+Det'` | 2 |
| 10 | `DPI_REIC_*`, CTI `A_claraboia` | 7 |
| 11 | CTI `SistemaControloFumo='Com'` | 50 |
| 12 | `ESCI_EXT_Aplica='Aplica'`, `EXT_Extintores='Existe'` | 1 |
| 13 | `ESCI_RIA_Aplica='Aplica'`, `RIA_CS='RIA'`, `RIA='Existe'` | 25 |
| 14 | `ESCI_RIA_CS='RIA+CS'`, `RIA='Existe'` | 25 |
| 15 | CTI `SistemaExtincao='Com'` | 150 |
| 16 | `DPI_OGS_OGS`, `ESCI_EXT_OGS`, `ESCI_RIA_OGS` → R+PP | 5 |
| 17 | `DPI_OGS_OGS='Plano de Segurança e Simulacro'`, `ESCI_SID_OGS='Existe'` | 5 |
| 18 | `ESCI_EXT_Formacao='Com Formacao'`, `ESCI_RIA_Formacao`, `DPI_OGS_Formacao` | 5 |
| 19 | `POI_CC_Infiltracoes='Sem'` | 50 |
| 20 | `POI_FA_*`, `DPI_EI_*` | 15 |
| 21 | Redução de efetivo (flag para CTI) | 50 |
| 22 | `POI_EF_ElemConstr='Respeita'`, `DPI_VDGF_*` | 20 |
| 23 | `POI_EA_*`, `DPI_PE_*` | 7 |
| 24 | `POI_FA_Lajes='Respeita'` | 10 |
| 25 | `POI_FA_Vaos` com caixa enclausurada, `DPI_REIC_*` | 50 |
| 26 | `POI_FA_Condutas='Respeita'` | 100 |
| 27 | `POI_FA_EnvolvFrac='Respeita'`, `DPI_EI_*` | 15 |
| 28 | `POI_CC_Revestimento='Adequado'`, CTI reação ao fogo | 20 |
| 29 | `POI_IEE_Regul='Respeita'`, `POI_FA_Condutas` | 20 |
| 30 | `POI_ILGC_*` | 5 |
| 31 | `POI_IVCA_*` | 5 |
| 32 | `POI_IA_*` (aquecimento autónomo elétrico) | 5 |
| 33 | `POI_IA_TipoInst='Centrais Term'` | 2 |
| 34 | `POI_ICONFA_*`, `POI_ICONSA_*` | 10 |

### Conjuntos por UT (estrutura)
- Conjunto1 a Conjunto6, cumulativos (6 inclui tudo de 5, etc.)
- Filtrados por `POI_ATIV_TipoEdif` (I-Habitação, III/VII-Administrativos/Hotéis, IV/V-Escolas/Hospitais, VI-Espetáculos, VIII-Comércio, XI-Bibliotecas/Arquivos, XII-Oficinas/Armazéns, II-Estacionamento)

### Novos ficheiros necessários no active build
- `app/backend/Chichorro_RI_inter.py` — novo módulo Python
- `app/backend/Flask.py` — novo endpoint `/RI/interv`
- `app/frontend/src/pages/InterventionsPage.tsx` — nova página
- Router — nova rota `/app/intervencoes`

---

## Resumo de impacto por ficheiro

| Ficheiro | Tipo | Ação |
|---|---|---|
| `app/backend/Chichorro_POI.py` | backend | Substituir por 3.1 (add `POI_CC_Idade`) |
| `app/backend/Chichorro_ESCI.py` | backend | Substituir por 3.1 (GP_Auto, EXT_Formacao, RIA_Formacao+CS) |
| `app/backend/Chichorro_DPI.py` | backend | Substituir por 3.1 (OGS redesenhado) |
| `app/backend/Chichorro_RI.py` | backend | Substituir por 3.1 (nova escala + RI_RIA via Idade) |
| `app/backend/Chichorro_RI_inter.py` | backend | Criar (novo módulo intervenções) |
| `app/backend/Flask.py` | backend | Atualizar endpoints afetados + novo `/RI/interv` |
| `app/frontend/.../poiDefinitions.ts` | frontend | Adicionar `POI_CC_Idade` ao subfator CC |
| `app/frontend/.../esciDefinitions.ts` | frontend | GP_Auto; EXT_Formacao + OGS rename; RIA_Formacao+CS + OGS rename |
| `app/frontend/.../dpiDefinitions.ts` | frontend | OGS: substituição completa 7→4 campos |
| `app/frontend/src/pages/RiPage.tsx` | frontend | Nova escala A++..F; remover seletor período |
| `app/frontend/src/pages/InterventionsPage.tsx` | frontend | Criar (nova feature) |

---

## O que requer análise adicional antes de implementar

1. **`Chichorro_CTI.py` 3.1** — confirmar se há alterações vs 3.0 (CPIVHE, CPIVVE)
2. **`Flask.py` 3.1** — ler os payloads completos de cada endpoint para garantir matchup exato
3. **Frontend de intervenções** — definir UX (modo de seleção: conjuntos predefinidos vs individual?)
