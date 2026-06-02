# ADR 0003 — Modelo de Comunicação entre Microserviços

**Título:** Modelo de Comunicação Síncrono e Assíncrono no HealthSync

**Status:** Aceito

**Contexto:**
Com a adoção de microserviços, a escolha do modelo de comunicação impacta diretamente a latência percebida pelo usuário (RNF-5: < 3 s), o acoplamento temporal entre serviços (RNF-3: disponibilidade ≥ 99,5 %), a integridade dos dados clínicos (RNF-4) e a rastreabilidade exigida pela LGPD (RNF-1). Nenhum modelo único atende a todos os fluxos do sistema — fluxos que exigem resposta imediata ao usuário têm necessidades diferentes de fluxos tolerantes a atraso.

**Decisão:**
Adotar um **modelo híbrido**: comunicação **síncrona via REST/HTTPS** para fluxos que exigem resposta imediata e **assíncrona via Kafka e SQS** para fluxos tolerantes a atraso.

**Fluxos síncronos (REST/HTTPS):** autenticação e autorização, carregamento de prontuário durante consulta, confirmação de agendamento e recomendação de IA em consulta ativa (com timeout de 2 s e fallback  ver ADR 0002). Nesses casos o usuário aguarda a resposta na UI, tornando a comunicação síncrona obrigatória.

**Fluxos assíncronos (Kafka / SQS):** ingestão de dados de wearables (Kafka, alto volume contínuo), alertas e notificações ao paciente (SQS + SNS, entrega eventual aceitável), atualização de prontuário pós-consulta (SQS FIFO, garante ordem dos eventos) e auditoria de acesso a dados clínicos (Kafka com retenção de 5 anos, atendendo rastreabilidade LGPD). Nesses fluxos o receptor pode falhar sem bloquear o chamador.

As alternativas rejeitadas foram: síncrono total, pois cria acoplamento temporal — uma falha no serviço de notificações derrubaria o agendamento; e assíncrono total, pois fluxos como autenticação e prontuário exigem resposta imediata, tornando a UX imprevisível.

**Consequências:**

(+) Fluxos síncronos garantem resposta imediata onde o médico e o paciente precisam, cumprindo o requisito de latência < 3 s.

(+) Fluxos assíncronos toleram falhas temporárias de serviços consumidores sem afetar o chamador, contribuindo para a disponibilidade de 99,5 %.

(+) O Kafka como log imutável de eventos atende nativamente ao requisito de rastreabilidade e auditoria da LGPD.

(-) Consistência eventual nos fluxos assíncronos: dados de wearables e prontuário pós-consulta podem estar levemente defasados — a equipe clínica deve ser orientada sobre essa característica.

(-) A operação do Kafka exige familiaridade com particionamento, consumer groups e gestão de offsets, aumentando a complexidade operacional da equipe.
