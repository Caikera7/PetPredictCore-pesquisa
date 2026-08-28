# 🛁 Banho e Tosa: De Problema para Regra

Este documento reúne problemas reais observados na operação de banho e tosa de petshops, suas causas raiz, as regras de negócio derivadas e como cada uma se traduz em algo implementável.

**Como ler cada caso:** as duas primeiras seções (Problema e Causa raiz) explicam *o que* acontece e *por quê*. As seções seguintes (Regra, Requisitos, Critérios de aceite, Pontos de atenção) são o material de trabalho para implementação.

---

## 1. Ocorrências durante o atendimento sem registro formal

### 🔍 Problema real observado
Quando acontece uma ocorrência durante o banho ou tosa um corte na pelagem, um machucado, qualquer intercorrência grave com o pet a loja tem uma resposta operacional (avisar o superior, encaminhar para uma unidade com clínica, tratar o tutor com atenção, cautela e profissionalismo), mas não existe nenhum registro formal do que aconteceu: sem foto, sem ficha de ocorrência, sem histórico salvo em lugar nenhum.

### 🧩 Causas raiz identificadas

**a) Ausência de processo de documentação**
A resposta ao problema é toda baseada em ação imediata (cuidar do pet, cuidar do tutor), mas não existe uma etapa de registrar o que aconteceu — nem em papel, nem em sistema.

**b) Sem histórico do pet acumulado ao longo do tempo**
Sem registro de ocorrências anteriores, a loja não tem como saber se um determinado pet já teve problema antes, se reage mal a um tipo de tosa específica, ou se um padrão está se repetindo com o mesmo profissional.

**c) Ausência de prova em caso de questionamento**
Sem foto ou registro do estado do pet antes e depois do atendimento, a loja fica numa posição frágil se o tutor questionar depois o que realmente aconteceu — não há como comprovar nem para a loja, nem para o tutor.

**d) Sem visibilidade de padrão para a gestão**
Sem dado registrado, a gestão não consegue identificar se ocorrências estão concentradas em um profissional específico, um horário de maior correria, ou um tipo de procedimento — informação que ajudaria a agir na causa, não só no caso isolado.

### 📏 Regra de negócio derivada
Toda ocorrência durante um atendimento de banho e tosa precisa ser registrada formalmente no momento em que acontece — não pode depender apenas da resposta verbal ao tutor. O registro deve incluir o que aconteceu, quando, com qual profissional, e deve ficar vinculado ao histórico daquele pet especificamente.

### ⚙️ Requisitos de sistema
- O sistema deve ter um tipo de registro específico para ocorrências, separado do agendamento comum — algo que possa ser preenchido no momento em que o problema acontece.
- Deve ser possível anexar foto ao registro de ocorrência.
- Toda ocorrência deve ficar vinculada ao histórico do pet, visível em atendimentos futuros — não apenas registrada e esquecida.
- O sistema deve permitir visualizar ocorrências agrupadas por profissional, por tipo de procedimento ou por período, para apoiar a gestão a identificar padrões.
- O tutor deveria poder visualizar (via Portal do Tutor, já previsto no projeto) o registro da ocorrência relacionada ao atendimento do seu pet, reforçando transparência.

### ✅ Critérios de aceite
- **Dado** que uma ocorrência acontece durante um atendimento, **quando** o profissional ou responsável registra o caso, **então** o sistema deve permitir anexar uma descrição, uma ou mais fotos, e vincular automaticamente ao pet e ao atendimento em questão.
- **Dado** que um pet tem uma ocorrência registrada, **quando** um novo agendamento é aberto para esse mesmo pet, **então** o sistema deve exibir um alerta visível mostrando que existe histórico de ocorrência anterior.
- **Dado** que existem múltiplas ocorrências registradas, **quando** a gestão consulta o painel de banho e tosa, **então** deve ser possível filtrar ocorrências por profissional, tipo de procedimento e período.
- **Dado** que uma ocorrência foi registrada e vinculada a um atendimento, **quando** o tutor acessa o histórico do pet no Portal do Tutor, **então** o registro da ocorrência (ou uma versão apropriada dela) deve estar disponível para consulta.

### ⚠️ Pontos de atenção na implementação
- É importante decidir com cuidado o que do registro de ocorrência é visível para o tutor e o que é interno da operação — nem tudo que a equipe registra internamente (ex: observações sobre o comportamento do pet) precisa ou deve aparecer da mesma forma no Portal do Tutor.
- O registro de ocorrência deve ter carimbo de data/hora e autoria (quem registrou) de forma que não possa ser editado silenciosamente depois — isso é o que dá valor de prova ao registro.
- Vale considerar diferentes níveis de gravidade na ocorrência (ex: observação leve x incidente que exigiu encaminhamento para clínica) — tratar tudo como o mesmo tipo de registro pode diluir a atenção que casos mais sérios merecem.
- Esse histórico de ocorrência por pet é um dado valioso não só para segurança, mas também para a experiência do tutor — pode se conectar futuramente com o conceito de repurchase/fidelização já previsto no projeto (Cashback, Portal do Tutor).

---

## 2. Agendamento com horário batendo (conflito de agenda)

### 🔍 Problema real observado
Ocorre com frequência de dois pets ficarem marcados para o mesmo horário com o mesmo profissional, gerando conflito na hora do atendimento — um dos tutores precisa esperar, ser remarcado, ou o atendimento é encaixado à força, afetando a qualidade do serviço.

### 🧩 Causa raiz identificada

O agendamento não considera o tempo real que cada atendimento vai ocupar. Um horário é reservado sem levar em conta variáveis que influenciam diretamente a duração do serviço — como porte do animal, peso e tipo de pelagem — fazendo com que o sistema (ou processo manual) trate todo agendamento como se ocupasse o mesmo tempo padrão, o que não reflete a realidade.

### 📏 Regra de negócio derivada
O tempo reservado na agenda para cada atendimento deve refletir a realidade do serviço, considerando características do pet que afetam a duração (porte, peso, pelagem). A agenda deve avisar previamente quando um novo agendamento for entrar em conflito ou apertar demais um horário já ocupado, em vez de permitir a marcação sem nenhum alerta.

### ⚙️ Requisitos de sistema
- O sistema deve permitir cadastrar, por pet, informações que influenciam o tempo de atendimento (porte, peso, tipo de pelagem).
- Ao agendar, o sistema deve estimar a duração do atendimento com base nessas características, e não usar um tempo fixo igual para todo pet.
- Se um novo agendamento for conflitar com a duração estimada de um atendimento já marcado, o sistema deve avisar antes de confirmar — permitindo repensar o horário.
- Quando um conflito acontecer mesmo assim (por decisão do atendente ou por urgência), deve existir uma forma de marcar esse agendamento como "encaixe", diferenciando-o de um agendamento normal.
- Para agendamentos marcados como "encaixe", o sistema deve apoiar o aviso ao tutor em diferentes momentos: antes de confirmar (se o tutor está tentando agendar em horário já cheio), no momento em que o pet é deixado na loja (aviso presencial), e reforçado depois por mensagem.

> A ideia de o sistema analisar automaticamente porte, peso e pelagem para estimar duração é uma sugestão inicial e precisa ser validada tecnicamente pelo time de desenvolvimento quanto à viabilidade e ao nível de precisão possível — o critério exato de cálculo é uma decisão de implementação, não uma regra fechada aqui.

### ✅ Critérios de aceite
- **Dado** que um pet tem porte, peso e tipo de pelagem cadastrados, **quando** um novo agendamento é criado para ele, **então** o sistema deve calcular e reservar uma duração estimada compatível com essas características, em vez de um tempo fixo padrão.
- **Dado** que um horário já está ocupado ou perto de ficar cheio, **quando** um tutor ou atendente tenta agendar nesse mesmo horário, **então** o sistema deve exibir um aviso antes de confirmar o agendamento, dando a chance de escolher outro horário.
- **Dado** que um agendamento é confirmado mesmo com conflito de horário, **quando** o registro é salvo, **então** ele deve ser marcado como "encaixe", diferenciado visualmente de um agendamento normal na agenda.
- **Dado** que um agendamento está marcado como "encaixe", **quando** o pet é deixado na loja, **então** o sistema deve apoiar o aviso presencial ao tutor e permitir o reforço posterior por mensagem.

### ⚠️ Pontos de atenção na implementação
- O cálculo de duração estimada precisa de um período inicial de calibração — os primeiros tempos estimados provavelmente não vão bater exatamente com a realidade, e o sistema deve permitir ajuste manual pelo profissional até que o cálculo automático fique mais confiável (se essa abordagem automática for validada como viável).
- Vale considerar se cada profissional tem seu próprio ritmo de atendimento (dois tosadores diferentes podem demorar tempos diferentes para o mesmo porte de pet) — isso pode exigir que a estimativa considere não só o pet, mas também quem vai atender.
- A comunicação de "encaixe" ao tutor precisa ser clara sobre o motivo (evitar parecer descaso) — o texto da mensagem/aviso deve deixar claro que é uma prioridade por conflito de agenda, mantendo a transparência que já é um valor do projeto (reforçado no caso de ocorrências, documentado anteriormente).
