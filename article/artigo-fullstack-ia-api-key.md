# Full Stack com IA: Como Proteger Sua API Key e Construir Projetos Prontos para Produção

> **O erro que 90% dos iniciantes cometem ao integrar IA em aplicações web — e como evitá-lo com arquitetura profissional**

![Capa do Artigo](https://raw.githubusercontent.com/matheusflorindo32/dio-artigo-ia-fullstack-seguro/main/images/capa-artigo.jpg)

---

## Introdução

Você acabou de aprender a usar a API da OpenAI. Conseguiu fazer o chatbot responder. Funciona localmente. E agora? Hora de colocar no ar e mostrar ao mundo, certo?

**Errado.**

Antes de apertar o botão de deploy, existe uma pergunta que separa projetos amadores de portfólios profissionais:

> **Onde está sua API Key agora?**

Se a resposta for "no meu código JavaScript do frontend", você está a um `Ctrl+Shift+I` de ter sua conta drenada por alguém do outro lado do mundo.

Este artigo é um guia prático, passo a passo, para construir aplicações Full Stack com IA de forma **segura, profissional e pronta para produção**. Não é teoria — é o que eu apliquei no meu próprio projeto de desafio da DIO.

**O que você vai aprender:**
- ✅ Por que a API Key nunca deve ficar no frontend
- ✅ Como separar frontend e backend corretamente
- ✅ Uso correto de variáveis de ambiente
- ✅ Proteção com rate limiting
- ✅ Deploy profissional (Railway + Vercel)
- ✅ Como transformar um projeto em portfólio

---

## 1. O Problema: A API Key Exposta

### Cenário real

Imagine: você está animado. O chatbot funciona. Você faz deploy no Vercel. Manda o link para um amigo testar. Ele testa. Funciona. Ótimo.

Mas o que ele (e qualquer pessoa) pode fazer em segundos:

1. Apertar `F12` → aba **Network**
2. Enviar uma mensagem no seu chat
3. Ver a requisição para `api.openai.com`
4. Ler sua **API Key** no header `Authorization`

```
Authorization: Bearer sk-su4per-secr3t-key-exposed-12345
```

### Consequências do vazamento

| Problema | Impacto |
|---------|---------|
| Uso indevido | Alguém usa sua key → você paga a conta |
| Drenagem financeira | Requisições em massa = custo exponencial |
| Revogação de acesso | OpenAI pode banir sua conta |
| Dados comprometidos | Seus prompts e dados expostos |

> ⚠️ **Alerta**: Em um teste de segurança, pesquisadores encontraram mais de 10.000 API keys expostas publicamente no GitHub em apenas um dia.

---

## 2. Arquitetura Segura: Frontend vs Backend

### O fluxo errado (comum entre iniciantes)

```
Usuário → Frontend → OpenAI API (com API key exposta)
```

### O fluxo correto (profissional)

```
Usuário → Frontend → Backend (seguro, com API key) → OpenAI API
```

### Por que separar?

| Frontend | Backend |
|---------|---------|
| Código visível ao usuário | Código inacessível ao usuário |
| Não pode ter segredos | É o único lugar para segredos |
| React, Vue, Angular | Node.js, Python, Go |
| Deploy no Vercel/Netlify | Deploy no Railway/Heroku |

**Regra de ouro**: A API Key vive **exclusivamente** no servidor. O frontend nunca a toca.

### Diagrama da arquitetura segura

![Arquitetura Frontend vs Backend](https://raw.githubusercontent.com/matheusflorindo32/dio-artigo-ia-fullstack-seguro/main/images/frontend-backend.jpg)

![Diagrama de Arquitetura](https://raw.githubusercontent.com/matheusflorindo32/dio-artigo-ia-fullstack-seguro/main/images/diagrama-arquitetura.jpg)

---

## 3. Implementação: Variáveis de Ambiente

### O que são?

Variáveis de ambiente são **configurações que ficam fora do código**. O arquivo `.env` armazena segredos que o código lê em tempo de execução.

### Como configurar

**1. Crie o arquivo `.env` no backend:**

```bash
# server/.env
OPENAI_API_KEY=sk-sua-chave-aqui
OPENAI_MODEL=gpt-3.5-turbo
PORT=5000
```

**2. Carregue no código:**

```javascript
require('dotenv').config();

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY, // Segura!
});
```

**3. NUNCA commit no GitHub:**

```bash
# .gitignore
.env
.env.local
```

![Proteção da API Key](https://raw.githubusercontent.com/matheusflorindo32/dio-artigo-ia-fullstack-seguro/main/images/api-key-segura.jpg)

> ⚠️ **NUNCA faça isso**:
```javascript
// ❌ ERRADO - expõe a key no frontend
const apiKey = 'sk-sua-chave-real-aqui';
```

---

## 4. Proteção Contra Abuso: Rate Limiting

### Por que rate limit?

Mesmo com a key no backend, se alguém descobrir seu endpoint público, pode fazer milhares de requisições e esvaziar sua conta.

### Implementação simples (Node.js)

```javascript
const rateLimit = new Map();

const rateLimitMiddleware = (req, res, next) => {
  const ip = req.ip;
  const now = Date.now();
  const window = 60000; // 1 minuto
  const max = 30; // 30 requisições
  
  if (!rateLimit.has(ip)) {
    rateLimit.set(ip, { count: 1, start: now });
    return next();
  }
  
  const data = rateLimit.get(ip);
  if (now - data.start > window) {
    data.count = 1;
    data.start = now;
    return next();
  }
  
  if (data.count >= max) {
    return res.status(429).json({
      error: 'Limite atingido',
      details: 'Máximo 30 requisições/minuto.'
    });
  }
  
  data.count++;
  next();
};

app.post('/api/chat', rateLimitMiddleware, async (req, res) => {
  // ...
});
```

> 💡 **Dica**: Em produção com múltiplos servidores, use Redis ou `express-rate-limit` para persistência.

---

## 5. Deploy Profissional

### Backend no Railway

1. Acesse https://railway.app
2. Conecte seu repositório GitHub
3. Adicione as variáveis de ambiente no dashboard
4. Deploy automático a cada push

### Frontend no Vercel

1. Acesse https://vercel.com
2. Importe seu repo
3. Configure `VITE_API_URL` apontando para o Railway
4. Deploy com um clique

### CORS configurado

```javascript
app.use(cors({
  origin: process.env.FRONTEND_URL || '*',
  methods: ['GET', 'POST'],
}));
```

![Fluxo Seguro](https://raw.githubusercontent.com/matheusflorindo32/dio-artigo-ia-fullstack-seguro/main/images/fluxo-seguro-ia.jpg)

---

## 6. Do Projeto ao Portfólio

### README profissional

- Banner visual no topo
- Badges de tecnologia
- Screenshots do projeto funcionando
- Link do deploy
- Instruções de instalação
- Estrutura de pastas

![GitHub Portfolio](https://raw.githubusercontent.com/matheusflorindo32/dio-artigo-ia-fullstack-seguro/main/images/github-portfolio.jpg)

### LinkedIn

Poste com:
- Imagem do projeto
- Explicação técnica breve
- Link do repositório
- O que você aprendeu

### Entrevistas técnicas

Quando perguntarem sobre o projeto, você pode falar:
> "Implementei arquitetura separada com backend seguro, variáveis de ambiente, rate limiting e deploy em produção."

---

## 7. Checklist de Segurança

- [ ] API Key está no `.env` do backend
- [ ] `.env` está no `.gitignore`
- [ ] Frontend nunca acessa a OpenAI diretamente
- [ ] Backend valida o input antes de enviar
- [ ] Rate limiting implementado
- [ ] CORS configurado corretamente
- [ ] Deploy separado (frontend ≠ backend)
- [ ] Hard limit de gastos na OpenAI configurado
- [ ] README profissional com screenshots
- [ ] Repositório organizado e documentado

---

## Conclusão

A diferença entre um "projeto que funciona no meu PC" e um "portfólio profissional" está nos detalhes que ninguém vê — mas que todo recrutador nota.

Segurança não é um "extra". É **obrigatório**.

Arquitetura separada não é "complicar". É **profissionalismo**.

Deploy não é o último passo. É **onde o projeto ganha vida**.

---

## Call to Action

🚀 **Veja o projeto funcionando:** [LINK_DO_DEPLOY]

📁 **Acesse o repositório:** [dio-artigo-ia-fullstack-seguro](https://github.com/matheusflorindo32/dio-artigo-ia-fullstack-seguro)

💼 **Conecte-se comigo no LinkedIn:** [LINK_DO_LINKEDIN]

💬 **O que você está construindo com IA?** Compartilhe nos comentários!

---

## Sobre o Autor

**Matheus Florindo** — Desenvolvedor em formação | Projetos com IA | Full Stack | Portfólio DIO

Construindo projetos reais para aprender de verdade.
