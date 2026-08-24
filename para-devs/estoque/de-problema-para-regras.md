# 📦 Estoque: De Problema para Regra

Este documento reúne problemas reais observados na gestão de estoque de petshops, suas causas raiz, as regras de negócio derivadas e como cada uma se traduz em algo implementável.

**Como ler cada caso:** as duas primeiras seções (Problema e Causa raiz) explicam *o que* acontece e *por quê*. As seções seguintes (Regra, Requisitos, Critérios de aceite, Pontos de atenção) são o material de trabalho para implementação.

---

## 1. Estoque parado / produto encalhado sem giro

### 🔍 Problema real observado
Produtos ficam parados em estoque por longos períodos (mais de 60 dias, em casos observados) sem gerar venda, representando dinheiro parado em prateleira. Ao mesmo tempo, produtos que realmente vendem bem (o "principal" da loja) acabam faltando, porque a compra não segue nenhum critério baseado em dado real de venda.

### 🧩 Causas raiz identificadas

**a) Compra sem análise de dado de venda**
As decisões de compra são feitas sem considerar quanto cada produto realmente vende, com que frequência, ou há quanto tempo está girando. O resultado é comprar produto novo em excesso enquanto o produto que sustenta a loja fica em falta.

**b) Ausência de visibilidade sobre estoque parado**
Não existe uma forma de identificar, de forma simples e confiável, quais produtos estão parados há muito tempo. A percepção de "vendeu bem" ou "não vendeu" é feita de forma informal, sem dado que confirme.

**c) Decisão de recompra sem base em porcentagem de venda real**
A recompra de um produto acontece sem saber, de fato, qual a taxa de saída dele — reforçando o ciclo de comprar mais do que já está parado ou menos do que realmente vende.

**d) Consequência sobre a precisão do próprio estoque**
Produto parado por muito tempo também aumenta o risco de erro de contagem e divergência (relacionado aos casos já documentados no módulo de PDV) — quanto mais tempo um produto fica sem movimentação, maior a chance de ele estar fisicamente diferente do que o sistema registra (perda, avaria não percebida, etc).

> Esse problema tem uma camada de decisão de gestão (como e quando comprar) que é tratada no módulo `/para-donos`. Aqui, o foco é a camada de sistema: como dar visibilidade de dado suficiente para que essa decisão pare de ser feita no "achismo".

### 📏 Regra de negócio derivada
O sistema precisa mostrar, de forma clara, há quanto tempo cada produto está sem vender e qual sua taxa real de saída. Decisão de compra não deve depender de impressão ou memória de quem compra — precisa de dado visível e fácil de consultar.

### ⚙️ Requisitos de sistema
- Cada produto deve ter um indicador de tempo desde a última venda registrada.
- O sistema deve permitir listar produtos parados acima de um período configurável (ex: 60 dias sem venda).
- Deve existir um indicador de giro de estoque por produto (quantidade vendida em um período, relacionada à quantidade comprada no mesmo período).
- O sistema deve permitir comparar produtos por taxa de saída, para apoiar decisão de recompra.
- Produtos com estoque parado por tempo excessivo devem gerar um alerta visível para quem faz a gestão de compras.

### ✅ Critérios de aceite
- **Dado** que um produto não tem nenhuma venda registrada dentro do período configurado (ex: 60 dias), **quando** a listagem de estoque é consultada, **então** esse produto deve aparecer destacado como "parado" ou "sem giro".
- **Dado** que um usuário quer decidir uma recompra, **quando** ele consulta um produto específico, **então** o sistema deve mostrar a quantidade vendida nos últimos X dias e a quantidade atualmente em estoque, lado a lado.
- **Dado** que existem produtos parados há mais tempo que o limite configurado, **quando** o painel de gestão é acessado, **então** esses produtos devem aparecer automaticamente em uma lista de atenção, sem precisar de busca manual.
- **Dado** que um produto tem alta taxa de giro (vende rápido e frequentemente), **quando** o estoque desse produto está abaixo de um nível mínimo, **então** o sistema deve alertar para reposição — evitando que o "produto principal" fique em falta.

### ⚠️ Pontos de atenção na implementação
- O período configurável ("parado há X dias") não deve ser fixo no código — cada loja pode ter uma noção diferente do que é "muito tempo parado" dependendo do tipo de produto (ração tem giro diferente de acessório, por exemplo).
- Cuidado ao calcular "giro": produto recém-cadastrado não deve aparecer como "parado" só porque ainda não teve tempo de vender — a métrica precisa considerar a data de entrada do produto no estoque, não só a ausência de venda.
- Esse indicador de giro é a base para o conceito de "estoque preditivo" já previsto no projeto principal — vale desenhar o modelo de dados pensando que essa informação vai alimentar previsão futura, não só o alerta simples de "parado ou não".
