# Guia de Utilização — FireRiskApp

O FireRiskApp permite avaliar o risco de incêndio num edifício existente com recurso ao método CHICHORRO (FEUP). O resultado final — o **Índice de Risco de Incêndio (RI)** — é obtido a partir de quatro módulos que devem ser preenchidos pela ordem seguinte:

```
POI → CTI → DPI → ESCI → RI → Intervenções (opcional)
```

---

## Acesso

A aplicação requer autenticação. Introduz as credenciais fornecidas pelo administrador no ecrã de login.

---

## Fluxo de trabalho

### 1. Preencher cada módulo

Cada módulo (POI, CTI, DPI, ESCI) está dividido em **subfatores**. Para cada subfator:

1. Expande o painel clicando no título (os subfatores podem estar colapsados por defeito)
2. Preenche os campos solicitados
3. Clica em **Calcular** para obter o valor do subfator
4. O resultado aparece no painel — se alterares um campo após calcular, aparece um aviso âmbar a pedir que recalcules

Os valores são guardados automaticamente na sessão — podes navegar entre páginas sem perder dados.

### 2. Calcular o RI

Depois de calculares todos os módulos, vai à página **RI** e clica em **Calcular RI**. O resultado aparece com:

- O valor numérico do RI
- A classe de risco na escala A++ … F
- A aceitabilidade de risco por utilização tipo

Se algum módulo não estiver calculado, a página indica qual falta.

### 3. Módulo de Intervenções (opcional)

Depois de ter um RI calculado, podes ir à página **Intervenções** para explorar medidas de redução de risco:

- Seleciona intervenções individualmente (34 disponíveis, ativas e passivas)
- Ou usa os **conjuntos predefinidos** adequados ao tipo de utilização do edifício
- Clica em **Calcular** para ver o novo RI após as intervenções e o custo estimado (€/m²)

---

## Módulos — o que preencher

### POI — Potencial Ocorrência de Incêndio

Avalia a probabilidade de ocorrência de incêndio com base em características do edifício e das suas instalações.

**Subfatores:**
- **CC** — Construção e Conservação (tipo de edifício, ano de construção, estado de conservação)
- **EF** — Instalações de Energia e Fluidos (instalação elétrica, gás, AVAC, aquecimento)
- **IA** — Instalações de Apoio (confeção de alimentos, armazenagem de combustíveis)
- **ATIV** — Tipo de Atividade (utilização-tipo do edifício — I a XII)

### CTI — Consequências do Incêndio para os Utilizadores

Avalia o risco para as pessoas em caso de incêndio, considerando as vias de evacuação.

**Subfatores:**
- **VHE** — Via Horizontal de Evacuação (corredor de saída ao mesmo nível)
- **VVE** — Via Vertical de Evacuação (caixa de escadas)

> O campo **Tipo de Edifício** (TipoEdif) é partilhado com o POI — é preenchido automaticamente a partir do módulo POI se já tiver sido calculado.

### DPI — Desenvolvimento e Propagação do Incêndio

Avalia a facilidade de propagação do incêndio pelo edifício.

**Subfatores:**
- **REIC** — Resistência ao Incêndio de Elementos Construtivos
- **EI** — Estanquidade e Integridade
- **VDGF** — Vãos, Diedros e Guarda-fogos
- **PE** — Paredes Exteriores
- **OGS** — Organização e Gestão da Segurança (planos de prevenção, formação, simulacros)

### ESCI — Eficácia de Socorro e Combate ao Incêndio

Avalia a capacidade de deteção e combate ao incêndio.

**Subfatores:**
- **GP** — Grau de Prontidão (deteção automática)
- **SID** — Sinalização, Iluminação e Deteção
- **AE** — Acessibilidade Exterior
- **HE** — Hidrantes Exteriores
- **EXT** — Extintores
- **RIA** — Rede de Incêndio Armada / Coluna Seca
- **CPB** — Corpo de Bombeiros

---

## Escala de classificação

| Classe | RI       | Significado |
|--------|----------|-------------|
| A++    | ≤ 0,50   | Risco muito reduzido |
| A+     | ≤ 0,75   | |
| A      | ≤ 1,00   | Risco reduzido |
| B+     | ≤ 1,10   | |
| B      | ≤ 1,25   | Risco moderado |
| B-     | ≤ 1,50   | |
| C+     | ≤ 1,75   | |
| C      | ≤ 2,00   | Risco elevado |
| C-     | ≤ 2,25   | |
| D      | ≤ 2,50   | Risco muito elevado |
| E      | ≤ 3,00   | |
| F      | > 3,00   | Risco extremo |

---

## Sessão — guardar e retomar

Os dados são guardados automaticamente no browser enquanto a sessão está ativa. Para guardar e retomar numa sessão futura:

- **Exportar sessão** — guarda um ficheiro `.json` com todos os valores preenchidos
- **Importar sessão** — carrega um ficheiro `.json` exportado anteriormente

A opção **Limpar sessão** apaga todos os dados e reinicia o formulário.

---

## Notas importantes

- O valor `0,00` em qualquer subfator significa **"Não se aplica"** — este subfator é excluído da média do módulo
- Se alterares um campo após calcular um subfator, o resultado fica marcado como desatualizado (cinzento) até recalculares
- Todos os módulos devem estar calculados antes de poder calcular o RI final
