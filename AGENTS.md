# AGENTS.md

## Visao do projeto

Finpy e um sistema de controle financeiro pessoal voltado a separar, categorizar e rastrear transacoes feitas para terceiros. A especificacao define uma arquitetura futura com tres servicos Python:

- Admin e CRM: Django, Django REST Framework e PostgreSQL.
- Motor de ingestao: FastAPI, SQLAlchemy e PostgreSQL.
- Worker de IA: Celery ou consumidor customizado, integracao com LLM e mensageria.

A arquitetura descrita ainda e planejamento. Nao presuma que servicos, filas, bancos ou pipelines ja existam no repositorio.

## Documentacao de referencia

- [Especificacao do produto e arquitetura](docs/spec.md): publico-alvo, responsabilidades dos servicos, fluxo de dados, observabilidade e qualidade.
- [Plano de execucao](docs/steps.md): fases de descoberta, especificacao, desenvolvimento e deploy.
- [Instrucao de seguranca financeira](.github/instructions/financial-security.instructions.md): regras obrigatorias para valores monetarios e tipagem em Python.

Consulte a documentacao relevante antes de implementar uma etapa e atualize-a quando uma decisao de arquitetura ou contrato mudar.

## Estado atual e comandos

- O repositorio ainda nao possui codigo de aplicacao, testes, configuracao de lint, pipeline CI/CD ou comando oficial de build.
- O ambiente local inclui `venv/`; nao versionar esse diretorio nem arquivos `.env`.
- Antes de introduzir um comando de execucao, teste ou lint, documente-o no README e mantenha-o reproduzivel para o ambiente local e para CI.
- Nao invente servicos ou dependencias para validar uma mudanca enquanto a etapa correspondente da arquitetura nao tiver sido implementada.

## Fluxo de trabalho para agentes

- Comece pela especificacao e identifique o servico, contrato ou regra de negocio afetado.
- Preserve as fronteiras entre Admin/CRM, ingestao e Worker de IA; evite acoplamento direto entre responsabilidades.
- Para mudancas de API, dados ou mensageria, defina ou atualize primeiro o contrato e cubra o comportamento com testes.
- Use alteracoes pequenas e valide com o teste, lint ou type checker especifico assim que esses comandos existirem.
- Nao altere documentacao ou configuracao sem necessidade para a tarefa atual.

## Regras financeiras

Toda mudanca Python que lide com dinheiro ou transacoes deve seguir [a instrucao de seguranca financeira](.github/instructions/financial-security.instructions.md). Em particular, nunca use `float` para valores monetarios, use `Decimal` com escala definida ou inteiros em centavos, e declare type hints em todos os parametros e retornos de funcoes.
