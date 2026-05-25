# Estratégia de Backup: Segurança dos Dados

Este guia detalha como configurar backups automáticos e externos para bancos de dados no Coolify. Backup reduz perda de dados, mas só é confiável quando o restore é testado periodicamente.

## 1. Onde Salvar (Storage Externo)
**NUNCA** salve o backup apenas na mesma máquina do banco de dados. Se o disco da VPS morrer, o backup morre junto.
Recomendo usar um serviço compatível com **S3**:
- **Cloudflare R2:** Melhor custo-benefício (Grátis até 10GB, sem taxa de download/egress).
- **AWS S3:** Padrão de mercado.
- **Backblaze B2:** Muito barato.

## 2. Configurando o Storage no Coolify
1. No painel do Coolify, acesse **Settings** -> **Storages**.
2. Clique em **Add Storage**.
3. Preencha os dados do seu Bucket S3:
   - **Name:** Ex: `Cloudflare-R2-Backups`
   - **Endpoint:** A URL do seu bucket (ex: `https://<account_id>.r2.cloudflarestorage.com`)
   - **Region:** `auto` (para R2) ou a região da AWS.
   - **Bucket:** Nome do bucket criado no provedor.
   - **Access Key** e **Secret Key:** Geradas no painel do provedor de storage.

## 3. Ativando o Backup no Banco de Dados
1. Vá até o seu Banco de Dados no Coolify (PostgreSQL, MySQL, etc.).
2. Clique na aba **Backups**.
3. Escolha o **Storage** que você acabou de criar.
4. Defina o **Frequency (CRON):**
   - Recomendado: `0 3 * * *` (Todo dia às 3h da manhã).
5. Defina o **Retention:** Quantos backups manter.
   - Recomendado: `7` ou `30` dias (dependendo da importância dos dados).

## 4. RPO e RTO

Defina metas antes de escolher frequência e retenção:

- **RPO (Recovery Point Objective):** quanto dado você aceita perder. Exemplo: backup diário permite perder até 24h de dados.
- **RTO (Recovery Time Objective):** quanto tempo você aceita ficar indisponível até restaurar o serviço.

Sugestão inicial para projetos pequenos:

- RPO: 24h.
- RTO: 2h a 4h.
- Frequência: diária.
- Retenção: 7 a 30 dias.

Projetos com transações críticas devem usar frequência maior, réplicas ou estratégia específica de point-in-time recovery.

## 5. Runbook Mínimo de Restore

Teste restore em ambiente isolado antes de precisar dele em produção.

1. Baixe o backup mais recente do bucket.
2. Crie uma instância temporária de banco.
3. Restaure o arquivo na instância temporária.
4. Valide conexão, tabelas principais e contagem aproximada de registros.
5. Execute um smoke test da aplicação apontando para o banco restaurado.
6. Documente tempo total de restauração e problemas encontrados.

## 6. Frequência de Teste

- Teste restore mensalmente para sistemas ativos.
- Teste restore antes de migrações destrutivas.
- Teste restore após trocar storage, credenciais ou versão do banco.

## 7. Checklist

- [ ] Backup externo configurado em storage S3 compatível.
- [ ] Porta pública do banco desabilitada.
- [ ] Retenção definida.
- [ ] Credenciais do bucket guardadas em local seguro.
- [ ] Restore testado em ambiente isolado.
- [ ] RPO/RTO registrados.
