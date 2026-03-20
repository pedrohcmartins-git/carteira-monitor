# Carteira Monitor — Guia de Setup

## Arquivos do projeto

```
carteira-app/
├── index.html    ← Dashboard principal (após login)
├── login.html    ← Tela de login
├── config.js     ← Suas credenciais (preencher!)
└── README.md     ← Este arquivo
```

---

## Passo 1 — Preencher config.js

Abra o arquivo `config.js` e substitua os valores:

```js
export const SUPABASE_URL      = 'https://SEU-PROJETO.supabase.co'
export const SUPABASE_ANON_KEY = 'sua-anon-key-aqui'
```

Onde encontrar:
- Supabase → seu projeto → ⚙️ Settings → API
- Copie **Project URL** e **anon public key**

---

## Passo 2 — Subir no Vercel (gratuito)

### Opção A — via GitHub (recomendado)

1. Crie conta em [github.com](https://github.com)
2. Crie repositório novo: `carteira-monitor`
3. Faça upload dos 4 arquivos (index.html, login.html, config.js, README.md)
4. Acesse [vercel.com](https://vercel.com) → **Add New Project**
5. Conecte ao GitHub e selecione o repositório
6. Clique **Deploy** — pronto!
7. Seu app estará em: `https://carteira-monitor.vercel.app`

### Opção B — via Vercel CLI

```bash
npm i -g vercel
cd carteira-app
vercel --prod
```

---

## Passo 3 — Criar primeiro usuário admin

1. No Supabase → **Authentication** → **Users** → **Invite user**
2. Digite seu e-mail e clique Invite
3. Você receberá um e-mail para definir sua senha
4. Após definir a senha, execute no SQL Editor:

```sql
-- Substitua pelo seu user_id (aparece em Authentication → Users)
-- e pelo id do grupo criado anteriormente
INSERT INTO public.group_members (group_id, user_id, role)
VALUES (
  'ID-DO-SEU-GRUPO',   -- copie de: SELECT id FROM public.groups;
  'ID-DO-SEU-USUARIO', -- copie de Authentication → Users
  'admin'
);
```

---

## Passo 4 — Convidar outros usuários

### Via painel Supabase (mais simples):
1. Authentication → Users → Invite user
2. Após o usuário aceitar o convite, execute o SQL acima com `role = 'viewer'`

### Via app (na aba Admin):
- A aba Admin aparece somente para usuários com `role = 'admin'`
- Permite convidar por e-mail diretamente

---

## Passo 5 — Configurar chaves de API de cotações

Após fazer login como admin:
1. Vá na aba **Admin**
2. Em "Chaves de API", cole seu token do brapi.dev e finnhub.io
3. Clique Salvar — ficam guardadas no seu navegador

**Obter chaves gratuitas:**
- brapi.dev → Criar conta → Dashboard → Copiar token
- finnhub.io → Criar conta → Dashboard → API key

---

## Formato do arquivo de carteira

CSV ou XLSX com duas colunas:

```
TICKER,QUANTIDADE
PETR4,100
WEGE3,50
VALE3,200
AAPL,10
NVDA,5
```

- Ativos com número no ticker → B3 (ex: PETR4)
- Ativos só com letras → EUA (ex: AAPL)

---

## Alertas (configuração futura — WhatsApp/Email automático)

Os alertas são salvos no banco. Para envio automático:

1. No Supabase → **Edge Functions** → New function: `send-alerts`
2. A função roda via cron (a cada hora) e verifica as condições
3. Para e-mail: integrar com [Resend](https://resend.com) (gratuito até 3.000/mês)
4. Para WhatsApp: integrar com [Twilio](https://twilio.com) ou [Z-API](https://z-api.io)

Me peça o código da Edge Function quando quiser configurar isso!

---

## Adicionar mais fundos (grupos)

```sql
-- Crie um novo grupo
INSERT INTO public.groups (name, slug)
VALUES ('Fundo Beta', 'fundo-beta');

-- Adicione um admin para esse grupo
INSERT INTO public.group_members (group_id, user_id, role)
VALUES ('ID-DO-NOVO-GRUPO', 'ID-DO-USUARIO', 'admin');
```

Cada grupo é completamente isolado — usuários de um fundo nunca veem dados de outro.

---

## Custos estimados

| Serviço | Gratuito inclui | Quando pagar |
|---|---|---|
| Supabase | 2 projetos, 500MB, 50k usuários | Mais projetos → $25/mês |
| Vercel | Ilimitado (hobby) | Domínio customizado → apenas custo do domínio |
| brapi.dev | Cotações B3 básicas | Volume alto → ~R$50/mês |
| Finnhub | 60 req/min | Alto volume → $0–50/mês |

**Para 2–5 fundos com uso normal: custo zero ou próximo de zero.**
