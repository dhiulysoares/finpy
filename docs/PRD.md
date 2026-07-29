# Product Requirements Document (PRD) — Finpy

## 1. Visão do Produto

### 1.1 Declaração de Visão
O **Finpy** é uma plataforma de controle financeiro pessoal focada em resolver a **mistura de fluxos de caixa e o rastreamento de despesas de terceiros**. O sistema permite isolar, categorizar e alocar transações de forma transparente, eliminando a gestão manual complexa em planilhas e fornecendo clareza imediata sobre quem deve o quê.

### 1.2 Problema Central
Usuários que centralizam cartões de crédito e pagamentos da família ou grupo de convivência enfrentam atrito ao tentar separar seus gastos pessoais do que foi pago em nome de terceiros (cônjuge, parentes, amigos). Conforme o volume de transações cresce, torna-se inviável manter o histórico discriminado, entender o saldo devedor de cada pessoa e realizar acertos financeiros sem retrabalho manual.

### 1.3 Proposta Única de Valor (USP)
Ingestão e categorização automatizada de extratos aliada a um **Ledger de Reembolsos Itemizado**, permitindo transformar transações brutas de cartão ou conta em saldos devedores auditáveis entre contatos e usuários da plataforma.

---

## 2. Personas

### 2.1 Persona Principal: O Organizador Financeiro ("Dhiuly")
* **Perfil:** Centraliza as contas da casa e assume a responsabilidade de pagar faturas de cartão de crédito e despesas recorrentes da família.
* **Necessidades:**
  * Importar extratos bancários sem ter que digitar transação por transação.
  * Selecionar compras específicas e vincular o valor total ou parcial a um terceiro.
  * Ter uma visão consolidada de quanto cada pessoa deve a ele, com o detalhamento item a item.
* **Dores:** Atualização manual desgastante em planilhas, esquecimento de cobranças picadas, dificuldade de provar o histórico do saldo devedor.

### 2.2 Persona Secundária: O Parceiro/Membro da Casa ("Leticia")
* **Perfil:** Compartilha contas da casa ou utiliza o cartão do titular para compras específicas.
* **Necessidades:**
  * Transparência sobre o que lhe está sendo cobrado.
  * Acompanhar seu saldo devedor de forma clara, podendo ter sua própria conta para gerenciar finanças pessoais.
* **Dores:** Incerteza sobre o valor total devido e falta de discriminação dos itens consumidos.

### 2.3 Entidade Terceira: Devedor Não Cadastrado ("Nei / Lurdinha")
* **Perfil:** Amigos ou familiares que não possuem conta na plataforma, mas tomaram valores emprestados ou dividiram compras esporádicas.
* **Necessidades:** N/A (representado como um contato/entidade no perfil do Organizador).

---

## 3. Requisitos Funcionais e Regras de Negócio (MVP)

### RF-01: Ingestão e Categorização com IA
* O sistema deve permitir o upload de arquivos de extrato bancário (CSV e OFX).
* O usuário deve receber feedback visual de que o arquivo foi processado e está sendo categorizado.
* O usuário pode registrar manualmente uma transação caso não haja extrato ou o arquivo não seja suportado.
* O **Motor de IA** deve categorizar automaticamente a transação bruta (ex: *"Ifood"* -> *Alimentação*).
* A categorização deve ser **assíncrona**, permitindo que o usuário continue usando a plataforma enquanto o processamento ocorre.
* Caso não seja possível categorizar uma transação, o sistema deve permitir que o usuário faça a categorização manualmente.
* A ingestão e categorização devem focar exclusivamente no **passado/realizado** (transações já efetivadas ou faturas fechadas/atuais).

### RF-02: Divisão Manual de Despesas (Split)
* O usuário pode selecionar qualquer transação importada e realizar um **Split (divisão)**.
* O Split permite atribuir o valor integral ou frações da transação para um ou mais terceiros (cadastrados ou contatos).
* A parcela atribuída ao terceiro é removida da métrica de despesa pessoal do titular e adicionada ao saldo devedor do terceiro.

### RF-03: Matriz de Saldos Devedores (Ledger de Reembolsos)
* O sistema deve manter um livro-razão (ledger) individualizado por pessoa/contato.
* Cada item devido deve manter o vínculo com a transação de origem (data, estabelecimento, valor original e valor alocado).
* Não há projeção de estimativas futuras no MVP; o saldo reflete estritamente o acumulado das transações já divididas.

### RF-04: Gestão Híbrida de Terceiros e Cobrança P2P
* **Terceiro Não Cadastrado:** O Organizador pode criar um contato (apenas nome/identificador) e vincular dívidas a ele.
* **Usuário Cadastrado:** Se o terceiro possuir conta no Finpy, o Organizador pode enviar a solicitação de cobrança do item.
* Ao receber a cobrança, o usuário destinatário pode aceitar/registrar o item em seu próprio painel para controle de seu passivo.

### RF-05: Liquidação e Baixa Manual (Settlement)
* O acerto de contas é realizado **manualmente** pelo usuário.
* Ao receber o reembolso (PIX/transferência), o Organizador acessa o perfil/ledger do terceiro e registra a **Baixa** (parcial ou total).
* O histórico de baixas deve ser preservado para auditoria de pagamentos passados.

---

## 4. Fluxo de Valor Principal (End-to-End)

```
[1. Ingestão]               [2. Categorização IA]         [3. Divisão / Split]
Upload de CSV/OFX  ----->   IA categoriza o gasto  -----> User seleciona gasto
do extrato bancário         bruto (ex: Mercado R$104)     e aloca R$52 para "Leticia"
                                                                  |
                                                                  v
[5. Baixa / Quitação]       [4b. Notificação P2P]         [4a. Atualização Ledger]
User clica em "Registrar    Se Leticia tem conta,        O saldo devedor de Leticia
Baixa" ao receber PIX   <-- recebe cobrança e aprova <-- aumenta R$52 com item discriminado
```

### Etapas do Fluxo:
1. **Importação:** O Organizador carrega o extrato do cartão/conta no Finpy.
2. **Processamento Assíncrono:** O Motor de Ingestão e o Worker de IA limpam e categorizam as transações brutas.
3. **Triagem e Split:** Na interface, o Organizador revisa as compras e clica nas que pertencem a terceiros ou foram compartilhadas, definindo o valor/porcentagem de cada devedor.
4. **Registro no Ledger:** O sistema atualiza instantaneamente o saldo devedor daquele contato, listando a nova dívida itemizada. Caso o devedor seja usuário registrado, uma notificação/cobrança P2P pode ser enviada.
5. **Encerramento / Baixa:** Quando o devedor realiza o pagamento externo (PIX), o Organizador registra a baixa no ledger do contato, liquidando o saldo pendente.
