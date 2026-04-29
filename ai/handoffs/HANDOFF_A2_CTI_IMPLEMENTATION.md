# Handoff A2 — Implementação CTI 3.1

Data: 2026-04-20
Produzido por: Claude (arquiteto técnico)
Para: Codex (implementador)
Depende de: A1 ✅ concluído

---

## Tipo de tarefa
`active_build` + `v3_1_matchup`

## Objetivo
Atualizar `app/backend/Chichorro_CTI.py` para incorporar as diferenças identificadas na análise CTI 3.1, **preservando os refactors locais do ativo**, e corrigir o bug crítico de sympy.

Esta tarefa é um **merge**, não uma substituição de ficheiro.

## Estado atual (antes desta tarefa)
- `app/backend/Chichorro_CTI.py` — versão 3.0, sem `RI_interv_21_efetivo`, `VHE_Dispositivos`, `VVE_Dispositivos`; tem bug sympy; tem refactors locais (fumo, clarabóia, VVE ascendente) não presentes na referência 3.1
- `reference/chichorro-3.1-rs/chichorro-html/Chichorro_CTI.py` — referência com a nova assinatura
- `app/backend/Flask.py` — chama `cti.CTI(...)` com a assinatura antiga

## Zona do repositório a usar
- `app/backend/Chichorro_CTI.py` → **editar**
- `app/backend/Flask.py` → **editar** (endpoint CTI)
- `reference/chichorro-3.1-rs/chichorro-html/Chichorro_CTI.py` → **apenas leitura**, fonte da assinatura 3.1
- `reference/chichorro-3.1-rs/chichorro-html/Flask.py` → **apenas leitura**, verificar payload CTI

## Módulos envolvidos
- `app/backend/Chichorro_CTI.py`
- `app/backend/Flask.py`

---

## Factos estabelecidos (da análise A1)

### O que já corresponde entre ativo e referência 3.1
- Subfatores CI, VHE, VVE presentes
- Tabelas `CPI_VHE_Fator`, `CPI_VVE_Fator` — valores corretos
- Revestimentos VHE/VVE — valores corretos
- Fórmulas de agregação CTI por cenário — corretas

### Diferenças a aplicar

**1. Nova assinatura — 3 novos parâmetros:**
```python
# 3.0 ativo (simplificado):
def CTI(..., efetivo, saidas, Dispositivos, ..., ...)

# 3.1 referência:
def CTI(..., efetivo, RI_interv_21_efetivo, saidas, Dispositivos, ..., VHE_Dispositivos, ..., VVE_Dispositivos, ...)
```

**2. Lógica `RI_interv_21_efetivo`:**
Imediatamente após receber `efetivo`, antes de qualquer cálculo de densidade:
```python
if RI_interv_21_efetivo == 1:
    efetivo = efetivo / 2
```

**3. `VHE_Dispositivos` no cenário só-VHE:**
No ramo onde só existe VHE (não VVE), a velocidade de evacuação da VHE deve usar `VHE_Dispositivos` em vez de `Dispositivos`.

**4. `VVE_Dispositivos` no cenário só-VVE:**
No ramo onde só existe VVE (não VHE), a velocidade de evacuação da VVE deve usar `VVE_Dispositivos` em vez de `Dispositivos`.

**5. Cenário VHE + VVE:**
Verificar na referência 3.1 se usa `VVE_Dispositivos` ou `VHE_Dispositivos` para cada veia neste ramo. Aplicar conforme a referência.

### Bug crítico a corrigir
No ramo `SistemaExtincao == 'Com'`, há chamadas a `Symbol(...)` e `solve(...)` sem `import sympy`.
**Correção:** adicionar `from sympy import Symbol, solve` no topo do ficheiro (ou no início da função, se for uma importação condicional por performance).

### Refactors locais a PRESERVAR (não apagar)
- Lógica de cálculo de fumo (helpers locais não presentes na referência 3.1)
- Lógica de clarabóia
- Ajuste de percurso VVE ascendente (quando não há pisos abaixo)
Estes não devem ser removidos — são melhorias do ativo face à referência literal.

---

## Decisões fechadas
- `RI_interv_21_efetivo` defaulta a `0` no endpoint Flask CTI normal — não é input do utilizador
- `VHE_Dispositivos` e `VVE_Dispositivos`: verificar no `Flask.py` da referência se são inputs separados ou herdam de `Dispositivos`; implementar conforme a referência
- O bug sympy é crítico — deve ser corrigido nesta tarefa, não deixado para depois

## Restrições
- **Não substituir** o ficheiro inteiro pela referência — é um merge
- **Não remover** os refactors locais (fumo, clarabóia, VVE ascendente)
- **Não tocar em** `CtiPage.tsx` nesta tarefa (frontend fica para bloco B)
- `RI_interv_21_efetivo` não deve aparecer como campo no formulário CTI

---

## Ficheiros a ler antes de implementar

1. `app/backend/Chichorro_CTI.py` — ler na íntegra (ativo atual)
2. `reference/chichorro-3.1-rs/chichorro-html/Chichorro_CTI.py` — ler na íntegra (referência)
3. `reference/chichorro-3.1-rs/chichorro-html/Flask.py` — ler secção do endpoint CTI para confirmar se `VHE_Dispositivos`/`VVE_Dispositivos` são enviados separadamente pelo frontend ou derivados

---

## Próximo passo pedido ao Codex

1. Ler os três ficheiros acima
2. Confirmar se `VHE_Dispositivos`/`VVE_Dispositivos` são inputs separados no Flask 3.1
3. Aplicar as 5 alterações descritas em "Diferenças a aplicar" ao ficheiro ativo
4. Corrigir o bug sympy
5. Preservar todos os refactors locais identificados
6. Atualizar o endpoint CTI em `app/backend/Flask.py` para aceitar os novos parâmetros
7. Atualizar `docs/migration/V3_1_MATCHUP_MATRIX.md` — mudar estado CTI de `analysed` para `done` após conclusão

## Output esperado
- `app/backend/Chichorro_CTI.py` atualizado (merge 3.1 + fixes)
- `app/backend/Flask.py` com endpoint CTI atualizado
- `V3_1_MATCHUP_MATRIX.md` com CTI marcado como `done`

## Riscos / pontos de atenção
- A lógica CTI usa numpy extensivamente — atenção a operações sobre arrays ao modificar o efetivo
- O ramo `SistemaExtincao == 'Com'` com sympy é o mais complexo — testar com um valor conhecido após a correção
- Se `VHE_Dispositivos`/`VVE_Dispositivos` não forem encontrados no Flask 3.1 como inputs separados, usar `Dispositivos` como default e deixar nota nos docs
- Após esta tarefa, o endpoint Flask CTI muda de assinatura — qualquer chamada com a assinatura antiga vai falhar
