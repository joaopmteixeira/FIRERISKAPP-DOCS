# Frontend UX Modifications

Registo cronológico de todas as alterações de UX/UI significativas no frontend.
Deve ser atualizado sempre que houver alterações relevantes de experiência de utilizador.

Última atualização: 2026-04-29

---

## UX1 — Validação e invalidação de resultados nos subfatores

**Ficheiros:**
- `app/frontend/src/components/poi/PoiFactorSection.tsx`
- `app/frontend/src/components/dpi/DpiFactorSection.tsx`
- `app/frontend/src/components/esci/EsciFactorSection.tsx`
- `app/frontend/src/components/ui/ModuleGlobalValueCard.tsx`
- `app/frontend/src/lib/resultsStore.ts`

**Descrição:**
- Campos obrigatórios em falta ao clicar "Calcular" mostram mensagem de erro específica: `Preenche "<campo>" antes de calcular.`
- O input em falta recebe destaque visual com `ring-2 ring-red-400`
- Quando um campo já calculado é alterado, o resultado local mantém o valor antigo mas fica em cinza translúcido (`isResultStale`)
- Aviso âmbar aparece: `Um ou mais valores foram alterados ou apagados. Por favor, volte a calcular.`
- O card global do módulo (canto superior direito) mantém o último valor válido em cinza e mostra `Valor desatualizado`
- Ao recalcular com sucesso, todos os estados de stale são limpos

---

## UX2 — Aviso de RI desatualizado na RiPage

**Ficheiro:** `app/frontend/src/pages/RiPage.tsx`

**Descrição:**
- Quando qualquer input de subfator é alterado após um cálculo de RI, aparece aviso vermelho: `Os dados de entrada foram alterados desde o último cálculo. O valor de RI apresentado pode não estar atualizado — recalcule para obter o resultado correto.`
- Implementado via `SESSION_DATA_UPDATED_EVENT` + `loadingRef` (suprime o evento durante o próprio cálculo)
- Aviso limpa ao recalcular RI com sucesso, ao importar sessão ou ao limpar sessão

---

## UX3 — Colapsar/expandir subfatores em POI, DPI e ESCI

**Ficheiros:**
- `app/frontend/src/components/poi/PoiFactorSection.tsx`
- `app/frontend/src/components/dpi/DpiFactorSection.tsx`
- `app/frontend/src/components/esci/EsciFactorSection.tsx`
- `app/frontend/src/components/ui/Card.tsx`

**Descrição:**
- Botão "Colapsar menu" / "Expandir menu" com chevron animado adicionado ao cabeçalho de cada card de subfator via `CardHeader.right`
- Animação de altura com `grid-rows-[0fr]` / `grid-rows-[1fr]` + `transition-[grid-template-rows]` (sem `max-height` fixo)
- Ao receber `highlightKey` (deep link da RiPage), o card expande automaticamente e faz scroll até si

---

## UX4 — Persistência do estado colapsado entre navegações

**Ficheiros:**
- `app/frontend/src/components/poi/PoiFactorSection.tsx`
- `app/frontend/src/components/dpi/DpiFactorSection.tsx`
- `app/frontend/src/components/esci/EsciFactorSection.tsx`

**Descrição:**
- O estado colapsado/expandido de cada subfator é persistido em `sessionStorage` com chave `collapse:{formKey}`
- O estado é lido no inicializador lazy do `useState` → survive navigation (desmontagem/remontagem do componente)
- Exemplo de chave: `collapse:poi:iee`

---

## UX5 — Melhorias UX na RiPage (notice fade + gate de erros)

**Ficheiro:** `app/frontend/src/pages/RiPage.tsx`

**Descrição:**
- **Aviso de sucesso com fade:** a box verde ("Sessão limpa com sucesso", "Sessão importada com sucesso") aparece após os botões de ação e desaparece automaticamente com fade de 500ms após 3 segundos (2 timers: `fadeTimer` a 3s, `clearTimer` a 3.5s)
- **Gate de erros:** as boxes de aviso de "módulos em falta" e "inputs em falta" só aparecem depois de o utilizador clicar "Calcular RI" pela primeira vez (`hasAttemptedCalculate`), evitando exibição prematura ao entrar na página
- O gate é resetado ao importar ou limpar sessão

---

## UX6 — Persistência de erros/avisos entre navegações nos subfatores

**Ficheiros:**
- `app/frontend/src/components/poi/PoiFactorSection.tsx`
- `app/frontend/src/components/dpi/DpiFactorSection.tsx`
- `app/frontend/src/components/esci/EsciFactorSection.tsx`

**Descrição:**
- Os estados `error`, `warning`, `missingFieldKey` e `isResultStale` são agora persistidos em `sessionStorage` com chaves prefixadas: `err:`, `warn:`, `miss:`, `stale:`
- Inicialização lazy lê do sessionStorage → ao regressar à página o utilizador vê o mesmo estado que deixou
- `useEffect` por estado garante persistência imediata a cada mudança
- `clearAll()` e `calculate()` limpam estes estados (via setters que disparam os effects)
- `clearComputedResult(resultKey)` foi movido do `setField` para o `catch` do `calculate()`: o resultado antigo permanece em storage para ser exibido como stale ao regressar

---

## UX7 — Warning no subfator ao limpar + ERRO na RiPage

**Ficheiros:**
- `app/frontend/src/components/poi/PoiFactorSection.tsx`
- `app/frontend/src/components/dpi/DpiFactorSection.tsx`
- `app/frontend/src/components/esci/EsciFactorSection.tsx`
- `app/frontend/src/pages/RiPage.tsx`

**Descrição:**
- `clearAll()` em qualquer subfator:
  1. Guarda o valor atual do módulo como stale (`setStaleModuleResult`) antes de limpar
  2. Usa `{ recomputeModule: false }` → o módulo fica `undefined` em vez de ser recalculado com dados parciais
  3. Mostra aviso âmbar: `"<Título do subfator> por preencher. Por favor, preencha corretamente e volte a calcular."`
- **RiPage:** o card de cada módulo (POI/DPI/ESCI) mostra `ERRO` com fundo vermelho quando `stored.X === undefined && staleResults.X !== undefined`
- Banner vermelho por módulo: `"Erro encontrado em [POI/DPI/ESCI]. Reveja a página correspondente."`
- Ao recalcular o subfator com sucesso, `setModuleResult` limpa o stale → card e banner voltam ao normal automaticamente

---

## UX8 — Auto-atualização de resultados na RiPage + remoção do botão "Atualizar resultados"

**Ficheiro:** `app/frontend/src/pages/RiPage.tsx`

**Descrição:**
- `onSessionUpdated` (listener de `SESSION_DATA_UPDATED_EVENT`) passa a chamar `setStored(getModuleResults())` e `setStaleResults(getStaleModuleResults())` → os cards POI/CTI/DPI/ESCI da RiPage atualizam em tempo real sem interação do utilizador
- Botão "Atualizar resultados" removido (era redundante)
- `staleResults` propagado a `handleImportFile` e `confirmClearSession` para resetar corretamente após essas ações
- Layout dos botões simplificado: `flex-wrap gap-3` com "Limpar sessão" alinhado à direita via `ml-auto`
