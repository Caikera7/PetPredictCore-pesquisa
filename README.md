# 🐾 PetPredict-Pesquisa

> Um repositório de conhecimento sobre gestão de petshops: erros comuns, processos que quebram e as regras de negócio que evitam que eles quebrem.

![Status](https://img.shields.io/badge/status-em%20constru%C3%A7%C3%A3o-yellow)
![Tipo](https://img.shields.io/badge/tipo-conhecimento%20%2F%20pesquisa-blue)
![Licença](https://img.shields.io/badge/uso-livre%20para%20consulta-lightgrey)

---

## 💡 Por que este repositório existe

Eu não sou dono de petshop. Sou desenvolvedor. Mas passei tempo o suficiente dentro da operação de um petshop pra ver, na prática, onde as coisas dão errado: venda de produto que já saiu de estoque, agendamento de banho e tosa batendo horário, caixa fechando com diferença e ninguém sabe por quê.

Esse repositório nasceu da tentativa de traduzir esses problemas reais em algo reaproveitável — tanto para quem constrói sistemas para petshops quanto para quem administra um. Ele é a base de conhecimento por trás do **[PetPredictCore](https://github.com/Caikera7/PetPredictCore)**, mas não depende dele: você pode usar o conteúdo aqui mesmo sem tocar em uma linha de código.

---

## 👥 Para quem é este repositório

| Público | O que encontra aqui |
|---|---|
| 🧑‍💻 **Desenvolvedores** | O *porquê* por trás das regras de negócio de sistemas de gestão para petshop (PDV, estoque, agendamento, financeiro) — não só o *como* técnico |
| 🐕 **Donos e gestores de petshop** | Onde os processos costumam falhar e como organizar a operação para reduzir erros, sem precisar entender de tecnologia |

---

## 🗂️ Como o repositório está organizado

O conteúdo é dividido em duas pastas principais, cada uma com um público-alvo definido — isso evita textos que tentam servir todo mundo e acabam não servindo bem ninguém:

```
📁 para-devs/
   📁 pdv               → regras de venda, erros comuns no caixa
   📁 estoque           → controle de estoque, divergências, estoque preditivo
   📁 banho-e-tosa      → agendamento, fluxo de atendimento, gargalos
   📁 financeiro        → fluxo de caixa, precificação, erros financeiros comuns

📁 para-donos/          → guias práticos de gestão, sem linguagem técnica
```

**`/para-devs`** é escrito para desenvolvedores cada subpasta representa um módulo do sistema, e o foco é o raciocínio completo, do problema real até o requisito técnico de sistema. Espera alguma familiaridade com vocabulário de desenvolvimento.

**`/para-donos`** é escrito para quem gerencia a operação no dia a dia sem termos técnicos, com foco em "como evitar isso na prática", com ou sem sistema envolvido.

Dentro de cada módulo em `/para-devs`, o documento central é o `de-problema-para-regra.md`, que segue sempre a mesma lógica:

1. **🔍 Problema real observado** — o que acontece na prática
2. **🧩 Causa raiz** — por que isso acontece
3. **📏 Regra de negócio derivada** — o que precisa ser garantido para o problema não se repetir
4. **⚙️ Requisito de sistema** — como isso se traduz em algo que um sistema deveria garantir (sem código, sem implementação)

---

## 🔗 Relação com o PetPredictCore

Este repositório é a **camada de raciocínio**. O [PetPredictCore](https://github.com/Caikera7/PetPredictCore) é a **implementação**. Uma regra de negócio documentada aqui pode levar meses para virar código lá — e está tudo bem, porque entender o problema direito é o que evita reescrever o sistema depois.

---

## 🤝 Como contribuir

Por enquanto este repositório é mantido por mim, mas a estrutura já está pensada para receber contribuições no futuro — de desenvolvedores e de donos de petshop que queiram compartilhar situações reais.

Se você tem uma experiência (boa ou ruim) de gestão de petshop que poderia virar uma regra documentada aqui, [abra uma issue](../../issues).
