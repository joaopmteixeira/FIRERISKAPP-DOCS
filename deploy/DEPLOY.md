# Colocar o CHICHORRO no seu domínio

## Arranque local rápido

1. Na raiz do projeto: `python -m pip install -r requirements.txt`
2. Com [Node.js](https://nodejs.org/) (LTS): `cd frontend`, `npm install`, `npm run build` (gera `frontend/dist/`).
3. Na pasta `PYTHON`: `python Flask.py`
4. No browser: **http://127.0.0.1:50/** — **nova interface** (React compilada) e **API** no mesmo servidor.

**Desenvolvimento com recarregamento rápido:** com o Flask na porta 50, noutro terminal `cd frontend && npm run dev` e abra **http://localhost:5173/** (o Vite faz proxy da API).

O ficheiro `index.html` na raiz do repositório é a UI antiga; **já não é servido pelo Flask** (fica só como arquivo de referência, se precisar).

## Arquitetura recomendada

1. **Frontend** — build estático do projeto `frontend/` (Vite + React), servido por nginx ou CDN.
2. **Backend** — `PYTHON/Flask.py` atrás de **gunicorn** na mesma máquina (ou noutro host).
3. **DNS** — um registo `A` (ou `AAAA`) do domínio para o IP do servidor.

Com o nginx de exemplo em `deploy/nginx-chichorro.example.conf`, o browser fala sempre com `https://SEU_DOMÍNIO`: ficheiros estáticos em `/`, autenticação em `/auth/*` e pedidos `/POI`, `/CTI`, `/DPI`, `/ESCI`, `/RI` são enviados para o Flask. Não precisa de CORS entre origens diferentes.

Se o frontend estiver noutro domínio (ex.: Netlify), defina `VITE_API_BASE=https://api.SEU_DOMÍNIO` no build e configure `CHICHORRO_CORS_ORIGINS` no servidor (ver abaixo).

## Backend (Linux típico)

```bash
cd PYTHON
python -m venv .venv
source .venv/bin/activate
pip install -r ../requirements.txt
gunicorn -w 4 -b 127.0.0.1:8000 wsgi:app
```

Em Windows para desenvolvimento pode continuar a usar `python Flask.py` (porta 50), alinhada com o proxy do Vite.

### Variáveis de ambiente do backend (obrigatórias em produção)

```bash
# Chave de sessão Flask (obrigatória: usar valor longo/aleatório)
export CHICHORRO_SECRET_KEY="trocar-por-valor-seguro-e-longo"

# Credenciais aceites no login backend (até 8 pares)
export CHICHORRO_AUTH_USER_1="admin"
export CHICHORRO_AUTH_PASS_1="trocar-por-password-forte"
# export CHICHORRO_AUTH_USER_2="..."
# export CHICHORRO_AUTH_PASS_2="..."
```

Sem `CHICHORRO_AUTH_USER_*` / `CHICHORRO_AUTH_PASS_*`, o backend só permite fallback em modo debug (dev local). Em produção, configure sempre estes pares.

## Frontend

```bash
cd frontend
npm install
cp .env.example .env
npm run dev
```

Produção:

```bash
npm run build
```

Pode **(A)** servir só o Flask/gunicorn: este projeto já entrega `frontend/dist/` em `/` (e a API nos mesmos URLs de sempre), ou **(B)** copiar `frontend/dist/` para o nginx e fazer proxy só dos caminhos `/POI`, `/CTI`, etc., para o backend.

## CORS (só se UI e API forem origens diferentes)

No servidor, defina por exemplo:

```bash
export CHICHORRO_CORS_ORIGINS="https://app.seudominio.pt,https://seudominio.pt"
```

O `Flask.py` lê esta variável quando existir.

## Certificados HTTPS

Use [Let’s Encrypt](https://letsencrypt.org/) (certbot) com o bloco `server` na porta 443 do exemplo nginx.

## UI antiga

O `index.html` na raiz não é usado pelo servidor; a interface atual é a de `frontend/`.
