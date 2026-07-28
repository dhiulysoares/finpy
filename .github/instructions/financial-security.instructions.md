---
name: "Seguranca Financeira e Tipagem"
description: "Use when writing or reviewing Python code, database models, migrations, APIs, or tests that handle monetary values or financial transactions. Enforces exact monetary arithmetic and complete type hints."
applyTo: "**/*.py"
---
# Seguranca Financeira e Tipagem

Este projeto lida com dados financeiros. Erros de arredondamento ou perda de precisao sao inaceitaveis.

## Valores monetarios

- Nunca use `float` para valores monetarios, nem no dominio, nem em calculos intermediarios, serializacao, persistencia ou testes.
- Prefira `Decimal` da biblioteca padrao do Python, com precisao e escala definidas, ou inteiros que representem a menor unidade monetaria (por exemplo, centavos).
- Ao usar `Decimal`, defina explicitamente a escala e quantize os resultados conforme a regra de negocio. Nao dependa da conversao implicita ou do contexto global sem configuracao documentada.
- Ao persistir em banco relacional, use um tipo decimal/numerico com `precision` e `scale` explicitas, ou um campo inteiro para centavos. Mantenha a mesma convencao em modelos, schemas, serializers e migrations.
- Nunca construa um `Decimal` monetario a partir de um `float`; leia valores de entradas textuais ou de inteiros.
- Defina e teste as regras de arredondamento. Nao use `round()` como substituto de uma politica financeira explicita.

Exemplo preferido:

```python
from decimal import Decimal

CENT = Decimal("0.01")
amount = Decimal("10.25").quantize(CENT)
```

## Tipagem obrigatoria

- Toda funcao e metodo deve declarar type hints para todos os parametros, incluindo o retorno com `-> None` quando aplicavel.
- Use tipos precisos para valores monetarios, como `Decimal` ou um alias de dominio para centavos; nao use `Any` para contornar a tipagem de valores financeiros.
- Atualize os type hints quando alterar contratos de servicos, schemas, modelos ou funcoes de calculo.

## Verificacao

- Para cada nova operacao monetaria, adicione ou atualize testes que cubram precisao, escala, arredondamento e limites relevantes.
- Revise conversoes de entrada e saida para garantir que nenhum `float` seja introduzido no caminho.
- Antes de concluir uma alteracao financeira, execute os testes focados e a verificacao de tipos/lint disponivel no projeto.
