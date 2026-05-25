# Gestão de Variáveis de Ambiente (ENVs) no Coolify

Este guia explica como o Coolify gerencia Variáveis de Ambiente, com foco em uma arquitetura de Monorepo separando uma API (Fastify/Node.js) e um Frontend (Next.js).

## 1. O Conceito Principal
Diferente do ambiente local onde você possui arquivos `.env` soltos nas pastas, no Coolify **você não deve commitar arquivos `.env` no Git**. 
Como configuramos o seu monorepo em duas "Aplicações" distintas dentro do painel do Coolify, **cada aplicação terá seu próprio ambiente de variáveis isolado**.

---

## 2. Backend (API - Fastify)
A API Fastify consome variáveis exclusivamente em **tempo de execução (Runtime)** através de `process.env`.

### Como Configurar:
1. No Coolify, acesse a aplicação correspondente à sua **API Backend**.
2. Acesse a aba **Environment Variables**.
3. Adicione suas chaves (ex: `JWT_SECRET`, `API_PORT`, `DATABASE_URL`).
4. **Comunicação Segura (Banco de Dados):**
   - **NÃO** use a URL pública do banco de dados na sua API.
   - Ao criar o banco no Coolify, ele fornece uma URL interna (ex: `postgresql://user:pass@coolify-db-name:5432/db`). Use **esta URL** na sua `DATABASE_URL`. Isso garante que o tráfego não saia para a internet, sendo muito mais rápido e seguro através da rede interna do Docker.

---

## 3. Frontend (Web - Next.js)
O Next.js é mais complexo, pois necessita de variáveis em duas fases distintas: no momento do build (geração de páginas estáticas e injeção para o navegador) e no momento da execução (SSR - Server Side Rendering).

### Como Configurar:
1. No Coolify, acesse a aplicação correspondente ao seu **Web Frontend**.
2. Acesse a aba **Environment Variables**.
3. Adicione as chaves necessárias.

### A Regra do "Build Variable":
Ao adicionar uma variável no Coolify, há uma opção/checkbox chamada **"Build Variable"** (Variável de Build).

- **Variáveis Públicas (`NEXT_PUBLIC_*`):** Se a variável precisa ir para o navegador (ex: `NEXT_PUBLIC_API_URL`), o Webpack precisa conhecê-la *durante o build* da aplicação. Portanto, você **DEVE marcar** a opção "Build Variable".
- **Variáveis Secretas Server-Side:** Se for uma chave usada apenas em `getServerSideProps`, `Server Actions` ou Rotas de API internas do Next.js (ex: `STRIPE_SECRET_KEY`), você **NÃO precisa** marcá-la como "Build Variable". Ela será injetada com segurança em tempo de execução.

### Comunicação com a API:
- Para o Frontend no navegador se comunicar com sua API Fastify, ele precisa da **URL Pública** da API.
- A variável `NEXT_PUBLIC_API_URL` deve conter o domínio real que você configurou (ex: `https://api.seudominio.com`). **Nunca** use nomes internos de containers Docker no frontend.

---

## 4. Matriz Recomendada de ENVs

| Variável | Aplicação | Fase | Visibilidade | Observação |
| --- | --- | --- | --- | --- |
| `DATABASE_URL` | API | Runtime | Privada | Use a URL interna do banco no Coolify. |
| `JWT_SECRET` | API | Runtime | Privada | Gere um segredo forte por ambiente. |
| `API_PORT` | API | Runtime | Privada | Deve bater com a porta exposta pela aplicação. |
| `NEXT_PUBLIC_API_URL` | Web | Build | Pública | Vai para o bundle do navegador. Use a URL pública da API. |
| `STRIPE_SECRET_KEY` | Web/API | Runtime | Privada | Nunca marque como pública. |
| `TURBO_TOKEN` | API/Web | Build | Privada | Necessário para remote cache. |
| `TURBO_TEAM` | API/Web | Build | Privada | Identificador do time/projeto no Turborepo. |
| `TURBO_API` | API/Web | Build | Privada | Necessário quando o cache remoto é self-hosted. |

## 5. Modo Raw com Curadoria

O modo **Raw** do Coolify acelera a edição, mas não deve receber o `.env` local inteiro sem revisão.

Antes de colar variáveis, remova:

- credenciais de desenvolvimento local;
- URLs `localhost`;
- flags de debug;
- tokens temporários;
- variáveis que pertencem a outra aplicação.

Prefira manter templates versionados sem valores sensíveis, como:

```text
apps/api/.env.production.example
apps/web/.env.production.example
```

## 6. Erros Comuns

- Usar hostname interno do Docker em `NEXT_PUBLIC_API_URL`. O navegador do usuário não resolve nomes internos de containers.
- Marcar segredo server-side como **Build Variable** sem necessidade.
- Reutilizar segredos de staging em produção.
- Configurar `DATABASE_URL` pública quando a API e o banco estão na mesma rede interna do Coolify.
