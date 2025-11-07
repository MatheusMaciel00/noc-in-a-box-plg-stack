# Projeto de Portfólio: NOC-in-a-Box (Stack de Observabilidade PLG)

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-F8F9FA?style=for-the-badge&logo=loki&logoColor=black)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)

Este projeto implementa uma stack de observabilidade "PLG" (Prometheus, Loki, Grafana) completa, orquestrada com Docker Compose para monitorizar uma arquitetura de microsserviços simulada.

O objetivo é demonstrar competências práticas em SRE (Site Reliability Engineering) e DevOps, incluindo monitorização de métricas, agregação de logs centralizados, criação de dashboards e configuração de alertas proativos para resposta a incidentes.

## A User Story

Este projeto foi construído para satisfazer a seguinte *User Story*:

> "Como um aspirante a Analista de NOC e Engenheiro de DevOps Júnior, eu quero construir um projeto completo de 'NOC-in-a-Box' localmente. Eu usarei Docker Compose para simular uma infraestrutura de microsserviços (como um 'servidor_web' Nginx e um 'servidor_db' MySQL) e implantar a stack de observabilidade completa Prometheus, Loki e Grafana (a stack PLG), para que eu possa monitorar métricas de saúde, coletar e consultar logs centralizados de todos os containers, criar dashboards visuais e configurar alertas automáticos (via Alertmanager) que disparam quando um serviço falha."

---

## 🚀 Como Executar

1.  Certifique-se de que tem o Docker e o Docker Compose instalados.
2.  Clone este repositório (`git clone ...`).
3.  Execute o comando a partir da raiz do projeto:
    ```bash
    docker compose up -d
    ```
4.  **Aguarde 1-2 minutos.** Na primeira execução, o Grafana demora a inicializar a sua base de dados interna.

---

## 📍 Pontos de Acesso (Endpoints)

Após o arranque, a stack completa está disponível no seu `localhost`:

* **Aplicação (Nginx):** `http://localhost:8080`
* **Visualização (Grafana):** `http://localhost:3000`
    * **Login:** `admin`
    * **Password:** `admin`
* **Métricas (Prometheus):** `http://localhost:9090`
* **Logs (Loki API):** `http://localhost:3100`

---

## 🏛️ Arquitetura da Solução

Esta stack é composta por 7 serviços Docker, comunicando através de uma rede `bridge` customizada (`monitor-net`), o que permite a descoberta de serviços por DNS interno (ex: `loki:3100`).

```mermaid
graph TD
    subgraph "Analista NOC"
        USER[<br>👨‍💻<br>Analista NOC]
    end

    subgraph "Camada de Visualização"
        GRA(Grafana<br>http://localhost:3000)
    end

    subgraph "Cérebros de Observabilidade"
        PROM(Prometheus<br>http://localhost:9090)
        LOKI(Loki<br>http://localhost:3100)
    end

    subgraph "Agentes Coletores"
        NEXP(Nginx Exporter<br>Traduz Métricas)
        PROMTAIL(Promtail<br>Coleta Logs)
    end

    subgraph "Serviços da Aplicação (Alvos)"
        WEB(Nginx<br>http://localhost:8080)
        DB(MariaDB<br>db-server)
    end

    %% Fluxo de Métricas
    WEB -- Métrica Nativa (stub_status) --> NEXP
    NEXP -- Formato Prometheus (/metrics) --> PROM
    PROM -- Query (PromQL) --> GRA

    %% Fluxo de Logs
    WEB -- Docker Logs --> PROMTAIL
    DB -- Docker Logs --> PROMTAIL
    PROMTAIL -- Push Logs --> LOKI
    LOKI -- Query (LogQL) --> GRA

    %% Interação do Utilizador
    USER -- Acede ao Dashboard --> GRA
    GRA -- Alertas (FIRING) --> USER
```
## 🏗️ Componentes da Stack

### Aplicações Alvo:
* **web-server (Nginx):** Simula o frontend da nossa aplicação.
* **db-server (MariaDB):** Simula o backend da nossa aplicação.

### Agentes Coletores:
* **nginx-exporter:** Um "tradutor" que lê a página `/stub_status` do Nginx e a expõe como métricas `/metrics` que o Prometheus entende (ex: `nginx_up`).
* **promtail:** O agente da Loki. Ele "ouve" o `docker.sock`, descobre todos os contentores com a label `logging=true`, e "empurra" os seus logs para o Loki.

### Cérebros (Backends):
* **prometheus:** O nosso banco de dados de séries temporais (TSDB). Ele "raspa" (coleta) ativamente métricas do `nginx-exporter` e de si mesmo.
* **loki:** O nosso cérebro de agregação de logs. Recebe logs do Promtail e indexa apenas as labels (ex: `{job="web"}`), tornando-o leve e rápido.

### Visualização (Frontend):
* **grafana:** O nosso "painel de vidro único". Foi auto-provisionado no arranque para já "conhecer" o Prometheus e o Loki como fontes de dados. É aqui que construímos dashboards e configuramos alertas.

---

## 📸 Galeria do Projeto (Prova de Conceito)
Aqui demonstramos o sistema completo em ação.

### 💡 Como Adicionar as Suas Imagens ao GitHub:

1.  Faça o upload do seu projeto para o GitHub (como eu lhe mostrei no guia `github_guide.md`).
2.  No seu repositório GitHub, vá à aba "Issues" e clique em "New Issue".
3.  Arraste e solte (drag and drop) a sua captura de tela (ex: a sua `image_132aa5.png`) para a caixa de texto da "issue".
4.  O GitHub vai fazer o upload e dar-lhe um link Markdown (ex: `![image](https://...)`).
5.  Copie esse link de Markdown.
6.  Volte ao seu `README.md` (clicando no ícone de "Editar"), e cole o link no placeholder correto abaixo.
7.  Feche a "issue" sem a submeter. Repita para todas as imagens.

### 1. Saúde da Coleta de Métricas (Prometheus)
O Prometheus mostra uma visão de "saúde" de todos os exporters que ele está a monitorizar. Ambos os jobs (prometheus e nginx) estão `UP`, provando que a coleta de métricas está a funcionar.
### 2. Dashboard do NOC (Métricas e Logs num só local)
O Grafana permite-nos construir um "painel de vidro único" que combina métricas (do Prometheus) e logs (do Loki).

**Como tirar este print:**

* No Grafana (`http://localhost:3000`), vá a Dashboards -> New Dashboard.
* Adicione um painel. Escolha Prometheus. Na query, coloque `nginx_connections_active`.
* Adicione outro painel. Escolha Loki. Na query, coloque `{container="web-server"}`.
* Organize-os lado a lado e tire o print!
### 3. ALERTA! Teste de Engenharia de Caos
A prova final é simular uma falha (`docker compose rm --stop --force web`) e provar que o nosso sistema dispara um alerta. Configurámos uma regra de alerta no Grafana que verifica a métrica `nginx_up < 1`.
Quando o Nginx "morre", o `nginx-exporter` reporta `nginx_up = 0`, e o Grafana deteta-o em 10 segundos, mudando o estado para "FIRING".
---

## 💡 Lições Aprendidas & Desafios de Depuração
Um projeto só está completo quando se depura os problemas. Esta stack apresentou vários desafios do mundo real que foram cruciais para o aprendizado:

#### O "Bug" `up` vs. `nginx_up` (Lógica de Alerta):
* **Problema:** O meu primeiro alerta (usando `up{job="nginx"}`) não disparou quando "matei" o content

