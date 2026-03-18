# 🏥 HealthSync – Arquitetura de Software

## 📌 CICLO 1: Visão e Requisitos (Fase 1)

---

## 1.1 Resumo do Cenário de Negócio

A arquitetura proposta para o sistema HealthSync tem como objetivo desenvolver uma plataforma de telemedicina inteligente que integre consultas online, monitoramento remoto de pacientes por meio de dispositivos wearables e um sistema de recomendação de tratamentos baseado em Inteligência Artificial. A plataforma permitirá a realização de atendimentos virtuais com segurança, incluindo videochamadas, troca de mensagens e acesso a prontuários eletrônicos em tempo real.

A solução visa oferecer um serviço de saúde personalizado, acessível e seguro, possibilitando que médicos acompanhem continuamente os sinais vitais e dados clínicos dos pacientes, mesmo à distância. Além disso, os algoritmos de IA auxiliarão na análise preditiva, identificação de padrões e apoio à tomada de decisão clínica.

---

## ⚙️ 1.2 Requisitos Funcionais (RFs)

Requisitos Funcionais do Sistema HealthSync:

1. **Cadastro e Autenticação de Usuários:** O sistema deve permitir o cadastro e login de pacientes e profissionais de saúde de forma segura.  
2. **Agendamento de Consultas Online:** O sistema deve permitir que pacientes agendem, remarquem e cancelem consultas com médicos.  
3. **Realização de Teleconsultas:** O sistema deve possibilitar consultas por videochamada com suporte a áudio, vídeo e chat em tempo real.  
4. **Monitoramento de Dados de Wearables:** O sistema deve coletar e exibir dados de dispositivos de monitoramento (como frequência cardíaca e pressão).  
5. **Sistema de Recomendação com IA:** O sistema deve analisar dados clínicos e fornecer recomendações ou alertas para apoio à decisão médica.  
6. **Notificações e Alertas:** O sistema deve enviar lembretes de consultas e alertas relacionados à saúde dos pacientes.  
7. **Acesso e Atualização de Prontuário:** O sistema deve permitir que médicos acessem e atualizem os prontuários dos pacientes.  

---

## 🔒 1.3 Atributos de Qualidade (RNFs)

1 **Segurança e Privacidade:** O sistema deve possuir criptografia ponta a ponta. Estando em conformidade com LGPD.  
2 **Escalabilidade:** A arquitetura deve ser baseada em microserviços e infraestrutura em nuvem, permitindo escalabilidade horizontal automática para suportar picos de consultas.  
3 **Disponibilidade:** O sistema deve garantir 99,5% ou mais de disponibilidade diária.  
4 **Integridade de Dados:** O sistema deve garantir que os dados clínicos coletados não sejam alterados.  
5 **Desempenho:** O tempo de resposta do sistema deve ser inferior a 3 segundos para carregamento de consultas, prontuários e recomendações automáticas.  

---

## 🧩 1.3 Diagrama de Contexto (C4 Nível 1)

Diagrama está na pasta /diagrams

---

## 🎯 1.4 Classificação da Estratégia

• Classificação: Balanceada  

• Justificativa: A estratégia do HealthSync é considerada balanceada pois combina tecnologias já consolidadas, como microserviços em nuvem e telemedicina, com elementos inovadores, como o uso de Inteligência Artificial para suporte à decisão clínica. Isso reduz riscos ao utilizar práticas maduras de mercado, ao mesmo tempo em que incorpora inovação de forma controlada. Em relação à natureza do software descrita por Roger Pressman, o sistema evolui de forma incremental, lidando com complexidade e mudanças contínuas no domínio da saúde. Além disso, a adoção de padrões conhecidos aumenta a confiabilidade e facilita a manutenção, enquanto a IA agrega valor competitivo sem comprometer a estabilidade do sistema.

---

## 🧾 1.5 ADR

1. Título: Arquitetura do Sistema baseada em Microserviços.  
2. Status: Aceito.  

3. Contexto: O HealthSync precisa processar simultaneamente chamadas de vídeo de telemedicina, ingestão de dados em tempo real e gerar recomendações de IA. A arquitetura monolítica apresenta riscos de acoplamento, na qual havendo falha no serviço de IA possibilita derrubar todo o portal de consultas. Além dos requisitos de escalabilidade e conformidade com a LGPD exigir isolamento de dados e serviços.  

4. Decisão: A escolha utilizar a arquitetura de Microserviços está relacionada onde cada funcionalidade principal como telemedicina, prontuário e monitoramento de Wearables são operados como serviço independente, comunicando-se via API REST e mensageira assíncrona.  

5. Consequências: Ganhos(+); Perdas(-)  

(+)Performance: Utilizando a arquitetura de Microserviços permite o escalonamento independente do módulos dos sinais vitais, garantido que o tempo de resposta permaneça inferior a 3 segundos.  

(+)Segurança: Isolamento de dados clínicos em serviços específicos, facilitando a implementação de criptografia ponta a ponta e auditoria LGPD.  

(+)Confiabilidade: Como o escalonamento dos serviços são independentes, se o serviço de IA falhar ou apresentar atraso, o médico ainda consegue realizar consulta e acessar o prontuário sem interrupções no sistema.  

(-)Custo: Maior gasto com múltiplas instâncias de servidores, custo elevado de transferência de dados entre Microserviços e dispositivos externos.  

(-)Complexidade: Necessidade de gerenciar a comunicação entre serviços e garantir a consistência dos dados clínicos.  

6. Alternativas: Considerando mas não escolhida-Monolito Modular. Inicialmente considerado para acelerar o desenvolvimento inicial. Porém foi rejeitado pela dificuldade com a escalabilidade futura do sistema e de não conseguir atingir a disponibilidade de 99,5% exigido.

(-)Complexidade: Necessidade de gerenciar a comunicação entre serviços e garantir a consistência dos dados clínicos.  

6. Alternativas: Considerando mas não escolhida-Monolito Modular. Inicialmente considerado para acelerar o desenvolvimento inicial. Porém foi rejeitado pela dificuldade com a escalabilidade futura do sistema e de não conseguir atingir a disponibilidade de 99,5% exigido.
