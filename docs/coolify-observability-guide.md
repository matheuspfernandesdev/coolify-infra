# Guia do Coolify e Stack de Observabilidade na VPS

Este documento detalha o funcionamento do Coolify, a arquitetura da stack leve de observabilidade e o passo a passo de instalação e configuração na VPS.

---

## 1. Como o Coolify Funciona e Ferramentas Gerenciadas

O **Coolify** é uma alternativa PaaS (*Platform as a Service*) open-source e auto-hospedada a serviços como Heroku, Netlify ou Render. Ele roda diretamente na sua VPS e simplifica o gerenciamento de aplicações, bancos de dados e serviços sem exigir que você gerencie manualmente arquivos do Docker Compose ou Nginx/Traefik via terminal.

### Funcionamento no Servidor
- **Agente e Docker Core**: O Coolify se instala no servidor e conversa diretamente com o **Docker Engine** local através do socket (`/var/run/docker.sock`).
- **Proxy Reverso Automático**: Por padrão, o Coolify instala e gerencia um proxy (como Traefik ou Caddy). Quando você adiciona um novo container ou serviço, o Coolify configura automaticamente o domínio, roteamento e certificados SSL/TLS (Let's Encrypt).
- **Gerenciamento de recursos**: Ele permite visualizar status, consumo de memória/CPU, reiniciar containers, ver logs diretamente na interface web e configurar deploys automáticos via GitHub/GitLab webhooks.

### Ferramentas e Containers Suportados
1. **Aplicações Web / Repositórios Git**: Node.js, Python, Go, PHP, Rust, Java, Dockerfile customizado, etc.
2. **Bancos de Dados Prontos**: PostgreSQL, MySQL, MariaDB, MongoDB, Redis, ClickHouse, Meilisearch, etc.
3. **Serviços One-Click (Docker Compose)**: Suporta a implantação de qualquer pilha definida via `docker-compose.yml` (como MinIO, Grafana, Nginx, WordPress, Supabase, Plausible, etc.).
4. **Containers Existentes**: O Coolify identifica containers existentes na VPS e permite monitorar status. Para gerenciamento completo de ciclo de vida (rebuild/deploy/proxy automático), recomenda-se cadastrar o serviço via Docker Compose ou Git no painel.

---

## 2. Stack Leve de Observabilidade Recomendada

A stack escolhida para monitoramento de métricas e centralização de logs é composta por:

1. **Prometheus**: Coleta e armazena métricas de séries temporais.
2. **Node Exporter**: Coleta métricas do SO da VPS (CPU, RAM, Disco, Rede).
3. **Loki**: Mecanismo de agregação e armazenamento de logs ultraleve da Grafana.
4. **Promtail**: Agente que lê o socket do Docker (`/var/run/docker.sock`) e envia os logs de todos os containers (atuais e futuros) para o Loki.
5. **Grafana**: Painel visual para dashboards de métricas e visualização de logs em tempo real.

Os arquivos de configuração dessa stack estão disponíveis na pasta `observability/`:
- `observability/docker-compose.yml`
- `observability/prometheus/prometheus.yml`
- `observability/promtail/promtail-config.yml`
- `observability/grafana/provisioning/datasources/datasources.yml`
- `observability/grafana/provisioning/dashboards/dashboards.yml`

---

## 3. Passo a Passo Prático de Instalação e Configuração

### Passo 1: Instalar o Coolify na VPS
No terminal da sua VPS (como `root` ou usuário com permissão `sudo`), execute:

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

Após a instalação:
1. Acesse no navegador: `http://IP_DA_SUA_VPS:8000`
2. Crie a conta de administrador.

### Passo 2: Implantar a Stack de Observabilidade no Coolify
1. No painel do Coolify, acesse **Projects** -> **+ New**.
2. Selecione o ambiente (`production`).
3. Clique em **+ New Resource** e escolha **Docker Compose**.
4. Apunte para este repositório Git e defina o diretório base como `observability` (ou cole o conteúdo de `observability/docker-compose.yml`).
5. Clique em **Deploy**.

### Passo 3: Acessar o Grafana e Configurar Dashboards
1. Acesse o Grafana no navegador (`http://IP_DA_SUA_VPS:3000`).
2. Login inicial: `admin` / `admin`.
3. **Logs (Loki)**:
   - Vá no menu lateral -> **Explore**.
   - Escolha o data source **Loki**.
   - No campo de busca por rótulos (labels), selecione `container` = `<nome_do_container>`.
4. **Métricas (Prometheus + Node Exporter)**:
   - Vá em **Dashboards** -> **New** -> **Import**.
   - Informe o ID `1860` (Node Exporter Full) e clique em **Load**.
   - Selecione a fonte **Prometheus** e salve.

---

## 4. Integração com Containers Existentes

- O **Promtail** mapeia o socket do Docker (`/var/run/docker.sock`), permitindo coletar logs de **todos** os containers rodando na máquina, inclusive os ~6 containers que já existiam antes da instalação do Coolify.
- Para gerenciar o ciclo de vida completo dos containers antigos pelo Coolify, basta importar suas configurações no painel.
