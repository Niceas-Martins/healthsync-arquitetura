# ADR 0001 — Estratégia de Nuvem e Escalabilidade

**Título:** Estratégia de Nuvem e Escalabilidade do HealthSync

**Status:** Aceito

**Contexto:**
O HealthSync precisa sustentar picos imprevisíveis de carga — surtos de teleconsultas em horários comerciais, ingestão contínua de dados de wearables 24 h/dia e inferência de IA sob demanda. Os requisitos não funcionais estabelecem disponibilidade ≥ 99,5 % (RNF-3), tempo de resposta inferior a 3 segundos para consultas, prontuários e recomendações (RNF-5), escalabilidade horizontal automática com microserviços (RNF-2) e conformidade com a LGPD, exigindo que os dados clínicos permaneçam em território nacional. A decisão de adotar microserviços já foi registrada no ADR anterior; este ADR define "onde" e "como" esses serviços serão hospedados e escalados.

**Decisão:**
Adotar uma estratégia **híbrida PaaS + Serverless** sobre um provedor de nuvem pública com presença em data center no Brasil (AWS São Paulo — `sa-east-1` ou Google Cloud `southamerica-east1`), organizando a infraestrutura em camadas conforme a natureza de cada serviço:

- **Serviços de negócio** (consulta, prontuário, agendamento): **PaaS — Kubernetes gerenciado (EKS / GKE)**, com controle fino de recursos, deploys Blue/Green e Horizontal Pod Autoscaler (HPA) acionado quando a utilização de CPU ultrapassar 70 % ou a latência P95 superar 1,5 s.
- **Ingestão de dados de wearables**: **Serverless (AWS Lambda / Cloud Run)**, pois os eventos são esporádicos e o modelo por invocação elimina custo ocioso.
- **IA / Recomendação**: **PaaS — instâncias GPU gerenciadas (SageMaker / Vertex AI)**, com autoscaling por fila de requisições via KEDA.
- **Armazenamento de prontuários**: **PaaS — banco gerenciado (Aurora PostgreSQL / Cloud SQL)**, com alta disponibilidade multi-AZ nativa e criptografia em repouso.
- **Entrega de mídia (vídeo)**: **SaaS — SDK especializado (Twilio / Daily.co)**, reduzindo risco operacional de WebRTC e garantindo SLA de vídeo.

A **escalabilidade horizontal** é a estratégia principal para os serviços de negócio. A **escalabilidade vertical** fica restrita ao banco de dados relacional, onde transações ACID não se distribuem trivialmente de forma horizontal.

As alternativas avaliadas e não escolhidas foram: **IaaS puro**, rejeitado pelo alto custo operacional de patching e provisionamento manual incompatível com a escalabilidade automática exigida; **Serverless total**, rejeitado porque cold starts violam a latência de 3 s em fluxos síncronos como prontuário e videochamada; e **PaaS puro (Kubernetes para tudo)**, descartado pelo custo elevado em cargas esparsas como alertas noturnos e coleta de sinais vitais de baixa frequência.

**Consequências:**

(+) Disponibilidade:* A infraestrutura multi-AZ com autoscaling automático reduz o MTTR e aumenta a probabilidade de atingir os 99,5 % de disponibilidade exigidos.

(+) Custo otimizado:* O modelo Serverless para wearables elimina custo de instâncias ociosas nos períodos de baixo volume, como madrugadas.

(+) Conformidade LGPD:* A região `sa-east-1` garante por padrão que dados clínicos não trafeguem ou sejam armazenados fora do Brasil.

(+) Agilidade:* O PaaS remove a responsabilidade de gerência de sistema operacional e hardware da equipe, que pode focar em desenvolvimento de produto.

(-) Vendor lock-in:* Serviços gerenciados como RDS e SageMaker aumentam a dependência do provedor escolhido. Mitigado com uso de Terraform e imagens OCI portáteis.

(-) Complexidade de billing:* Coexistem três modelos de custo  por hora de instância, por requisição Serverless e por hora de GPU, o que exige monitoramento FinOps contínuo para evitar surpresas orçamentárias.

(-) Cold starts pontuais:* Funções Serverless de ingestão podem apresentar latência extra na primeira invocação após período ocioso. Aceitável dado o contexto assíncrono da coleta de wearables, onde atrasos de segundos não impactam a decisão clínica imediata.
