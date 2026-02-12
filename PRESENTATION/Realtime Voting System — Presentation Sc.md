Sistema de Votação em Tempo Real — Roteiro da Apresentação
[~1 min] Abertura & Definição do Problema

Então, o que estamos construindo aqui é um sistema de votação global em tempo real. Pensem nele como uma plataforma SaaS que poderia ser usada para eleições, tomada de decisão corporativa, cooperativas, pesquisas — qualquer situação onde você precise que pessoas votem de forma segura e em larga escala.

Os números para os quais estamos projetando são sérios: até 300 milhões de usuários registrados, com picos de tráfego de 240.000 requisições por segundo. Para colocar em perspectiva, isso é cerca de 100x a carga base atingindo o sistema de uma só vez — pensem no dia da eleição, um grande debate, notícias de última hora.

Os principais desafios se resumem a cinco coisas: integridade dos dados nessa escala, prevenção contra fraudes e bots, lidar com esses picos massivos de tráfego, garantir exatamente um voto por pessoa por pesquisa, e mostrar os resultados em tempo real.

[~1 min] Princípios de Design

Temos sete princípios fundamentais guiando cada decisão.

Primeiro, segurança em primeiro lugar — defesa em profundidade. Cada camada assume que houve uma violação e implementa seus próprios controles de forma independente. Isso não é uma funcionalidade que adicionamos no final; é a base de tudo.

Escalabilidade por padrão — tudo é horizontal e stateless desde o primeiro dia. Não podemos nos dar ao luxo de reprojetar quando os picos de tráfego chegarem.

Arquitetura orientada a eventos — desacoplamos tudo através de fluxos de eventos. Requisição-resposta síncrona a 240K RPS simplesmente esmagaria qualquer banco de dados. Então as operações críticas são assíncronas.

Computação stateless — nenhum servidor mantém estado persistente. Qualquer instância pode morrer e ser substituída sem perda de dados.

Também temos anti-abuso em múltiplas camadas, dados auditáveis com logs append-only à prova de adulteração, e tratamos falhas como uma condição normal — o sistema se auto-recupera automaticamente.

[~1,5 min] Estratégia de Segurança & Anti-Bot

Segurança é provavelmente a parte mais interessante, então deixem-me apresentar as camadas.

Na borda, temos o CloudFront e o AWS WAF cuidando da proteção contra DDoS, limitação de taxa e bloqueio de ataques comuns como injeção SQL e XSS. O AWS Shield Advanced fica na frente para ataques volumétricos.

Para identidade, estamos usando login social OAuth — Google, Apple, Facebook — com um serviço de autenticação customizado em Java. Deliberadamente escolhemos não usar Auth0 ou Keycloak. O tradeoff é mais esforço de implementação, mas ganhamos controle total, sem vendor lock-in, e é muito mais barato com 300 milhões de usuários.

Para verificação de identidade, integramos o SumSub para biometria facial e detecção de vivacidade. Os usuários fazem um vídeo selfie e enviam um documento de identidade oficial. Isso previne contas falsas e deepfakes. Nenhum dado biométrico bruto chega ao nosso backend.

No lado do dispositivo, o FingerprintJS nos dá fingerprinting passivo do dispositivo — detecta emuladores, VMs, fazendas de bots e abuso de múltiplas contas. E o Cloudflare Turnstile atua como um substituto invisível do CAPTCHA.

Então vocês têm seis camadas trabalhando juntas: rede, identidade, inteligência de dispositivo, segurança da aplicação, criptografia de dados e monitoramento de auditoria. Um atacante teria que vencer todas elas.

[~1 min] Arquitetura & Serviços Principais

A arquitetura geral é de microsserviços no EKS — escolhemos Kubernetes ao invés do ECS pela flexibilidade de autoscaling, especificamente HPA e KEDA para métricas customizadas.

Temos cinco serviços principais:

O Serviço de Autenticação lida com toda a autenticação — é nossa autoridade de identidade interna integrando com os provedores OAuth e o SumSub.

O Serviço de Votação é o motor principal. Ele é tanto produtor quanto consumidor Kafka. Recebe votos, valida-os, impõe idempotência através do Redis, publica no Kafka, e então no lado consumidor persiste no PostgreSQL e atualiza contadores em tempo real.

O Serviço de Notificação consome eventos de votos contabilizados e transmite atualizações em tempo real para os clientes via SSE — Server-Sent Events. Escolhemos SSE ao invés de WebSockets porque é unidirecional, usa menos memória por conexão, tem reconexão automática integrada e funciona melhor através de firewalls. O tradeoff de latência é cerca de 100ms vs 50ms — totalmente aceitável para transmissão de resultados.

Depois temos o Serviço de Auditoria e o Serviço de Backoffice para operações administrativas.

[~1 min] Armazenamento de Dados & Garantia de Voto Único

Para o banco de dados, escolhemos PostgreSQL ao invés do MySQL. Os principais motivos: o PostgreSQL tem um planejador de consultas mais sofisticado, particionamento declarativo nativo, replicação binária a nível de WAL com zero perda de dados, funções criptográficas nativas e forte suporte a auditoria com pgAudit.

Usamos relacionamentos aplicados pela aplicação ao invés de chaves estrangeiras no banco de dados. No mundo de microsserviços, cada serviço é dono do seu schema, e chaves estrangeiras no banco não conseguem abranger múltiplos bancos. Elas também adicionam 10-30% de overhead nas escritas, e na nossa escala isso importa.

A garantia de voto único acontece em múltiplos níveis: tokens de votação globalmente únicos, chaves criptográficas de voto de uso único e fortes restrições de unicidade no nível do banco de dados com escritas condicionais atômicas.

[~1 min] Auditabilidade

Isso é crítico para confiança e conformidade legal. Usamos uma abordagem híbrida: PostgreSQL para dados operacionais e S3 Object Lock para a trilha de auditoria imutável.

Por que não apenas PostgreSQL? Porque é um banco de dados transacional — administradores podem atualizar, deletar, até dropar tabelas. Não existe uma verdadeira capacidade WORM. Exploramos triggers, revogação de permissões, abordagens baseadas em WAL — todas falham porque alguém com acesso suficiente pode adulterar os dados.

O S3 Object Lock em modo Compliance nos dá um verdadeiro write-once-read-many. Nem mesmo a conta root da AWS pode deletar objetos durante o período de retenção.

Construímos uma cadeia de hashes por cima — o hash de cada voto inclui o hash do voto anterior, criando uma cadeia à prova de adulteração. Se alguém modificar um único registro, quebra a sequência inteira. Checkpoints a cada 10.000 votos permitem verificação rápida.

O custo? Cerca de $100 por ano para 300 milhões de votos. O QLDB seria 3-5x mais caro.

[~1 min] Observabilidade

Seguimos os três pilares — métricas, logs e traces — tudo open source.

Prometheus para métricas com Thanos para armazenamento de longo prazo. Grafana como frontend unificado. Jaeger para rastreamento distribuído — essencial em uma arquitetura de microsserviços para seguir uma requisição entre os serviços. Loki para agregação de logs. E OpenTelemetry como o padrão de instrumentação vendor-neutral unindo tudo.

Tudo é correlacionado através de trace IDs. Você pode clicar em uma métrica no Grafana, pular para os traces relacionados e depois pular para os logs relacionados. É assim que você depura um sistema rodando a 240K RPS.

Nossos SLOs são agressivos: 99,99% de taxa de sucesso para submissão de votos — isso é apenas cerca de 26 minutos de budget de erro por mês.

[~30 seg] Stack Tecnológica & Encerramento

Na stack tecnológica, estamos usando Scala com ZIO para o backend — tipagem forte, excelente suporte a concorrência e tratamento funcional de erros. SBT para builds, K6 para testes de performance e ZIO Test para testes unitários e de integração.

Para encerrar — este é um sistema projetado para confiança em escala. Cada decisão, desde o modelo de segurança em seis camadas até a trilha de auditoria com cadeia de hashes até a arquitetura orientada a eventos, serve a um único objetivo: garantir que cada voto conte, exatamente uma vez, e que todos possam verificar.

Ficarei feliz em responder perguntas.

Total: ~8 minutos
