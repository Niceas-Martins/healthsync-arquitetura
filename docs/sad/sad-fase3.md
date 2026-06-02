# SAD — Software Architecture Document
## HealthSync — Plataforma de Telemedicina Inteligente

> **Versão:** 1.0 · **Ciclo:** 4 · **Status:** Aprovado

---

## 1. Visão Executiva

### O que é o HealthSync

O HealthSync é uma plataforma de telemedicina inteligente que conecta pacientes e profissionais de saúde de forma segura e acessível. A solução integra três pilares funcionais: **teleconsultas por videochamada**, **monitoramento remoto de sinais vitais via wearables** e **recomendações clínicas geradas por Inteligência Artificial**.

### Qual problema resolve

O acesso à saúde de qualidade ainda é limitado por barreiras geográficas, filas de espera e falta de continuidade no acompanhamento do paciente. O HealthSync endereça esses problemas ao:

- Eliminar a necessidade de deslocamento para consultas de acompanhamento e triagem;
- Permitir que médicos monitorem sinais vitais de pacientes crônicos em tempo real, mesmo à distância;
- Apoiar a tomada de decisão clínica com análise preditiva, reduzindo erros por sobrecarga de informação.

### Estado atual — Fase 4

Na Fase 4 (Ciclo 3), a arquitetura está consolidada com todas as decisões estruturais registradas em ADRs aceitos. Os microserviços principais estão definidos e mapeados, a estratégia de nuvem e escalabilidade foi estabelecida (ADR 0001), os padrões de resiliência estão especificados (ADR 0002) e o modelo de comunicação híbrido foi formalizado (ADR 0003). O projeto encontra-se na transição entre a fase de design arquitetural e o início da implementação incremental dos serviços de maior valor de negócio — Consulta, Prontuário e Agendamento.

---

## 2. Visões Arquiteturais

### 2.1 Visão de Contexto — C4 Nível 1

Apresenta o sistema como uma caixa preta e identifica todos os atores e sistemas externos que interagem com ele.

```mermaid
C4Context
    title Diagrama de Contexto — HealthSync

    Person(paciente, "Paciente", "Agenda consultas, participa de teleconsultas e visualiza seus dados de saúde via app ou web.")
    Person(medico, "Médico / Profissional de Saúde", "Realiza teleconsultas, acessa prontuários e recebe recomendações clínicas da IA.")

    System(healthsync, "HealthSync", "Plataforma de telemedicina com teleconsulta, monitoramento de wearables e recomendação por IA.")

    System_Ext(wearable, "Dispositivo Wearable", "Envia continuamente sinais vitais como frequência cardíaca e pressão arterial.")
    System_Ext(twilio, "Twilio / Daily.co", "Provedor SaaS de infraestrutura WebRTC para videochamadas.")
    System_Ext(notificacao, "Serviço de Email / Push", "Entrega lembretes de consulta e alertas de saúde aos usuários.")

    Rel(paciente, healthsync, "Usa", "HTTPS")
    Rel(medico, healthsync, "Usa", "HTTPS")
    Rel(wearable, healthsync, "Envia sinais vitais", "HTTPS / MQTT")
    Rel(healthsync, twilio, "Cria e gerencia salas de vídeo", "API REST")
    Rel(healthsync, notificacao, "Envia notificações e alertas", "API REST / SMTP")
```

---

### 2.2 Visão de Contêineres — C4 Nível 2

Detalha os contêineres que compõem o HealthSync — aplicações, serviços e armazenamentos de dados.

```mermaid
C4Container
    title Diagrama de Contêineres — HealthSync

    Person(paciente, "Paciente", "App mobile ou navegador web.")
    Person(medico, "Médico", "App mobile ou navegador web.")

    System_Boundary(healthsync, "HealthSync") {

        Container(gateway, "API Gateway", "Kong + AWS API Gateway", "Ponto de entrada único. Gerencia autenticação JWT, rate limiting, timeout global de 5s e retry com backoff.")

        Container(svc_consulta, "Serviço de Consulta", "Node.js / Spring Boot", "Gerencia o ciclo de vida das teleconsultas. Integra o SDK de vídeo e orquestra prontuário e IA durante a sessão.")
        Container(svc_prontuario, "Serviço de Prontuário", "Spring Boot", "CRUD de prontuários eletrônicos com controle de acesso por perfil profissional.")
        Container(svc_agendamento, "Serviço de Agendamento", "Node.js", "Gerencia agendamento, remarcação e cancelamento de consultas.")
        Container(svc_ia, "Serviço de IA", "Python / FastAPI", "Analisa dados clínicos e gera recomendações e alertas preditivos para apoio à decisão médica.")
        Container(svc_wearable, "Serviço de Wearables", "AWS Lambda (Serverless)", "Ingere dados de sinais vitais de forma assíncrona e os publica no broker de eventos.")
        Container(svc_notificacao, "Serviço de Notificações", "Node.js", "Consome eventos do broker e entrega lembretes e alertas via email e push.")

        ContainerDb(db_prontuario, "Banco de Prontuários", "Aurora PostgreSQL Multi-AZ", "Armazena prontuários eletrônicos com criptografia em repouso e alta disponibilidade.")
        ContainerDb(cache, "Cache", "Redis (ElastiCache)", "Cache de leitura de prontuários. Usado como fallback em caso de indisponibilidade do banco.")
        ContainerDb(broker, "Broker de Eventos", "Amazon MSK (Kafka) + SQS FIFO", "Comunicação assíncrona entre serviços. Kafka para wearables e auditoria; SQS FIFO para notificações e atualização de prontuário.")
    }

    System_Ext(twilio, "Twilio / Daily.co", "Infraestrutura WebRTC para videochamadas.")
    System_Ext(notif_ext, "Email / Push Provider", "Entrega final de notificações.")

    Rel(paciente, gateway, "Usa", "HTTPS")
    Rel(medico, gateway, "Usa", "HTTPS")

    Rel(gateway, svc_consulta, "Roteia", "REST / HTTP")
    Rel(gateway, svc_prontuario, "Roteia", "REST / HTTP")
    Rel(gateway, svc_agendamento, "Roteia", "REST / HTTP")
    Rel(gateway, svc_wearable, "Roteia eventos", "REST / HTTP")

    Rel(svc_consulta, svc_ia, "Solicita recomendação", "REST síncrono + Circuit Breaker")
    Rel(svc_consulta, svc_prontuario, "Lê/atualiza prontuário", "REST síncrono")
    Rel(svc_consulta, twilio, "Cria sala de vídeo", "API REST")

    Rel(svc_prontuario, db_prontuario, "Lê e grava", "SQL")
    Rel(svc_prontuario, cache, "Lê (fallback)", "Redis protocol")

    Rel(svc_wearable, broker, "Publica sinais vitais", "Kafka")
    Rel(svc_ia, broker, "Consome dados para análise batch", "Kafka")
    Rel(svc_agendamento, broker, "Publica evento de agendamento", "SQS FIFO")
    Rel(svc_prontuario, broker, "Publica atualização pós-consulta", "SQS FIFO")

    Rel(svc_notificacao, broker, "Consome eventos", "SQS / Kafka")
    Rel(svc_notificacao, notif_ext, "Entrega notificação", "SMTP / API")
```

---

### 2.3 Visão de Componentes — C4 Nível 3

Detalha os componentes internos do **Serviço de Consulta**, núcleo funcional da plataforma.

```mermaid
C4Component
    title Diagrama de Componentes — Serviço de Consulta

    Container_Boundary(svc_consulta, "Serviço de Consulta") {
        Component(ctrl, "Controller de Teleconsulta", "REST Controller", "Expõe endpoints para iniciar, encerrar e consultar o estado de uma teleconsulta.")
        Component(session_mgr, "Gerenciador de Sessão", "Service", "Orquestra o ciclo de vida da sessão: cria sala de vídeo, carrega prontuário e solicita recomendação da IA.")
        Component(video_client, "Cliente de Vídeo", "SDK Client", "Integra a API do Twilio/Daily.co para criar e encerrar salas WebRTC.")
        Component(prontuario_client, "Cliente de Prontuário", "HTTP Client", "Realiza chamadas síncronas REST ao Serviço de Prontuário.")
        Component(ia_client, "Cliente de IA", "HTTP Client + Circuit Breaker", "Solicita recomendações ao Serviço de IA com timeout de 2s e fallback de degradação graciosa via Resilience4j.")
    }

    Container_Ext(svc_prontuario, "Serviço de Prontuário", "Spring Boot")
    Container_Ext(svc_ia, "Serviço de IA", "Python / FastAPI")
    System_Ext(twilio, "Twilio / Daily.co", "WebRTC SaaS")

    Rel(ctrl, session_mgr, "Delega orquestração")
    Rel(session_mgr, video_client, "Solicita criação de sala")
    Rel(session_mgr, prontuario_client, "Solicita prontuário do paciente")
    Rel(session_mgr, ia_client, "Solicita recomendação clínica")

    Rel(video_client, twilio, "Cria sala de vídeo", "API REST / HTTPS")
    Rel(prontuario_client, svc_prontuario, "GET/PUT prontuário", "REST / HTTPS")
    Rel(ia_client, svc_ia, "POST dados clínicos", "REST / HTTPS")
```

---

### 2.4 Visão de Implantação

Descreve a distribuição dos contêineres na infraestrutura AWS `sa-east-1` (São Paulo), garantindo conformidade com a LGPD.

```mermaid
C4Deployment
    title Diagrama de Implantação — HealthSync (AWS sa-east-1)

    Deployment_Node(aws, "AWS — sa-east-1 (São Paulo)") {

        Deployment_Node(eks, "Amazon EKS (Kubernetes)", "Orquestrador de contêineres com HPA") {
            Container(gateway, "API Gateway", "Kong", "Borda de entrada com autenticação e resiliência.")
            Container(svc_consulta, "Serviço de Consulta", "Pod — Node.js", "Escalado horizontalmente via HPA.")
            Container(svc_prontuario, "Serviço de Prontuário", "Pod — Spring Boot", "Escalado horizontalmente via HPA.")
            Container(svc_agendamento, "Serviço de Agendamento", "Pod — Node.js", "Escalado horizontalmente via HPA.")
            Container(svc_notificacao, "Serviço de Notificações", "Pod — Node.js", "Consome eventos do broker.")
            Container(svc_ia, "Serviço de IA", "Pod GPU — FastAPI", "Escalado via KEDA por tamanho de fila.")
        }

        Deployment_Node(serverless, "AWS Lambda (Serverless)") {
            Container(svc_wearable, "Serviço de Wearables", "Lambda Function", "Ingestão de eventos esporádicos sem custo ocioso.")
        }

        Deployment_Node(dados, "Camada de Dados") {
            ContainerDb(db, "Aurora PostgreSQL", "Multi-AZ", "Prontuários com failover automático e criptografia.")
            ContainerDb(redis, "ElastiCache Redis", "Cache", "Fallback de leitura de prontuários.")
            ContainerDb(kafka, "Amazon MSK (Kafka)", "Broker", "Eventos de wearables e auditoria LGPD — retenção 5 anos.")
            ContainerDb(sqs, "Amazon SQS FIFO", "Fila", "Notificações e atualizações de prontuário com exactly-once.")
        }
    }

    System_Ext(twilio, "Twilio / Daily.co", "SaaS WebRTC externo.")

    Rel(gateway, svc_consulta, "Roteia", "HTTP interno")
    Rel(gateway, svc_prontuario, "Roteia", "HTTP interno")
    Rel(gateway, svc_agendamento, "Roteia", "HTTP interno")
    Rel(svc_consulta, svc_ia, "Recomendação", "HTTP + Circuit Breaker")
    Rel(svc_consulta, twilio, "Vídeo WebRTC", "HTTPS")
    Rel(svc_prontuario, db, "Lê/Grava", "SQL")
    Rel(svc_prontuario, redis, "Cache fallback", "Redis")
    Rel(svc_wearable, kafka, "Publica sinais vitais", "Kafka")
    Rel(svc_ia, kafka, "Consome para análise", "Kafka")
    Rel(svc_notificacao, sqs, "Consome eventos", "SQS")
```

---

## 3. Decisões Arquiteturais (ADRs)

| ADR | Título | Decisão Central | Status |
|-----|--------|-----------------|--------|
| ADR 0000 | Arquitetura de Microserviços | Cada funcionalidade principal opera como serviço independente via REST e mensageria assíncrona | Aceito |
| ADR 0001 | Estratégia de Nuvem e Escalabilidade | PaaS (EKS) + Serverless (Lambda) com escalabilidade horizontal via HPA/KEDA na AWS `sa-east-1` | Aceito |
| ADR 0002 | Padrões de Resiliência | API Gateway centralizado + Circuit Breaker (Resilience4j) + Bulkhead por thread pool | Aceito |
| ADR 0003 | Modelo de Comunicação | REST síncrono para fluxos imediatos; Kafka + SQS assíncrono para fluxos tolerantes a atraso | Aceito |

---

## 4. Atributos de Qualidade

| Atributo | Requisito (RNF) | Estratégia Arquitetural |
|---|---|---|
| **Disponibilidade** | ≥ 99,5 % | Multi-AZ, HPA, Circuit Breaker, liveness/readiness probes |
| **Desempenho** | < 3 s de resposta | REST síncrono nos fluxos de usuário, cache Redis, escalabilidade horizontal |
| **Segurança / LGPD** | Criptografia ponta a ponta | JWT no gateway, criptografia em repouso no Aurora, dados restritos à região `sa-east-1` |
| **Escalabilidade** | Horizontal automática | Kubernetes HPA para serviços de negócio, Lambda para wearables, KEDA para IA |
| **Integridade** | Dados clínicos não alterados | SQS FIFO com exactly-once, Kafka com retenção imutável para auditoria |
