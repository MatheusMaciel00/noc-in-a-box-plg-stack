# noc-in-a-box-plg-stack
rojeto de portfólio a implementar a stack de observabilidade Prometheus, Loki e Grafana com Docker Compose
Projeto de Portfólio: NOC-in-a-Box (Stack de Observabilidade PLG)

Este projeto implementa uma stack de observabilidade "PLG" (Prometheus, Loki, Grafana) completa, usando Docker Compose para monitorizar uma arquitetura de microsserviços simulada (um servidor web e um banco de dados).

O objetivo é demonstrar habilidades práticas em SRE (Site Reliability Engineering) e DevOps, incluindo monitorização de métricas, agregação de logs centralizados, criação de dashboards e configuração de alertas automáticos para resposta a incidentes.

User Story

"Como um aspirante a Analista de NOC e Engenheiro de DevOps Júnior, eu quero construir um projeto completo de 'NOC-in-a-Box' localmente. Eu usarei Docker Compose para simular uma infraestrutura de microsserviços (como um 'servidor_web' Nginx e um 'servidor_db' MySQL) e implantar a stack de observabilidade completa Prometheus, Loki e Grafana (a stack PLG), para que eu possa monitorar métricas de saúde, coletar e consultar logs centralizados de todos os containers, criar dashboards visuais e configurar alertas automáticos (via Alertmanager) que disparam quando um serviço falha."

Arquitetura Final (7 Serviços)

Este docker-compose.yml lança 7 contentores que trabalham em conjunto, demonstrando uma arquitetura de observabilidade moderna.

1. Serviços da Aplicação (Os Alvos)

web-server (Nginx): A nossa "aplicação" simulada que serve uma página web.

db-server (MariaDB): O nosso "banco de dados" simulado.

2. A Stack de Observabilidade (A Stack PLG)

prometheus (Prometheus): O "cérebro" de métricas. Um banco de dados de séries temporais que "raspa" (coleta) métricas dos nossos exporters.

loki (Loki): O "cérebro" de logs. Um sistema de agregação de logs leve, inspirado no Prometheus.

grafana (Grafana): O "painel de vidro único". A nossa UI de dashboarding e alertas, ligada ao Prometheus e ao Loki.

3. Os Agentes (Os Coletores)

nginx-exporter: O "tradutor" que expõe as métricas do Nginx (como nginx_up) num formato que o Prometheus entende.

promtail (Promtail): O "agente" que "ouve" os logs de todos os outros contentores Docker e os "empurra" (push) para o Loki.

Como Executar

Certifique-se de que tem o Docker e o Docker Compose instalados.

Clone este repositório.

Execute o comando a partir da raiz do projeto:

docker compose up -d


A stack completa pode demorar 1-2 minutos a arrancar na primeira vez, especialmente o Grafana (que precisa de inicializar a sua base de dados interna).

Endpoints (Pontos de Acesso)

Após o arranque, os serviços estão disponíveis no seu localhost:

Aplicação (Nginx): http://localhost:8080

Visualização (Grafana): http://localhost:3000 (Login: admin / admin)

Métricas (Prometheus): http://localhost:9090

Logs (Loki API): http://localhost:3100

Habilidades Demonstradas

Este projeto valida um conjunto de competências essenciais para funções de NOC, SRE e DevOps:

Docker & Docker Compose: Orquestração de uma stack complexa de 7 serviços multi-contentor.

Redes Docker: Criação de uma rede bridge customizada (monitor-net) para permitir a descoberta de serviços por DNS interno (ex: http://web/stub_status).

Gestão de Recursos: Definição de limites de memória (deploy.resources.limits) para garantir que os serviços rodem de forma estável num ambiente com recursos limitados (8GB RAM).

Monitorização de Métricas (Prometheus):

Configuração de jobs de "scrape" (coleta) no prometheus.yml.

Uso de Exporters (como o nginx-prometheus-exporter) para "traduzir" métricas de serviços que não são nativos do Prometheus.

Compreensão da diferença entre métricas de "saúde" do exporter (up) e métricas de "saúde" do serviço (nginx_up).

Agregação de Logs (Loki & Promtail):

Configuração do Promtail para auto-descobrir e "raspar" (coletar) logs de outros contentores Docker usando o docker.sock.

Configuração do Loki para receber e armazenar logs de forma eficiente.

Visualização (Grafana):

Provisioning as Code: Uso de datasource.yml para auto-configurar as fontes de dados (Prometheus e Loki) no arranque.

Dashboarding: Demonstração de como criar painéis que combinam métricas (do Prometheus) e logs (do Loki) num único "painel de vidro".

Alertas (Grafana Alerting):

Configuração de "Contact Points" e "Notification Policies".

Criação de regras de alerta (Alert Rules) baseadas em queries de PromQL (ex: nginx_up < 1).

Engenharia de Caos & Resposta a Incidentes:

Simulação de uma falha de serviço (docker compose rm --stop web).

Verificação do ciclo de vida completo do alerta: Falha -> Deteção -> Firing (Alerta) -> Correção (docker compose up -d) -> Resolução (Normal).

Depuração (Debugging) de Sistemas:

Análise de logs de contentores (docker compose logs ...) para encontrar a causa-raiz de falhas.

Depuração de erros de montagem de volumes (not a directory).

Resolução de problemas de "estado preso" (stale state) e volumes corrompidos (docker compose down, docker volume rm).

Identificação de ficheiros não salvos (o "bug" do ●) como causa de falhas de configuração.
