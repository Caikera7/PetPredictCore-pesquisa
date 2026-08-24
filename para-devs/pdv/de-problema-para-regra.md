# 🧾 PDV: De Problema para Regra

Este documento reúne problemas reais observados na operação de ponto de venda (PDV) de petshops, suas causas raiz, as regras de negócio derivadas e como cada uma se traduz em algo implementável.

**Como ler cada caso:** as duas primeiras seções (Problema e Causa raiz) explicam *o que* acontece e *por quê*. As seções seguintes (Regra, Requisitos, Critérios de aceite, Pontos de atenção) são o material de trabalho para implementação — dá pra ir direto nelas se você já entende o contexto do problema.

---

## 1. Venda de produto sem estoque real disponível

### 🔍 Problema real observado
O sistema (ou a percepção da equipe) indica que um produto está disponível para venda, mas na prática ele não está mais na loja — ou o inverso: o produto existe fisicamente, mas consta como indisponível. Isso gera venda cancelada na hora da entrega, cliente insatisfeito, ou produto parado sem ser vendido por engano.

### 🧩 Causas raiz identificadas

**a) Contagem física falha ou pouco frequente**
Sem uma rotina recorrente de contagem, ninguém sabe com certeza se o número no sistema ainda reflete o que está na prateleira. O erro só é descoberto quando já causou um problema.

**b) Falha no processo de saída manual**
Saída de produto esquecida, ou lançada duas vezes, por falta de um controle padronizado e falta de acompanhamento da gestão sobre esse processo.

**c) Divergência por avaria não formalizada**
Produtos vencidos, danificados ou rasgados saem do estoque "vendável" na prática, mas continuam contando como disponíveis no sistema porque não existe um processo formal de baixa por avaria.

**d) Canais de venda externos desincronizados (ex: iFood)**
Quando a loja vende por mais de um canal, falta tanto a disciplina de lançar a baixa manualmente quanto uma integração real entre os sistemas. Mesmo quando existe um processo manual definido, ele falha por depender inteiramente da disciplina humana, sem nenhuma rede de segurança.

### 📏 Regra de negócio derivada
O estoque exibido para decisão de venda nunca deve depender de um único evento manual não verificado. Toda saída de produto por venda, avaria ou canal externo precisa ser registrada no momento em que acontece, e o sistema deve permitir auditar e comparar o estoque teórico com o estoque físico com frequência definida.

### ⚙️ Requisitos de sistema
- O sistema deve permitir contagens de estoque parciais e frequentes, com registro de divergência quando o número contado for diferente do esperado.
- Toda venda, independentemente do canal, deve gerar uma baixa de estoque no mesmo fluxo — sem depender de um segundo lançamento manual separado.
- Deve existir um tipo de baixa específico para "avaria/perda", separado da baixa por venda, registrando a causa da divergência.
- O sistema deve sinalizar produtos com alta frequência de divergência entre estoque teórico e físico.

### ✅ Critérios de aceite
- **Dado** que uma venda é concluída em qualquer canal, **quando** o pagamento é confirmado, **então** o sistema deve gerar automaticamente a baixa de estoque correspondente, sem exigir lançamento manual adicional.
- **Dado** que um produto é identificado como vencido ou danificado, **quando** o funcionário registra essa condição, **então** o sistema deve dar baixa no estoque com o motivo "avaria" separado do motivo "venda".
- **Dado** que uma contagem física é registrada, **quando** o valor contado difere do valor teórico do sistema, **então** o sistema deve criar um registro de divergência vinculado ao produto e à data da contagem — não apenas sobrescrever o número.
- **Dado** que um produto acumula divergências acima de um limite configurável em um período, **quando** esse limite é atingido, **então** o sistema deve sinalizar o produto para revisão da gestão.

### ⚠️ Pontos de atenção na implementação
- Cuidado com **condição de corrida**: se dois canais (loja física e iFood, por exemplo) tentarem dar baixa no mesmo produto quase ao mesmo tempo, a baixa de estoque precisa ser atômica para não permitir estoque negativo.
- O registro de divergência deve guardar *histórico*, não sobrescrever — perder o histórico de contagens anteriores impede identificar padrões (ex: sempre falta o mesmo produto).
- Definir claramente se uma baixa por avaria exige aprovação de um nível superior (relaciona-se com o caso 2, de autorização) ou se qualquer funcionário pode registrar.

---

## 2. Desconto aplicado sem autorização

### 🔍 Problema real observado
Descontos são aplicados no PDV por funcionários sem a autorização que deveria ser exigida, sem limite de valor respeitado, e muitas vezes sem relação com uma causa legítima. Isso só é percebido no fechamento de caixa, na forma de quebra de caixa.

### 🧩 Causas raiz identificadas

O problema não é ausência de regra — é regra existente que não está sendo aplicada na operação real.

**a) Controle de autorização não ativado na prática**
Mesmo o sistema prevendo a exigência de senha para desconto de funcionário, essa trava não estava em uso na loja.

**b) Ausência de limite de valor no sistema**
Não existe um teto de desconto configurado. A referência de "até 50%" é informal, e deveria valer apenas para causas específicas.

**c) Motivo do desconto não é diferenciado**
Desconto por falta de troco, por cortesia e por causa legítima de produto parecem ser lançados da mesma forma, sem campo que diferencie a causa.

**d) Detecção tardia**
A consequência só aparece no fechamento — não existe alerta no momento em que o desconto é aplicado fora do padrão esperado.

### 📏 Regra de negócio derivada
Autorização para desconto precisa ser imposta pelo sistema no momento da ação, não verificada depois. Todo desconto deve ter uma causa identificável e limites diferentes dependendo dessa causa.

### ⚙️ Requisitos de sistema
- A exigência de autorização para desconto de funcionário deve ser uma trava do sistema, não uma configuração opcional sem visibilidade da gestão.
- Todo desconto deve exigir a seleção de um motivo — valor livre sem causa associada não deveria ser permitido.
- O sistema deve permitir configurar limites de desconto diferentes por motivo.
- Descontos fora do padrão esperado devem gerar um registro sinalizado para a gestão revisar.
- O fechamento de caixa deve exibir o total de descontos segmentado por motivo.

### ✅ Critérios de aceite
- **Dado** que um caixa está aberto no perfil de funcionário, **quando** um desconto é solicitado, **então** o sistema deve exigir autorização de um perfil com permissão de gerente antes de aplicar o desconto.
- **Dado** que um desconto está sendo aplicado, **quando** o funcionário confirma o valor, **então** o sistema deve obrigar a seleção de um motivo antes de permitir a conclusão da venda.
- **Dado** que um motivo de desconto tem um limite configurado (ex: avaria = até 50%), **quando** o valor solicitado ultrapassa esse limite, **então** o sistema deve bloquear a aplicação ou exigir uma segunda autorização de nível superior.
- **Dado** que um desconto foi aplicado sem motivo válido ou acima do limite (caso a trava falhe ou seja contornada), **quando** o caixa é fechado, **então** esse desconto deve aparecer destacado num relatório de exceções, não apenas somado ao total.

### ⚠️ Pontos de atenção na implementação
- A trava de autorização não pode depender só de front-end (esconder o botão) — precisa ser validada no back-end, senão vira segurança de fachada.
- Definir se a autorização é por senha simples, por login separado do gerente, ou por aprovação remota (alguém aprova de outro dispositivo) — isso muda bastante a modelagem.
- O campo de "motivo do desconto" deve ser um catálogo fechado (enum/tabela), não texto livre, para permitir agregação confiável no relatório de fechamento.

---

## 3. Caixa fechando com diferença de valor

### 🔍 Problema real observado
No fechamento de caixa, o valor conferido não bate com o que o sistema esperava ter recebido — gerando falta ou sobra de valor e retrabalho para identificar a origem, muitas vezes sem sucesso.

### 🧩 Causas raiz identificadas

**a) TEF (débito e crédito à vista) — falha na emissão do comprovante**
A filipeta às vezes não é impressa, ou é impressa mas não indica claramente débito ou crédito.

**b) Crédito parcelado — depende inteiramente de lançamento manual**
Não gera comprovante automático — precisa ser lançado manualmente, sem verificação cruzada.

**c) PIX — dois fluxos diferentes, com riscos diferentes**
PIX via maquininha gera um código (CV) de identificação. PIX via link não gera TEF nem CV — depende de três ações manuais separadas (lançar a venda, emitir Nota Fiscal, anotar no caixa).

**d) iFood/delivery — sem integração com o sistema**
Lançado manualmente como "crediário", sem nenhuma forma de o sistema flagar uma venda não lançada.

**e) Ausência de verificação cruzada automática**
Cada categoria é somada manualmente e só depois comparada com o valor esperado — erros só aparecem no fechamento.

**Observação sobre trocas:** trocas de produto aparecem no fechamento como registro informativo, sem entrar na soma que precisa bater com o valor financeiro.

### 📏 Regra de negócio derivada
Toda venda precisa sair do PDV já com a forma de pagamento claramente definida — sem depender de etapas frágeis, como a impressão de um comprovante que pode falhar, nem de um lançamento manual feito à parte, sem conferência depois. Cada forma de pagamento precisa deixar um registro claro no sistema, com informação suficiente para ser conferido depois — mesmo sem integração automática com o meio de pagamento.

### ⚙️ Requisitos de sistema
- A impressão de comprovante não deveria ser descartável no fluxo da venda, ou o sistema deve manter um registro digital equivalente mesmo sem impressão física.
- O comprovante de TEF deve sempre indicar débito ou crédito claramente.
- Toda venda lançada manualmente deve exigir a seleção obrigatória da forma de pagamento e, quando aplicável, um identificador (como o código CV).
- Vendas de PIX via link devem ter um fluxo guiado único que amarre lançamento, emissão de Nota Fiscal e marcação na contagem.
- Vendas do iFood devem ter categoria própria estruturada, não misturada genericamente com "crediário".
- O fechamento de caixa deve exibir o total esperado por categoria de pagamento lado a lado com o valor conferido.
- O total de trocas deve aparecer separado, sem ser somado ao valor financeiro que precisa bater.

### ✅ Critérios de aceite
- **Dado** que uma venda é processada via TEF, **quando** a transação é aprovada pela maquininha, **então** o sistema deve registrar débito ou crédito de forma explícita, independentemente de o comprovante físico ter sido impresso.
- **Dado** que uma venda é lançada manualmente (parcelado, PIX via link ou iFood), **quando** o funcionário tenta concluir o lançamento, **então** o sistema deve exigir a seleção da forma de pagamento e, se aplicável, o código identificador (CV) antes de permitir salvar.
- **Dado** que uma venda por PIX via link é iniciada, **quando** o funcionário confirma o pagamento, **então** o sistema deve conduzir em um único fluxo o lançamento da venda, a emissão da Nota Fiscal e o registro da forma de pagamento para a contagem do caixa.
- **Dado** que o caixa está sendo fechado, **quando** o relatório é gerado, **então** o sistema deve exibir o valor esperado e o valor conferido separados por categoria (TEF débito, TEF crédito, parcelado, PIX maquininha, PIX link, iFood), e não apenas um total único.
- **Dado** que existem trocas registradas no dia, **quando** o fechamento é gerado, **então** o total de trocas deve aparecer em seção separada, sem impactar a soma que precisa bater com o valor financeiro do caixa.

### ⚠️ Pontos de atenção na implementação
- Se houver integração futura direta com a maquininha (captura eletrônica da transação, não dependente de leitura de comprovante impresso), isso elimina de vez a causa "b" e "TEF" — vale considerar isso como evolução natural após a primeira versão manual.
- Ao desenhar a categoria "iFood" separada de "crediário", pensar já na estrutura de dados de forma que uma futura integração automática com o iFood não exija remodelar tudo — deixar o campo de "canal de venda" como conceito genérico, não amarrado a um método de pagamento específico.
- O identificador CV deve ser único por transação para evitar duplicidade de contagem se o mesmo comprovante for lançado duas vezes por engano.
