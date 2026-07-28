Público-Alvo: B2C  (Pessoas que atuam como organizadores financeiros de suas famílias, casais e indivíduos com dependentes informais).

A Dor Principal: Mistura de fluxos de caixa. Quando o usuário empresta o cartão de crédito ou dinheiro para terceiros, o gasto aparece na fatura, bagunçando as métricas pessoais e dificultando o rastreamento do que já foi pago ou do que ainda precisam receber.

Proposta de Valor Única (USP): Um sistema de controle financeiro que permite isolar, categorizar e rastrear transações de terceiros dentro das contas do titular, garantindo previsibilidade sobre o próprio dinheiro e sobre o que está "emprestado".

---

# 📄 Spec do Projeto: Sistema Financeiro Distribuído & Inteligente

## 1. Visão Geral

Um ecossistema financeiro pessoal baseado em microsserviços. Ele permite o gerenciamento de contas, orçamentos (CRM Financeiro) e a ingestão assíncrona de extratos bancários, que são processados e categorizados automaticamente por Inteligência Artificial.

## 2. Arquitetura de Microsserviços

Dividiremos o sistema em responsabilidades claras para justificar o uso de diferentes frameworks.

### Serviço 1: O "Admin & CRM" (Foco: Framework Digital)

* **Tecnologia:** Python + **Django** + Django REST Framework + Banco de Dados PostgreSQL (DB 1).
* **Responsabilidade:** Gerenciamento do estado principal do usuário.
* **Funcionalidades:**
* Cadastro de usuários e autenticação (JWT).
* CRUD de Contas Bancárias, Cartões de Crédito e Metas de Orçamento.
* Painel administrativo nativo do Django configurado para gerenciar os dados mestre.
* Geração de relatórios consolidados no fim do mês.



### Serviço 2: O "Motor de Ingestão" (Foco: CI&T)

* **Tecnologia:** Python + **FastAPI** + **SQLAlchemy** + Banco de Dados PostgreSQL (DB 2).
* **Responsabilidade:** Receber e processar grandes volumes de dados (ex: upload de arquivos CSV/OFX de extratos bancários) de forma rápida e assíncrona.
* **Funcionalidades:**
* Endpoints de alta performance para receber os arquivos de extrato.
* Validação de payload ultrarrápida usando **Pydantic**.
* Em vez de processar o arquivo na hora (o que deixaria a API lenta), este serviço lê as linhas do extrato e as publica em uma **Fila de Mensagens**.



### Serviço 3: O "Worker de IA" (Foco: Airbnb e TCS)

* **Tecnologia:** Python + Celery (ou consumidor customizado) + Integração com LLM (OpenAI/Gemini).
* **Responsabilidade:** Processamento em background e inteligência.
* **Funcionalidades:**
* Consome as transações brutas da fila de mensagens.
* Envia a descrição da transação para a IA ("Categorize o gasto: PGTO UBER *EATS").
* Salva a transação categorizada e notifica o Serviço 1 (via API ou publicando em outra fila) de que o saldo do usuário precisa ser atualizado.



## 3. Mensageria e Orquestração

* **Fila de Mensagens (Message Broker):** **RabbitMQ** ou **Kafka** (RabbitMQ é mais fácil para começar, Kafka brilha se quiser falar de altíssima escala). Aqui você prova que sabe desacoplar sistemas.
* **Orquestração de Containers:**
* Crie um `Dockerfile` individual para cada serviço (Django, FastAPI, Worker).
* Use o **Docker Compose** para orquestrar tudo localmente. Seu `docker-compose.yml` vai subir os bancos de dados, o RabbitMQ e os três serviços em uma rede virtual, simulando um ambiente real.



## 4. Observabilidade e Logs (O Diferencial de Senioridade)

Para cobrir a necessidade da CI&T (Azure Log Analytics) e Framework Digital (Governança):

* **Logs Estruturados:** Configure as aplicações para cuspir logs no formato JSON.
* **Correlation ID:** Gere um ID único quando o extrato for feito o upload no FastAPI e repasse esse ID para a Fila, e da Fila para o Worker. *Por que?* Se der erro na IA, você consegue rastrear o log desde o exato momento em que o usuário clicou em "Enviar Extrato". Recrutadores amam isso.
* **Opcional de Ouro:** Suba um container do **Prometheus** ou **Elasticsearch** só para coletar esses logs e provar que você sabe montar uma esteira de monitoramento.

## 5. Qualidade e CI/CD (Foco: Airbnb)

* **Testes (Pytest):** Faça testes unitários para a lógica de categorização no Django e testes de integração no FastAPI (usando o `TestClient` do FastAPI) com **mocking** (simulação) do RabbitMQ e da API da OpenAI.
* **Pipeline Automatizada:** Configure um arquivo `.github/workflows/main.yml` no GitHub Actions. Toda vez que fizer um commit, a pipeline deve subir os containers (usando Docker), rodar o *flake8/black* (linters) e executar todos os testes.

---

### O Fluxo de Dados (A História para contar na Entrevista)

Imagine que o recrutador pergunte como seu sistema funciona. Você desenhará mentalmente este fluxo:

1. *"O usuário entra no portal feito em **Django** e cadastra que sua meta de gastos com comida é R$ 500."*
2. *"Ele faz o upload do seu extrato bancário no endpoint **FastAPI**. O FastAPI valida o arquivo instantaneamente, quebra as transações e joga tudo em uma fila do **RabbitMQ**, retornando um 'Status 202: Processando' para o usuário não ficar esperando."*
3. *"O meu **Worker assíncrono** pega essas transações da fila, se comunica com a IA para categorizar os gastos e salva no banco de dados com **SQLAlchemy**."*
4. *"Se o limite de R$ 500 for ultrapassado, o sistema registra um evento de alerta em formato JSON, que é capturado pela minha ferramenta de **monitoramento de logs**, garantindo rastreabilidade total da operação, exatamente como é exigido em ambientes de nuvem."*

Esse é um projeto que demonstra conhecimento de ponta a ponta: do banco de dados relacional e APIs RESTful até filas assíncronas, inteligência artificial e cultura DevOps!