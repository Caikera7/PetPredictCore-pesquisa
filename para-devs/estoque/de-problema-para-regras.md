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

---

## 2. Contagem de estoque sem processo estruturado

### 🔍 Problema real observado
A contagem de estoque acontece sem frequência definida — é "solta no escuro", sem um cronograma. Mesmo existindo o conceito de curva ABC para priorizar quais produtos contar primeiro, a liberação da contagem não é alinhada com o setor de compras, o que gera um erro recorrente: a contagem é iniciada e, no meio do processo, chega mercadoria nova, fazendo o número contado não bater com o que o sistema esperava — não por erro de contagem, mas por falta de sincronismo no momento de contar.

### 🧩 Causas raiz identificadas

**a) Ausência de cronograma de contagem**
Não existe uma rotina mensal (ou outra frequência definida) de contagem. Isso significa que, na prática, cada contagem é um evento isolado, sem previsibilidade.

**b) Curva ABC sem sincronismo com compras**
A ideia de contar por prioridade (curva ABC) existe, mas falta um processo de conversa com quem faz as compras antes de liberar a contagem — o resultado é contagem e chegada de mercadoria acontecendo ao mesmo tempo, o que distorce o resultado sem ser um erro real de estoque.

**c) Processo manual com transcrição em duas etapas**
A contagem é feita em papel (planilha) e só depois passada para o sistema, onde a diferença entre o contado e o registrado é calculada. Essa transcrição é um ponto a mais onde erro humano pode entrar, além do erro de contagem em si.

**d) Investigação só em divergências grandes**
Divergências pequenas não costumam ser investigadas — só quando o valor é muito discrepante é que alguém pergunta o motivo. Isso significa que pequenos erros recorrentes (que, somados, também custam dinheiro) nunca são entendidos nem corrigidos na causa.

**e) Ausência de conferência cruzada**
A contagem é feita por qualquer funcionário, sozinho, sem uma segunda pessoa conferindo o resultado antes de ele virar número oficial no sistema.

> Este módulo exige atenção tanto aqui (estrutura de sistema) quanto no `/para-donos` (processo operacional) — um bom sistema de contagem não resolve sozinho se a rotina de quando e como contar não for definida pela gestão.

### 📏 Regra de negócio derivada
Contagem de estoque não pode ser um evento solto — precisa ter cronograma, precisa ser coordenada com quem compra (pra não contar no meio de uma chegada de mercadoria), e toda divergência, grande ou pequena, precisa ficar registrada com sua causa possível, não só corrigida silenciosamente no número final.

### ⚙️ Requisitos de sistema
- O sistema deve permitir agendar contagens (por produto, categoria ou curva ABC) com data prevista, associando isso a um calendário — não só disparar contagem manualmente sem controle.
- Antes de liberar uma contagem, o sistema deve alertar se existe recebimento de mercadoria previsto ou em andamento para os mesmos produtos, evitando contar no meio da chegada.
- O lançamento de contagem deve poder ser feito direto no sistema (aplicativo ou tela simples), reduzindo a necessidade de transcrição de papel para sistema como etapa separada.
- Toda divergência encontrada — mesmo pequena — deve ser registrada com um campo de motivo (mesmo que "não identificado"), em vez de o número ser simplesmente ajustado sem rastro.
- O sistema deve permitir registrar quem fez a contagem e, quando houver, quem conferiu — abrindo espaço para conferência cruzada mesmo que não seja obrigatória em todos os casos.

### ✅ Critérios de aceite
- **Dado** que uma contagem está agendada para uma data, **quando** essa data chega, **então** o sistema deve notificar os responsáveis pela contagem automaticamente, sem depender de alguém lembrar manualmente.
- **Dado** que existe um recebimento de mercadoria em andamento para um produto, **quando** uma contagem desse mesmo produto é iniciada, **então** o sistema deve alertar sobre o conflito antes de permitir a finalização da contagem.
- **Dado** que uma contagem é registrada com valor diferente do estoque teórico, **quando** o resultado é salvo, **então** o sistema deve exigir a seleção de um motivo (mesmo que "não identificado"), independentemente do tamanho da diferença.
- **Dado** que uma contagem foi realizada por um funcionário, **quando** outro funcionário realiza a conferência do mesmo produto, **então** o sistema deve permitir registrar essa segunda contagem vinculada à primeira, para comparação.

### ⚠️ Pontos de atenção na implementação
- O alerta de "contagem em conflito com recebimento" precisa considerar a janela de tempo real do processo de recebimento (do momento que a nota chega até a mercadoria estar fisicamente conferida), não só a data do pedido.
- Vale pensar se o lançamento direto no sistema (sem papel) vai depender de dispositivo móvel/coletor — isso influencia o desenho de interface e pode exigir uma etapa de transição enquanto a operação se adapta.
- O campo de "motivo da divergência" deve ser um catálogo pré-definido (igual sugerido no módulo de PDV para desconto), permitindo depois analisar quais motivos são mais recorrentes — texto livre dificulta esse tipo de análise.
- Esse processo de contagem estruturada se conecta diretamente com o caso de "produto parado sem giro" e com "transferências" — a mesma base de dado de estoque teórico x físico alimenta os três. Vale desenhar pensando em reaproveitamento, não como módulos isolados.
