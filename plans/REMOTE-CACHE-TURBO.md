# Setup: Turborepo Remote Cache no Coolify

Para que seus builds no Monorepo sejam instantâneos, utilizaremos um servidor de Cache Remoto. Quando a sua API ou Web for buildada uma vez, o resultado fica salvo e o próximo build só processa o que realmente mudou.

## Opção A: Usando o Vercel (Grátis para projetos pequenos)
Use esta opção se aceitar armazenar metadados e artefatos de cache na infraestrutura da Vercel.

1. No seu terminal local, rode `npx turbo login`.
2. Rode `npx turbo link`.
3. Nas ENVs do Coolify (API e Web), adicione:
    - `TURBO_TOKEN`: Seu token de acesso (gerado no painel da Vercel).
    - `TURBO_TEAM`: O slug do seu time na Vercel.

## Opção B: Self-Hosted no Coolify (Recomendado para total controle)
Use esta opção se quiser controlar onde os artefatos de cache ficam armazenados. O trade-off é consumir disco, CPU e rede da mesma VPS que roda suas aplicações.

1. No Coolify, vá em **New -> Service**.
2. Procure por **Turborepo Remote Cache** (ou use a imagem Docker `ducktors/turborepo-remote-cache`).
3. Configure as seguintes ENVs para o serviço de cache:
    - `STORAGE_PATH`: `/data`
    - `AUTH_TOKEN`: Token forte gerado em gerenciador de senhas.
4. Atrele um domínio ao serviço de cache (ex: `cache.seudominio.com`).
5. **Nas ENVs das suas Aplicações (API e Web) no Coolify**, adicione:
    - `TURBO_TOKEN`: O mesmo valor de `AUTH_TOKEN`.
    - `TURBO_API`: `https://cache.seudominio.com`
    - `TURBO_TEAM`: Um identificador estável do projeto ou time.

## Comparativo

| Critério | Vercel | Self-hosted |
| --- | --- | --- |
| Setup | Mais simples | Requer serviço extra |
| Controle dos artefatos | Menor | Maior |
| Carga na VPS | Nenhuma | Usa disco, CPU e rede |
| Resiliência | Depende da Vercel | Depende da sua VPS |
| ENVs obrigatórias | `TURBO_TOKEN`, `TURBO_TEAM` | `TURBO_TOKEN`, `TURBO_TEAM`, `TURBO_API` |

### Resultado Esperado
No log do build do Coolify, você verá algo como:
`>>> FULL TURBO: 12 packages cached, 0 computed`
Isso significa que o build levou apenas alguns segundos porque ele aproveitou o cache remoto.

## Cuidados

- Não reutilize senhas humanas como `AUTH_TOKEN`.
- Monitore uso de disco se o cache for self-hosted.
- Mantenha o cache como otimização. O deploy deve continuar funcionando mesmo se o cache estiver indisponível.
