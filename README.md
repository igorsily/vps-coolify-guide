# Documentação de Infraestrutura e Deploy

Este repositório documenta como provisionar, operar e evoluir uma VPS com Coolify para aplicações em monorepo.

Use o `PASSO-A-PASSO-COOLIFY.md` como fonte de verdade para o setup inicial. Os arquivos em `plans/` complementam o runbook com decisões de deploy, operação e evolução.

## Como Usar

1. Execute o setup inicial da VPS.
2. Configure deploy, variáveis e monorepo.
3. Ative backup antes de colocar dados críticos em produção.
4. Evolua observabilidade e cache remoto conforme a carga crescer.

## Setup Inicial

- [Runbook: VPS segura + Coolify](./PASSO-A-PASSO-COOLIFY.md): comandos e validações para preparar a VPS.
- [Visão arquitetural da infra](./plans/infra-coolify.md): componentes, fluxo macro e modelo de segurança.

## Deploy

- [Plano de deploy de serviços](./plans/PLANO-DEPLOY-SERVICOS.md): ordem de provisionamento, validação pós-deploy e rollback básico.
- [Deploy de monorepo com Turborepo](./plans/DEPLOY-MONOREPO-TURBO.md): configuração de `Base Directory`, `Watch Paths` e impacto de arquivos compartilhados.
- [Gestão de variáveis de ambiente](./plans/ENVS-COOLIFY.md): separação entre build/runtime, público/privado e API/Web.

## Operação

- [Estratégia de backup e restore](./plans/ESTRATEGIA-BACKUP.md): storage externo, retenção, RPO/RTO e teste de recuperação.
- [Remote cache do Turborepo](./plans/REMOTE-CACHE-TURBO.md): opções Vercel e self-hosted no Coolify.
- [Observabilidade futura](./plans/OBSERVABILIDADE-FUTURA.md): Prometheus, Loki, Grafana e Grafana Alloy.

## Princípios

- Segurança administrativa via SSH por chave e acesso preferencial pela Tailnet.
- Bancos sem porta pública, acessíveis pela rede interna do Docker/Coolify.
- Backups externos testados periodicamente.
- Deploys validados com healthcheck, logs e rollback documentado.
- Versões pinadas em exemplos de produção para reduzir drift operacional.
