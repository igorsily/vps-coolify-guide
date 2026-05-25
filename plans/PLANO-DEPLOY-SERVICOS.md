# Plano de Deploy de Serviços no Coolify

## Objetivo
Estabelecer o fluxo correto para subir bancos de dados, APIs (Backend) e aplicações Web (Frontend) no Coolify, garantindo a comunicação segura entre os containers e a exposição correta para a internet.

## 1. Preparação: Integração com o Git
Antes de subir suas aplicações, é necessário conectar o Coolify ao seu provedor de versionamento (GitHub, GitLab, etc.).
- No painel do Coolify, acesse **Sources**.
- Adicione uma nova fonte escolhendo **GitHub App** (Recomendado, pois gerencia webhooks e tokens automaticamente).
- Siga o fluxo de autorização no GitHub para instalar o App na sua conta ou organização.

## 2. Ordem de Provisionamento
Para evitar erros de dependência (como a API não subir por falta do banco), siga estritamente esta ordem:
1. Bancos de Dados e Caches (PostgreSQL, MySQL, Redis).
2. Backend / APIs.
3. Frontend / Aplicações Web.

---

## 3. Subindo os Bancos de Dados
O Coolify gerencia instâncias de bancos de dados como containers nativos usando templates "1-click".

1. Navegue até **Projects** -> Escolha seu ambiente (ex: `Production`) -> **New** -> **Databases**.
2. Selecione o mecanismo (ex: PostgreSQL).
3. **Configuração de Segurança Crítica:**
   - **Public Port:** Deixe **VAZIO** ou desligado. Isso garante que o banco não seja acessível pela internet, restringindo o acesso apenas à rede interna do Docker (onde suas APIs estarão).
4. Guarde as credenciais geradas automaticamente. O formato interno para uso na API será semelhante a: `postgresql://user:password@nome-do-container:5432/db`.

---

## 4. Subindo o Backend (API)
1. No seu ambiente, clique em **New** -> **Public Repository** ou **Private Repository**.
2. Selecione o repositório da sua API e a branch alvo (ex: `main`).
3. **Estratégia de Build:**
   - **Nixpacks (Padrão e Recomendado):** Analisa o código fonte (Node, Python, Go, etc.) e cria o ambiente ideal automaticamente, sem necessidade de `Dockerfile`.
   - **Dockerfile:** Selecione apenas se você já tiver um Dockerfile altamente customizado e otimizado na raiz do projeto.
4. **Variáveis de Ambiente (Environment Variables):**
   - Na aba *Environment Variables*, insira a URL do banco de dados (ex: `DATABASE_URL` = url-interna-do-passo-anterior) e outras chaves (JWT, APIs externas).
5. **Configuração de Domínio (Domains):**
   - Defina o domínio público da API. Exemplo: `https://api.seudominio.com`.
6. Clique em **Deploy**. O Coolify fará o pull, build e subirá a API roteando o tráfego pelo Traefik.

---

## 5. Subindo o Frontend (React, Vue, Next.js)
1. Crie um novo recurso apontando para o repositório do frontend.
2. **Estratégia de Build:**
   - **Aplicações Estáticas (React/Vite, Vue, Angular):** O Nixpacks fará o build estático. Certifique-se de que o diretório de saída (Publish Directory) esteja correto (geralmente `dist` ou `build`).
   - **Server-Side Rendering (Next.js, Nuxt):** O Nixpacks iniciará um servidor Node.js (ex: `npm start`) para servir a aplicação.
3. **Variáveis de Ambiente:**
   - Defina as variáveis apontando para a sua API pública (ex: `VITE_API_URL` ou `NEXT_PUBLIC_API_URL` apontando para `https://api.seudominio.com`).
4. **Configuração de Domínio (Domains):**
   - Defina o domínio final da aplicação. Exemplo: `https://seudominio.com`.
5. Clique em **Deploy**.

---

## 6. Operações e Boas Práticas (Pós-Deploy)

- **Continuous Deployment (CD):** Com o GitHub App configurado, commits na branch selecionada disparam deploys automáticos. O Coolify pode reduzir downtime, mas isso depende de healthchecks, tempo de boot, compatibilidade de migrações e comportamento da aplicação.
- **Backups de Banco de Dados:** Configure backup externo antes de armazenar dados críticos. Consulte [Estratégia de Backup](./ESTRATEGIA-BACKUP.md).
- **Monitoramento de Logs:** Use a aba **Logs** de cada serviço para investigação inicial. Para operação contínua, evolua para a stack de observabilidade em [Observabilidade](./OBSERVABILIDADE-FUTURA.md).

## 7. Validação Pós-Deploy

Após cada deploy, valide:

- serviço está `Running` no Coolify;
- domínio público responde com HTTPS válido;
- healthcheck da API responde com status 2xx;
- logs não mostram erro de inicialização, conexão com banco ou migração;
- frontend consome a API pelo domínio público correto;
- banco continua sem porta pública exposta.

## 8. Rollback Básico

Use este fluxo quando um deploy quebrar produção:

1. Identifique o último commit ou imagem estável.
2. Reverta o commit problemático no Git ou redeploye a versão anterior pelo Coolify, se disponível.
3. Evite rollback de aplicação sem avaliar mudanças de schema no banco.
4. Se houve migração destrutiva, acione o runbook de restore antes de promover tráfego.
5. Valide healthcheck, logs e fluxo crítico de negócio.

## 9. Cuidados com Migrações

- Prefira migrações compatíveis com versão anterior da aplicação.
- Faça backup antes de mudanças de schema críticas.
- Evite remover colunas ou constraints usadas pela versão anterior sem janela planejada.
- Documente o caminho de rollback quando a migração não for reversível.
