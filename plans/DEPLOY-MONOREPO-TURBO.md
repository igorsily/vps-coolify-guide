# Estratégia de Deploy: Monorepo (Turborepo) no Coolify

## O Desafio do Monorepo
Em um monorepo com Turborepo, Next.js e Fastify, todo o código vive em um único repositório Git. O comportamento padrão de CI/CD seria *buildar* tudo a cada commit.
O objetivo aqui é configurar o Coolify para tratar seu repositório como aplicações separadas e **só disparar o build do serviço que foi realmente alterado**.

## Como o Coolify Resolve Isso
O Coolify possui suporte nativo para monorepos através da combinação de duas configurações em cada aplicação:
1. **Base Directory (Build Directory):** define a raiz da aplicação para Nixpacks ou Docker.
2. **Watch Paths (ou Git Filters):** define quais mudanças devem disparar deploy daquela aplicação.

O ponto crítico é equilibrar custo e segurança operacional. Filtros muito restritos economizam build, mas podem ignorar mudanças em arquivos compartilhados. Filtros mais amplos disparam mais builds, mas reduzem o risco de deploy perdido.

---

## Passo a Passo da Configuração

Vamos assumir a seguinte estrutura de pastas comum em Turborepos:
```text
meu-monorepo/
├── apps/
│   ├── web/        (Next.js Frontend)
│   └── api/        (Fastify Backend)
├── packages/
│   ├── ui/         (Componentes compartilhados)
│   └── config/     (Configs do TS, ESLint)
├── turbo.json
└── package.json
```

### 1. Criando a Aplicação do Backend (Fastify)
No painel do Coolify:
1. Adicione uma nova aplicação (New -> Public/Private Repository).
2. Selecione o seu repositório `meu-monorepo`.
3. Vá nas configurações (Settings) dessa nova aplicação:
    - **Name:** `API Backend`
    - **Base Directory:** `/apps/api` (Isso isola o build apenas para a API).
    - **Build Command:** O Coolify (via Nixpacks) tentará rodar `npm run build` na pasta base. Como é um Turborepo, garanta que o script de build no `/apps/api/package.json` funcione isoladamente ou ajuste o comando de build aqui para algo como `cd ../.. && npx turbo run build --filter=api`.
    - **Advanced -> Watch Paths:** inclua a API, pacotes compartilhados e arquivos raiz que alteram o build.

Exemplo de `Watch Paths` para API:

```text
/apps/api/**
/packages/**
/package.json
/package-lock.json
/pnpm-lock.yaml
/yarn.lock
/turbo.json
/tsconfig.json
/eslint.config.*
/nixpacks.toml
/Dockerfile
```

### 2. Criando a Aplicação do Frontend (Next.js)
Repita o processo, mas apontando para o Frontend:
1. Adicione **outra** nova aplicação, selecionando **o mesmo repositório** `meu-monorepo`.
2. Nas configurações dessa segunda aplicação:
    - **Name:** `Web Frontend`
    - **Base Directory:** `/apps/web`
    - **Advanced -> Watch Paths:** inclua o frontend, pacotes compartilhados e arquivos raiz que alteram o build.

Exemplo de `Watch Paths` para Web:

```text
/apps/web/**
/packages/**
/package.json
/package-lock.json
/pnpm-lock.yaml
/yarn.lock
/turbo.json
/tsconfig.json
/eslint.config.*
/nixpacks.toml
/Dockerfile
```

## Matriz de Impacto Recomendada

| Mudança | API | Web | Motivo |
| --- | --- | --- | --- |
| `/apps/api/**` | Sim | Não | Código específico da API |
| `/apps/web/**` | Não | Sim | Código específico do frontend |
| `/packages/**` | Sim | Sim | Bibliotecas compartilhadas |
| `/turbo.json` | Sim | Sim | Pipeline e cache do monorepo |
| Lockfile | Sim | Sim | Versões de dependências |
| `package.json` raiz | Sim | Sim | Scripts, workspaces e dependências globais |
| Configs raiz | Sim | Sim | TypeScript, lint, bundling ou build |

---

## Dica de Ouro com Turborepo: Remote Caching

Como você já está usando o **Turborepo**, a melhor forma de garantir builds extremamente rápidos no Coolify é ativando o **Remote Caching**.

Quando o Coolify fizer o build da sua API ou Web, o Turborepo verificará se aquele exato código já foi compilado antes. Se sim, ele baixa o artefato do cache e termina o build em segundos, ao invés de minutos.

**Como implementar:**
1. Escolha uma estratégia em [Remote Cache do Turborepo](./REMOTE-CACHE-TURBO.md).
2. Nas **Variáveis de Ambiente** de ambas as aplicações, configure as variáveis do cenário escolhido.

Dessa forma, mesmo que o Coolify acabe rodando um webhook que você não queria, o Turborepo dirá "Eu já fiz esse build, pegue do cache" e a operação levará poucos segundos, não consumindo CPU da sua VPS.

## Validação

- Alterar somente `/apps/api/**` deve disparar deploy da API.
- Alterar somente `/apps/web/**` deve disparar deploy da Web.
- Alterar `/packages/**`, lockfile ou `turbo.json` deve disparar ambos.
- Os logs do Coolify devem mostrar quando um deploy foi ignorado por filtro.
