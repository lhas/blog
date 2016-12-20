---
title: 'NoSQL para Leigos, Parte 1'
date: 2016-12-15 21:41:08
tags: ['nosql']
cover: cover.jpg
share_cover: /2016/12/15/NoSQL-para-Leigos-Parte-1/cover.jpg
---

# Introdução

Estava eu, no meu cantinho, fazendo o que eu sei fazer de melhor: caçar bugs. Até que o CTO da empresa que trabalho me fez um pedido muito interessante:

"Explique para o setor de Marketing como eles podem rodar consultas na nossa base de dados em MongoDB."

Situação perfeita para eu realinhar o meu conhecimento com NoSQL e escrever mais um pouco para este bendito blog.

# Contexto

Temos uma base de dados com diversos CNPJs, que contém informações como: atividades, data de registro, razão social, natureza jurídica, etc.

A ideia é que através de alguns snippets o pessoal possa *mineirar* estes dados.

![](mineiros.jpg)
*São bombeiros ou mineiros? Eis a questão.*

Então antes de sair espalhando *receitas de miojo*, gostaria de introduzir a ferramenta com **20%** teoria e **80%** de prática.

![](pareto.jpg)
*Valeu Pareto! Tamo junto.*

# O que é MongoDB?

É o software da base de dados que nós usamos. Apesar de usarmos esta solução, existem dezenas, talvez centenas, de outras soluções concorrentes ao MongoDB, com o foco de solucionar o mesmo problema: **NoSQL**.

# O que é NoSQL?

Sempre que falamos de base de dados, temos como referência uma tabela ou uma planilha.

A magia do NoSQL é justamente essa. Ela é totalmente o oposto de uma planilha.

NoSQL é um tipo de base de dados que fornece o armazenamento e recuperação de dados em diversos modelos (chave-valor, grafos ou documentos).

O MongoDB especificamente trabalha com o modelo de **Documentos**.

Colocando um exemplo prático da coisa:

1 base de dados MongoDB tem:

- Diversas Collections ("tabelas");
- Dentro das collections, temos os documentos;
- Documentos são a entidade mais básica da base, é o objeto que nós queremos manipular por si só.

Exemplo prático:

![](1.png)

Temos a nossa base *CNPJ Robot AWS*.

Dentro da nossa base, temos a coleção de *CNPJs*.

Dentro da nossa coleção, temos os nossos documentos:

![](2.png)

Dentro destes documentos, temos nossos dados, no formato de chave-valor:

![](3.png)

Esta é a nossa estrutura de dados.

Através destes documentos, que nós iremos rodar nossas *consultas* para identificar somente os documentos que desejamos, semelhante as funções do Excel.

# Como acessar a base de dados?

Você irá precisar de um cliente. Existem diversos clientes hoje para MongoDB, mas o que eu recomendo é o [Robomongo](https://robomongo.org/).

Ele está disponível para Windows, Mac e Linux. Você pode baixá-lo [aqui](https://robomongo.org/download).

Após instalá-lo, você vai precisar:

- Endereço IP do banco;
- Usuário;
- Senha.

Para adquirir estas credenciais, solicite o responsável.

Voltando para nossa base de dados, para enxergamos estes dados, precisamos rodar *consultas*.

# O que é uma consulta?

É algo parecido com isto:

```sql
db.getCollection('cnpj').find({})
```

Basicamente, estamos dizendo ao banco de dados para retornar todos os documentos da coleção de CNPJ.

# E se quisesessemos...

## Recuperar somente empresas ativas?

```sql
db.getCollection('cnpj').find({"situacao": "ATIVA"})
```

Como pode ver, a diferença ocorreu dentro das chaves do `find`.

A chave, é o nome da coluna que você deseja filtrar a informação.

O valor, é o que você deseja pesquisar.

Vamos para outros exemplos.

## Recuperar somente empresas ativas do DF?

```sql
db.getCollection('cnpj').find({"situacao": "ATIVA", "uf": "DF"})
```

Como você pode ver, basta adicionar um novo chave-valor separado por vírgula, para você fazer uma consulta em múltiplas colunas.

## Recuperar empresas em uma atividade principal?

Como as empresas podem ter mais de uma atividade principal, a estrutura de dados para este campo, é no formtao de *Listas* (Array).

Pelo fato de ser uma lista, para nós consultarmos este tipo de dado, precisamos fazer de uma forma um pouco diferente.

Veja o exemplo abaixo:

```sql
db.getCollection('cnpj').find({"atividade_principal.code": "43.30-4-05"})
```

Basicamente, indicamos a segunda coluna através de um ponto final.

Exemplos:

- atividade_principal.code
- atividade_principal.text
- qsa.qual
- qsa.nome

## Recuperar empresas com um capital social MAIOR que X

Para podermos fazer esta consulta em campos que são números, como é o caso do capital social, precisamos usar algumas palavras-chaves.

No caso de uma consulta com empresas ativas que tenham o capital social maior que dez mil reais:

```sql
db.getCollection('cnpj').find({"situacao": "ATIVA", "capital_social": { $gt: "10000.00" }  })
```
