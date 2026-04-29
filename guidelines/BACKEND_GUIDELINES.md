# Backend Guidelines

## Objetivo
Manter o backend em `app/backend/` como núcleo técnico do sistema, alinhado com os domínios de risco e preparado para matchup com a referência 3.1.

Estrutura de ficheiros em `docs/ARCHITECTURE.md`.

## Regras principais

### 1. Os módulos `Chichorro_*` são a fonte principal da lógica por domínio
Toda a evolução de cálculo e regras deve preservar esta separação funcional enquanto a estrutura atual existir.

### 2. `Flask.py` não deve concentrar toda a lógica
A camada Flask deve:
- receber pedidos
- validar e fazer parsing inicial
- chamar os módulos certos
- devolver respostas estruturadas

### 3. Matching com a referência 3.1
No modo `v3_1_matchup`:
- usar `reference/chichorro-3.1-rs/` para perceber comportamento esperado
- implementar ou adaptar no backend ativo `app/backend/`
- não duplicar o backend antigo sem necessidade real

### 4. Comparação com a referência v3.0
No modo `v3_0_legacy_compare`:
- usar `reference/chichorro-3.0-jt/` como referência histórica e funcional
- não usar essa pasta como destino principal de desenvolvimento novo

### 5. Preparação futura para v4.0
No modo `v4_0_research`:
- usar `reference/chichorro-2.0-rf/` como referência metodológica
- não introduzir refactors grandes no backend ativo sem decisão explícita

## Não fazer
- não espalhar scripts principais fora de `app/backend/`
- não misturar lógica de cálculo com lógica HTTP sem controlo
- não duplicar regras entre módulos e frontend
- não alterar resultados esperados sem validação ou paridade

## Refactor futuro desejável
Quando a v3.1 estiver estabilizada, a estrutura poderá evoluir para algo como:
- `app/backend/api/`
- `app/backend/services/`
- `app/backend/core/`
- `app/backend/schemas/`

Até lá, a prioridade é:
1. estabilidade
2. clareza funcional
3. matchup correto com a referência 3.1