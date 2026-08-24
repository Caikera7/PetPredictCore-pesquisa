# 🧾 PDV: De Problema para Regra

Este documento reúne problemas reais observados na operação de ponto de venda (PDV) de petshops, suas causas raiz e as regras de negócio derivadas para evitá-los.

---

## 1. Venda de produto sem estoque real disponível

### 🔍 Problema real observado
O sistema (ou a percepção da equipe) indica que um produto está disponível para venda, mas na prática ele não está mais na loja — ou o inverso: o produto existe fisicamente, mas consta como indisponível. Isso gera venda cancelada na hora da entrega, cliente insatisfeito, ou produto parado sem ser vendido por engano.

### 🧩 Causas raiz identificadas

Esse problema não tem uma única causa — na prática, ele nasce de falhas em pontos diferentes da operação:

**a) Contagem física falha ou pouco frequente**
Sem uma rotina recorrente de contagem, ninguém sabe com certeza se o número no sistema ainda reflete o que está na prateleira. O erro só é descoberto quando já causou um problema (venda cancelada, cliente na loja sem o produto).

**b) Falha no processo de saída manual**
Saída de produto esquecida, ou lançada duas vezes, por falta de um controle padronizado (ex: planilha sem regra clara) e falta de acompanhamento da gestão sobre esse processo.

**c) Divergência por avaria não formalizada**
Produtos vencidos, danificados ou rasgados saem do estoque "vendável" na prática, mas continuam contando como disponíveis no sistema porque não existe um processo formal de baixa por avaria.

**d) Canais de venda externos desincronizados (ex: iFood)**
Quando a loja vende por mais de um canal (loja física + delivery), pode faltar tanto a disciplina de lançar a baixa manualmente quanto uma integração real entre os sistemas. Na prática, os dois problemas coexistem: mesmo quando existe um processo manual definido, ele falha porque depende inteiramente da disciplina humana, sem nenhuma rede de segurança por trás.

### 📏 Regra de negócio derivada
O estoque exibido para decisão de venda **nunca deve depender de um único evento manual não verificado**. Toda saída de produto — seja por venda, avaria ou canal externo — precisa ser registrada no momento em que acontece, e o sistema deve permitir auditar e comparar o estoque teórico com o estoque físico com frequência definida.

### ⚙️ Requisitos de sistema
- O sistema deve permitir contagens de estoque parciais e frequentes (não só o inventário geral), com registro de divergência quando o número contado for diferente do número esperado.
- Toda venda, independentemente do canal (loja, delivery, app), deve gerar uma baixa de estoque no mesmo fluxo — o sistema não deve depender de um segundo lançamento manual separado para refletir a saída.
- Deve existir um tipo de baixa específico para "avaria/perda" (vencimento, dano), separado da baixa por venda, para que a causa da divergência fique registrada e não apenas o número final.
- O sistema deve sinalizar produtos com alta frequência de divergência entre estoque teórico e físico, para que a gestão identifique padrões (ex: um produto que sempre diverge pode indicar erro de processo recorrente, não coincidência).

---

## 2. Desconto aplicado sem autorização
*(em construção)*

## 3. Caixa fechando com diferença de valor
*(em construção)*
