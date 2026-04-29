# Page ↔ Backend Mapping

## Objetivo
Este ficheiro serve como mapa operacional rápido entre:
- páginas ativas do frontend
- módulos do backend
- auth/rotas relacionadas
- referências a consultar durante matchup e comparação

É mais direto e prático do que a matriz de matchup.

---

## 1. App shell e autenticação

### `app/frontend/src/pages/AppLayout.tsx`
**Papel**
- layout principal da aplicação
- estrutura de navegação interna
- shell após login

**Relacionados**
- `app/frontend/src/routes/RequireAuth.tsx`
- `app/frontend/src/auth/session.ts`
- `app/frontend/src/auth/AuthPendingScreen.tsx`

**Backend**
- normalmente sem módulo de cálculo direto

**Referências**
- `reference/chichorro-3.1-rs/` → layout, navegação, ordem de módulos
- `reference/chichorro-3.0-jt/` → fluxo legacy após login

---

### `app/frontend/src/pages/LoginPage.tsx`
**Papel**
- entrada do utilizador
- autenticação
- arranque de sessão

**Relacionados**
- `app/frontend/src/auth/legacyLogin.ts`
- `app/frontend/src/auth/session.ts`
- `app/frontend/src/auth/AuthPendingScreen.tsx`
- `app/frontend/src/routes/RequireAuth.tsx`

**Backend**
- auth/session endpoint ou lógica equivalente, se existir

**Referências**
- `reference/chichorro-3.1-rs/` → fluxo funcional de login
- `reference/chichorro-3.0-jt/` → comportamento legacy de autenticação

**Notas**
- `legacyLogin.ts` deve ser visto como compatibilidade transitória até prova em contrário

---

## 2. Módulos de risco

### `app/frontend/src/pages/CtiPage.tsx`
**Papel**
- formulário e resultado CTI
- recolha de inputs
- envio para backend
- apresentação do resultado

**Backend principal**
- `app/backend/Chichorro_CTI.py`

**Integração**
- `app/backend/Flask.py`

**UI provável**
- `Button.tsx`
- `Card.tsx`
- `Field.tsx`
- `ModuleGlobalValueCard.tsx`

**Referências**
- `reference/chichorro-3.1-rs/` → módulo CTI
- `reference/chichorro-3.0-jt/` → comparação legacy

**Lacuna provável**
- ainda não há componentes CTI dedicados listados

---

### `app/frontend/src/pages/DpiPage.tsx`
**Papel**
- formulário e cálculo do módulo DPI

**Componentes**
- `app/frontend/src/components/dpi/DpiFactorSection.tsx`

**Definitions**
- `app/frontend/src/components/dpi/dpiDefinitions.ts`

**Backend principal**
- `app/backend/Chichorro_DPI.py`

**Integração**
- `app/backend/Flask.py`

**UI provável**
- `Button.tsx`
- `Card.tsx`
- `Field.tsx`
- `ModuleGlobalValueCard.tsx`

**Referências**
- `reference/chichorro-3.1-rs/` → módulo DPI
- `reference/chichorro-3.0-jt/` → comparação legacy

---

### `app/frontend/src/pages/EsciPage.tsx`
**Papel**
- formulário e cálculo do módulo ESCI

**Componentes**
- `app/frontend/src/components/esci/EsciFactorSection.tsx`

**Definitions**
- `app/frontend/src/components/esci/esciDefinitions.ts`

**Backend principal**
- `app/backend/Chichorro_ESCI.py`

**Integração**
- `app/backend/Flask.py`

**UI provável**
- `Button.tsx`
- `Card.tsx`
- `Field.tsx`
- `ModuleGlobalValueCard.tsx`

**Referências**
- `reference/chichorro-3.1-rs/` → módulo ESCI
- `reference/chichorro-3.0-jt/` → comparação legacy

---

### `app/frontend/src/pages/PoiPage.tsx`
**Papel**
- avaliação POI
- inputs/fatores
- cálculo e apresentação do resultado

**Componentes**
- `app/frontend/src/components/poi/PoiFactorSection.tsx`

**Definitions**
- `app/frontend/src/components/poi/poiDefinitions.ts`

**Backend principal**
- `app/backend/Chichorro_POI.py`

**Integração**
- `app/backend/Flask.py`

**UI provável**
- `Button.tsx`
- `Card.tsx`
- `Field.tsx`
- `ModuleGlobalValueCard.tsx`

**Referências**
- `reference/chichorro-3.1-rs/` → módulo POI
- `reference/chichorro-3.0-jt/` → comparação legacy

---

### `app/frontend/src/pages/RiPage.tsx`
**Papel**
- apresentação do resultado final do risco de incêndio
- atualização
- import/export de sessão
- visualização do resultado global

**Backend principal**
- `app/backend/Chichorro_RI.py`

**Integração**
- `app/backend/Flask.py`

**UI provável**
- `Button.tsx`
- `Card.tsx`
- `ModuleGlobalValueCard.tsx`

**Referências**
- `reference/chichorro-3.1-rs/` → módulo RI
- `reference/chichorro-3.0-jt/` → comparação legacy

**Lacuna provável**
- ainda não há componentes RI dedicados listados

---

## 3. Página temporária

### `app/frontend/src/pages/PlaceholderPage.tsx`
**Papel**
- zona temporária para funcionalidade ainda em migração ou desenvolvimento

**Backend**
- idealmente sem backend próprio

**Referências**
- `reference/chichorro-3.1-rs/` → identificar o que falta migrar
- `reference/chichorro-3.0-jt/` → identificar o que existia e desapareceu

**Risco**
- esconder dívida funcional durante demasiado tempo

---

## 4. Route protection e auth

### `app/frontend/src/routes/RequireAuth.tsx`
**Papel**
- proteger acesso a páginas internas
- redirecionar quando não existe sessão válida

### `app/frontend/src/auth/AuthPendingScreen.tsx`
**Papel**
- estado intermédio enquanto sessão/auth está a ser resolvida

### `app/frontend/src/auth/session.ts`
**Papel**
- gestão de sessão
- persistência
- leitura e invalidação de estado autenticado

### `app/frontend/src/auth/legacyLogin.ts`
**Papel**
- compatibilidade temporária com fluxo antigo de autenticação

---

## 5. UI base reutilizável

### `app/frontend/src/components/ui/Button.tsx`
- botões reutilizáveis

### `app/frontend/src/components/ui/Card.tsx`
- contentores visuais reutilizáveis

### `app/frontend/src/components/ui/Field.tsx`
- campos/input reutilizáveis

### `app/frontend/src/components/ui/ModuleGlobalValueCard.tsx`
- apresentação de valor global agregado por módulo

---

## 6. Backend por domínio

### `app/backend/Chichorro_CTI.py`
- regras e cálculo CTI

### `app/backend/Chichorro_DPI.py`
- regras e cálculo DPI

### `app/backend/Chichorro_ESCI.py`
- regras e cálculo ESCI

### `app/backend/Chichorro_POI.py`
- regras e cálculo POI

### `app/backend/Chichorro_RI.py`
- cálculo e composição do risco final

### `app/backend/Flask.py`
- ponte HTTP/API entre frontend e lógica técnica

### `app/backend/wsgi.py`
- serving/deploy

### `app/backend/parity_runner.py`
- comparação/paridade entre versões, métodos ou resultados

---

## 7. Lacunas já visíveis

### Frontend
- CTI ainda não mostra componentes dedicados como DPI/ESCI/POI
- RI ainda não mostra componentes dedicados
- `PlaceholderPage.tsx` pode esconder áreas relevantes ainda não migradas
- `legacyLogin.ts` pode ser dívida técnica transitória

### Documentação
Ainda falta mapear:
- page → endpoint Flask
- page → payload de entrada
- page → payload de saída
- page → referência exata em `chichorro-3.1-rs`
- status por módulo: `done / partial / missing / unknown`