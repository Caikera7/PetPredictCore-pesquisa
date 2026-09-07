# 💰 Financeiro: De Problema para Regra

Este documento reúne problemas reais observados na gestão financeira de petshops, suas causas raiz, as regras de negócio derivadas e como cada uma se traduz em algo implementável.

**Como ler cada caso:** as duas primeiras seções (Problema e Causa raiz) explicam *o que* acontece e *por quê*. As seções seguintes (Regra, Requisitos, Critérios de aceite, Pontos de atenção) são o material de trabalho para implementação.

---

## 1. Contas a pagar sem controle de vencimento

### 🔍 Problema real observado
Pagamentos a fornecedores não têm um controle claro de vencimento, resultando em atrasos. Isso gera consequências que vão além do simples atraso: juros e multas desnecessários, desgaste na relação com o fornecedor, e em casos mais graves, risco de o fornecedor suspender novos pedidos até a situação ser regularizada — o que, por sua vez, alimenta os problemas de estoque já documentados (produto em falta por falta de reposição).

### 🧩 Causas raiz identificadas

**a) Ausência de registro centralizado de obrigações**
Sem um lugar único que reúna todas as contas a pagar com suas datas de vencimento, o controle depende de memória, papel avulso ou verificação manual de boletos/notas fiscais uma por uma.

**b) Sem alerta antecipado de vencimento**
Mesmo quando existe algum registro, não há um aviso automático chegando perto da data — o vencimento só é percebido quando já está em cima da hora, ou depois de já ter passado.

**c) Falta de visão consolidada do fluxo de saída de dinheiro**
Sem saber quanto e quando precisa pagar nos próximos dias/semanas, fica difícil planejar se há caixa suficiente disponível para cobrir todos os compromissos — decisões de compra (já discutidas no módulo de estoque) podem ser tomadas sem considerar compromissos financeiros já assumidos.

**d) Consequência em cadeia com outros módulos**
Atraso recorrente com um fornecedor pode levar a esse fornecedor exigir pagamento antecipado ou suspender entregas — o que reconecta diretamente ao problema de estoque parado/em falta já documentado, mostrando que esses módulos não são isolados na prática do negócio.

### 📏 Regra de negócio derivada
Toda obrigação de pagamento (a fornecedores, prestadores de serviço, etc.) deve ser registrada em um único lugar no sistema, com data de vencimento clara, e deve gerar aviso automático com antecedência suficiente para que o pagamento seja programado — sem depender de verificação manual ou memória de quem gerencia o financeiro.

### ⚙️ Requisitos de sistema
- O sistema deve permitir cadastrar contas a pagar com fornecedor, valor, data de vencimento e status (pendente, pago, atrasado).
- Deve existir um alerta configurável (ex: 3 dias antes do vencimento) notificando o responsável financeiro.
- O sistema deve exibir uma visão consolidada de todas as contas a pagar dos próximos dias/semanas, permitindo enxergar o compromisso financeiro total antes de decisões de compra.
- Contas vencidas e não pagas devem ser sinalizadas de forma destacada, diferenciando-as visualmente das que ainda estão dentro do prazo.
- O sistema deve manter histórico de pagamentos por fornecedor, permitindo identificar se atrasos são recorrentes com um fornecedor específico.

### ✅ Critérios de aceite
- **Dado** que uma conta a pagar é cadastrada com uma data de vencimento, **quando** essa data se aproxima dentro do prazo de alerta configurado, **então** o sistema deve notificar o responsável financeiro automaticamente.
- **Dado** que uma conta a pagar passa da data de vencimento sem ser marcada como paga, **quando** essa condição é detectada, **então** o sistema deve sinalizar essa conta como atrasada, destacando-a nas listagens.
- **Dado** que o usuário quer planejar decisões de compra, **quando** ele consulta o painel financeiro, **então** o sistema deve mostrar o total de compromissos de pagamento previstos para os próximos dias/semanas, junto com o saldo disponível, se essa informação existir no sistema.
- **Dado** que um fornecedor tem múltiplos atrasos registrados em um período, **quando** o histórico desse fornecedor é consultado, **então** esses atrasos devem aparecer destacados, permitindo à gestão identificar um padrão de risco.

### ⚠️ Pontos de atenção na implementação
- O alerta de vencimento não deve depender só de uma notificação dentro do próprio sistema — vale considerar reforço por outro canal (e-mail, WhatsApp) para garantir que o aviso realmente chegue a quem precisa agir, mesmo que essa pessoa não esteja logada no sistema naquele momento.
- É importante distinguir contas a pagar recorrentes (ex: aluguel, assinatura de serviço) de contas pontuais (compra avulsa de mercadoria) — o modelo de dados deve suportar lançamento recorrente automático, sem exigir recadastro manual todo mês.
- O cruzamento entre "compromissos de pagamento futuros" e "decisão de compra de estoque" (mencionado no requisito de visão consolidada) é uma conexão importante entre os módulos financeiro e estoque — vale pensar nessa integração desde o início do desenho de dados, em vez de tratar os dois como sistemas totalmente separados.
- Ao sinalizar contas atrasadas, considerar também o impacto reputacional: seria útil um campo de observação vinculado ao fornecedor (ex: "fornecedor passou a exigir pagamento antecipado após atrasos"), documentando consequências que vão além do valor financeiro em si.

---

## 2. Ausência de reserva para imprevistos

### 🔍 Problema real observado
Quando um imprevisto acontece — equipamento quebrando, por exemplo — a loja não tem dinheiro reservado especificamente para isso. O resultado é sempre a mesma escolha forçada: optar pela solução mais barata e simples possível no momento, mesmo sabendo que não é a ideal, o que frequentemente gera o mesmo problema se repetindo pouco tempo depois — e, no fim, mais dinheiro é gasto do total (conserto barato + conserto de novo + conserto de novo) do que se tivesse sido feita uma solução mais durável desde o início.

### 🧩 Causas raiz identificadas

**a) Ausência de uma categoria de reserva financeira**
Todo o dinheiro disponível é tratado como uma única "massa" de caixa, sem uma parte reservada especificamente para emergências. Isso significa que qualquer imprevisto compete diretamente com outros compromissos já programados (fornecedores, contas fixas).

**b) Decisão reativa em vez de preventiva**
Sem reserva, a decisão sobre o que fazer diante do imprevisto é tomada sob pressão, no momento em que o problema já aconteceu — o que naturalmente empurra para a opção mais barata e imediata, não para a mais vantajosa a longo prazo.

**c) Ciclo de gasto repetido não visível como um todo**
Como cada conserto é tratado como um gasto pontual e isolado, ninguém costuma somar quanto, no total, já foi gasto tentando resolver o mesmo problema de forma barata repetidas vezes — se essa soma fosse visível, ficaria mais claro que uma solução definitiva, mesmo mais cara à vista, teria sido mais barata no total.

> Este caso se conecta diretamente com o caso 1 (contas a pagar sem controle de vencimento): sem visão consolidada de compromissos futuros, fica ainda mais difícil saber se existe "sobra" de caixa que poderia formar uma reserva.

### 📏 Regra de negócio derivada
O sistema deve permitir tratar reserva de emergência como uma categoria financeira própria, separada do caixa de operação do dia a dia, e deve ajudar a visualizar o custo acumulado de problemas recorrentes — para que a decisão entre "resolver barato de novo" e "resolver definitivo" seja baseada em dado, não em urgência do momento.

### ⚙️ Requisitos de sistema
- O sistema deve permitir configurar uma categoria de "reserva para imprevistos", separada visualmente e financeiramente do saldo de operação normal.
- Deve ser possível registrar contribuições periódicas para essa reserva (ex: um percentual do faturamento mensal), mesmo que o valor inicial seja pequeno.
- Gastos com manutenção/reparo devem poder ser categorizados e vinculados a um mesmo "ativo" (ex: todos os gastos relacionados à mesma máquina de tosa), permitindo somar o total gasto com aquele item ao longo do tempo.
- O sistema deve exibir esse total acumulado por ativo, tornando visível quando o padrão de "conserto barato repetido" já ultrapassou o que custaria uma solução definitiva.

### ✅ Critérios de aceite
- **Dado** que existe uma categoria de reserva para imprevistos configurada, **quando** o usuário consulta o painel financeiro, **então** esse valor deve aparecer separado do saldo de operação do dia a dia, não misturado em um único número.
- **Dado** que um gasto de manutenção é registrado e vinculado a um ativo específico (ex: "máquina de tosa 1"), **quando** um novo gasto de manutenção do mesmo ativo é registrado posteriormente, **então** o sistema deve somar ao histórico já existente daquele ativo.
- **Dado** que o total acumulado de manutenção de um ativo ultrapassa um valor de referência (configurável, ex: o custo estimado de substituição), **quando** essa condição é detectada, **então** o sistema deve alertar o usuário sobre esse padrão.

### ⚠️ Pontos de atenção na implementação
- O valor de referência para "ultrapassou o que valeria a pena trocar" pode ser difícil de estimar automaticamente sem dado de mercado — uma abordagem inicial mais simples é permitir que o próprio usuário cadastre esse valor de referência manualmente por tipo de ativo, evoluindo depois para uma sugestão mais automática.
- Reserva de emergência não deve ser apenas um número "de gaveta" — vale considerar se esse valor precisa estar de fato segregado (ex: referenciando uma conta bancária separada) ou se é só uma categoria informativa dentro do sistema; isso muda a complexidade da implementação.
- Esse conceito de "gasto acumulado por ativo" pode ser reaproveitado further para outros contextos do negócio (ex: custo total de manutenção de um veículo de delivery, se esse serviço existir) — vale desenhar como um conceito genérico de "ativo com histórico de custo", não amarrado só a equipamento de banho e tosa.
