# Frontend Guidelines

## Objetivo
Evoluir a app moderna em `app/frontend/` com consistência funcional e estrutural, alinhando-a com os módulos de risco e com as referências v3.0 e v3.1 quando necessário.

## Stack
- React + Vite + TypeScript + Tailwind CSS

Estrutura de ficheiros em `docs/ARCHITECTURE.md`.

## Regras principais

### 1. O frontend ativo vive em `app/frontend/`
Nova implementação deve ser feita aí.

Não desenvolver funcionalidades principais dentro de:
- `reference/chichorro-3.0-jt/`
- `reference/chichorro-3.1-rs/`
- `reference/chichorro-2.0-rf/`

salvo tarefa explícita de análise ou comparação.

### 2. Organização por páginas funcionais
As páginas principais devem refletir os domínios do sistema:
- CTI
- DPI
- ESCI
- POI
- RI

Isto deve continuar a ser a base da navegação e da estrutura mental da app.

### 3. Organização por domínio
Quando um domínio tiver lógica de UI própria, manter o padrão:
- `FactorSection.tsx`
- `definitions.ts`

Exemplos atuais:
- `DpiFactorSection.tsx` + `dpiDefinitions.ts`
- `EsciFactorSection.tsx` + `esciDefinitions.ts`
- `PoiFactorSection.tsx` + `poiDefinitions.ts`

Se CTI e RI precisarem do mesmo padrão, seguir esta convenção.

### 4. Layout, autenticação e sessão
- `AppLayout.tsx` concentra o layout global da aplicação
- `LoginPage.tsx` trata a entrada do utilizador
- `RequireAuth.tsx` protege zonas autenticadas
- `AuthPendingScreen.tsx` trata estados intermédios de autenticação
- `session.ts` deve concentrar gestão de sessão
- `legacyLogin.ts` deve ser encarado como compatibilidade transitória até eventual substituição

### 5. Placeholder
`PlaceholderPage.tsx` deve ser usado apenas como transição.
Não deixar funcionalidades reais escondidas indefinidamente em placeholders.

### 6. Integração com backend
A comunicação com backend deve ser centralizada em `src/lib/` ou numa futura pasta `services/`.
Evitar:
- fetch solto em vários componentes
- duplicação de transformação de payloads
- parsing repetido

### 7. UI reutilizável
- `Button.tsx`, `Card.tsx` e `Field.tsx` devem ser a base dos elementos comuns
- `ModuleGlobalValueCard.tsx` deve ser reutilizado quando fizer sentido apresentar valores agregados por módulo
- evitar recriar variantes paralelas sem necessidade real

### 8. Resultados calculados e estado desatualizado
Quando um input de um fator/módulo já calculado é alterado ou apagado:
- o resultado local antigo deve permanecer visível, mas em cinza translúcido
- deve aparecer aviso âmbar: `Um ou mais valores foram alterados ou apagados. Por favor, volte a calcular.`
- o resultado atual deve ser removido do storage usado por cálculos dependentes
- o último valor global válido deve ser guardado como desatualizado para o `ModuleGlobalValueCard`
- o cartão global deve mostrar o valor antigo em cinza translúcido e a nota `Valor desatualizado`
- ao recalcular com sucesso, o estado desatualizado deve ser limpo

Implementação atual:
- `resultsStore.ts`: `getStaleModuleResults`, `setStaleModuleResult`, `clearStaleModuleResult`
- `ModuleGlobalValueCard.tsx`: renderiza valor atual ou último valor desatualizado
- POI/DPI/ESCI: limpar fator com `clearModuleFactorResult(module, factorKey, { recomputeModule: false })` em alteração de input
- CTI: limpar `module-results.cti` e preservar o último valor válido em `module-results-stale`

## Não fazer
- não copiar HTML/CSS antigo diretamente da referência 3.1 como base do frontend moderno
- não mover lógica técnica de risco para o frontend
- não criar novas páginas fora do padrão modular existente
- não duplicar definitions e fatores em vários sítios
- não perpetuar dependência de `legacyLogin.ts` sem necessidade

## Modos de trabalho

### `active_build`
Editar apenas `app/frontend/`

### `v3_0_legacy_compare`
Usar `reference/chichorro-3.0-jt/` apenas como comparação histórica ou funcional

### `v3_1_matchup`
Ler `reference/chichorro-3.1-rs/` como referência funcional, mas implementar em `app/frontend/`

### `v4_0_research`
Não alterar o frontend ativo sem decisão explícita
