# Observabilidade com Prometheus, Loki, Grafana e Alloy

Este documento planeja uma stack de observabilidade para aplicações Fastify/Next.js e para a VPS gerenciada pelo Coolify.

## Arquitetura

- **Prometheus:** coleta métricas por pull em endpoints como `/metrics`.
- **Loki:** armazena logs com indexação por labels.
- **Grafana Alloy:** coleta logs e métricas do host/containers e envia para Loki/Prometheus. Use Alloy para novos setups; Promtail está em fim de vida.
- **Grafana:** visualiza métricas, logs e dashboards.

## Deploy via Coolify

No Coolify, crie um recurso **Docker Compose** para a stack. Evite `:latest` em produção. Os números abaixo são exemplos; revise as versões antes de aplicar.

```yaml
services:
  prometheus:
    image: prom/prometheus:v2.54.1
    volumes:
      - prometheus-data:/prometheus
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --storage.tsdb.retention.time=15d

  loki:
    image: grafana/loki:3.2.0
    volumes:
      - loki-data:/loki
    command: -config.file=/etc/loki/local-config.yaml

  alloy:
    image: grafana/alloy:v1.4.2
    volumes:
      - ./config.alloy:/etc/alloy/config.alloy:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    command:
      - run
      - /etc/alloy/config.alloy
      - --storage.path=/var/lib/alloy/data

  grafana:
    image: grafana/grafana:11.2.0
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}

volumes:
  prometheus-data:
  loki-data:
  grafana-data:
```

## Configuração Mínima do Prometheus

Crie `prometheus.yml` no recurso do Docker Compose.

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: api-backend
    metrics_path: /metrics
    static_configs:
      - targets: ["api-backend:3000"]
```

Ajuste `api-backend:3000` para o nome DNS interno e porta real do container no Coolify.

## Configuração Mínima do Alloy

Crie `config.alloy` para coletar logs Docker e enviar ao Loki.

```hcl
discovery.docker "containers" {
  host = "unix:///var/run/docker.sock"
}

loki.source.docker "containers" {
  host       = "unix:///var/run/docker.sock"
  targets    = discovery.docker.containers.targets
  forward_to = [loki.write.default.receiver]
}

loki.write "default" {
  endpoint {
    url = "http://loki:3100/loki/api/v1/push"
  }
}
```

Montar `/var/run/docker.sock` ou `/var/lib/docker/containers` dá ao coletor visibilidade ampla do host. Restrinja o acesso ao stack de observabilidade e revise permissões antes de produção.

## Aplicações

### Logs

As aplicações devem escrever logs em `stdout`/`stderr`. Use logs estruturados em JSON quando possível, especialmente no Fastify com Pino.

### Métricas Fastify

O Prometheus precisa de uma rota HTTP de métricas.

```javascript
const fastify = require('fastify')();
const metricsPlugin = require('fastify-metrics');

fastify.register(metricsPlugin, { endpoint: '/metrics' });

fastify.get('/health', async () => ({ status: 'ok' }));
```

### Métricas Next.js

Para Next.js SSR, avalie OpenTelemetry ou bibliotecas compatíveis com Prometheus. Comece com métricas de infraestrutura e logs antes de instrumentar métricas customizadas.

## Troubleshooting

1. No Grafana, verifique picos de latência ou erro no dashboard de métricas.
2. No Explore, consulte logs no Loki com labels do container.
3. Correlacione horário, release e logs da API.
4. Valide conexão com banco e healthcheck do serviço afetado.

## Checklist de Validação

- [ ] Grafana acessível por HTTPS.
- [ ] Prometheus mostra targets `UP`.
- [ ] Loki recebe logs dos containers.
- [ ] Alloy está rodando sem erro de permissão.
- [ ] Dashboards mostram CPU, memória, latência e erros.
- [ ] Retenção de Prometheus e Loki está definida.
- [ ] Senha admin do Grafana não usa valor padrão.
