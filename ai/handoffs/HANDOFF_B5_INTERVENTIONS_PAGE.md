# Handoff B5 — InterventionsPage.tsx

Data: 2026-04-21
Produzido por: Claude (arquiteto técnico)
Para: Codex (implementador)
Depende de: B1–B4 ✅ concluído

---

## Tipo de tarefa
`active_build` + `v3_1_matchup`

## Objetivo
Criar `app/frontend/src/pages/InterventionsPage.tsx` — nova página para o módulo de intervenções de redução de risco de incêndio.

A página permite ao utilizador:
1. Selecionar intervenções individualmente (34 checkboxes)
2. Usar botões de conjunto predefinidos (atalhos que selecionam grupos de intervenções)
3. Ver o RI antes e após as intervenções selecionadas
4. Ver o custo estimado das intervenções

---

## Ficheiros a criar/editar
- `app/frontend/src/pages/InterventionsPage.tsx` — **criar** (novo)
- `app/frontend/src/App.tsx` (ou ficheiro de rotas) — **editar**: adicionar rota `/interventions`
- Verificar se há entrada de menu a criar (consultar como as outras páginas estão registadas)

---

## Dados estáticos a incluir no ficheiro

### Lista das 34 intervenções

```typescript
const INTERVENTIONS: { id: number; label: string; type: "ativa" | "passiva" }[] = [
  { id: 1,  label: "Hidrantes exteriores < 30m do edifício", type: "ativa" },
  { id: 2,  label: "Redução do estacionamento condicionado pela Câmara", type: "ativa" },
  { id: 3,  label: "Sinalização nas vias de evacuação", type: "ativa" },
  { id: 4,  label: "Iluminação nas vias de evacuação", type: "ativa" },
  { id: 5,  label: "Sinalização e iluminação na CI", type: "ativa" },
  { id: 6,  label: "Deteção dentro das frações com média fiabilidade (100 seg)", type: "ativa" },
  { id: 7,  label: "Deteção dentro das frações com grande fiabilidade (50 seg)", type: "ativa" },
  { id: 8,  label: "Deteção dentro das frações com elevada fiabilidade (30 seg)", type: "ativa" },
  { id: 9,  label: "Deteção nas vias de evacuação", type: "ativa" },
  { id: 10, label: "Controlo de fumo — clarabóias regulamentares e entrada de ar passivo nas vias de evacuação", type: "ativa" },
  { id: 11, label: "Controlo de fumo — CI", type: "ativa" },
  { id: 12, label: "Extintores 1ª intervenção", type: "ativa" },
  { id: 13, label: "Rede de intervenção armada 1ª intervenção", type: "ativa" },
  { id: 14, label: "Rede de intervenção armada e coluna seca 2ª intervenção", type: "ativa" },
  { id: 15, label: "REA Sprinklers", type: "ativa" },
  { id: 16, label: "OGS — Registos + Plano de prevenção", type: "ativa" },
  { id: 17, label: "OGS — Plano de emergência + Simulacro", type: "ativa" },
  { id: 18, label: "OGS — Formação", type: "ativa" },
  { id: 19, label: "Redução de infiltrações", type: "passiva" },
  { id: 20, label: "Acesso à cave por acesso distinto do resto do edifício ou proteção porta CF ou CCF", type: "passiva" },
  { id: 21, label: "Instalação ou reparação de escadas de salvação", type: "passiva" },
  { id: 22, label: "Proteção dos vãos para edifícios fronteiros", type: "passiva" },
  { id: 23, label: "Proteção de cobertura e empena para edifícios vizinhos", type: "passiva" },
  { id: 24, label: "Compartimentação — RF lajes", type: "passiva" },
  { id: 25, label: "Compartimentação — Enclausuramento caixa de escadas", type: "passiva" },
  { id: 26, label: "Selagem dos ductos piso a piso", type: "passiva" },
  { id: 27, label: "Compartimentação — Portas CF nos CI", type: "passiva" },
  { id: 28, label: "Pinturas e acabamentos nos CHE e CVE", type: "passiva" },
  { id: 29, label: "Revisão da instalação elétrica", type: "passiva" },
  { id: 30, label: "Revisão da instalação de gás", type: "passiva" },
  { id: 31, label: "Revisão da instalação AVAC", type: "passiva" },
  { id: 32, label: "Revisão pequena da instalação de aquecimento", type: "passiva" },
  { id: 33, label: "Revisão grande da instalação de aquecimento", type: "passiva" },
  { id: 34, label: "Revisão da instalação de confeção e conservação de alimentos", type: "passiva" },
];
```

### Conjuntos predefinidos por tipo de utilização

Os conjuntos são UT-dependentes. Extrair os dados de:
`reference/chichorro-3.1-rs/frontend_reference/MainPage/script/RI_I.js`

Funções: `RI_interv_Conj_01()` … `RI_interv_Conj_06()` (a partir da linha 1367).
Cada função define, por `POI_ATIV_TipoEdif`, quais intervenções ficam `checked = true`.

Estrutura TypeScript a usar:

```typescript
type TipoEdifGroup =
  | "habitacao"
  | "administrativos_hoteis"
  | "escolas_hospitais"
  | "desporto_museus"
  | "espetaculo_restauracao_comercio_bibliotecas_oficinas"
  | "estacionamento_industria_armazem";

const CONJUNTOS: Record<TipoEdifGroup, Record<1|2|3|4|5|6, number[]>> = {
  habitacao: {
    1: [3, 4, 19, 29],           // extraído de RI_interv_Conj_01 → "I - Habitacao"
    2: [3, 4, 19, 29, 34],       // extraído de RI_interv_Conj_02 → "I - Habitacao"
    3: [...],                    // ler de RI_I.js
    4: [...],
    5: [...],
    6: [...],
  },
  administrativos_hoteis: { 1: [...], 2: [...], 3: [...], 4: [...], 5: [...], 6: [...] },
  escolas_hospitais:      { 1: [...], 2: [...], 3: [...], 4: [...], 5: [...], 6: [...] },
  desporto_museus:        { 1: [...], 2: [...], 3: [...], 4: [...], 5: [...], 6: [...] },
  espetaculo_restauracao_comercio_bibliotecas_oficinas: { 1: [...], 2: [...], 3: [...], 4: [], 5: [], 6: [] },
  estacionamento_industria_armazem: { 1: [...], 2: [...], 3: [], 4: [], 5: [], 6: [] },
};

// Mapeamento de POI_ATIV_TipoEdif para grupo
function getTipoEdifGroup(tipoEdif: string): TipoEdifGroup | null {
  if (tipoEdif === "I - Habitacao") return "habitacao";
  if (["III - Administrativos", "VII - Hoteis, Hosteis, Pensoes ou Albergarias"].includes(tipoEdif))
    return "administrativos_hoteis";
  if (["IV - Escolas, Creches ou Jardins de Infancia", "V - Hospitais, Enfermarias, Lares, Consultorios ou Clinicas"].includes(tipoEdif))
    return "escolas_hospitais";
  if (["IX - Desporto e de Lazer", "X - Museus ou Galerias de Arte"].includes(tipoEdif))
    return "desporto_museus";
  if (["VI - Salas de Espetaculo e Reunioes Publicas", "VII - Restauracao", "VIII - Comercio",
       "XI - Bibliotecas", "XI - Arquivos", "XII - Oficinas"].includes(tipoEdif))
    return "espetaculo_restauracao_comercio_bibliotecas_oficinas";
  if (["II - Estacionamento", "XII - Industria", "XII - Laboratorios Quimicos", "XII - Armazem"].includes(tipoEdif))
    return "estacionamento_industria_armazem";
  return null;
}

// Número de conjuntos visíveis por grupo
const CONJUNTOS_VISIVEIS: Record<TipoEdifGroup, number> = {
  habitacao: 6,
  administrativos_hoteis: 6,
  escolas_hospitais: 6,
  desporto_museus: 6,
  espetaculo_restauracao_comercio_bibliotecas_oficinas: 3,
  estacionamento_industria_armazem: 2,
};
```

**⚠️ Preencher os arrays `[...]` lendo RI_I.js.** Os dados para `habitacao` Conjuntos 1 e 2 já estão acima confirmados. Os restantes ler das funções `RI_interv_Conj_0X` para cada `POI_ATIV_TipoEdif`.

---

## Estado React e lógica de conjuntos

```typescript
const [activeConjuntos, setActiveConjuntos] = useState<Set<number>>(new Set());
const [checked, setChecked] = useState<Set<number>>(new Set());
```

### Toggle de um conjunto N

```typescript
function toggleConjunto(n: number) {
  const group = getTipoEdifGroup(tipoEdif);
  if (!group) return;
  const conjInterventions = new Set(CONJUNTOS[group][n as 1|2|3|4|5|6]);

  if (activeConjuntos.has(n)) {
    // Desselecionar: recalcular checked = união dos conjuntos restantes
    const remaining = new Set(activeConjuntos);
    remaining.delete(n);
    const newChecked = new Set<number>();
    remaining.forEach(c => CONJUNTOS[group][c as 1|2|3|4|5|6].forEach(i => newChecked.add(i)));
    setActiveConjuntos(remaining);
    setChecked(newChecked);
  } else {
    // Selecionar: adicionar intervenções deste conjunto
    const newChecked = new Set(checked);
    conjInterventions.forEach(i => newChecked.add(i));
    const newActive = new Set(activeConjuntos);
    newActive.add(n);
    setActiveConjuntos(newActive);
    setChecked(newChecked);
  }
}
```

### Toggle manual de uma intervenção

```typescript
function toggleIntervention(id: number) {
  const group = getTipoEdifGroup(tipoEdif);
  const newChecked = new Set(checked);
  if (newChecked.has(id)) {
    // Desmarcar manualmente → desselecionar todos os conjuntos que contêm esta intervenção
    newChecked.delete(id);
    const newActive = new Set(activeConjuntos);
    if (group) {
      newActive.forEach(c => {
        if (CONJUNTOS[group][c as 1|2|3|4|5|6].includes(id)) newActive.delete(c);
      });
    }
    setActiveConjuntos(newActive);
  } else {
    newChecked.add(id);
    // Marcar manualmente não activa nenhum conjunto
  }
  setChecked(newChecked);
}
```

### Limpar Conjuntos

```typescript
function clearConjuntos() {
  setActiveConjuntos(new Set());
  setChecked(new Set());
}
```

---

## Dados necessários do session storage

A página precisa de todos os inputs já preenchidos nas outras páginas (POI, CTI, DPI, ESCI), mais o campo `POI_ATIV_TipoEdif` especificamente para determinar o grupo de conjuntos.

Usar `getModuleInputs` / `getFormValues` (padrão das outras páginas) para recolher o payload completo.

---

## Chamada ao backend

Endpoint: `POST /RI/interv`

Payload: todos os campos do `/RI` + `Custo` (valor fixo `0`) + `RI_interv_01` … `RI_interv_34` (cada um `"true"` ou `"false"` conforme `checked`).

```typescript
const payload = {
  ...allRiInputs,           // mesmo payload do /RI
  Custo: 0,
  ...Array.from({ length: 34 }, (_, i) => ({
    [`RI_interv_${String(i + 1).padStart(2, "0")}`]: checked.has(i + 1) ? "true" : "false",
  })).reduce((a, b) => ({ ...a, ...b }), {}),
};
```

Resposta esperada: mesmo dict que `/RI` (com `RI`, `RI_Escala`, `RI_RIA`, etc.) — o backend recalcula o RI com as intervenções aplicadas.

---

## Layout da página

Seguir a estrutura visual da referência (screenshot partilhado):
- Título "Intervenções"
- Dois painéis lado a lado: "Intervenções Ativas" (ids 1-18) e "Intervenções Passivas" (ids 19-34)
- Cada intervenção: checkbox + número + label
- Secção de conjuntos:
  - Título "Conjunto de medidas de intervenção para a redução do risco de incêndio"
  - Botões "Conjunto 1" … "Conjunto N" (N = `CONJUNTOS_VISIVEIS[group]`, 0 se `group` null)
  - Botão estilizado como selecionado quando `activeConjuntos.has(n)`
  - Botão "Limpar Conjuntos"
- Dois painéis de resultado lado a lado:
  - "Classificação antes das intervenções" — RI e escala do `/RI` (ler do sessionStorage se disponível)
  - "Classificação após as intervenções" — resultado do `/RI/interv`
  - Custo da Intervenção (€/m²) — campo de output do backend
- Botão "Calcular"

---

## Restrições
- **Não modificar** nenhum outro ficheiro de página ou componente existente
- Os conjuntos visíveis dependem de `POI_ATIV_TipoEdif` — se não estiver em sessão, mostrar mensagem a pedir que o POI seja preenchido primeiro
- O botão de conjunto fica visualmente ativo (e.g. `bg-green-600`) quando está selecionado, inativo (e.g. `bg-gray-200`) quando não

---

## Validação esperada
- Selecionar Conjunto 1 → checkboxes corretos marcados
- Selecionar Conjunto 2 (com 1 ativo) → união das intervenções
- Desmarcar conjunto já ativo → apenas intervenções exclusivas desse conjunto desaparecem
- Desmarcar manualmente intervenção de conjunto ativo → botão do conjunto fica inativo
- "Limpar Conjuntos" → todos os checkboxes e botões limpos
- Calcular → resultado aparece no painel "após intervenções"
- TypeScript build sem erros

## Output esperado
- `app/frontend/src/pages/InterventionsPage.tsx` criado
- Rota `/interventions` registada
- `docs/migration/V3_1_MATCHUP_MATRIX.md` — Intervenções de `partial` para `done`
- `docs/ai/NEXT_STEPS.md` — B5 marcado como concluído
