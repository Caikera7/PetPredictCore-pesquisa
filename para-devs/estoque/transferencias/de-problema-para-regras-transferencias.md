# 🔄 Estoque: Transferências entre lojas — De Problema para Regra

Este documento é dedicado especificamente a problemas de **transferência de estoque entre lojas**, separado do documento geral de estoque pela complexidade própria do processo (envolve autorização, emissão fiscal e custo por operação).

**Como ler cada caso:** as duas primeiras seções (Problema e Causa raiz) explicam *o que* acontece e *por quê*. As seções seguintes (Regra, Requisitos, Critérios de aceite, Pontos de atenção) são o material de trabalho para implementação. O objetivo aqui é apontar caminhos de solução — não entregar a implementação pronta.

---

## 1. Transferências reativas, sem dado, em alto volume

### 🔍 Problema real observado
Transferências de produto entre lojas acontecem de forma reativa: uma loja percebe que está sem um produto, verifica se outra loja tem, e só então inicia o processo de transferência. Isso já foi registrado gerando mais de 20 transferências separadas chegando em uma única loja no mesmo dia. Cada transferência tem um custo fiscal (imposto pago por operação), então repetir esse processo várias vezes por semana, sem necessidade, é dinheiro saindo da operação de forma evitável.

### 🧩 Causas raiz identificadas

**a) Decisão de transferência é reativa, não planejada**
A transferência só é cogitada depois que o produto já está em falta na loja — não existe uma análise prévia que antecipe essa necessidade, o que resulta em pedidos avulsos e repetidos em vez de uma transferência única e bem calculada.

**b) Processo depende de uma cadeia manual de pessoas**
O fluxo até hoje segue: loja verifica estoque de outra loja → pede autorização de um superior → aciona o RH para emitir a Nota Fiscal de transferência. Cada etapa dessa cadeia depende de uma pessoa lembrar de fazer sua parte.

**c) Falhas de comunicação entre as etapas**
O RH pode esquecer de emitir a NF mesmo depois de autorizado. A loja pode esquecer de avisar o RH depois de conseguir a autorização. Como não existe um sistema amarrando essas etapas, qualquer uma delas pode simplesmente não acontecer, e ninguém percebe até o produto não chegar.

**d) Ausência de visão consolidada de necessidade**
Não existe um jeito de ver, de uma vez, quais produtos estão com estoque baixo e alta saída ao mesmo tempo — o que permitiria agrupar necessidades e fazer transferências mais estratégicas, em vez de uma por vez conforme a falta aparece.

### 📏 Regra de negócio derivada
A decisão de transferir estoque entre lojas não deve nascer da falta já acontecida — o sistema deve ajudar a antecipar a necessidade, cruzando estoque baixo com produtos de alta saída, para que a transferência seja pensada com antecedência e agrupada, reduzindo o número de operações separadas (e, consequentemente, o custo fiscal repetido). Além disso, o processo de autorização e emissão não pode depender só da memória das pessoas envolvidas — o sistema deve acompanhar cada etapa até ela ser concluída.

### ⚙️ Requisitos de sistema
- O sistema deve ter um indicador que cruze estoque baixo com alta frequência de saída por produto, sinalizando candidatos a transferência antes da falta acontecer.
- Deve ser possível visualizar, de forma consolidada, quais produtos precisam de reposição entre lojas — permitindo agrupar vários produtos em uma transferência só, em vez de uma transferência por produto.
- O processo de transferência (pedido → autorização → emissão de NF) deve ter status visível e rastreável, para que fique claro em qual etapa cada transferência está parada.
- Se uma transferência ficar parada em alguma etapa por tempo além do esperado, o sistema deve alertar os responsáveis envolvidos — em vez de depender de alguém perceber por conta própria.

### ✅ Critérios de aceite
- **Dado** que um produto está com estoque baixo e alta saída recente em uma loja, **quando** o painel de transferências é consultado, **então** esse produto deve aparecer como candidato sugerido para transferência, mesmo antes de estar em falta.
- **Dado** que uma transferência é solicitada e autorizada, **quando** a autorização é confirmada, **então** o sistema deve notificar automaticamente o setor responsável pela emissão da Nota Fiscal — sem depender de aviso manual da loja.
- **Dado** que uma transferência autorizada não teve a Nota Fiscal emitida dentro de um prazo configurável, **quando** esse prazo é ultrapassado, **então** o sistema deve gerar um alerta para o responsável, sinalizando a etapa pendente.
- **Dado** que existem múltiplos produtos precisando de reposição na mesma loja de origem e destino, **quando** o usuário for criar uma transferência, **então** o sistema deve permitir agrupar todos esses produtos em uma única operação de transferência.

### ⚠️ Pontos de atenção na implementação
- O agrupamento de produtos em uma única transferência precisa considerar que cada transferência gera custo fiscal — vale desenhar o modelo pensando nisso como um critério de decisão (ex: sugerir esperar acumular mais itens antes de disparar uma nova transferência, dentro de um limite de tempo razoável, sem deixar a loja em falta).
- O rastreamento de status (pedido → autorização → NF emitida → recebido) deve deixar claro *quem* é o responsável por cada etapa, para que o alerta de atraso vá para a pessoa certa, não para todos genericamente.
- Vale considerar, no desenho do fluxo, que múltiplas lojas podem competir pelo mesmo produto ao mesmo tempo — o sistema precisa lidar com o caso de duas lojas solicitando o mesmo item da mesma origem simultaneamente.
- Esse indicador de "estoque baixo + alta saída" é o mesmo tipo de dado usado no caso de produto parado sem giro (documentado no módulo geral de estoque) — faz sentido que os dois compartilhem a mesma base de cálculo de giro, em vez de serem implementados como métricas separadas.
