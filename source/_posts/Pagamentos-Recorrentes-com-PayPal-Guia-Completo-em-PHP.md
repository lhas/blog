---
title: 'Pagamentos Recorrentes com PayPal: Guia Completo! (em PHP)'
date: 2016-11-21 01:42:32
tags: ['recurring payments', 'paypal', 'php']
---

![](never-give-up_o_1608723.jpg)

# Introdução


Faz um tempo que não escrevo por aqui. Hoje resolvi escrever a respeito de um assunto que tirou o meu sono durante algumas noites.

O motivo é simples. *"PayPal is a mess."* disse um comentário perdido de 2014 no blog deles. E é uma bagunça mesmo.

Só quem já precisou trabalhar com este gateway sabe como a documentação deles é estranha.

Atualmente, o PayPal tem 2 métodos de integração: **(a) Clássico** ou **(b) REST API**.

Nós iremos abordar aqui o segundo método, através da REST API deles.

**Por que a REST e não a Clássica?**
R: A Clássica é uma bosta para configurarmos o retorno automático, para fazer as coisas funcionarem você não tem uma biblioteca padrão suportada pelo PayPal e a qualquer momento este método será abandonado.

Para quem já fez integração com eles alguma vez, sabe que existe o Express Checkout, o IPN (Instant Payment Notification), entre outros.

Então, não usaremos nada disto, hehe. >:)

Tópicos do nosso papo de hoje:

- **Introdução** (estamos aqui)
- **Tokens de Acesso** (OAuth)
- **Instalando SDK** (Composer)
- **Definindo nossos modelos de cobrança** (Billing Plans)
- **Criando um acordo para nosso usuário** (Billing Agreements)
- **Notificações Automáticas** (Webhooks)
- **Saindo de Sandbox para o mundo** (Environments)

"*Talk is cheap, show me the code!*"

Antes de qualquer coisa, iremos precisar de...

# Tokens de Acesso

![](oauth.jpg)

1) Acesse https://developer.paypal.com/
2) Faça Login, depois clique em [Dashboard](https://developer.paypal.com/developer/applications/)
3) Em *"REST API apps"*, clique em **Create App**

![](0.png)

4) Insira um nome em *App Name*, guarde o *Sandbox developer account*. Clique em *Create App*.

![](1.jpg)
![](2.png)

5) Guarde o **Client ID** e  o **Secret**. Precisa clicar em *Show* para ver o Secret Key.

Com nosso token em mãos, vamos seguir para o próximo tópico...