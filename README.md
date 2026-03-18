# healthsync-arquitetura
Arquitetura de Software - HealthSync

# 🏥 HealthSync – Arquitetura de Software

## 📌 CICLO 1: Visão e Requisitos (Fase 1)

---

## 1.1 Resumo do Cenário de Negócio

A arquitetura proposta para o sistema HealthSync tem como objetivo desenvolver uma plataforma de telemedicina inteligente que integre consultas online, monitoramento remoto de pacientes por meio de dispositivos wearables e um sistema de recomendação de tratamentos baseado em Inteligência Artificial.

A plataforma permitirá a realização de atendimentos virtuais com segurança, incluindo videochamadas, troca de mensagens e acesso a prontuários eletrônicos em tempo real.

A solução visa oferecer um serviço de saúde personalizado, acessível e seguro, possibilitando que médicos acompanhem continuamente os sinais vitais e dados clínicos dos pacientes, mesmo à distância. Além disso, os algoritmos de IA auxiliarão na análise preditiva, identificação de padrões e apoio à tomada de decisão clínica.

---

## ⚙️ 1.2 Requisitos Funcionais (RFs)

1. **Cadastro e Autenticação de Usuários:** O sistema deve permitir o cadastro e login de pacientes e profissionais de saúde de forma segura.  
2. **Agendamento de Consultas Online:** O sistema deve permitir que pacientes agendem, remarquem e cancelem consultas com médicos.  
3. **Realização de Teleconsultas:** O sistema deve possibilitar consultas por videochamada, com suporte a áudio, vídeo e chat em tempo real.  
4. **Monitoramento de Dados de Wearables:** O sistema deve coletar e exibir dados de dispositivos de monitoramento, como frequência cardíaca e pressão arterial.  
5. **Sistema de Recomendação com IA:** O sistema deve analisar dados clínicos e fornecer recomendações ou alertas para apoio à decisão médica.  
6. **Notificações e Alertas:** O sistema deve enviar lembretes de consultas e alertas relacionados à saúde dos pacientes.  
7. **Acesso e Atualização de Prontuário:** O sistema deve permitir que médicos acessem e atualizem os prontuários dos pacientes.  

---

## 🔒 1.3 Atributos de Qualidade (RNFs)

1. **Segurança e Privacidade:** O sistema deve possuir criptografia ponta a ponta, estando em conformidade com a LGPD.  
2. **Escalabilidade:** A arquitetura deve ser baseada em microserviços e infraestrutura em nuvem, permitindo escalabilidade horizontal automática para suportar picos de consultas.  
3. **Disponibilidade:** O sistema deve garantir 99,5% ou mais de disponibilidade diária.  
4. **Integridade de Dados:** O sistema deve garantir que os dados clínicos coletados não sejam alterados indevidamente.  
5. **Desempenho:** O tempo de resposta do sistema deve ser inferior a 3 segundos para carregamento de consultas, prontuários e recomendações automáticas.  

---

## 🧩 1.4 Diagrama de Contexto (C4 – Nível 1)

![Diagrama de Contexto](./diagrams/context-diagram.png)

---

## 🎯 1.5 Classificação da Estratégia

**Classificação:** Balanceada  

**Justificativa:**  
A estratégia do HealthSync é considerada balanceada, pois combina tecnologias consolidadas, como microserviços em nuvem e telemedicina, com elementos inovadores, como o uso de Inteligência Artificial para suporte à decisão clínica.

Essa abordagem reduz riscos ao utilizar práticas maduras de mercado, ao mesmo tempo em que incorpora inovação de forma controlada. Em relação à natureza do software descrita por Roger Pressman, o sistema evolui de forma incremental, lidando com complexidade e mudanças contínuas no domínio da saúde.

Além disso, a adoção de padrões conhecidos aumenta a confiabilidade e facilita a manutenção, enquanto a IA agrega valor competitivo sem comprometer a estabilidade do sistema.

---

## 🧾 1.6 ADR – Arquitetura de Microserviços

**Título:** Arquitetura do Sistema baseada em Microserviços  
**Status:** Aceito  

### Contexto
O HealthSync precisa processar simultaneamente chamadas de vídeo de telemedicina, ingestão de dados em tempo real e geração de recomendações por Inteligência Artificial.

A arquitetura monolítica apresenta riscos de alto acoplamento, onde a falha em um serviço (como o de IA) pode comprometer todo o sistema. Além disso, os requisitos de escalabilidade e conformidade com a LGPD exigem isolamento de dados e serviços.

### Decisão
A escolha pela arquitetura de microserviços se deve ao fato de que cada funcionalidade principal — como telemedicina, prontuário eletrônico e monitoramento de wearables — será implementada como um serviço independente, comunicando-se por meio de APIs REST e mensageria assíncrona.

### Consequências

**Ganhos (+):**
- Performance com escalabilidade independente  
- Segurança com isolamento de dados  
- Confiabilidade com tolerância a falhas  

**Perdas (-):**
- Maior custo de infraestrutura  
- Maior complexidade de comunicação e consistência de dados  

### Alternativa considerada
Monólito modular, descartado por limitações de escalabilidade, disponibilidade (≥ 99,5%) e maior acoplamento.
