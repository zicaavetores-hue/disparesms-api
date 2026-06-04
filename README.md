# 🚀 DisparesSMS — Plataforma SaaS de Disparo de SMS

> Plataforma multi-tenant completa para disparo de SMS em massa, transacional e recuperação de vendas.

---

## 📁 Estrutura do Projeto

```
disparesms/
├── apps/
│   ├── api/                  ← Backend NestJS (esta pasta)
│   │   ├── prisma/
│   │   │   ├── schema.prisma ← Banco de dados (todas as tabelas)
│   │   │   └── seed.ts       ← Dados iniciais (planos + admin)
│   │   ├── src/
│   │   │   ├── modules/
│   │   │   │   ├── auth/           ← Login, JWT, API Key
│   │   │   │   ├── tenants/        ← Configurações da empresa
│   │   │   │   ├── users/          ← Gestão de equipe
│   │   │   │   ├── contacts/       ← Contatos + importação CSV
│   │   │   │   ├── campaigns/      ← Campanhas + disparo + fila
│   │   │   │   ├── messages/       ← Histórico + dashboard
│   │   │   │   ├── billing/        ← Créditos + Asaas/Stripe
│   │   │   │   ├── admin/          ← Painel administrativo
│   │   │   │   ├── sms-providers/  ← Zenvia / Twilio / Infobip
│   │   │   │   └── webhooks/       ← Callbacks dos provedores
│   │   │   └── common/
│   │   │       ├── prisma/         ← PrismaService global
│   │   │       └── decorators/     ← @CurrentUser, @CurrentTenant
│   │   └── .env.example      ← Todas as variáveis necessárias
│   └── web/                  ← Frontend Next.js (a criar)
└── package.json
```

---

## ⚡ Como rodar em 5 minutos

### 1. Pré-requisitos
- Node.js 18+
- PostgreSQL (use [Supabase](https://supabase.com) — gratuito)
- Redis (use [Upstash](https://upstash.com) — gratuito)

### 2. Instalar dependências
```bash
cd apps/api
npm install
```

### 3. Configurar variáveis de ambiente
```bash
cp .env.example .env
# Edite o .env com suas credenciais
```

**Mínimo para funcionar:**
```env
DATABASE_URL="postgresql://..."   # Supabase
REDIS_URL="redis://..."           # Upstash
JWT_SECRET="qualquer-string-longa-aqui"
```

### 4. Criar banco de dados
```bash
npm run db:migrate    # Cria todas as tabelas
npm run db:seed       # Cria planos + usuário admin
```

### 5. Rodar o servidor
```bash
npm run dev
```

✅ API rodando em: http://localhost:3001/api/v1
📚 Documentação: http://localhost:3001/docs

---

## 🔑 Credenciais iniciais (após seed)

| Usuário | E-mail | Senha |
|---------|--------|-------|
| Admin da plataforma | admin@disparesms.com.br | Admin@2025! |
| Demo (Fruitfy) | dev@fruitfy.com.br | Fruitfy@2025! |

---

## 📡 Endpoints principais

### Autenticação
```
POST /api/v1/auth/register   ← Cadastrar nova empresa
POST /api/v1/auth/login      ← Login
GET  /api/v1/auth/me         ← Perfil do usuário logado
```

### Dashboard
```
GET  /api/v1/messages/dashboard  ← Métricas gerais
```

### Contatos
```
GET    /api/v1/contacts           ← Listar contatos
POST   /api/v1/contacts           ← Criar contato
POST   /api/v1/contacts/import/csv ← Importar CSV
GET    /api/v1/contacts/lists/all ← Listar grupos
POST   /api/v1/contacts/lists     ← Criar grupo
```

### Campanhas
```
GET    /api/v1/campaigns          ← Listar campanhas
POST   /api/v1/campaigns          ← Criar campanha
POST   /api/v1/campaigns/:id/launch ← Disparar agora
GET    /api/v1/campaigns/:id/stats  ← Relatório
POST   /api/v1/campaigns/send/transactional ← SMS avulso
```

### Financeiro
```
GET  /api/v1/billing/balance      ← Saldo de créditos
GET  /api/v1/billing/packages     ← Pacotes disponíveis
POST /api/v1/billing/buy          ← Comprar créditos (Pix)
GET  /api/v1/billing/transactions ← Histórico
```

### Admin (apenas admin@disparesms.com.br)
```
GET  /api/v1/admin/stats          ← Estatísticas da plataforma
GET  /api/v1/admin/tenants        ← Todas as empresas
PUT  /api/v1/admin/tenants/:id/status  ← Suspender/ativar
POST /api/v1/admin/tenants/:id/credits ← Adicionar créditos
```

### Webhooks (configurar nos provedores)
```
POST /api/v1/webhooks/zenvia      ← Status Zenvia
POST /api/v1/webhooks/twilio      ← Status Twilio
POST /api/v1/webhooks/infobip     ← Status Infobip
POST /api/v1/webhooks/asaas       ← Pagamento Asaas
```

---

## 📱 Formato do CSV para importação

```csv
nome,telefone,email,tags
João Silva,44999999999,joao@email.com,"vip,cliente"
Maria Santos,44988888888,,prospects
```

**Colunas aceitas:** `nome/name`, `telefone/phone/numero/celular`, `email`, `tags`

---

## 💬 Como usar variáveis na mensagem

```
Olá {{nome}}! Seu pedido #{{pedido}} foi aprovado.
Acesse: {{link}}
```

As variáveis `{{nome}}`, `{{telefone}}` são preenchidas automaticamente
com os dados do contato na hora do disparo.

---

## 🔌 Configurar provedores SMS

### Zenvia (recomendado para Brasil)
1. Acesse [app.zenvia.com](https://app.zenvia.com)
2. Configurações → API → Criar token
3. Adicione no `.env`:
   ```env
   ZENVIA_API_TOKEN=seu_token_aqui
   ZENVIA_SENDER=NomeDaMarca
   ```
4. Configure webhook em: Configurações → Webhooks
   URL: `https://sua-api.com/api/v1/webhooks/zenvia`

### Twilio (fallback / internacional)
1. Acesse [console.twilio.com](https://console.twilio.com)
2. Copie Account SID e Auth Token
3. Compre um número SMS-capable
4. Adicione no `.env`:
   ```env
   TWILIO_ACCOUNT_SID=ACxxxx
   TWILIO_AUTH_TOKEN=xxxx
   TWILIO_FROM_NUMBER=+15551234567
   ```

---

## 💰 Configurar pagamentos (Asaas)

1. Acesse [asaas.com](https://www.asaas.com)
2. Configurações → Integrações → API Keys
3. Adicione no `.env`:
   ```env
   ASAAS_API_KEY=$aact_xxxx
   ASAAS_ENVIRONMENT=sandbox   # mude para production quando lançar
   ```
4. Configure webhook: Configurações → Notificações
   URL: `https://sua-api.com/api/v1/webhooks/asaas`

---

## 🚀 Deploy em produção

### Backend (Railway — recomendado)
```bash
# 1. Crie conta em railway.app
# 2. Conecte o repositório GitHub
# 3. Configure as variáveis de ambiente no painel
# 4. Deploy automático a cada push
```

### Banco de dados (Supabase — gratuito)
```bash
# 1. Crie projeto em supabase.com
# 2. Copie a connection string
# 3. Adicione em DATABASE_URL no Railway
```

### Redis (Upstash — gratuito)
```bash
# 1. Crie database em upstash.com
# 2. Copie a Redis URL
# 3. Adicione em REDIS_URL no Railway
```

---

## 🗺️ Próximos passos

- [ ] Frontend Next.js (painel do cliente)
- [ ] Frontend admin (painel administrativo)  
- [ ] Autenticação 2FA
- [ ] Relatórios exportáveis (PDF/Excel)
- [ ] Integração Shopify (carrinho abandonado)
- [ ] Integração Nuvemshop
- [ ] App mobile (React Native)

---

## 📞 Suporte

**DisparesSMS** — (44) 99144-5556  
contato@disparesms.com.br
