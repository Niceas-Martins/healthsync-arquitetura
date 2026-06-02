# ADR 0002 — Padrões de Resiliência

**Título:** Padrões de Resiliência para Microserviços do HealthSync

**Status:** Aceito

**Contexto:**
A arquitetura de microserviços cria dependências entre serviços independentes que, sem isolamento, podem gerar falhas em cascata. Um serviço de IA sobrecarregado bloqueando o serviço de consultas, por exemplo, comprometeria o requisito de disponibilidade ≥ 99,5 % (RNF-3). É necessário definir padrões centralizados de resiliência para garantir comportamento uniforme e degradação graciosa em todo o sistema.

**Decisão:**
Adotar um conjunto em camadas de padrões de resiliência, implementados via **API Gateway** na borda e **Resilience4j** nos serviços internos:

API Gateway (Kong + AWS API Gateway): rate limiting por usuário/IP, validação de JWT, timeout global de 5 s e retry com backoff exponencial (até 2 tentativas em erros 5xx).
Circuit Breaker (Resilience4j): limiar de abertura em 50 % de falhas em janela de 10 requisições, com estado OPEN por 30 s. Aplicado obrigatoriamente no fluxo Consultas → IA, com fallback de degradação graciosa (prontuário exibido sem recomendação). No fluxo Prontuário → Banco, o fallback é cache read-only em Redis.
Bulkhead: thread pools dedicados por grupo de chamadas — IA (10 threads), banco de prontuário (20), vídeo (15) e notificações (8) — impedindo que um serviço lento consuma recursos dos demais.
Health Checks:liveness e readiness probes no Kubernetes, com alertas no Grafana quando qualquer Circuit Breaker permanecer aberto por mais de 60 s.

As alternativas rejeitadas foram: retry ilimitado sem Circuit Breaker (amplifica falhas); timeout apenas no cliente (inconsistência entre serviços); e Service Mesh com Istio (complexidade operacional não justificada no volume atual — reavaliado quando os serviços superarem 15).

**Consequências:**

(+) Circuit Breaker previne falhas em cascata, sustentando a meta de 99,5 % de disponibilidade.

(+) Médicos continuam atendendo e acessando prontuários mesmo com o serviço de IA fora do ar.

(+) Bulkhead garante que picos em um serviço não degradem os demais.

(-) Parâmetros de Circuit Breaker mal calibrados podem gerar falsos positivos e degradar a experiência desnecessariamente.

(-) O fallback com cache Redis pode exibir dados levemente desatualizados — aceitável para leitura de histórico, mas os endpoints que admitem cache devem ser documentados e sinalizados na interface.
