# Handoff A1 — Análise CTI 3.1

Data: 2026-04-20
Produzido por: Claude (arquiteto técnico)
Para: Codex (implementador)

---

## Tipo de tarefa
`v3_1_matchup`

## Objetivo
Comparar o módulo CTI da referência 3.1 com o módulo CTI ativo e determinar se existem diferenças funcionais.
Esta é uma tarefa de **análise e reporte** — não de implementação.
No final, atualizar `docs/migration/V3_1_MATCHUP_MATRIX.md` com o estado real do CTI.

## Estado atual
- `app/backend/Chichorro_CTI.py` — módulo ativo, baseado na v3.0
- `reference/chichorro-3.1-rs/chichorro-html/Chichorro_CTI.py` — referência 3.1, **ainda não lida**
- `app/frontend/src/pages/CtiPage.tsx` — página ativa, sem componentes dedicados (sem `CtiFactorSection.tsx` / `ctiDefinitions.ts`)
- Estado na matriz: `unknown`

## Zona do repositório a usar
- `reference/chichorro-3.1-rs/` → **apenas leitura**, fonte de comparação
- `app/backend/Chichorro_CTI.py` → leitura para comparação
- `docs/migration/V3_1_MATCHUP_MATRIX.md` → atualizar no final
- **Não tocar em** `app/frontend/` nem em `app/backend/` para além da atualização dos docs

## Módulos envolvidos
- `app/backend/Chichorro_CTI.py`
- `reference/chichorro-3.1-rs/chichorro-html/Chichorro_CTI.py`
- `docs/migration/V3_1_MATCHUP_MATRIX.md`

## Factos conhecidos
- A dissertação de Rui Sobral menciona melhorias nos subfatores CPIVHE e CPIVVE do CTI
- Os outros módulos (POI, ESCI, DPI, RI) já foram analisados — as diferenças estão documentadas em `docs/migration/V3_1_DIFF_ANALYSIS.md`
- O CTI é o único módulo com estado `unknown` na matriz
- O CTI na app ativa tem inputs numéricos complexos (área, pé-direito, efetivo, saídas, dimensões VHE/VVE) — não usa o padrão `definitions.ts` dos outros módulos

## Decisões fechadas
- Não implementar nada antes de confirmar o que mudou
- Se não houver diferenças, o estado passa a `done` (assumindo que o ativo já está correto)
- Se houver diferenças, documentar exatamente o que muda e deixar a implementação para o handoff seguinte

## Restrições
- **Não editar** `app/backend/Chichorro_CTI.py` nesta tarefa
- **Não editar** `app/frontend/` nesta tarefa
- Tratar `reference/chichorro-3.1-rs/` apenas como fonte de leitura

## Ficheiros a ler (por esta ordem)

1. `reference/chichorro-3.1-rs/chichorro-html/Chichorro_CTI.py` — ler na íntegra
2. `app/backend/Chichorro_CTI.py` — ler na íntegra para comparação
3. `docs/migration/V3_1_DIFF_ANALYSIS.md` — contexto (leitura rápida da secção CTI, §7)

## Próximo passo pedido ao Codex

1. Ler ambos os ficheiros CTI (referência 3.1 e ativo)
2. Comparar:
   - assinaturas das funções
   - subfatores presentes (CI, VHE, VVE, CPIVHE, CPIVVE e outros)
   - tabelas de valores / condicionais
   - parâmetros novos, removidos ou renomeados
3. Produzir um relatório curto com:
   - lista de diferenças encontradas (ou confirmação de que não há)
   - para cada diferença: campo afetado, valor antigo, valor novo
4. Atualizar a secção CTI em `docs/migration/V3_1_MATCHUP_MATRIX.md`:
   - mudar estado de `unknown` para `done`, `partial`, `analysed` ou `missing` conforme o resultado
   - preencher "O que já corresponde" e "O que falta" na secção de detalhe CTI

## Output esperado
- Relatório de diferenças CTI (pode ser inline no ficheiro de matchup ou num comentário)
- `V3_1_MATCHUP_MATRIX.md` atualizado com estado e detalhe do CTI
- **Não iniciar implementação** — apenas análise e documentação

## Riscos / pontos de atenção
- CTI é o módulo mais complexo do modelo (física, numpy, cálculos iterativos) — ler com cuidado
- Se existirem novos subfatores (CPIVHE/CPIVVE alterados), as mudanças podem ser significativas
- Não confundir parâmetros CTI com parâmetros que o módulo de Intervenções passa ao CTI (esses são modificações temporárias para recálculo, não alterações ao modelo base)
