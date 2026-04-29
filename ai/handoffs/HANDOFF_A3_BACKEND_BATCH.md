# Handoff A3 — Backend Batch: POI + ESCI + DPI + RI completion

Data: 2026-04-20
Produzido por: Claude (arquiteto técnico)
Para: Codex (implementador)
Depende de: A2 ✅ concluído

---

## Tipo de tarefa
`active_build` + `v3_1_matchup`

## Objetivo
Substituir os módulos POI, ESCI e DPI do backend pelos equivalentes 3.1, e completar a atualização do RI.py (que ficou a meio no A2).

**⚠️ Executar como batch.** Os três módulos (POI/ESCI/DPI) mudam assinaturas que `Chichorro_RI.py` chama. Se executados separadamente, `Chichorro_RI.py` fica inconsistente até ao último ficheiro. Fazer tudo no mesmo passo mantém o estado sempre coerente.

## Estado atual (antes desta tarefa)
- `Chichorro_POI.py` — versão 3.0 (sem `POI_CC_Idade`)
- `Chichorro_ESCI.py` — versão 3.0 (sem `ESCI_GP_Auto`, `ESCI_EXT_Formacao`, `ESCI_RIA_Formacao`, `ESCI_RIA_CS`)
- `Chichorro_DPI.py` — versão 3.0 (DPI_OGS com 7 campos antigos)
- `Chichorro_RI.py` — parcialmente atualizado: escala 3.1 ✅, CTI call ✅, mas ainda com assinaturas 3.0 para POI/ESCI/DPI internamente

## Zona do repositório a usar
- `app/backend/` → **editar** (4 ficheiros)
- `reference/chichorro-3.1-rs/chichorro-html/` → **apenas leitura** (fonte das versões 3.1)

---

## Tarefa 1 — Substituir Chichorro_POI.py
**Tipo:** substituição direta (sem refactors locais a preservar)
**Fonte:** `reference/chichorro-3.1-rs/chichorro-html/Chichorro_POI.py`
**Destino:** `app/backend/Chichorro_POI.py`

**Diferença chave:**
- `POI_CC(Comb, Infiltracoes, Revestimento, Idade)` — novo 4.º parâmetro `POI_CC_Idade`
- O parâmetro é aceite mas **não altera o cálculo de POI_CC** — é passado até ao RI para `RI_RIA`
- Assinatura `POI(...)` também recebe `POI_CC_Idade` como parâmetro extra

**Ação:** copiar o ficheiro de referência diretamente.

---

## Tarefa 2 — Substituir Chichorro_ESCI.py
**Tipo:** substituição direta
**Fonte:** `reference/chichorro-3.1-rs/chichorro-html/Chichorro_ESCI.py`
**Destino:** `app/backend/Chichorro_ESCI.py`

**Diferenças chave:**
- `ESCI_GP(Distancia, Tempo, DetAler, Auto)` — novo 4.º parâmetro `ESCI_GP_Auto`
- `ESCI_EXT(Aplica, OGS, Formacao, Extintores)` — novo `ESCI_EXT_Formacao` entre OGS e Extintores; OGS values mudam de `PP+F` → `R+PP`
- `ESCI_RIA(Aplica, OGS, Formacao, CS, RIA)` — novos `ESCI_RIA_Formacao` e `ESCI_RIA_CS`; OGS values mudam
- `ESCI(...)` — assinatura agregadora actualizada

**Ação:** copiar o ficheiro de referência diretamente.

---

## Tarefa 3 — Substituir Chichorro_DPI.py
**Tipo:** substituição direta
**Fonte:** `reference/chichorro-3.1-rs/chichorro-html/Chichorro_DPI.py`
**Destino:** `app/backend/Chichorro_DPI.py`

**Diferença chave:**
- `DPI_OGS(Aplica, OGS, Formacao, Regulamento)` — substitui completamente a assinatura antiga de 7 parâmetros
- Lógica com ajuste pós-score: após calcular o valor base, subtrai 0.1 (`Regulamento == 'Existe'`), 0.2 (`Respeita Totalmente`) ou 0.0 (`Nao Respeita`)
- `DPI(...)` — assinatura actualizada

**Ação:** copiar o ficheiro de referência diretamente.

---

## Tarefa 4 — Completar Chichorro_RI.py
**Tipo:** merge (o ficheiro já foi editado no A2 — não substituir)
**Ficheiro:** `app/backend/Chichorro_RI.py`

### O que já está correto (não tocar):
- Escala 3.1: A++, A+, A, B+, B, B-, C+, C, C-, D, E, F
- Chamada `cti.CTI(...)` com nova assinatura (passando `0` como `RI_interv_21_efetivo`, e `Dispositivos` como fallback para `VHE_Dispositivos`/`VVE_Dispositivos`)

### O que falta completar na assinatura `def RI(...)`:

**1. Adicionar `POI_CC_Idade`:**
Inserir `POI_CC_Idade` imediatamente a seguir a `POI_CC_Revestimento` na assinatura.
A chamada `poi.POI_CC(...)` dentro do `RI()` também deve receber `POI_CC_Idade`.
A chamada `poi.POI(...)` também deve receber `POI_CC_Idade`.
Adicionar o cálculo de `RI_RIA` no final:
```python
if POI_CC_Idade == '>2008':
    RI_RIA = 1.00
elif POI_CC_Idade == '1991-2008':
    RI_RIA = 1.05
elif POI_CC_Idade == '1975-1990':
    RI_RIA = 1.10
elif POI_CC_Idade == '1968-1974':
    RI_RIA = 1.15
elif POI_CC_Idade == '1951-1967':
    RI_RIA = 1.20
elif POI_CC_Idade == '<1951':
    RI_RIA = 1.25
```
E incluir no return: `RI_RI["RI_RIA"] = RI_RIA`

**2. Substituir DPI_OGS na assinatura (7 campos → 4):**
Remover: `DPI_OGS_P_Emergencia_Exigencia, DPI_OGS_P_Emergencia_Regul, DPI_OGS_Formacao_Simulacro_Exigencia, DPI_OGS_Formacao_Simulacro_Regul, DPI_OGS_Equipa_Exigencia, DPI_OGS_Equipa_Regul`
Adicionar: `DPI_OGS_OGS, DPI_OGS_Formacao, DPI_OGS_Regulamento`
Atualizar a chamada `dpi.DPI_OGS(...)` e `dpi.DPI(...)` dentro da função.

**3. Adicionar parâmetros ESCI 3.1 à assinatura:**
- Após `ESCI_GP_DetAler`, adicionar: `ESCI_GP_Auto`
- Após `ESCI_EXT_OGS`, adicionar: `ESCI_EXT_Formacao`
- Após `ESCI_RIA_OGS`, adicionar: `ESCI_RIA_Formacao, ESCI_RIA_CS`
Atualizar as chamadas a `esci.ESCI_GP(...)`, `esci.ESCI_EXT(...)`, `esci.ESCI_RIA(...)` e `esci.ESCI(...)`.

---

## Decisões fechadas
- POI/ESCI/DPI: substituição direta da referência, sem preservar nada do ativo (não há refactors locais nestes ficheiros)
- RI: merge — a escala e o CTI call já estão corretos, não regredir
- `RI_RIA` é o campo de aceitabilidade — deve ser incluído no dict de retorno de `RI()`

## Restrições
- **Não tocar em** `Flask.py` nesta tarefa — será o handoff seguinte (A4), depois de todos os módulos estarem corretos
- **Não tocar em** frontend nesta tarefa
- **Não alterar** a escala de RI já implementada

---

## Validação esperada
Após as 4 tarefas, correr:
```bash
cd app/backend && python -c "import Chichorro_POI, Chichorro_ESCI, Chichorro_DPI, Chichorro_RI; print('imports ok')"
```
Não deve haver erros de import ou assinatura.

## Output esperado
- 4 ficheiros backend atualizados
- Sem erros de import
- `docs/migration/V3_1_MATCHUP_MATRIX.md` — atualizar estados de POI, ESCI, DPI e RI para `done` (backend)

## Riscos / pontos de atenção
- `Chichorro_RI.py` chama `poi.POI_CC`, `poi.POI`, `dpi.DPI_OGS`, `dpi.DPI`, `esci.ESCI_GP`, `esci.ESCI_EXT`, `esci.ESCI_RIA`, `esci.ESCI` — todas estas chamadas devem ser atualizadas para as novas assinaturas 3.1
- Os valores de OGS mudaram de string (`PP+F` → `R+PP`) — certificar que os módulos ESCI e RI usam os novos valores
- `parity_runner.py` pode falhar após estas mudanças se os payloads de teste não forem atualizados — aceitar falha temporária e notar nos docs; será corrigido no handoff Flask
