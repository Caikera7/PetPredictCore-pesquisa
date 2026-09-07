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
