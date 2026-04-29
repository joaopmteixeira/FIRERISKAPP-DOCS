# V3.1 Matchup Matrix

Última atualização: 2026-04-27
Análise detalhada: `docs/migration/V3_1_DIFF_ANALYSIS.md`

## Legenda de estado
- `done` → comportamento já reproduzido e validado
- `partial` → implementação parcial ou sem validação suficiente
- `missing` → não implementado na base ativa
- `unknown` → não analisado
- `analysed` → diferenças identificadas, implementação pendente

## Matriz principal

| Área | Frontend ativo | Backend ativo | Referência v3.1 | Estado | Notas |
|---|---|---|---|---|---|
| App shell / layout | `AppLayout.tsx` | n/a | layout em `reference/chichorro-3.1-rs/` | unknown | Validar menu, ordem dos módulos, navegação pós-login |
| Login / sessão | `LoginPage.tsx` | auth/session | login flow em `reference/chichorro-3.1-rs/` | partial | Rever `RequireAuth.tsx`, `AuthPendingScreen.tsx`, `legacyLogin.ts`, `session.ts` |
| CTI | `CtiPage.tsx` | `Chichorro_CTI.py` | `Chichorro_CTI.py` em referência | done | Backend CTI 3.1 aplicado por merge; frontend usa fallback até expor VHE/VVE separados |
| DPI | `DpiPage.tsx` + `dpiDefinitions.ts` | `Chichorro_DPI.py` | `Chichorro_DPI.py` em referência | done | Backend, Flask e frontend alinhados para OGS 3.1 |
| ESCI | `EsciPage.tsx` + `esciDefinitions.ts` | `Chichorro_ESCI.py` | `Chichorro_ESCI.py` em referência | done | Backend, Flask e frontend alinhados para GP/EXT/RIA 3.1 |
| POI | `PoiPage.tsx` + `poiDefinitions.ts` | `Chichorro_POI.py` | `Chichorro_POI.py` em referência | done | Backend, Flask e frontend alinhados com `POI_CC_Idade` |
| RI | `RiPage.tsx` | `Chichorro_RI.py` | `Chichorro_RI.py` em referência | done | Backend, Flask e frontend alinhados; aceitabilidade vem de `RI_RIA` |
| Intervenções | `InterventionsPage.tsx` | `Chichorro_RI_inter.py` + `/RI/interv` | `Chichorro_RI_inter.py` + `RI_I.js` em referência | done | Backend e frontend implementados; conjuntos extraídos de `RI_I.js` |
| Placeholder / áreas pendentes | `PlaceholderPage.tsx` | n/a | áreas na referência | unknown | Identificar o que falta e eliminar placeholder |
| Route protection | `RequireAuth.tsx` | n/a | comportamento protegido | unknown | Confirmar bloqueio e redirecionamentos |
| Auth pending | `AuthPendingScreen.tsx` | n/a | comportamento equivalente | unknown | Confirmar quando aparece e como resolve |
| Session management | `session.ts` | n/a | persistência/sessão | unknown | Confirmar persistência, restauração, invalidação |
| Legacy login bridge | `legacyLogin.ts` | n/a | login antigo | unknown | Avaliar se ainda é necessário ou é dívida técnica |
| UI base | `Button.tsx`, `Card.tsx`, `Field.tsx`, `ModuleGlobalValueCard.tsx` | n/a | padrões visuais da referência | partial | `ModuleGlobalValueCard` já suporta valor global desatualizado; ainda validar coerência visual geral |

---

## Detalhe por módulo

### POI
- **Estado:** `done`
- **Ficheiros ativos:** `PoiPage.tsx`, `poiDefinitions.ts`, `Chichorro_POI.py`
- **Referência lida:** `reference/chichorro-3.1-rs/backend_reference/Chichorro_POI.py`
- **O que já corresponde:** todos os subfatores existentes (CC, IEE, IA, ICONFA, ICONSA, IVCA, ILGC, EF, EA, FA, PPP, ATIV)
- **Implementado no backend:**
  - `Chichorro_POI.py` substituído pela versão 3.1
  - `POI_CC(...)` e `POI(...)` aceitam `POI_CC_Idade`
  - `POI_EF`: naming das distâncias corrigido — `'<=16m'` → `'<=4m'` em 7 ocorrências
- **Implementado no frontend/Flask:**
  1. `POI_CC_Idade` adicionado ao subfator CC em `poiDefinitions.ts`
  2. Endpoint `/POI/CC` e agregador `/POI` alinhados em `Flask.py`
  3. `POI_EF_Altura`: valores `"Menor9m"`/`"Maior9m"` → `"<=9m"`/`">9m"`; `visibleWhen` exclui UT=XII + CI > 5000
  4. Ordem dos campos POI_EF: Aplica → UT → ElemConstr → CI → Altura → DistEdif
  5. `POI_IA_TipoInst2`: `getOptions` dinâmico filtrado por `TipoInst`

---

### DPI
- **Estado:** `done`
- **Ficheiros ativos:** `DpiPage.tsx`, `DpiFactorSection.tsx`, `dpiDefinitions.ts`, `Chichorro_DPI.py`
- **Referência lida:** `reference/chichorro-3.1-rs/backend_reference/Chichorro_DPI.py`
- **O que já corresponde:** REIC, EI, VDGF, PE — sem alterações em 3.1
- **Implementado no backend:** `DPI_OGS` completamente redesenhado
  - 3.0: 7 campos (`P_Emergencia_*`, `Formacao_Simulacro_*`, `Equipa_*`)
  - 3.1: 4 campos (`DPI_OGS_OGS`, `DPI_OGS_Formacao`, `DPI_OGS_Regulamento`)
  - Lógica de ajuste pós-score: Regulamento afeta score final (-0.1 / -0.2 / 0)
- **Implementado no frontend/Flask:**
  1. Subfator `ogs` substituído em `dpiDefinitions.ts` (7→4 campos)
  2. Endpoints `/DPI/OGS` e `/DPI` alinhados em `Flask.py`

---

### ESCI
- **Estado:** `done`
- **Ficheiros ativos:** `EsciPage.tsx`, `EsciFactorSection.tsx`, `esciDefinitions.ts`, `Chichorro_ESCI.py`
- **Referência lida:** `reference/chichorro-3.1-rs/backend_reference/Chichorro_ESCI.py`
- **O que já corresponde:** SID, AE, HE, CPB — sem alterações em 3.1
- **Implementado no backend:**

  **GP:**
  - Novo campo `ESCI_GP_Auto` (Termovelocimétrico / Ótico / Aspiração)
  - Visível apenas quando `DetAler == 'Automatico'`

  **EXT:**
  - `ESCI_EXT_OGS`: `PP+F` renomeado para `R+PP` (quebra strings existentes)
  - Novo campo `ESCI_EXT_Formacao` (Sem / Com)
  - Assinatura backend muda de 3 para 4 parâmetros

  **RIA:**
  - `ESCI_RIA_OGS`: `PP+F` renomeado para `R+PP`
  - Novo campo `ESCI_RIA_Formacao` (Sem / Com)
  - Novo campo `ESCI_RIA_CS` (RIA / RIA+CS)
  - Assinatura backend muda de 3 para 5 parâmetros

- **Implementado no frontend/Flask:**
  1. Subfatores `gp`, `ext`, `ria` atualizados em `esciDefinitions.ts`
  2. Endpoints `/ESCI/GP`, `/ESCI/EXT`, `/ESCI/RIA` e `/ESCI` alinhados em `Flask.py`

---

### RI
- **Estado:** `done`
- **Ficheiros ativos:** `RiPage.tsx`, `Chichorro_RI.py`
- **Referência lida:** `reference/chichorro-3.1-rs/backend_reference/Chichorro_RI.py`
- **O que já corresponde:** fórmula geral `RI = POI × CTI × ((DPI + ESCI) / 2)`
- **Implementado no backend:**
  - Escala de 12 classes já preservada: A++/A+/A/B+/B/B-/C+/C/C-/D/E/F
  - `POI_CC_Idade` passado para POI e usado para devolver `RI_RIA`
  - Chamadas internas atualizadas para assinaturas 3.1 de POI, DPI e ESCI
  - Chamada CTI preservada com `RI_interv_21_efetivo = 0` e `Dispositivos` como fallback VHE/VVE
- **Implementado no frontend/Flask:**
  1. `RiPage.tsx` removeu o seletor local de período
  2. Aceitabilidade passa a usar `RI_RIA` devolvido pelo backend
  3. Endpoint `/RI` alinhado em `Flask.py`

---

### CTI
- **Estado:** `done`
- **Ficheiros ativos:** `CtiPage.tsx`, `Chichorro_CTI.py`
- **Referência lida:** `reference/chichorro-3.1-rs/backend_reference/Chichorro_CTI.py`
- **O que já corresponde:**
  - Subfatores base presentes: CI, VHE e VVE
  - Tabelas de fatores `CPI_VHE_Fator` e `CPI_VVE_Fator` coincidem com a referência 3.1
  - Revestimentos VHE/VVE (`Melhor`, `Respeita`, `<= 1 Classe`, `> 1 Classe`) mantêm os mesmos valores da referência
  - Fórmulas finais de agregação coincidem por cenário:
    - sem VHE/VVE: `CTI = CPI_CI`
    - só VHE: `CTI = (2 * CPI_CI + CPI_VHE) / 3`
    - só VVE: `CTI = (2 * CPI_CI + CPI_VVE) / 3`
    - VHE + VVE: `CTI = (2 * CPI_CI + (CPI_VHE + CPI_VVE) / 2) / 3`
- **Diferenças implementadas:**
  - Assinatura da função:
    - 3.1: `CTI(..., efetivo, RI_interv_21_efetivo, saidas, Dispositivos, ..., VHE_Dispositivos, ..., VVE_Dispositivos, ...)`
    - antes do merge: `CTI(..., efetivo, saidas, Dispositivos, ..., ...)`
  - Intervenção 21:
    - 3.1: quando `RI_interv_21_efetivo == 1`, reduz `efetivo` para `efetivo / 2` antes de calcular densidade e velocidades no CI
    - ativo: `RI_interv_21_efetivo` aceite e aplicado; endpoint CTI normal defaulta para `0`
  - Dispositivos VHE:
    - 3.1: no cenário só VHE, a velocidade da VHE usa `VHE_Dispositivos`
    - ativo: usa `VHE_Dispositivos`; Flask usa fallback para `Dispositivos` enquanto o frontend não envia o campo
  - Dispositivos VVE:
    - 3.1: no cenário só VVE, a velocidade da VVE usa `VVE_Dispositivos`
    - ativo: usa `VVE_Dispositivos`; Flask usa fallback para `Dispositivos` enquanto o frontend não envia o campo
  - Cenário VHE + VVE:
    - 3.1: a referência usa `VVE_Dispositivos` nas velocidades VHE e VVE deste ramo
    - ativo: segue a referência e usa `VVE_Dispositivos` neste ramo; Flask usa fallback para `Dispositivos` enquanto o frontend não envia o campo
  - Integração ativa:
    - `Flask.py` chama `cti.CTI(...)` com a assinatura 3.1
    - `CtiPage.tsx` ainda só recolhe `Dispositivos` global; `VHE_Dispositivos`/`VVE_Dispositivos` ficam para o bloco frontend
- **Diferenças locais do ativo face à referência literal:**
  - O ativo contém helpers/refatoração para cálculo de fumo e clarabóia, não presentes no ficheiro 3.1 literal
  - O ativo altera parte da lógica de percurso vertical VVE quando não há pisos abaixo, aparentemente para tratar percurso ascendente fora de estacionamento
  - O bug `Symbol`/`solve` sem import foi corrigido com `from sympy import Symbol, solve`
- **Correções aplicadas:**
  1. `Chichorro_CTI.py` aceita `RI_interv_21_efetivo`, `VHE_Dispositivos` e `VVE_Dispositivos`
  2. Refatorações/fixes locais do ativo foram preservados
  3. Endpoints CTI em `Flask.py` aceitam e passam os novos campos
  4. `Chichorro_RI.py` foi ajustado minimamente para continuar a chamar CTI com a assinatura nova até à migração RI 3.1
- **Fica para o bloco frontend CTI:**
  - Expor `VHE_Dispositivos` e `VVE_Dispositivos` em `CtiPage.tsx`

---

### Intervenções
- **Estado:** `done`
- **Referências lidas:** `reference/chichorro-3.1-rs/backend_reference/Chichorro_RI_inter.py`, `reference/chichorro-3.1-rs/frontend_reference/MainPage/script/RI_I.js`
- **Resumo:** 34 intervenções booleanas, organizadas em 6 conjuntos por tipo de utilização (UT)
- **Implementado no backend:**
  1. `app/backend/Chichorro_RI_inter.py` criado a partir da referência 3.1
  2. Endpoint `/RI/interv` adicionado em `Flask.py`
  3. Alias `/RI_interv` mantido para compatibilidade com a referência
- **Implementado no frontend:**
  1. `app/frontend/src/pages/InterventionsPage.tsx` criado
  2. Rota `/app/interventions` adicionada ao router e ao menu
  3. Seleção por conjuntos predefinidos e seleção individual das 34 intervenções
  4. Resultado antes/depois com chamada a `/RI` e `/RI/interv`

---

## Itens transversais

### UX de resultados desatualizados
- **Estado:** `done` para POI/DPI/ESCI/CTI e RI
- **Ficheiros ativos:** `resultsStore.ts`, `ModuleGlobalValueCard.tsx`, `PoiFactorSection.tsx`, `DpiFactorSection.tsx`, `EsciFactorSection.tsx`, `CtiPage.tsx`, `RiPage.tsx`
- **Comportamento implementado (subfatores):**
  1. Alterar/apagar input invalida o resultado calculado atual.
  2. O valor antigo permanece visível no cartão local, mas em cinza translúcido.
  3. Surge aviso âmbar para obrigar o utilizador a recalcular.
  4. O cartão global mantém o último valor válido em cinza translúcido e mostra `Valor desatualizado`.
  5. O valor global atual é removido de `module-results`, evitando consumo por cálculos dependentes.
  6. O último valor válido é guardado separadamente em `chichorro:module-results-stale`.
  7. Recalcular com sucesso limpa o estado desatualizado.
- **Comportamento implementado (RI):**
  1. Quando qualquer input de subfator muda após RI calculado, surge aviso vermelho na RiPage.
  2. Implementado via `SESSION_DATA_UPDATED_EVENT` + `loadingRef` (suprime o evento emitido pelo próprio cálculo).
  3. Aviso limpa ao recalcular RI, importar sessão ou limpar sessão.
- **Validação:** `npm run build` em `app/frontend` passou.

### UX — Colapsar/expandir subfatores
- **Estado:** `done` para POI/DPI/ESCI
- **Ficheiros ativos:** `PoiFactorSection.tsx`, `DpiFactorSection.tsx`, `EsciFactorSection.tsx`, `Card.tsx`
- **Comportamento implementado:**
  1. Botão "Colapsar menu" / "Expandir menu" com chevron animado no `CardHeader.right`
  2. Animação de altura via `grid-rows-[0fr]` / `grid-rows-[1fr]` + `transition-[grid-template-rows]`
  3. Ao receber `highlightKey`, o cartão expande automaticamente e faz scroll até si
  4. Inline styles substituídos por `text-ink-500/55` (Tailwind opacity modifier)

### Autenticação e sessão
- `legacyLogin.ts`, `AuthPendingScreen.tsx`, `session.ts`, `RequireAuth.tsx` — todos `unknown`
- Não são bloqueadores para a migração 3.1 mas são dívida técnica a endereçar depois

### Integração frontend-backend (lacuna documental)
Ainda falta mapear por endpoint:
- payload de entrada exato
- payload de saída exato
- validações e estados de erro

---

## Análise de lacunas 3.1 (2026-04-27)

Revisão completa da dissertação de Rui Sobral (secções 3.4 e 7.2) e dos ficheiros de referência.

**Resultado:** B10 (VHE_Dispositivos / VVE_Dispositivos no frontend) é a **única** lacuna de modelo do 3.1. Todos os outros 7 itens da secção 3.4 estão implementados. Os itens da secção 7.2 são desenvolvimentos futuros propostos por Rui Sobral — fora do âmbito 3.1.

## Próximas ações recomendadas (ordem)
1. ⏳ Expor `VHE_Dispositivos` e `VVE_Dispositivos` em `CtiPage.tsx` (B10) — única lacuna 3.1 restante
2. ✅ `parity_runner.py` — 11/11 checks, 0 falhas
3. Fazer teste end-to-end com um caso conhecido da referência 3.1
