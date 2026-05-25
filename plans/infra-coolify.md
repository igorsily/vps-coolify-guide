# Visão Arquitetural: Infraestrutura com Coolify

Este documento descreve a arquitetura alvo. Use o [runbook de setup](../PASSO-A-PASSO-COOLIFY.md) para executar comandos na VPS.

## Objetivo

Operar aplicações, bancos e serviços auxiliares em uma VPS Ubuntu usando Coolify como PaaS self-hosted.

## Componentes

- VPS Ubuntu: host principal da infraestrutura.
- Coolify: painel de deploy, proxy, banco interno e orquestração Docker.
- Docker: runtime dos serviços gerenciados pelo Coolify.
- Tailscale: rede administrativa privada para SSH e bootstrap seguro.
- UFW: firewall de borda no host.
- Fail2Ban: mitigação de tentativas repetidas de autenticação.
- DNS público: roteamento de domínios para aplicações e painel.
- Storage S3 compatível: destino externo para backups.

## Modelo de Acesso

- SSH administrativo deve usar chave pública e preferencialmente IP Tailscale.
- A porta SSH pública deve ficar bloqueada no UFW.
- O painel do Coolify deve ser usado por HTTPS após a configuração do domínio.
- A conta admin inicial não deve ser criada por HTTP público.

## Modelo de Rede

- `80/tcp` e `443/tcp` ficam públicos para o proxy atender aplicações.
- Bancos de dados não devem expor porta pública.
- APIs acessam bancos pela rede interna Docker/Coolify.
- Frontends no navegador acessam APIs pelo domínio público da API.

## Fluxo Macro

1. Preparar VPS e acesso administrativo seguro.
2. Instalar Coolify.
3. Configurar DNS e HTTPS.
4. Criar banco de dados sem porta pública.
5. Subir API com variáveis de runtime.
6. Subir frontend com variáveis de build/runtime.
7. Ativar backup externo.
8. Evoluir cache remoto e observabilidade.

## Referências

- [Runbook de setup da VPS](../PASSO-A-PASSO-COOLIFY.md)
- [Deploy de serviços](./PLANO-DEPLOY-SERVICOS.md)
- [Gestão de ENVs](./ENVS-COOLIFY.md)
- [Backup e restore](./ESTRATEGIA-BACKUP.md)
- [Observabilidade](./OBSERVABILIDADE-FUTURA.md)
