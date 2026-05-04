# PRD — FireRiskApp / CHICHORRO

Última atualização: 2026-05-04  
Estado do produto: CHICHORRO v3.1 implementado; validação end-to-end final pendente  
Produto: FireRiskApp — avaliação de risco de incêndio em edifícios históricos

---

## 1. Resumo

O FireRiskApp é uma aplicação web para avaliação do risco de incêndio em edifícios existentes com valor patrimonial, baseada no método CHICHORRO desenvolvido no contexto da FEUP. A aplicação permite recolher dados técnicos do edifício, calcular os quatro fatores do modelo, obter o Índice de Risco de Incêndio (RI), classificar o risco numa escala A++ a F e simular medidas de intervenção que reduzem o risco.

O produto atual corresponde à implementação web do modelo CHICHORRO v3.1, com frontend em React/TypeScript e backend Flask/Python. A lógica de cálculo vive no backend; o frontend deve fornecer uma experiência orientada, validada e persistente para técnicos que precisam de preencher muitos campos sem perder contexto.

---

## 2. Problema

A avaliação de risco de incêndio em edifícios históricos exige um método técnico consistente, muitos parâmetros de entrada e cálculos encadeados entre módulos. As versões anteriores do CHICHORRO existem em implementações legacy, com maior dificuldade de manutenção, menor ergonomia de utilização e menos preparação para deployment moderno.

O FireRiskApp resolve este problema ao transformar o método CHICHORRO numa aplicação web utilizável, preservando a fidelidade do modelo matemático e tornando mais simples:

- preencher os fatores de risco por etapas;
- calcular subfatores e resultados globais;
- identificar campos em falta ou resultados desatualizados;
- guardar, importar e exportar sessões de avaliação;
- comparar o RI inicial com o RI após intervenções.

---

## 3. Objetivos

1. Disponibilizar uma aplicação web autenticada para aplicar o método CHICHORRO v3.1.
2. Garantir paridade funcional com a referência CHICHORRO v3.1 nos cálculos de POI, CTI, DPI, ESCI, RI e intervenções.
3. Reduzir erros de preenchimento através de validação, estados visuais claros e navegação entre módulos.
4. Permitir que uma avaliação seja interrompida e retomada por persistência local e export/import de sessão JSON.
5. Suportar simulação de intervenções ativas e passivas, incluindo conjuntos predefinidos, RI pós-intervenção e custo estimado.
6. Preparar a base técnica para fases futuras, incluindo modelo simplificado CHICHORRO 4.0 e eventual deployment em VPS.

---

## 4. Não Objetivos

Nesta fase, o produto não pretende:

- substituir a validação técnica por especialista de segurança contra incêndio;
- fornecer base de dados multiutilizador persistente no servidor;
- gerar relatórios oficiais assinados ou documentos PDF formais;
- integrar mapas, georreferenciação ou inventário patrimonial;
- implementar o modelo simplificado CHICHORRO 4.0;
- suportar edição colaborativa simultânea da mesma avaliação;
- garantir compatibilidade com sessões geradas pela versão legacy 3.0 (sessões antigas podem falhar ao importar).

---

## 5. Utilizadores

### Técnico avaliador

Profissional que recolhe dados do edifício e executa a avaliação CHICHORRO. Precisa de um fluxo previsível, validação forte, persistência de dados e resultados transparentes.

### Investigador ou docente

Utilizador que compara resultados com as dissertações e versões anteriores do método. Precisa de rastreabilidade, paridade e acesso claro aos subfatores.

### Decisor / dono do edifício

Interessa-se pelo resultado final, aceitabilidade do risco e impacto das intervenções. Pode não preencher o modelo, mas consome conclusões e cenários.

### Administrador técnico

Mantém deploy, credenciais, variáveis de ambiente, backend e frontend. Precisa de uma arquitetura simples de operar e diagnosticar.

---

## 6. Fluxo Principal

1. Utilizador autentica-se no ecrã de login.
2. Utilizador preenche e calcula o módulo POI.
3. Utilizador preenche e calcula o módulo CTI.
4. Utilizador preenche e calcula o módulo DPI.
5. Utilizador preenche e calcula o módulo ESCI.
6. Utilizador acede a Resultados e calcula o RI global.
7. A aplicação apresenta valor de RI, classe de risco e aceitabilidade face ao limite RI_RIA.
8. Opcionalmente, utilizador abre Intervenções, escolhe medidas individuais ou conjuntos predefinidos e calcula RI pós-intervenção.
9. Utilizador exporta a sessão para JSON, limpa a sessão ou termina sessão.

---

## 7. Requisitos Funcionais

### RF-01 — Autenticação

- A aplicação deve exigir autenticação antes de permitir acesso a `/app/*`.
- O backend deve proteger endpoints de cálculo contra pedidos não autenticados.
- O login deve suportar credenciais configuradas por variáveis de ambiente.
- Em desenvolvimento, a aplicação pode aceitar credenciais não vazias se a autenticação não estiver configurada e o backend estiver em modo debug.
- O logout deve invalidar a sessão no servidor e redirecionar para login.

### RF-02 — Navegação

- A aplicação deve disponibilizar navegação para POI, CTI, DPI, ESCI, Resultados e Intervenções.
- Ao entrar em `/app`, o utilizador deve ser redirecionado para POI.
- Rotas internas devem permanecer protegidas por `RequireAuth`.
- A navegação deve preservar dados introduzidos durante a sessão.

### RF-03 — POI

O módulo POI deve permitir preencher, calcular e persistir os subfatores:

- CC — Construção e Conservação;
- IEE — Instalações Elétricas e Equipamentos;
- IA — Instalações de Apoio;
- ICONFA — Instalações de Confeção Alimentar;
- ICONSA — Instalações de Conservação Sanitária;
- IVCA — Instalações de Ventilação, Climatização e Aquecimento;
- ILGC — Instalações de Líquidos e Gases Combustíveis;
- EF — Exposição ao Fogo;
- EA — Empenas e Afastamentos;
- FA — Fracionamento e Aberturas;
- PPP — Plano de Prevenção e Proteção;
- ATIV — Atividade / utilização-tipo.

Requisitos específicos:

- O campo `POI_CC_Idade` deve ser recolhido em CC e usado no cálculo do limite aceitável RI_RIA.
- Subfatores com valor `0.00` devem ser tratados como "Não se aplica" e excluídos da média global do módulo.
- O resultado global POI deve ser guardado em sessão e disponibilizado à página de Resultados.

### RF-04 — CTI

O módulo CTI deve calcular as consequências do incêndio para os utilizadores, incluindo:

- características gerais do edifício;
- carga de ocupação / efetivo;
- saídas;
- sistemas de deteção, extinção e controlo de fumo;
- via horizontal de evacuação (VHE);
- via vertical de evacuação (VVE).

Requisitos específicos:

- `VHE_Dispositivos` e `VVE_Dispositivos` devem ser recolhidos separadamente.
- O campo `TipoEdif` deve manter sincronização funcional com `POI_ATIV_TipoEdif`, sem bloquear indevidamente a edição.
- Inputs numéricos vindos do frontend devem ser normalizados pelo backend antes do cálculo.
- O resultado CTI deve incluir os resultados intermédios necessários e o valor global `CTI`.

### RF-05 — DPI

O módulo DPI deve permitir preencher, calcular e persistir:

- REIC — Resistência ao Incêndio de Elementos Construtivos;
- EI — Estanquidade e Integridade;
- VDGF — Vãos, Diedros e Guarda-fogos;
- PE — Paredes Exteriores;
- OGS — Organização e Gestão da Segurança.

Requisitos específicos:

- OGS deve seguir a lógica v3.1 com campos de organização, formação e regulamento.
- Resultados parciais e global devem ser persistidos.
- Alterações após cálculo devem marcar o resultado como desatualizado.

### RF-06 — ESCI

O módulo ESCI deve permitir preencher, calcular e persistir:

- GP — Grau de Prontidão;
- SID — Sinalização, Iluminação e Deteção;
- AE — Acessibilidade Exterior;
- HE — Hidrantes Exteriores;
- EXT — Extintores;
- RIA — Rede de Incêndio Armada / Coluna Seca;
- CPB — Corpo de Bombeiros.

Requisitos específicos:

- GP deve suportar `ESCI_GP_Auto` quando a deteção/alerta é automática.
- EXT deve suportar `ESCI_EXT_Formacao`.
- RIA deve suportar `ESCI_RIA_Formacao` e `ESCI_RIA_CS`.
- Resultados parciais e global devem ser persistidos.

### RF-07 — Resultado RI

A página de Resultados deve:

- mostrar os valores globais calculados de POI, CTI, DPI e ESCI;
- indicar módulos em falta ou com erro;
- validar se existem inputs necessários antes de calcular o RI;
- enviar payload completo para `/RI`;
- apresentar RI numérico, classe de risco e limite aceitável RI_RIA;
- indicar se o RI é aceitável ou não aceitável;
- marcar o RI como desatualizado quando inputs mudam após o cálculo.

Fórmula do produto:

```text
RI = POI * CTI * ((DPI + ESCI) / 2)
```

A classificação deve seguir a escala CHICHORRO v3.1 implementada e validada no backend, com classes de A++ a F.

### RF-08 — Intervenções

O módulo de Intervenções deve:

- permitir seleção individual de 34 intervenções ativas e passivas;
- permitir seleção por conjuntos predefinidos baseados na referência v3.1;
- enviar o estado completo da avaliação e intervenções para `/RI/interv`;
- devolver RI original, RI pós-intervenção, escala pós-intervenção e custo estimado;
- apresentar comparação clara entre cenário atual e cenário pós-intervenção.

### RF-09 — Persistência de Sessão

A aplicação deve guardar no browser:

- resultados globais por módulo;
- resultados parciais por subfator;
- inputs submetidos por módulo;
- valores atuais dos formulários;
- estados de erro, aviso, campos em falta e resultados desatualizados;
- estado colapsado/expandido dos subfatores.

O utilizador deve conseguir:

- exportar a sessão atual para JSON;
- importar uma sessão JSON válida;
- limpar a sessão;
- receber aviso antes de sair quando existem dados de sessão.

### RF-10 — Validação e Estados Visuais

- Ao calcular com campos obrigatórios em falta, a aplicação deve mostrar mensagem específica.
- O campo em falta deve ser destacado visualmente.
- Quando um input muda após cálculo, o resultado antigo deve permanecer visível, mas marcado como desatualizado.
- O valor global do módulo deve indicar "Valor desatualizado" quando aplicável.
- A página de Resultados deve guiar o utilizador para o campo em falta quando possível.

---

## 8. Requisitos Não Funcionais

### RNF-01 — Fidelidade de cálculo

O backend é a fonte de verdade para o modelo CHICHORRO. O frontend não deve replicar lógica matemática crítica, exceto validações de UI e preparação de payload.

### RNF-02 — Manutenibilidade

POI, DPI e ESCI devem usar o padrão de ficheiros `*definitions.ts` + `*FactorSection.tsx`, mantendo os campos, opções e visibilidade condicional declarativos.

### RNF-03 — Segurança

- Cookies de sessão devem ser `HttpOnly`.
- `SESSION_COOKIE_SAMESITE` e `SESSION_COOKIE_SECURE` devem ser configuráveis por ambiente.
- CORS deve ser limitado por `CHICHORRO_CORS_ORIGINS` em produção.
- Endpoints POST de cálculo devem exigir autenticação.

### RNF-04 — Compatibilidade

- A aplicação deve correr em browsers modernos.
- A sessão exportada deve ser JSON legível e versionável.
- O build frontend deve ser servível pelo Flask em produção, além de suportar Cloudflare Pages.

### RNF-05 — Performance

- Cálculos devem responder em tempo aceitável para utilização interativa (máximo 2 segundos em condições de backend ativo).
- O frontend deve manter UI responsiva durante chamadas ao backend.
- O backend deve expor health check para monitorização.

### RNF-06 — Operação

- Frontend: React + TypeScript + Vite + Tailwind CSS.
- Backend: Flask + Python.
- Deploy atual: Cloudflare Pages para frontend e Render para backend.
- Deploy futuro recomendado: VPS com nginx, React estático e Flask/Gunicorn em proxy reverso.

---

## 9. API Principal

| Endpoint | Método | Descrição |
|---|---:|---|
| `/auth/login` / `/login` | POST | Autenticação |
| `/auth/logout` / `/logout` | POST | Terminar sessão |
| `/auth/me` / `/me` | GET | Estado de autenticação |
| `/health` | GET | Health check |
| `/POI` | POST | Cálculo global POI |
| `/POI/<subfator>` | POST | Cálculo parcial POI |
| `/CTI` | POST | Cálculo CTI |
| `/DPI` | POST | Cálculo global DPI |
| `/DPI/<subfator>` | POST | Cálculo parcial DPI |
| `/ESCI` | POST | Cálculo global ESCI |
| `/ESCI/<subfator>` | POST | Cálculo parcial ESCI |
| `/RI` | POST | Cálculo final RI |
| `/RI/interv` / `/RI_interv` | POST | Cálculo de RI pós-intervenção |

---

## 10. Modelo de Dados de Sessão

Principais chaves em `sessionStorage`:

| Chave | Conteúdo |
|---|---|
| `chichorro:module-results` | últimos valores globais válidos de POI, CTI, DPI e ESCI |
| `chichorro:module-results-stale` | últimos valores globais marcados como desatualizados |
| `chichorro:module-factor-results` | resultados por subfator |
| `chichorro:module-inputs` | inputs submetidos por módulo |
| `chichorro:form:<formKey>` | estado atual de cada formulário |
| `chichorro:computed:<key>` | resultados intermédios calculados |
| `collapse:<module>:<subfactorKey>` | estado colapsado/expandido |
| `err:<formKey>` | erro persistido |
| `warn:<formKey>` | aviso persistido |
| `miss:<formKey>` | campo em falta |
| `stale:<formKey>` | estado desatualizado |

Eventos globais esperados:

- `chichorro:module-results-updated`;
- `chichorro:session-data-updated`.

---

## 11. UX e Interface

Princípios:

- fluxo por módulos, com navegação persistente no topo;
- cartões de subfator colapsáveis;
- mensagens de erro diretas e acionáveis;
- valores antigos preservados quando ficam desatualizados;
- resultados globais visíveis em cada módulo;
- ações de sessão visíveis na página de Resultados;
- confirmação antes de limpar sessão ou terminar sessão;
- import/export por JSON para avaliações longas.

Estados essenciais:

- vazio;
- preenchido mas não calculado;
- calculado;
- calculado mas desatualizado;
- erro de validação;
- erro de backend;
- carregamento;
- sessão importada/exportada/limpa com sucesso.

---

## 12. Critérios de Aceitação

### CA-01 — Autenticação

- Um utilizador não autenticado não acede a `/app/poi`.
- Um pedido POST não autenticado para `/RI` devolve erro 401.
- Após logout, voltar a `/app/*` redireciona para login.

### CA-02 — Cálculo dos módulos

- Cada subfator de POI, DPI e ESCI pode ser preenchido e calculado.
- CTI calcula com inputs numéricos válidos e dispositivos VHE/VVE separados.
- Cada módulo guarda resultado global em sessão.
- Alterar um campo após cálculo marca o resultado como desatualizado.

### CA-03 — RI

- O botão Calcular RI fica funcional quando todos os inputs necessários existem.
- Se faltar um campo, a página indica o campo e permite navegar para ele quando aplicável.
- O resultado apresenta RI, classe e aceitabilidade.
- Alterar dados após cálculo mostra aviso de RI desatualizado.

### CA-04 — Intervenções

- É possível selecionar qualquer uma das 34 intervenções.
- É possível aplicar conjuntos predefinidos.
- O cálculo devolve RI original, RI pós-intervenção e custo estimado.
- A comparação é compreensível para o utilizador.

### CA-05 — Sessão

- Exportar sessão gera JSON com estado suficiente para retomar a avaliação.
- Importar sessão repõe inputs, resultados e estados relevantes.
- Limpar sessão remove valores e resultados do browser.
- Sair com dados ativos mostra confirmação e opção de exportar.

### CA-06 — Paridade

- O `parity_runner.py` deve passar para os cenários de referência definidos.
- Alterações ao modelo devem ser comparadas com `reference/chichorro-3.1-rs/` antes de serem aceites.

---

## 13. Métricas de Sucesso

- 100% dos módulos CHICHORRO v3.1 disponíveis no frontend.
- 0 falhas nos testes de paridade backend definidos.
- Avaliação completa executável no browser sem edição manual de JSON.
- Utilizador consegue exportar e importar sessão sem perda de dados críticos.
- Teste end-to-end aprovado pelo João.
- Tempo de preenchimento percebido menor face à versão legacy, por validação e navegação mais claras.

---

## 14. Roadmap

### Fase 1 — CHICHORRO 3.0

Estado: concluída.

- Migração base para React.
- Backend Flask consolidado.
- Fluxo POI → CTI → DPI → ESCI → RI.
- Auth, sessão local e import/export.

### Fase 2 — CHICHORRO 3.1

Estado: implementação concluída; pendente teste end-to-end final.

- Atualizações v3.1 de CTI, DPI, ESCI e RI.
- Escala A++ a F.
- Aceitabilidade RI_RIA via idade de construção.
- Módulo de intervenções com 34 medidas.
- UX de validação e resultados desatualizados.

### Fase 3 — CHICHORRO 4.0 / Modelo Simplificado

Estado: futuro.

- Analisar referência v2.0 de Ricardo Ferreira e Bruno Silva.
- Definir inputs simplificados.
- Implementar cálculo simplificado no backend.
- Criar UI própria para avaliação rápida.
- Comparar resultados do modelo simplificado com o modelo completo.

### Fase 4 — Produto Operacional

Estado: futuro.

- Deployment em VPS.
- Autenticação mais robusta.
- Persistência servidor/base de dados.
- Exportação de relatórios.
- Gestão de avaliações por edifício.
- Eventual georreferenciação e histórico de intervenções.

Para detalhe de tarefas, estado atual e itens pendentes, ver [`docs/NEXT_STEPS.md`](NEXT_STEPS.md).

---

## 15. Riscos e Dependências

| Risco | Impacto | Mitigação |
|---|---|---|
| Divergência entre frontend e backend | Resultados errados ou impossíveis de calcular | Backend como fonte de verdade; contratos de payload documentados |
| Divergência face à referência v3.1 | Perda de validade técnica | Paridade com `reference/chichorro-3.1-rs/` |
| Sessões antigas incompatíveis com campos v3.1 | Importações podem falhar | Versionar formato de sessão e normalizar migrações futuras |
| Render free tier com cold start | Experiência inicial lenta | VPS/nginx em fase operacional |
| Fórmulas sensíveis a strings de opção | Erros silenciosos | Centralizar opções em definitions e validar no backend |
| CTI com física complexa | Risco de regressão | Testes unitários/paridade adicionais para cenários CTI |

---

## 16. Questões em Aberto

1. Deve existir um formato de relatório oficial exportável em PDF ou DOCX?
2. ~~O formato JSON de sessão deve passar a ter campo explícito de versão?~~ **Fechada — já implementado:** o JSON exportado inclui `version: 1` e `app: "chichorro"`; o import valida ambos os campos antes de aceitar o ficheiro.
3. Haverá perfis de utilizador distintos, como avaliador, revisor e administrador?
4. O produto final deve guardar avaliações em servidor ou continuar local/exportável?
5. Que cenários de teste end-to-end serão considerados suficientes para aprovação final?
6. O deploy final será mantido em Cloudflare/Render ou migrado para VPS único?
7. Devem ser documentados contratos completos de payload por endpoint?

---

## 17. Referências Internas

- `docs/PROJECT_OVERVIEW.md`
- `docs/ARCHITECTURE.md`
- `docs/USER_GUIDE.md`
- `docs/NEXT_STEPS.md`
- `docs/migration/V3_1_MATCHUP_MATRIX.md`
- `docs/migration/PAGE_BACKEND_MAPPING.md`
- `app/frontend/src/App.tsx`
- `app/frontend/src/pages/RiPage.tsx`
- `app/backend/Flask.py`
- `app/backend/Chichorro_RI.py`
- `app/backend/Chichorro_RI_inter.py`
