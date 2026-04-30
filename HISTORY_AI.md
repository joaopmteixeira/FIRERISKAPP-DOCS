# HISTORY_AI — Registo de Interações com IA

Registo cronológico das principais interações com agentes de IA (Claude, Codex) neste projeto.
Cada entrada cobre o objetivo, o que foi pedido, o que foi feito e a decisão resultante.

---

## 2026-04-18 — Setup inicial e definição de arquitetura

**Objetivo:** Estabelecer a estrutura conceptual do projeto e definir como trabalhar com o repositório.

**Prompt:** Análise da estrutura do repositório e definição dos modos de trabalho.

**Resposta:** Identificação das 4 zonas do repositório e criação dos docs base (`PROJECT_OVERVIEW.md`, `ARCHITECTURE.md`, guidelines).

**Resultado:** Estrutura conceptual definida, ficheiros de contexto criados.

**Decisões:**
- `app/frontend/` + `app/backend/` = base ativa; as pastas `reference/` são só leitura
- 4 modos de trabalho: `active_build`, `v3_0_legacy_compare`, `v3_1_matchup`, `v4_0_research`
- Padrão `FactorSection.tsx` + `definitions.ts` por domínio a preservar e expandir
- `legacyLogin.ts` e `AuthPendingScreen.tsx` tratados como compatibilidade transitória

---

## 2026-04-20 — A1: Análise CTI 3.0 → 3.1

**Objetivo:** Comparar `Chichorro_CTI.py` ativo com a referência 3.1 para identificar diferenças antes de qualquer implementação.

**Prompt:** Lê ambos os ficheiros CTI e produz relatório de diferenças. Não implementes nada.

**Resposta (Codex):** Relatório com 5 diferenças encontradas: nova assinatura (3 parâmetros novos), lógica `RI_interv_21_efetivo`, `VHE_Dispositivos`/`VVE_Dispositivos` por cenário, e bug crítico `sympy` sem import.

**Resultado:** `V3_1_MATCHUP_MATRIX.md` atualizado; estado CTI de `unknown` para `analysed`.

**Decisões:**
- CTI não pode ser substituição direta — requer merge (tem refactors locais: fumo, clarabóia, VVE ascendente)
- Bug sympy é crítico de produção — corrigir no mesmo handoff da implementação
- `RI_interv_21_efetivo` default `0` no fluxo normal; não é input do utilizador

---

## 2026-04-20 — A2: Implementação CTI 3.1

**Objetivo:** Atualizar `Chichorro_CTI.py` com a assinatura 3.1, preservando os refactors locais.

**Prompt:** Aplica as 5 diferenças identificadas em A1. Merge, não substituição. Corrige o bug sympy.

**Resposta (Codex):** CTI atualizado com nova assinatura, `RI_interv_21_efetivo`, `VHE_Dispositivos`/`VVE_Dispositivos`, bug sympy corrigido, refactors preservados. Bonus: `Chichorro_RI.py` também atualizado (escala 3.1 + CTI call corrigida).

**Resultado:** Backend CTI 3.1 funcional. Parity 11/11.

**Decisões:**
- `VHE_Dispositivos`/`VVE_Dispositivos` são inputs distintos por veia (confirmado na referência Flask 3.1)
- `Flask.py` recebeu fallback temporário `VHE_Dispositivos = VHE_Dispositivos or Dispositivos` (a remover quando o frontend expuser os campos — B10)

---

## 2026-04-20 — A3: Backend batch — POI + ESCI + DPI + RI

**Objetivo:** Substituir os 3 módulos restantes e completar `Chichorro_RI.py`.

**Prompt:** Substituição direta de POI/ESCI/DPI pela referência 3.1. Merge de RI (escala e CTI call já estão corretos; adicionar POI_CC_Idade, DPI_OGS 4 campos, ESCI novos parâmetros).

**Resposta (Codex):** 4 módulos Python atualizados. Imports OK. `RI_RIA` calculado e no dict de retorno.

**Resultado:** Backend 3.1 completo.

**Decisões:**
- POI/ESCI/DPI: substituição direta (sem refactors locais a preservar)
- `RI_RIA` incluído no dict de retorno do `RI()` — frontend vai ler daí (sem seletor local na RiPage)
- `DPI_OGS`: 7 campos → 4; valores OGS de `PP+F` → `R+PP` (quebra sessões 3.0)
- `ESCI_GP_Auto`, `ESCI_EXT_Formacao`, `ESCI_RIA_Formacao`, `ESCI_RIA_CS` adicionados

---

## 2026-04-20 — A4: Flask.py + Chichorro_RI_inter.py

**Objetivo:** Criar `Chichorro_RI_inter.py` e atualizar todos os endpoints de `Flask.py` para assinaturas 3.1.

**Prompt:** Cria `Chichorro_RI_inter.py` (cópia direta da referência). Atualiza endpoints `/POI`, `/POI/CC`, `/DPI`, `/DPI/OGS`, `/ESCI`, `/ESCI/GP`, `/ESCI/EXT`, `/ESCI/RIA`, `/RI`. Adiciona `/RI/interv`. Flask.py é merge — não tocar em auth routes nem em `_normalize_cti_payload`.

**Resposta (Codex):** `Chichorro_RI_inter.py` criado. Flask.py atualizado com todos os endpoints 3.1. `/RI/interv` adicionado.

**Resultado:** Backend 3.1 totalmente funcional. Parity runner: 11/11.

**Decisões:**
- `Flask.py` é merge — as rotas de auth e o helper `_normalize_cti_payload` mantêm-se
- `/RI/interv` recebe `RI_interv_21_efetivo` como input (não defaultado a 0 aqui — o módulo de intervenções pode modificar o efetivo)

---

## 2026-04-21 — B1–B4: Frontend definitions + RiPage

**Objetivo:** Alinhar o frontend com as assinaturas backend 3.1.

**Prompt:** Atualizar `poiDefinitions.ts` (B1), `esciDefinitions.ts` (B2), `dpiDefinitions.ts` (B3) e `RiPage.tsx` (B4) em batch.

**Resposta (Codex):**
- B1: `POI_CC_Idade` adicionado ao subfator CC
- B2: `ESCI_GP_Auto`, `ESCI_EXT_Formacao`, `ESCI_RIA_Formacao`, `ESCI_RIA_CS` adicionados; OGS values atualizados (`R+PP`)
- B3: `DPI_OGS` substituído de 7 para 4 campos
- B4: seletor de período removido; `RI_RIA` lido do backend; escala 3.1 (A++…F)

**Resultado:** Build TypeScript sem erros.

**Decisões:**
- `POI_CC_Idade` consolida-se no subfator CC do POI (removido da RiPage local)
- `ESCI_GP_Auto` value `"Aspiracao"` sem acento (corresponde ao backend)

---

## 2026-04-21 — B5: InterventionsPage.tsx

**Objetivo:** Criar nova página de intervenções de redução de risco.

**Prompt:** Cria `InterventionsPage.tsx` com 34 intervenções, conjuntos predefinidos por UT (lidos de `RI_I.js`), seleção individual, botão calcular via `/RI/interv`, dois painéis de resultado (antes/após), custo estimado.

**Resposta (Codex):** Página criada com seleção individual + conjuntos predefinidos. Rota `/app/interventions` registada.

**Resultado:** Módulo de intervenções funcional.

**Decisões:**
- Ambos os modos implementados em paralelo (seleção individual + conjuntos), como na referência 3.1
- `VHE_Dispositivos`/`VVE_Dispositivos` continuam com fallback para `Dispositivos` até B10

---

## 2026-04-22 — B7: Correções POI_EF e POI_IA

**Objetivo:** Corrigir inconsistências nos subfatores POI_EF e POI_IA identificadas em teste de uso.

**Prompt:** Corrigir valores de `POI_EF_Altura`, ordem de campos, `visibleWhen` de Altura para UT=XII + CI > 5000, e `POI_IA_TipoInst2` com opções dinâmicas por `TipoInst`.

**Resposta:** Frontend e backend atualizados em simultâneo.

**Resultado:** Build OK.

**Decisões:**
- `POI_EF_Altura` usa notação `"<=9m"` / `">9m"` (coerência com restantes campos de distância)
- Ordem dos campos POI_EF: Aplica → UT → ElemConstr → CI → Altura → DistEdif (reflete dependência lógica)
- `POI_IA_TipoInst2` usa `getOptions` dinâmico filtrado por `TipoInst` (opções irrelevantes ocultadas)
- `POI_EF_Altura.visibleWhen` exclui UT=XII + CI > 5000 (tabela 3.1 não discrimina por altura neste ramo)

---

## 2026-04-22 — B8: Sync bidirecional CTI↔POI_ATIV

**Objetivo:** Corrigir deadlock no sync do campo `TipoEdif` entre CTI e POI_ATIV após import de sessão.

**Prompt:** Corrigir comportamento em que `syncTipoEdifFromCti` e `syncTipoEdifFromPoi` escreviam ambos em `moduleInputs`, bloqueando o campo após import.

**Resposta:** `syncTipoEdifFromCti` passa a atualizar apenas o form (display) sem escrever em `moduleInputs.poi`. `clearAll` em CTI também limpa `POI_ATIV_TipoEdif`. Campo `POI_ATIV_TipoEdif` nunca fica bloqueado.

**Resultado:** Sync funcional sem deadlock.

**Decisão:** Sync é unidirecional a nível de `moduleInputs` — apenas o clique explícito ou "Calcular" escreve em `moduleInputs`.

---

## 2026-04-22 — UX1–UX3: Validação, stale results e colapso de subfatores

**Objetivo:** Melhorar UX de feedback ao utilizador em subfatores POI/DPI/ESCI.

**Prompt:** (1) Campos obrigatórios em falta → erro específico + destaque visual. (2) Input alterado após cálculo → resultado antigo em cinza + aviso âmbar. (3) Botão colapsar/expandir em cada cartão de subfator com animação.

**Resposta:** Implementados em `PoiFactorSection.tsx`, `DpiFactorSection.tsx`, `EsciFactorSection.tsx`, `ModuleGlobalValueCard.tsx`, `resultsStore.ts`, `Card.tsx`.

**Resultado:** Build OK.

**Decisões:**
- `clearModuleFactorResult(..., { recomputeModule: false })` impede recomputação parcial ao alterar inputs
- Estado desatualizado global em store separado (`chichorro:module-results-stale`)
- Animação de colapso via `grid-rows-[0fr]`/`grid-rows-[1fr]` + `transition-[grid-template-rows]` (sem `max-height` fixo)
- Ao receber `highlightKey` (deep link da RiPage), o cartão expande automaticamente

---

## 2026-04-22 — UX4–UX5: Persistência de colapso + gate de erros na RiPage

**Objetivo:** Persistir estado colapsado entre navegações; melhorar UX de avisos na RiPage.

**Prompt:** (1) Estado colapsado/expandido persistido em sessionStorage. (2) Aviso de sucesso (importar/limpar sessão) com fade automático. (3) Gate: avisos de falta de módulos/inputs só aparecem após primeiro clique em "Calcular RI".

**Resposta:** Implementado com chaves `collapse:{formKey}` em sessionStorage. Fade com dois timers (3s fade, 3.5s clear). Gate `hasAttemptedCalculate`.

**Resultado:** UX mais limpa na RiPage; estados preservados entre navegações.

---

## 2026-04-29 — UX6: Persistência de erros/avisos entre navegações nos subfatores

**Objetivo:** Erros e avisos nos subfatores (campo em falta, resultado stale) desapareciam ao navegar para outra página e voltar.

**Prompt:** Screenshot demonstrando o problema. Pedir que os estados `error`, `warning`, `missingFieldKey`, `isResultStale` sobrevivam a desmontagem/remontagem do componente.

**Resposta:** Estados persistidos em sessionStorage com chaves `err:`, `warn:`, `miss:`, `stale:`. Inicialização lazy lê do sessionStorage. `useEffect` persiste cada mudança. `clearComputedResult` movido do `setField` para o `catch` do `calculate()`.

**Resultado:** Estados preservados entre navegações. Build OK.

**Decisão:** `clearComputedResult(resultKey)` só corre no `catch` — o resultado antigo permanece em storage para ser exibido como stale ao regressar.

---

## 2026-04-29 — UX7: Aviso ao limpar subfator + ERRO na RiPage

**Objetivo:** Quando se clica "Limpar" num subfator, não aparecia nenhum aviso nem o card global ficava desatualizado. Na RiPage, não aparecia indicação de erro.

**Prompt:** (1) Limpar subfator → aviso âmbar na secção + card global mostra stale. (2) RiPage → card do módulo mostra "ERRO" em vermelho quando o módulo foi limpo após cálculo. (3) Banner vermelho por módulo com erro.

**Resposta:** `clearAll()` atualizado em POI/DPI/ESCI para: guardar stale antes de limpar, usar `{ recomputeModule: false }`, mostrar aviso âmbar. `RiPage.tsx` com `moduleErrors` (deteta `stored.X === undefined && staleResults.X !== undefined`), `ResultCard` com `hasError`, banners vermelhos por módulo.

**Resultado:** Build OK.

**Decisão:** A distinção "nunca calculado" (mostra "-") vs "limpo após cálculo" (mostra "ERRO") usa o mecanismo stale — se `stored.X === undefined && staleResults.X !== undefined`, é um ERRO.

---

## 2026-04-29 — UX8: Auto-atualização + remoção do botão "Atualizar resultados"

**Objetivo:** O botão "Atualizar resultados" na RiPage era redundante.

**Prompt:** Confirmar o que o botão fazia; remover se redundante; garantir que os cards da RiPage atualizam automaticamente.

**Resposta:** `onSessionUpdated` (listener de `SESSION_DATA_UPDATED_EVENT`) passou a chamar `setStored` e `setStaleResults` diretamente. Botão removido. Layout simplificado com `ml-auto` em "Limpar sessão".

**Resultado:** Build OK.

---

## 2026-04-29 — Criação de FRONTEND_UX_MODIFICATIONS.md + limpeza de NEXT_STEPS.md

**Objetivo:** Separar o registo de alterações UX/UI dos ficheiros de planeamento de tarefas.

**Prompt:** Cria `docs/FRONTEND_UX_MODIFICATIONS.md` com todas as UX/UI. Remove UX de `NEXT_STEPS.md`. Analisa `docs/planning/` para duplicados.

**Resposta:** `FRONTEND_UX_MODIFICATIONS.md` criado com UX1–UX8 cronológicos. `NEXT_STEPS.md` limpo. `docs/planning/` identificado como redundante face a `PROJECT_OVERVIEW.md`.

**Resultado:** Docs mais organizados. UX1–UX8 num único ficheiro de referência.

---

## 2026-04-29 — Reestruturação completa da pasta docs/

**Objetivo:** Eliminar duplicados, definir propósito único por ficheiro, limpar pasta `docs/ai/` (caminhos errados em `prompt.md`).

**Prompt:** Propõe reestruturação; avança com as alterações + compila informações duplicadas; renomeia `tmp_ch3.txt` para `tese_extract_3.0.txt` e move para `docs/research/`.

**Resposta:**
- `PROJECT_OVERVIEW.md` reescrito: modelo matemático + escala 3.1 + fases (com autores RF/Bruno Silva) + deploy + princípios
- `ARCHITECTURE.md` reescrito: árvore de ficheiros atualizada + padrão FactorSection + tabela sessionStorage keys + endpoints Flask
- `FRONTEND_GUIDELINES.md` / `BACKEND_GUIDELINES.md`: removidas listagens de ficheiros (passaram para ARCHITECTURE)
- `docs/ai/prompt.md`: caminhos corrigidos (`/docs/ai/X.md` → `/docs/X.md`)
- `docs/planning/` (3 ficheiros): eliminada — conteúdo consolidado em `PROJECT_OVERVIEW.md` e `NEXT_STEPS.md`
- `tese_extract_3.0.txt` movido para `docs/research/`

**Resultado:** Docs sem redundâncias. Cada ficheiro com propósito único e claro.

**Decisão:** `docs/planning/` eliminada definitivamente. `PROJECT_OVERVIEW.md` é o "o quê/porquê/estado". `ARCHITECTURE.md` é o "como o código está organizado". `NEXT_STEPS.md` é o "o que está a ser feito".

---

## 2026-04-30 — B10: Expor VHE_Dispositivos e VVE_Dispositivos no formulário CTI

**Objetivo:** O backend CTI 3.1 aceita `VHE_Dispositivos` e `VVE_Dispositivos` como parâmetros distintos, mas o frontend só enviava `Dispositivos` para todas as vias — resultado incorreto em edifícios com dispositivos diferentes por veia.

**Prompt:** Apresenta o plano, valida-o, avança. Remover o fallback do Flask.py (app nova, sem sessões legadas).

**Resposta:**
- `CtiValues`: adicionados `VHE_Dispositivos` e `VVE_Dispositivos`
- `defaultValues`: ambos inicializados com `""`
- `normalizeSavedValues`: ambos normalizados com `dispositivosValues`, condicionados a `Aplica_VHE/VVE === "Existe"`
- UI: `SelectField` para `VHE_Dispositivos` dentro da secção VHE (visível quando `showVHE`); idem `VVE_Dispositivos` em VVE
- `validate()`: validação adicionada nos blocos `if (showVHE)` e `if (showVVE)`
- Payload `onCalculate`: inclui ambos os campos
- `Flask.py`: fallback `VHE_Dispositivos = VHE_Dispositivos or Dispositivos` removido

**Resultado:** Build OK. B10 concluído. Fase 2 implementação 100% completa.

**Decisão:** Sem fallback no Flask — app nova não tem sessões legadas. Frontend é a única fonte de verdade para estes campos.

---

## 2026-04-30 — Setup de workflow e memória

**Objetivo:** Formalizar procedimentos de trabalho para sessões futuras.

**Prompt:** (1) Fala sempre em PT-PT. (2) Commits sempre em inglês. (3) Após cada prompt com alterações: atualiza os docs `.md` relevantes e apresenta título do commit.

**Resultado:** Memórias gravadas. Procedimento ativo a partir desta sessão.
