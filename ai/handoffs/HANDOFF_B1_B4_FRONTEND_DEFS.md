# Handoff B1–B4 — Frontend: definitions + RiPage

Data: 2026-04-21
Produzido por: Claude (arquiteto técnico)
Para: Codex (implementador)
Depende de: A4 ✅ concluído

---

## Tipo de tarefa
`active_build` + `v3_1_matchup`

## Objetivo
Atualizar 4 ficheiros frontend para alinhar com as assinaturas backend 3.1 já implementadas.

**Executar em batch** — as definitions (B1/B2/B3) alimentam `requiredPoi`/`requiredEsci`/`requiredDpi` em `RiPage.tsx`; se B4 correr antes, o formulário RI pode rejeitar campos que ainda não existem.

## Ficheiros a editar
1. `app/frontend/src/components/poi/poiDefinitions.ts` — B1
2. `app/frontend/src/components/esci/esciDefinitions.ts` — B2
3. `app/frontend/src/components/dpi/dpiDefinitions.ts` — B3
4. `app/frontend/src/pages/RiPage.tsx` — B4

---

## B1 — poiDefinitions.ts: adicionar `POI_CC_Idade`

**Localização:** subfator `cc` (id: "cc"), após o campo `POI_CC_Revestimento` (linha ~107).

Adicionar o seguinte campo à lista `fields` do subfator `cc`, imediatamente depois de `POI_CC_Revestimento`:

```typescript
{
  key: "POI_CC_Idade",
  label: "Idade de Construção do Edifício",
  options: [
    { value: ">2008", label: "Posterior a 2008" },
    { value: "1991-2008", label: "1991 – 2008" },
    { value: "1975-1990", label: "1975 – 1990" },
    { value: "1968-1974", label: "1968 – 1974" },
    { value: "1951-1967", label: "1951 – 1967" },
    { value: "<1951", label: "Anterior a 1951" },
  ],
},
```

**Nota:** os `value` strings devem corresponder exatamente ao que o backend espera em `Chichorro_RI.py` linhas 118-129: `'>2008'`, `'1991-2008'`, `'1975-1990'`, `'1968-1974'`, `'1951-1967'`, `'<1951'`.

---

## B2 — esciDefinitions.ts: 4 alterações

### B2.1 — Subfator `gp`: adicionar `ESCI_GP_Auto`

Após o campo `ESCI_GP_DetAler`, adicionar:

```typescript
{
  key: "ESCI_GP_Auto",
  label: "Tipo de deteção automática",
  options: [
    { value: "Termovelocimetrico", label: "Termovelocimétrico" },
    { value: "Otico", label: "Ótico" },
    { value: "Aspiracao", label: "Aspiração" },
  ],
  visibleWhen: (v) => v.ESCI_GP_DetAler === "Automatico",
},
```

### B2.2 — Subfator `ext`: rename OGS values + adicionar `ESCI_EXT_Formacao`

**Rename OGS values** (campo `ESCI_EXT_OGS`):
- `"Sem OGS - PP+F"` → `"Sem OGS - R+PP"` (value E label)
- `"Com OGS - PP+F"` → `"Com OGS - R+PP"` (value E label)
- `"Com OGS - PE+S"` mantém (sem alteração)

Labels sugeridos: `"Sem OGS: R + PP"`, `"Com OGS: R + PP"`, `"Com OGS: PE + S"`.

**Adicionar `ESCI_EXT_Formacao`** após `ESCI_EXT_OGS`:

```typescript
{
  key: "ESCI_EXT_Formacao",
  label: "Formação",
  options: [
    { value: "Sem Formacao", label: "Sem formação" },
    { value: "Com Formacao", label: "Com formação" },
  ],
  visibleWhen: (v) => v.ESCI_EXT_Aplica === "Aplica" && v.ESCI_EXT_Extintores !== "" && v.ESCI_EXT_Extintores !== "Nao Respeita",
},
```

### B2.3 — Subfator `ria`: rename OGS values + adicionar `ESCI_RIA_Formacao` e `ESCI_RIA_CS`

**Rename OGS values** (campo `ESCI_RIA_OGS`): mesmo processo que B2.2.
- `"Sem OGS - PP+F"` → `"Sem OGS - R+PP"`
- `"Com OGS - PP+F"` → `"Com OGS - R+PP"`

**Adicionar `ESCI_RIA_Formacao`** após `ESCI_RIA_OGS`:

```typescript
{
  key: "ESCI_RIA_Formacao",
  label: "Formação",
  options: [
    { value: "Sem Formacao", label: "Sem formação" },
    { value: "Com Formacao", label: "Com formação" },
  ],
  visibleWhen: (v) => v.ESCI_RIA_Aplica === "Aplica" && v.ESCI_RIA_RIA !== "" && v.ESCI_RIA_RIA !== "Nao Respeita",
},
```

**Adicionar `ESCI_RIA_CS`** após `ESCI_RIA_Formacao`:

```typescript
{
  key: "ESCI_RIA_CS",
  label: "RIA e coluna seca",
  options: [
    { value: "RIA", label: "RIA" },
    { value: "RIA+CS", label: "RIA + CS" },
  ],
  visibleWhen: (v) => v.ESCI_RIA_Aplica === "Aplica" && v.ESCI_RIA_RIA !== "" && v.ESCI_RIA_RIA !== "Nao Respeita",
},
```

---

## B3 — dpiDefinitions.ts: substituir subfator `ogs` (7→4 campos)

**Localização:** subfator com `id: "ogs"` (linha ~218). Substituição completa dos campos — manter apenas `DPI_OGS_Aplica` e substituir os 6 campos restantes pelos 3 novos.

Resultado final do subfator `ogs`:

```typescript
{
  id: "ogs",
  title: "OGS — Organização e gestão da segurança",
  endpoint: "/DPI/OGS",
  resultKey: "DPI_OGS",
  fields: [
    {
      key: "DPI_OGS_Aplica",
      label: "Aplicabilidade",
      options: [
        { value: "Nao Se Aplica", label: "Não se aplica" },
        { value: "Aplica", label: "Aplica-se" },
      ],
    },
    {
      key: "DPI_OGS_OGS",
      label: "Organização e gestão de segurança",
      options: [
        { value: "Plano de Segurança e Simulacro", label: "Plano de Segurança e Simulacro" },
        { value: "Registo e Plano de Prevençao", label: "Registo e Plano de Prevenção" },
        { value: "Registos e Procedimento de Prevençao", label: "Registos e Procedimento de Prevenção" },
      ],
      visibleWhen: (v) => v.DPI_OGS_Aplica === "Aplica",
    },
    {
      key: "DPI_OGS_Formacao",
      label: "Formação",
      options: [
        { value: "Sem Formacao", label: "Sem formação" },
        { value: "Com Formacao", label: "Com formação" },
      ],
      visibleWhen: (v) => v.DPI_OGS_Aplica === "Aplica",
    },
    {
      key: "DPI_OGS_Regulamento",
      label: "Legislação regulamentar",
      options: [
        { value: "Existe", label: "Existe (apesar de não exigido)" },
        { value: "Respeita Totalmente", label: "Respeita totalmente" },
        { value: "Nao Respeita", label: "Não respeita" },
      ],
      visibleWhen: (v) => v.DPI_OGS_Aplica === "Aplica",
    },
  ],
},
```

---

## B4 — RiPage.tsx: remover picker período + ler RI_RIA do backend

### O que remover

1. **`acceptabilityOptions`** — array de 6 objetos com `value/label/limit` (linhas 46-53): remover completamente.
2. **`acceptabilityFormKey`** — constante `"ri:acceptability"` (linha ~56): remover.
3. **`acceptabilityPeriod`** state e o `setAcceptabilityPeriod` (linha ~65): remover.
4. **`acceptabilityConfig`** useMemo (linhas ~89-91): remover.
5. **`riAcceptable`** computed (linhas ~93-94): remover.
6. **`updateAcceptabilityPeriod`** function (linha ~197): remover.
7. **Referências a `acceptabilityFormKey`** em `importAppSession` e `clearAppSession` callbacks: remover as linhas `setAcceptabilityPeriod(...)`.
8. **O componente `<Select>`** do período e o bloco que o envolve (linhas ~273-300): remover.

### O que adicionar

**No tipo `RiResponse`** (linha ~44), adicionar `RI_RIA`:
```typescript
type RiResponse = { RI?: number; RI_Escala?: string; RI_RIA?: number };
```

**Novo state** `riRia`:
```typescript
const [riRia, setRiRia] = useState<number | undefined>(undefined);
```

**No callback de sucesso do fetch `/RI`**, após `setEscala(...)`:
```typescript
setRiRia(typeof response.RI_RIA === "number" ? response.RI_RIA : undefined);
```

**No clear/import session**, adicionar `setRiRia(undefined)`.

**A lógica de aceitabilidade** passa a ser:
```typescript
const riAcceptable = typeof ri === "number" && riRia !== undefined ? ri <= riRia : undefined;
```

**No display**, onde antes estava o picker de período e o limit, usar `riRia`:
- "Risco aceitável" se `ri <= riRia`, "Não aceitável" se `ri > riRia`, sem mensagem se `riRia` não definido
- Mostrar o valor `riRia?.toFixed(2)` em vez de `acceptabilityConfig.limit.toFixed(2)`

### O que NÃO tocar

- `scaleClass` — já funciona para a escala 12 classes (`startsWith("A")`, `startsWith("B")`, etc. + fallback `text-rose-800` para E e F)
- Lógica de fetch, inputs, required fields — não alterar

---

## Decisões fechadas
- `POI_CC_Idade` value strings com hífens: `">2008"`, `"1991-2008"`, etc. — correspondem ao backend (não usar underscores)
- `ESCI_GP_Auto` value `"Aspiracao"` sem acento — corresponde ao backend; a referência HTML tem acento mas o backend não
- `RiPage.tsx` não guarda `riRia` em sessionStorage — vem sempre do backend no fetch
- O picker de período é removido completamente (não manter como fallback)

## Restrições
- **Não tocar em** `RiPage.tsx` lógica de fetch ou required fields
- **Não tocar em** `EsciPage.tsx`, `DpiPage.tsx`, `PoiPage.tsx` — apenas as definitions
- **Não tocar em** backend nesta tarefa

---

## Validação esperada
1. Abrir POI → subfator CC deve mostrar 4 campos (incluindo Idade)
2. Abrir ESCI → subfator GP com DetAler=Automático deve mostrar campo Auto; EXT/RIA devem mostrar novos campos e OGS com R+PP
3. Abrir DPI → subfator OGS deve mostrar 4 campos (OGS, Formacao, Regulamento)
4. Calcular RI → resultado deve mostrar aceitabilidade sem picker local
5. TypeScript compile sem erros: `npm run build` na pasta frontend

## Output esperado
- 4 ficheiros frontend atualizados
- Build TypeScript sem erros
- `docs/migration/V3_1_MATCHUP_MATRIX.md` — atualizar POI/ESCI/DPI/RI de `done` para `done` com nota "frontend alinhado"

## Riscos / pontos de atenção
- `visibleWhen` do `ESCI_EXT_OGS` e `ESCI_RIA_OGS` dependem de valores que mudam (`PP+F` → `R+PP`) — verificar se as condições de visibilidade ainda fazem sentido
- `ESCI_RIA_CS` só aparece quando `ESCI_RIA_RIA !== "Nao Respeita"` — confirmar `visibleWhen` idêntico ao `ESCI_RIA_Formacao`
- `RiPage.tsx` usa `requiredPoi`, `requiredDpi`, `requiredEsci` derivados das definitions — ao adicionar `POI_CC_Idade` torna-se required para calcular RI
