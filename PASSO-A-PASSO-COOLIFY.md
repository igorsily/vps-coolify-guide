# Runbook: VPS Segura + Coolify

Este é o runbook canônico para preparar uma VPS Ubuntu, restringir acesso administrativo e instalar o Coolify.

## Pré-requisitos

- VPS com Ubuntu 22.04 ou 24.04.
- Domínio com acesso ao provedor de DNS.
- Conta Tailscale e acesso à Tailnet.
- Chave SSH pública na máquina local.

## 1. Acesso Inicial e Atualização

Acesse a VPS como `root` usando o IP público fornecido pelo provedor:

```bash
ssh root@<IP_DA_VPS>
```

Atualize o sistema:

```bash
apt update && apt upgrade -y
```

## 2. Criar Usuário Administrativo

Crie um usuário administrativo para evitar operação diária como `root`:

```bash
adduser adminuser
usermod -aG sudo adminuser
```

No computador local, copie sua chave pública para o novo usuário:

```bash
ssh-copy-id adminuser@<IP_DA_VPS>
```

Abra uma segunda sessão antes de continuar e confirme que o usuário consegue usar `sudo`:

```bash
ssh adminuser@<IP_DA_VPS>
sudo whoami
```

O resultado esperado é `root`. Mantenha a sessão original aberta até terminar o hardening.

## 3. Instalar e Ativar Tailscale

Instale o Tailscale na VPS:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Autentique a máquina na Tailnet:

```bash
tailscale up
```

Anote o IP Tailscale da VPS:

```bash
tailscale ip -4
```

Teste o SSH pela Tailnet a partir da máquina local:

```bash
ssh adminuser@<IP_TAILSCALE>
```

## 4. Configurar Firewall com UFW

Siga esta ordem para evitar lockout.

Defina as políticas padrão:

```bash
ufw default deny incoming
ufw default allow outgoing
```

Permita HTTP e HTTPS públicos para o proxy do Coolify:

```bash
ufw allow 80/tcp
ufw allow 443/tcp
```

Permita SSH somente pela interface do Tailscale:

```bash
ufw allow in on tailscale0 to any port 22
```

Ative o firewall:

```bash
ufw enable
ufw status verbose
```

Valide em uma nova sessão local:

```bash
ssh adminuser@<IP_TAILSCALE>
```

O acesso via IP público deve falhar após o UFW bloquear a porta 22 pública.

## 5. Endurecer o SSH

Edite o arquivo de configuração do SSH:

```bash
nano /etc/ssh/sshd_config
```

Garanta estes valores:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Teste a configuração antes de reiniciar:

```bash
sshd -t
```

Reinicie o serviço:

```bash
systemctl restart ssh
```

Abra uma nova sessão pela Tailnet antes de encerrar sessões existentes:

```bash
ssh adminuser@<IP_TAILSCALE>
```

## 6. Instalar Fail2Ban

Instale o pacote:

```bash
apt install fail2ban -y
```

Crie um override mínimo para o SSH:

```bash
nano /etc/fail2ban/jail.local
```

Use uma configuração mínima:

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = %(sshd_log)s
maxretry = 5
bantime = 1h
findtime = 10m
```

Ative o serviço:

```bash
systemctl enable fail2ban
systemctl restart fail2ban
fail2ban-client status sshd
```

## 7. Instalar o Coolify

Use `root` ou eleve a sessão administrativa antes da instalação:

```bash
sudo -i
```

Execute o instalador oficial:

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

Scripts remotos aceleram o bootstrap, mas reduzem reprodutibilidade. Para ambientes com maior exigência de controle, valide o script e fixe versões antes de executar.

## 8. Configurar DNS

No provedor de DNS, crie os registros apontando para o IP público da VPS:

1. `A` para `coolify.seudominio.com`.
2. `A` wildcard para `*.seudominio.com`, se quiser domínios dinâmicos por aplicação.

## 9. Primeiro Acesso ao Painel

Não crie a conta administrativa inicial por HTTP público.

Use uma destas opções:

1. Acesse o painel pela Tailnet, se o Coolify estiver escutando em uma rota acessível por Tailscale.
2. Use túnel SSH a partir da máquina local:

```bash
ssh -L 8000:127.0.0.1:8000 adminuser@<IP_TAILSCALE>
```

Depois acesse localmente:

```text
http://127.0.0.1:8000
```

3. Configure o domínio `https://coolify.seudominio.com` no painel assim que possível e use HTTPS para acessos posteriores.

## 10. Checklist Final

- [ ] SSH com `root` bloqueado.
- [ ] Login SSH por senha bloqueado.
- [ ] SSH pelo IP público falha.
- [ ] SSH pelo IP Tailscale funciona.
- [ ] UFW permite apenas `80/tcp`, `443/tcp` e `22/tcp` via `tailscale0`.
- [ ] Fail2Ban está ativo para `sshd`.
- [ ] Coolify acessível por HTTPS após configuração do domínio.
- [ ] Conta admin inicial criada sem trafegar credenciais por HTTP público.
