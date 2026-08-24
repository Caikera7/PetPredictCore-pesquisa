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

### 🔍 Problema real observado
Descontos são aplicados no PDV por funcionários sem a autorização que deveria ser exigida, sem limite de valor respeitado, e muitas vezes sem relação com uma causa legítima (avaria, vencimento). Isso só é percebido — quando é percebido — no fechamento de caixa, na forma de quebra de caixa.

### 🧩 Causas raiz identificadas

Diferente do problema de estoque, aqui a regra formal já existe: o sistema prevê que caixa aberto como funcionário exija autorização (senha de gerente ou líder) para conceder desconto, enquanto caixa aberto como gerente tem liberdade. O problema não é ausência de regra — é **regra existente que não está sendo aplicada na operação real**.

**a) Controle de autorização não ativado na prática**
Mesmo o sistema prevendo a exigência de senha para desconto de funcionário, na loja essa trava não estava em uso — o desconto saía livre, independente de quem estivesse operando o caixa.

**b) Ausência de limite de valor no sistema**
Não existe um teto de desconto configurado. A referência de "até 50%" é uma prática informal, não uma regra imposta pelo sistema — e mesmo essa referência só deveria valer para causas específicas (vencimento, avaria), não para qualquer motivo.

**c) Motivo do desconto não é diferenciado**
Desconto por falta de troco, desconto para "agradar" o cliente e desconto por causa legítima de produto (avaria, vencimento) parecem ser lançados da mesma forma, sem um campo que diferencie a causa. Isso dificulta até para a própria gestão entender, depois, se o desconto teve uma razão operacional real ou foi decisão informal do momento.

**d) Detecção tardia**
A consequência (quebra de caixa) só aparece no fechamento — não existe um alerta no momento em que o desconto está sendo aplicado fora do padrão esperado.

### 📏 Regra de negócio derivada
Autorização para desconto não pode ser uma regra que existe apenas "no papel" — ela precisa ser **imposta pelo sistema no momento da ação**, não verificada depois. Além disso, todo desconto deve ter uma causa identificável e limites diferentes dependendo dessa causa, para que seja possível distinguir desconto operacional legítimo de desconto por decisão informal.

### ⚙️ Requisitos de sistema
- A exigência de autorização (senha de gerente/líder) para desconto de funcionário deve ser uma trava do sistema, não uma configuração opcional que pode ficar desativada sem visibilidade da gestão.
- Todo desconto deve exigir a seleção de um motivo (ex: avaria, vencimento, troco insuficiente, cortesia) — o valor livre sem causa associada não deveria ser permitido.
- O sistema deve permitir configurar limites de desconto diferentes por motivo (ex: até 50% para avaria, um teto bem menor ou zero para "cortesia"), em vez de um limite único genérico.
- Descontos aplicados fora do padrão esperado (sem motivo, acima do limite, ou por funcionário sem autorização) devem gerar um registro sinalizado para a gestão revisar — não só aparecer disperso no relatório de fechamento de caixa.
- O fechamento de caixa deve exibir o total de descontos segmentado por motivo, para que uma quebra de caixa possa ser investigada por causa, não apenas por valor total.

## 3. Caixa fechando com diferença de valor
*(em construção)*
