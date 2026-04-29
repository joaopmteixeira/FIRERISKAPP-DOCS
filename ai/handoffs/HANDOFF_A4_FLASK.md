# Handoff A4 — Flask + RI_inter: completar backend 3.1

Data: 2026-04-20
Produzido por: Claude (arquiteto técnico)
Para: Codex (implementador)
Depende de: A3 ✅ concluído

---

## Tipo de tarefa
`active_build` + `v3_1_matchup`

## Objetivo
Duas tarefas em batch:
1. Criar `app/backend/Chichorro_RI_inter.py` (cópia direta da referência)
2. Atualizar `app/backend/Flask.py` — alinhar endpoints existentes com assinaturas 3.1 + adicionar `/RI/interv`

**⚠️ Executar em batch.** `Flask.py` vai importar `riintv` que só existirá depois de criar o ficheiro. Criar o módulo primeiro.

## Estado atual (antes desta tarefa)
- `app/backend/Chichorro_RI_inter.py` — **não existe**
- `app/backend/Flask.py` — 1641 linhas; CTI endpoints ✅ já atualizados; todos os outros endpoints ainda com assinaturas 3.0:
  - `/POI/CC` — sem `POI_CC_Idade`
  - `/POI` — sem `POI_CC_Idade`
  - `/DPI/OGS` — 7 campos antigos
  - `/DPI` — 7 campos DPI_OGS antigos
  - `/ESCI/GP` — sem `ESCI_GP_Auto`
  - `/ESCI/EXT` — sem `ESCI_EXT_Formacao`
  - `/ESCI/RIA` — sem `ESCI_RIA_Formacao`, `ESCI_RIA_CS`
  - `/ESCI` — sem novos campos
  - `/RI` — sem `POI_CC_Idade`, DPI_OGS antigo, ESCI antigo
  - Sem `/RI/interv`

## Zona do repositório a usar
- `app/backend/Chichorro_RI_inter.py` → **criar** (novo ficheiro)
- `app/backend/Flask.py` → **editar** (não substituir — tem auth routes e helpers locais)
- `reference/chichorro-3.1-rs/backend_reference/Chichorro_RI_inter.py` → **apenas leitura** (fonte)
- `reference/chichorro-3.1-rs/backend_reference/Flask.py` → **apenas leitura** (referência de payloads)

---

## Tarefa 1 — Criar Chichorro_RI_inter.py
**Tipo:** cópia direta
**Fonte:** `reference/chichorro-3.1-rs/backend_reference/Chichorro_RI_inter.py`
**Destino:** `app/backend/Chichorro_RI_inter.py`

**Ação:** copiar o ficheiro de referência diretamente. Não adaptar.

---

## Tarefa 2 — Atualizar Flask.py

**⚠️ Flask.py é um merge, NÃO uma substituição.** O ficheiro ativo tem rotas de autenticação (`/auth/login`, `/auth/logout`, `/auth/me`, `/login`, `/logout`, `/me`) e a função helper `_normalize_cti_payload` que **não existem na referência 3.1**. Apagar essas partes quebraria a app. Apenas editar os endpoints listados abaixo.

### 2.1 — Adicionar import no topo

Após a linha `import Chichorro_RI as ri`, adicionar:
```python
import Chichorro_RI_inter as riintv
```

### 2.2 — Endpoint `/POI/CC`

**Localização:** linha ~221 (rota `@app.route('/POI/CC', ...)`)

Adicionar leitura do novo campo após `POI_CC_Revestimento`:
```python
POI_CC_Idade = data["POI_CC_Idade"]
```
Atualizar a chamada `poi.POI_CC(...)` para incluir `POI_CC_Idade` como 4.º argumento.

### 2.3 — Endpoint `/POI`

**Localização:** linha ~460

Adicionar leitura após `POI_CC_Revestimento`:
```python
POI_CC_Idade = data["POI_CC_Idade"]
```
Atualizar as chamadas `poi.POI_CC(...)` e `poi.POI(...)` para incluir `POI_CC_Idade` como 4.º argumento.

### 2.4 — Endpoint `/DPI/OGS`

**Localização:** linha ~1169

Substituir os 7 campos antigos por 4 campos novos:

Remover:
```python
DPI_OGS_P_Emergencia_Exigencia = data["DPI_OGS_P_Emergencia_Exigencia"]
DPI_OGS_P_Emergencia_Regul = data["DPI_OGS_P_Emergencia_Regul"]
DPI_OGS_Formacao_Simulacro_Exigencia = data["DPI_OGS_Formacao_Simulacro_Exigencia"]
DPI_OGS_Formacao_Simulacro_Regul = data["DPI_OGS_Formacao_Simulacro_Regul"]
DPI_OGS_Equipa_Exigencia = data["DPI_OGS_Equipa_Exigencia"]
DPI_OGS_Equipa_Regul = data["DPI_OGS_Equipa_Regul"]
```

Adicionar:
```python
DPI_OGS_OGS = data["DPI_OGS_OGS"]
DPI_OGS_Formacao = data["DPI_OGS_Formacao"]
DPI_OGS_Regulamento = data["DPI_OGS_Regulamento"]
```

Atualizar chamada `dpi.DPI_OGS(...)`:
```python
DPI__OGS = dpi.DPI_OGS(DPI_OGS_Aplica, DPI_OGS_OGS, DPI_OGS_Formacao, DPI_OGS_Regulamento)
```

### 2.5 — Endpoint `/DPI`

**Localização:** linha ~1192

Mesmo processo que 2.4 — substituir os 7 campos antigos pelos 4 novos, e atualizar as chamadas `dpi.DPI_OGS(...)` e `dpi.DPI(...)`.

### 2.6 — Endpoint `/ESCI/GP`

**Localização:** linha ~1253

Adicionar após `ESCI_GP_DetAler`:
```python
ESCI_GP_Auto = data["ESCI_GP_Auto"]
```
Atualizar chamada `esci.ESCI_GP(...)` para incluir `ESCI_GP_Auto` como 4.º argumento.

### 2.7 — Endpoint `/ESCI/EXT`

**Localização:** linha ~1328

Adicionar após `ESCI_EXT_OGS`:
```python
ESCI_EXT_Formacao = data["ESCI_EXT_Formacao"]
```
Atualizar chamada `esci.ESCI_EXT(...)`:
```python
ESCI__EXT = esci.ESCI_EXT(ESCI_EXT_Aplica, ESCI_EXT_OGS, ESCI_EXT_Formacao, ESCI_EXT_Extintores)
```

### 2.8 — Endpoint `/ESCI/RIA`

**Localização:** linha ~1347

Adicionar após `ESCI_RIA_OGS`:
```python
ESCI_RIA_Formacao = data["ESCI_RIA_Formacao"]
ESCI_RIA_CS = data["ESCI_RIA_CS"]
```
Atualizar chamada `esci.ESCI_RIA(...)`:
```python
ESCI__RIA = esci.ESCI_RIA(ESCI_RIA_Aplica, ESCI_RIA_OGS, ESCI_RIA_Formacao, ESCI_RIA_CS, ESCI_RIA_RIA)
```

### 2.9 — Endpoint `/ESCI`

**Localização:** linha ~1384

Adicionar os 4 novos campos (GP_Auto, EXT_Formacao, RIA_Formacao, RIA_CS) nas leituras do payload, e atualizar as chamadas `esci.ESCI_GP(...)`, `esci.ESCI_EXT(...)`, `esci.ESCI_RIA(...)` e `esci.ESCI(...)` para corresponder às assinaturas 3.1.
Verificar na referência `Flask.py` linha ~1176 o payload exato do `/ESCI` para confirmar a ordem.

### 2.10 — Endpoint `/RI`

**Localização:** linha ~1440

Adicionar `POI_CC_Idade` após `POI_CC_Revestimento`.
Substituir os 7 campos DPI_OGS antigos pelos 4 novos.
Adicionar `ESCI_GP_Auto`, `ESCI_EXT_Formacao`, `ESCI_RIA_Formacao`, `ESCI_RIA_CS`.
Adicionar `RI_interv_21_efetivo` se ainda não estiver (verificar — os CTI endpoints já têm, mas o `/RI` pode não ter).
Atualizar a chamada `ri.RI(...)` para a assinatura 3.1 completa.
Verificar na referência `Flask.py` linha ~1236 o payload exato.

### 2.11 — Novo endpoint `/RI/interv`

Adicionar após o endpoint `/RI`. Copiar diretamente da referência `Flask.py` (linhas ~1430+).

O endpoint:
- Recebe todos os campos de `/RI` + `Custo` + `RI_interv_01` … `RI_interv_34`
- Chama `riintv.RI_interv(...)`
- Retorna o dict de resultado

---

## Decisões fechadas
- `Flask.py` é merge — auth routes e `_normalize_cti_payload` não devem ser tocados
- `Chichorro_RI_inter.py` é cópia direta — não adaptar
- O endpoint `/RI/interv` recebe `RI_interv_21_efetivo` como parte do payload (não é defaultado a 0 aqui — o módulo de intervenções pode modificar o efetivo)

## Restrições
- **Não tocar em** frontend nesta tarefa
- **Não tocar em** nenhum outro módulo Python (`Chichorro_*.py`) — todos já estão atualizados
- **Não remover** as rotas de auth nem o helper `_normalize_cti_payload`
- **Não tocar nos** endpoints CTI — já estão corretos

---

## Validação esperada
Após as 2 tarefas, correr:
```bash
cd app/backend && python -c "import Chichorro_RI_inter; print('RI_inter ok')"
cd app/backend && python -c "import Flask; print('Flask ok')" 2>&1 | head -5
```
Sem erros de import.

Opcional (se `parity_runner.py` tiver payloads atualizados):
```bash
cd app/backend && python parity_runner.py
```

## Output esperado
- `app/backend/Chichorro_RI_inter.py` criado
- `app/backend/Flask.py` com todos os endpoints alinhados com assinaturas 3.1
- `docs/migration/V3_1_MATCHUP_MATRIX.md` — atualizar Intervenções de `missing` para `partial` (backend existe, frontend ainda por fazer)

## Riscos / pontos de atenção
- O `/RI` endpoint é o mais longo do ficheiro — ler a secção completa antes de editar para não perder variáveis intermédias
- Verificar se o `/RI` ativo já tem `RI_interv_21_efetivo` no payload (os CTI endpoints têm, mas o RI pode ter ficado de fora)
- O `/ESCI` endpoint agrega todos os subfatores — verificar se lê todos os campos novos antes de passar às chamadas internas
- `parity_runner.py` pode falhar se os payloads de teste não tiverem os novos campos — aceitar falha temporária
