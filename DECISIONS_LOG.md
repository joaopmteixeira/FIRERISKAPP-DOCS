# Decisions Log

## 2026-04-22 — Correções POI_EF / POI_IA, sync CTI↔POI_ATIV e aviso RI desatualizado

Decisão: valores de `POI_EF_Altura` usam notação `"<=9m"` / `">9m"` em vez de `"Menor9m"` / `"Maior9m"`.
Razão: consistência com os restantes campos de distância que usam operadores `<=`/`>`.

Decisão: ordem dos campos no subfator POI_EF alterada para: Aplicabilidade → UT → ElemConstr → CI → Altura → DistEdif.
Razão: reflete a dependência lógica — CI e Altura condicionam DistEdif; Altura depende de ElemConstr que depende de Aplica.

Decisão: `POI_IA_TipoInst2` passa de opções estáticas para `getOptions` dinâmico filtrado por `TipoInst`.
Razão: os subtipos só fazem sentido no contexto do tipo de instalação selecionado; apresentar opções irrelevantes induzia erro de preenchimento.

Decisão: `POI_EF_Altura.visibleWhen` exclui o caso UT=XII + CI > 5000.
Razão: quando UT=XII e CI > 5000, a tabela de referência 3.1 não discrimina por Altura — o campo é semanticamente irrelevante neste ramo.

Decisão: o sync CTI↔POI_ATIV para `TipoEdif` é unidirecional ao nível de `moduleInputs`.
Razão: antes, `syncTipoEdifFromCti` e `syncTipoEdifFromPoi` escreviam ambos em `moduleInputs`, criando um deadlock após import de sessão — nenhum dos campos ficava editável. A correção: `syncTipoEdifFromCti` atualiza apenas o form (display) sem escrever em `moduleInputs.poi`; só o clique explícito ou o botão "Calcular" escreve em `moduleInputs`.

Decisão: o botão "Limpar" de POI ATIV e CTI limpa ambos os lados da sincronização (moduleInputs + form).
Razão: sem isso, limpar um lado deixava o outro com o valor antigo, que na reativação seguinte causava re-lock do campo.

Decisão: quando qualquer input de subfator é alterado e já existe um resultado de RI calculado, aparece um aviso vermelho na RiPage a indicar que o RI pode não estar atualizado.
Razão: o utilizador pode alterar inputs em POI/CTI/DPI/ESCI sem recalcular o RI, tornando o valor apresentado desatualizado sem aviso visual. A implementação usa `SESSION_DATA_UPDATED_EVENT` com um `loadingRef` para suprimir o evento durante o próprio cálculo do RI.

## 2026-04-22 — UX de invalidação de resultados calculados

Decisão: ao alterar ou apagar qualquer input de POI, DPI, ESCI ou CTI, o resultado calculado anteriormente deixa de ser considerado atual, mas continua visível no frontend em cinza translúcido.
Razão: apagar o valor antigo remove contexto ao utilizador; manter o valor com estado visual desatualizado deixa claro que existe um cálculo anterior, mas obriga a recalcular antes de confiar nele.

Decisão: os cartões globais `ModuleGlobalValueCard` também devem mostrar estado desatualizado.
Razão: quando um subfator é alterado, o valor global do módulo deixa de representar os inputs atuais. O cartão global mantém o último valor válido em cinza translúcido e mostra `Valor desatualizado`.

Decisão: o estado desatualizado global fica num store separado em `resultsStore.ts` (`chichorro:module-results-stale`), em vez de permanecer em `module-results`.
Razão: isto evita que módulos dependentes consumam um resultado inválido como se fosse atual, mas permite ao UI renderizar o último valor válido para orientação visual.

Decisão: `clearModuleFactorResult(..., { recomputeModule: false })` é usado durante alterações de inputs em POI/DPI/ESCI.
Razão: recomputar automaticamente a média global com um subfator removido criaria um valor parcial potencialmente enganador. O estado correto após alteração é “desatualizado” até novo cálculo.

## 2026-04-20 — Análise 3.0 → 3.1 (v3_1_matchup)

Decisão: `POI_CC_Idade` é um novo campo em 3.1 que não altera o score do subfator CC mas é usado pela função RI() para calcular o limite de aceitabilidade `RI_RIA`.
Razão: o campo já existe na app ativa (em RiPage.tsx como seletor separado), mas em 3.1 pertence conceptualmente ao subfator CC do POI. A migração deve consolidá-lo aí.

Decisão: `ESCI_EXT_OGS` e `ESCI_RIA_OGS` têm valores renomeados em 3.1.
Razão: `PP+F` passa a `R+PP`. Não é só cosmético — os valores das strings mudam, o que quebra endpoints existentes se não forem atualizados em simultâneo no frontend e backend.

Decisão: `DPI_OGS` é uma substituição completa de 7 campos por 4 em 3.1.
Razão: o modelo simplificou o subfator OGS. Os campos antigos (`P_Emergencia_Exigencia`, etc.) desaparecem por completo. A lógica de cálculo é totalmente nova, incluindo um ajuste pós-score baseado em `DPI_OGS_Regulamento`.

Decisão: a escala RI passa de 6 classes (A1/A2/B/C/D/E) para 12 classes (A++/A+/A/B+/B/B-/C+/C/C-/D/E/F) em 3.1.
Razão: a referência `reference/chichorro-3.1-rs/Chichorro_RI.py` e `Chichorro_RI_inter.py` confirmam a nova escala. A mudança é de modelo, não estética.

Decisão: o Módulo de Intervenções é uma feature nova de raiz em 3.1 — não é uma extensão de algo existente.
Razão: `Chichorro_RI_inter.py` é um ficheiro independente que recebe todos os parâmetros do modelo + 34 flags booleanas, modifica os parâmetros conforme as intervenções selecionadas, e recalcula o RI completo. Requer novo endpoint Flask, novo módulo backend e nova página frontend.

Decisão: as alterações backend 3.0→3.1 devem ser feitas substituindo os `Chichorro_*.py` ativos, não criando versões paralelas.
Razão: o contrato de API deve evoluir (não bifurcar). A paridade pode ser validada com `parity_runner.py` após a substituição.

Decisão: `Chichorro_CTI.py` 3.1 não pode ser substituído diretamente — requer merge.
Razão: o ficheiro ativo tem refactors locais (fumo, clarabóia, VVE ascendente) ausentes na referência 3.1. A substituição direta apagaria correções funcionais. A estratégia é aplicar a nova assinatura 3.1 preservando os refactors do ativo.

Decisão: `RI_interv_21_efetivo` não é um input do utilizador no fluxo CTI normal.
Razão: é um flag injetado exclusivamente pelo módulo de intervenções (intervenção 21 = redução de efetivo). No endpoint CTI padrão, deve defaultar a `0`. Não deve aparecer no formulário `CtiPage.tsx`.

Decisão: o bug `sympy` no `Chichorro_CTI.py` ativo deve ser corrigido no mesmo handoff da atualização CTI 3.1.
Razão: `Symbol`/`solve` são chamados sem `import sympy` no ramo `SistemaExtincao == 'Com'`. Gera `NameError` em runtime para qualquer sessão com sistema de extinção ativo. É um bug crítico de produção.

## 2026-04-21 — Decisões UX frontend (v3_1_matchup)

Decisão: `POI_CC_Idade` move para o subfator CC do POI e é removido da RiPage (Opção A).
Razão: a referência 3.1 coloca o campo dentro do subfator CC. O backend calcula `RI_RIA` e devolve-o no response dict. Manter na RiPage criaria inconsistência e dívida técnica. As sessões 3.0 já ficam inválidas pelo rename dos OGS do ESCI.

Decisão: a página de Intervenções implementa ambos os modos em simultâneo — seleção individual das 34 intervenções E conjuntos predefinidos (1-6), como a referência 3.1.
Razão: a referência 3.1 já implementa os dois modos em paralelo. Não há razão para limitar.

---

## 2026-04-18
Decisão: `app/frontend/` é a base ativa principal da interface moderna.
Razão: já possui estrutura modular por páginas, rotas, autenticação, componentes de domínio e UI reutilizável.

Decisão: `app/backend/` é a base ativa principal do backend e da lógica técnica.
Razão: já possui separação clara por módulos CTI, DPI, ESCI, POI e RI.

Decisão: `reference/chichorro-3.0-jt/` é a referência legacy completa da v3.0.
Razão: serve de comparação histórica, funcional e de rastreabilidade.

Decisão: `reference/chichorro-3.1-rs/` é uma referência funcional viva para implementação da v3.1.
Razão: será necessária para matchup entre a stack moderna e o comportamento esperado da próxima fase.

Decisão: `reference/chichorro-2.0-rf/` é uma referência metodológica futura para a v4.0.
Razão: ajudará a recriar ou reinterpretar um método simplificado + completo.

Decisão: o projeto deve distinguir explicitamente quatro modos de trabalho:
- `active_build`
- `v3_0_legacy_compare`
- `v3_1_matchup`
- `v4_0_research`
Razão: evita confusão entre app ativa, referências funcionais e investigação futura.

Decisão: o padrão de componentes por domínio deve ser preservado.
Exemplos:
- `FactorSection.tsx`
- `definitions.ts`
Razão: melhora consistência e facilita expansão para novos módulos.

Decisão: a camada de UI reutilizável deve servir de base comum.
Exemplos:
- `Button.tsx`
- `Card.tsx`
- `Field.tsx`
- `ModuleGlobalValueCard.tsx`
Razão: evita duplicação visual e estrutural.

Decisão: o fluxo de autenticação atual inclui componentes transitórios/legacy.
Exemplos:
- `legacyLogin.ts`
- `AuthPendingScreen.tsx`
Razão: isso deve ser tratado como compatibilidade temporária e não como arquitetura final desejada.
