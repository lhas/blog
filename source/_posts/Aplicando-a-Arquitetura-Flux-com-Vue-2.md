---
title: Aplicando a Arquitetura Flux com Vue 2
date: 2017-01-10 22:46:07
tags: ['flux', 'vuex', 'vue']
cover: cover.png
share_cover: /2017/01/10/Aplicando-a-Arquitetura-Flux-com-Vue-2/cover.png
---

# Introdução

Olá a todos!

Depois de termos implementado um CRUD através do Vue 2, agora iremos profissionalizar a arquitetura do mesmo.

Caso você esteja desenvolvendo um SPA de médio a grande porte, é interessante que você tenha uma arquitetura que seja escalável.

E uma destas patterns escaláveis que temos hoje em dia é o [Flux](https://facebook.github.io/flux/).

Caso você queira conhecer os outros posts desta série de Vue 2:

- [Iniciando um projeto com Vue 2](http://blog.0e1dev.com/2016/12/11/Iniciando-um-projeto-com-Vue-2/)
- [Autenticação com JWT e Vue 2](http://blog.0e1dev.com/2016/12/11/Autenticacao-com-JWT-e-Vue-2/)
- [Montando CRUD com Vue 2](http://blog.0e1dev.com/2017/01/07/Montando-CRUD-com-Vue-2/)

# O que é Flux?

![](https://media.giphy.com/media/3o6wrdG8vt4X86Pauc/giphy.gif)

É um modelo de arquitetura para User Interfaces que mantém um fluxo de dados uni-direcional.

Este fluxo permite que o desenvolvedor gerencie de forma centralizada os dados de toda a aplicação.

Uma das regras é que você não pode alterar os dados diretamente. Você tem que fazer uma solicitação de mudança (commit, literalmente) para que aquele dado seja modificado. Esta solicitação é enfileirada e direcionada para a ação desejada.

Esta explicação é um resumo e é normal que não faça muito sentido até que coloquemos a mão no código.

# Quais os benefícios deste pattern?

Os benefícios são diversos:

- Dados da aplicação ficam centralizados em um lugar só (store);
- Os dados não podem ser modificados diretamente, prevenindo possíveis comportamentos anormais;
- É possível fazer uma viagem no tempo para debugar o que nossa aplicação está fazendo;
- Você pode importar/exportar os fluxos para debugar melhor.
- Entre outros...

# Eu realmente preciso disto?

Como dito anteriormente, só vale a  pena fazer este tipo de  implementação caso você tenha certeza de que a sua aplicação irá crescer com o tempo. Para projetos pequenos, isto pode ser matar uma formiga com uma bazuca. Uma frase dita por Dan Abramov, autor do Redu, define bem:

>Flux libraries are like glasses: you’ll know when you need them.
>**Dan Abramov**

# Implementando...

Caso você esteja vindo do nosso post anterior, [Montando CRUD com Vue 2](http://blog.0e1dev.com/2017/01/07/Montando-CRUD-com-Vue-2/), você terá a seguinte situação:

- Um componente de Index (listando todos os registros);
- Este componente consumindo uma API;
- Esta API está sendo acessada através do vue-resource, diretamente no Componente.

Antes de colocarmos a mão na massa quanto a arquitetura, precisamos fazer um isolamento de responsabilidade aí.

Precisamos que nosso Componente deixe de se comunicar diretamente com a API.

Para isto, vamos precisar de um Service, que fará esta comunicação.

# Criando Services com Vue

Para quem já tem um background de Angular 1, os Services aqui terão um comportamento semelhante.

A diferença aqui é que nosso Service será apenas um módulo JS, sem muita magia por baixo dos panos (ainda bem).

Já que estamos trabalhando com *Aulas* (Lessons), faremos um Service para o mesmo.

Para isto:
- Vamos criar uma pasta para armazenar nossos services;
- Criar um arquivo base que será usado por todos os services;
- Criar o service de Lessons.

Supondo que você está na pasta da nossa app:

```bash
# Cria a pasta
mkdir src/services

# Inicializa a base dos services
touch src/services/index.js

# Inicializa o service de Lessons
touch src/services/lessons.js
```

# Base dos Services

Este código-base é necessário caso você não queira ficar repetindo código entre os services.

É bem simples e ficará assim:

```js
import Vue from 'vue'
import VueResource from 'vue-resource'

Vue.use(VueResource)

export default Vue
```

Basicamente, ele cria uma instância do Vue e incorpora o vue-resource nesta instância.

Isto se faz necessário para podermos efetuar as requisições XHR nos nossos services.

# Service de Lessons

Este módulo precisa fazer todas as nossas operações de CRUD - Adicionar, Ler, Editar e Deletar.

Ficará assim:

```js
import service from './index'

const resource = service.resource('http://localhost:3000/lessons{/id}.json')
const currentProfile = JSON.parse(window.localStorage.getItem('current-profile'))

export default {
  // Read
  getAll () {
    return resource.get({})
  },
  getOne (id) {
    return resource.get({id: id})
  },
  // Delete
  deleteOne (lesson) {
    return resource.delete({id: lesson.id})
  },
  // Create
  createOne (lesson) {
    lesson['student_id'] = currentProfile.student.id
    return resource.save({}, lesson)
  },
  // Update
  updateOne (lesson) {
    return resource.update({id: lesson.id}, lesson)
  }
}
```

Caso você precise adicionar valores de campos padrão, este é um ótimo lugar. Eu mesmo fiz isso na linha 20. Pego o ID do estudante atual e acoplo ele ao novo registro de aula.

# Começando com Vuex

Primeiro, instale ele como dependência do seu projeto:

```bash
npm install vuex --save
```

Agora, vamos criar a nossa *Store*.

# O que é uma Store?

Toda aplicação que usufrua desta arquitetura terá uma Store.

Store é o nosso "centralizador de dados". Ele irá armazenar todos os estados dos dados atuais da nossa aplicação, como se fosse um objeto global.

É como se ele fosse a nossa base de dados front-end.

# Qual a diferença entre uma Store e um objeto global (plain global object)?

A primeira diferença que os dados dentro do Store são reativos. Então caso o componente X mude o dado Y, o componente Z que usufrui do mesmo dado Y também receberá a modificação.

A segunda diferença é que você não tem permissão alguma para modificar os dados dentro do Store diretamente.

Para você fazer qualquer tipo de mudança, você deve solicitar ela explicitamente através de **commits**.

Os *commits* funcionam de uma forma semelhante aos commits do Git, anexando mudanças.

Estas mudanças ocorrem através das **mutações** (mutations).

Isto garante que cada modificação no estado da aplicação seja traçável e navegável, permitindo que o desenvolvedor tenha um passo-a-passo do que a sua aplicação está de fato fazendo e evitando possíveis gargalos ou sobrescrever alguma informação.

### Exemplo de Store

![](https://media.giphy.com/media/jog2TsnJMXqX6/giphy.gif)

Eu vou re-aproveitar o exemplo da própria documentação do Vuex pois acho ela muito boa para dar uma visão do que ocorre por baixo dos panos:

Dado a seguinte store:

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

Imagine que o state seja na verdade o `data()`.

E os mutations sejam na verdade os `methods`.

Para você recuperar o count atual:

```js
store.state.count
```

Para você rodar o método `increment` e consequentemente modificar o count:

```js
store.commit('increment')
```

Após rodar a linha acima, o `store.state.count` será igual a `1`.

# O que é State?

Como dito anteriormente, o state é a nossa camada de dados.

Ela será a nossa `única fonte da verdade` (single source of truth).

Apesar de ser algo centralizado, nós podemos criar *módulos* que terão *states* próprios, como veremos mais a frente.

Após você injetar o Vuex e o Store na sua aplicação, você poderá acessá-lo assim:

```js
this.$store; // -> retornará todo o Store
this.$store.state; // -> retornará todos os States disponíveis
this.$store.commit('exemplo'); // -> rodará o commit 'exemplo'
```

Chega de teoria e vamos para a prática.

# Aplicando o Vuex

Vamos precisar de uma pasta para armazenar nosso Store.

Dentro desta pasta, teremos:

- Um módulo para nosso Store, que irá inicializá-lo;
- Um módulo para armazenar os nossos tipos de mutação (veremos mais a frente).

Para isto:

```bash
mkdir src/store
touch src/store/index.js
touch src/store/mutation-types.js
```

Com nossa pasta e arquivos criados, vamos começar pelo nosso módulo para Store.

## Módulo da Store

Ele ficará assim:

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const debug = process.env.NODE_ENV !== 'production'

export default new Vuex.Store({
  strict: debug
})
```

# Importando Store na App

Abra seu `main.js`.

Primeiro vamos precisar importar o Store que acabamos de criar.

```js
import store from './store'
```

Depois vamos injetá-lo dentro da nossa instância do Vue.

```js
new Vue({
  el: '#app',
  router,
  store
})
```

Pronto! Agora teremos acesso ao nosso Store através do `this.$store`.

# Módulos

Para nosso código ficar mais organizado possível, vamos separar nossa store em módulos.

Para isto, vamos precisar de uma pasta "modules":

```bash
mkdir src/store/modules
```

Vamos começar com um módulo simples, que será o de **Alertas**.

## Módulo de Alertas

![](https://media.giphy.com/media/11nAv843yJ4Flm/giphy.gif)

Este módulo será muito simples, ele apenas mostrará informações para nossos usuários com um `alert()`.

Para isto, vamos precisar de uma pasta para este módulo:

```bash
mkdir src/store/modules/alerts
```

Como ele será um módulo bem simples, podemos fazer toda implementação dentro de um único JS:

```bash
touch src/store/modules/alerts/index.js
```

Vamos editá-lo agora.

### State de Alertas

Nosso state será assim:

```js
const state = {
  message: null,
  delay: 4000
}
```

### Actions de Alertas

Nosso módulo terá uma ação chamada `createOne`, que fará a criação do alerta.

Nossa action será assim:

```js
const actions = {
  createOne ({ commit, state }, alert) {
    commit(types.ALERTS_CHANGE_MESSAGE, alert.message)
    commit(types.ALERTS_DISPLAY)
  }
}
```

Como você pode ver, nossa ação efetua **2 commits**.

- O primeiro para modificar o nosso estado de mensagem;
- O segundo para mostrar o alerta.

Só que para efetuar estes commits, ele utiliza 2 constantes: `ALERTS_CHANGE_MESSAGE` e `ALERTS_DISPLAY`.

Ambas oriúndas do `types`.

### Mutation Types de Alertas

Lembra que criamos um `mutation-types.js`? Vamos precisar modificá-lo e incluir o mesmo no nosso módulo de alertas.

Para incluir ele, insira na primeira linha do módulo:

```js
import * as types from '../../mutation-types'
```
Agora vamos preencher o **mutation types** com as constantes necessárias para nossas ações:

```js
export const ALERTS_CHANGE_MESSAGE = 'ALERTS_CHANGE_MESSAGE'
export const ALERTS_DISPLAY = 'ALERTS_DISPLAY'
```

**Pronto!** Até aqui, já temos um módulo de alertas com **2 states**, **2 commits**, **2 mutation types** e **1 action**.

**Observação:** Para cada state, você sempre terá um mutation/commit/mutation type.

No caso do nosso módulo de Alertas, ficou faltando somente as **Mutations**.

# O que são Mutations?

![](https://media.giphy.com/media/zpoDwr92f7UJ2/giphy.gif)

As mutações são a única forma que você tem para modificar os dados do **State**.

Como dito anteriormente, você não pode modificar os dados do **State** diretamente, você precisa antes efetuar um **commit** com as mudanças necessárias.

**Cada commit precisará de um mutation.**

### Mutations de Alertas

Para nosso módulo, estas serão as mutações necessárias:

```js
const mutations = {
  [types.ALERTS_CHANGE_MESSAGE] (state, message) {
    state.message = message
  },
  [types.ALERTS_DISPLAY] (state) {
    setTimeout(() => {
      alert(state.message)
    }, state.delay)
  }
}
```

**Observação:** Caso esta sintaxe pareça estranha para você, pesquise mais sobre ES6. Mas de forma resumida, o ES6 permite que criemos uma função com nome igual a uma constante. Para isto, basta encapsular a constante entre colchetes [].

Toda mutação recebe como primeiro parâmetro, o **state** da aplicação.

O segundo parâmetro é o **payload**. Ele é opcional e é através dele que você pode enviar os dados para seu commit.

No caso de efetuarmos um *commit* para mudar a mensagem, o nosso *payload* é a mensagem a se mostrada.

# Finalizando módulo de Alertas

Para finalizar, precisamos exportar ele como módulo de fato. Para isto, no fim do nosso arquivo, você deverá incluir:

```js
export default {
  namespaced: true,
  state,
  actions,
  mutations
}
```

# Resultado do Alertas

Você pode ver o resultado do código explicado acima abaixo. Clique em **Result**:

<script async src="//jsfiddle.net/7e0z2qye/3/embed/"></script>

# Voltando para o CRUD

Acima implementamos um módulo bem simples, mas agora faremos a implementação do módulo de Lessons, que envolve o CRUD criado no tópico anterior.

O primeiro passo é...

# Criar módulo de Lessons

Nossa Store precisa de um módulo só para nossas aulas. Aqui entra a questão de apesar do **Store** ser algo centralizado, nós podemos ter **States** independentes dentro de cada módulo.

Para criá-lo:

```bash
mkdir src/store/modules/lessons
touch src/store/modules/lessons/index.js
touch src/store/modules/lessons/actions.js
touch src/store/modules/lessons/getters.js
touch src/store/modules/lessons/index.js
touch src/store/modules/lessons/mutations.js
touch src/store/modules/lessons/state.js
```

Como este módulo pode vir a ser mais complexo no futuro, faremos o isolamento de cada camada em módulos separados.

Vamos começar pela base (index.js).

## Módulo base de Lessons

Ele irá basicamente centralizar os sub-módulos. Nenhum mistério aqui:

```js
import actions from './actions'
import getters from './getters'
import mutations from './mutations'
import state from './state'

export default {
  namespaced: true,
  state,
  getters,
  actions,
  mutations
}
```

## State de Lessons

Vamos para o State que são os nossos dados, é a segunda parte mais simples aqui:

```js
const state = {
  lessons: [],
  lesson: {}
}

export default state
```

## Getters de Lessons

![](https://media.giphy.com/media/AlRP5F1t3nLry/giphy.gif)

Existe duas formas de você recuperar os dados do **State**:

**1) Acessar o State diretamente**

A maneira mais simples:

```js
this.$store.states.NOME_DO_MODULO.NOME_DO_STATE;
// exemplo:
this.$store.states.lessons.lesson; // -> estamos acessando nosso módulo "lessons" e recuperando o state "lesson"; neste exemplo, fica meio redundante mesmo
```

**2) Usar um getter**

Getters são úteis quando você precisa fazer algum tipo de manipulação no dado do *state* antes de enviá-lo.

Um exemplo clássico seria:

```js
const getters = {
  lessonsDone: (state) => {
    return state.lessons.filter(lesson => lesson.done)
  }
}
```

Ou seja, ao invés de repetirmos este filtro em todo componente que quisermos mostrar esta informação, fazemos o filtro dentro de um Getter, e quando quisermos mostrar o mesmo, acessamos apenas o getter assim:

```js
this.$store.getters['lessons/lessonsDone']; // -> nome_do_modulo/nome_do_getter
```

## Mutation Types de Lessons

Como explicado anteriormente, para cada mutation, vamos precisar de uma constante que chamamos de mutation types para modificá-lo.

No caso, basta adicionar estas 2 linhas no nosso `mutation-types.js` atual:

```js
export const GET_ALL_LESSONS = 'GET_ALL_LESSONS'
export const GET_ONE_LESSON = 'GET_ONE_LESSON'
```

## Mutations de Lessons

Como explicado anteriormente, para cada dado no state, vamos precisar de um mutation para poder modificá-lo.

Como no nosso módulo temos 2 states (**lesson** e **lessons**), teremos 2 mutations:

```js
import * as types from '../../mutation-types'

const mutations = {
  [types.GET_ALL_LESSONS] (state, { lessons }) {
    state.lessons = lessons
  },
  [types.GET_ONE_LESSON] (state, { lesson }) {
    state.lesson = lesson
  }
}
export default mutations
```

**Pronto!** Já temos Getters, Mutations e State.

Ficou faltando as nossas queridas...

## Ações de Lessons

Aqui que o bixo costuma pegar. Caso você precise fazer manipulações assíncronas, este é o lugar. Em todos os outros se parte do princípio que serão manipulações síncronas.

No nosso exemplo, nossas ações serão bem semelhantes aos mesmos métodos do nosso **Service**. Inclusive, é aqui que iremos importá-lo.

```js
import * as types from '../../mutation-types'
import LessonsService from '../../../services/lessons'

const actions = {
  getAll ({ commit, state }) {
    LessonsService.getAll().then((response) => {
      commit(types.GET_ALL_LESSONS, {
        lessons: response.data
      })
    })
  },
  getOne ({ commit, state }, lesson) {
    commit(types.GET_ONE_LESSON, lesson)
  },
  deleteOne ({ commit, state }, lesson) {
    return LessonsService.deleteOne(lesson)
  },
  createOne ({ commit, state }, lesson) {
    return LessonsService.createOne(lesson)
  },
  updateOne ({ commit, state }, lesson) {
    return LessonsService.updateOne(lesson)
  }
}

export default actions
```

No caso do `getAll()`, por exemplo, primeiro fazemos a requisição XHR com o nosso Service.

Somente após esta requisição, caso a mesma seja bem sucedida, efetuamos o commit de `GET_ALL_LESSONS`, indicando que recuperamos todas as aulas.

Junto com nosso **commit**, enviamos no **payload** todas as aulas que nós queremos que apareça.

Os outros casos como deleteOne(), createOne() e updateOne(), não precisamos efetuar commits, pois estes não afetarão diretamente nosso **State**. Mais a frente você entenderá o por quê.

**Pronto!** Toda a implementação necessária quanto a arquitetura já foi feita.

![](https://media.giphy.com/media/3otPon0xX7E4KWwVlC/giphy.gif)

Agora podemos usufruir da mesma na nossa **View**.

## Usando o Flux nas Views

Esta é a parte mais fácil e definitivamente a que você vai ver vantagens.

Se antes a nossa View era responsável por se comunicar com a API, manipular e transferir os dados, agora ela só fará uma coisa:

**Despachará ações.**

Não entendeu? Eu explico. Nosso componente Index atualmente encontra-se assim:

```js
import InputsCreate from './Create'
import InputsUpdate from './Update'

export default {
  name: 'DashboardInputsIndex',
  data () {
    return {
      resource: this.$resource('http://localhost:3000/lessons{/id}.json'),
      lessons: [],
      lesson: {}
    }
  },
  methods: {
    initialize () {
      this.resource.get({}).then((response) => {
        this.lessons = response.data
      })
    },
    handleDelete (lesson) {
      this.resource.delete({id: lesson.id}).then((response) => {
        this.$emit('deleted')
      })
    },
    handleUpdate (lesson) {
      this.lesson = lesson
      window.jQuery('#modal2').modal('open')
    },
    eventDeleted () {
      this.initialize()
      // notifica o usuário
      let message = 'Aula removida!'
      alert(message)
    }
  },
  created () {
    this.initialize()
    this.$on('deleted', () => {
      this.eventDeleted()
    })
  },
  components: {
    'Inputs-Create': InputsCreate,
    'Inputs-Update': InputsUpdate
  }
}
```

Agora, com a arquitetura Flux implementada, ele funcionará assim:

```js
import InputsCreate from './Create'
import InputsUpdate from './Update'

export default {
  name: 'DashboardInputsIndex',
  computed: {
    lessons () {
      return this.$store.getters['lessons/lessons']
    },
    lesson () {
      return this.$store.getters['lessons/lesson']
    }
  },
  methods: {
    initialize () {
      this.$store.dispatch('lessons/getAll')
    },
    handleDelete (lesson) {
      this.$store.dispatch('lessons/deleteOne', lesson).then(() => {
        this.$store.dispatch('lessons/getAll')
        this.$store.dispatch('alerts/createOne', {
          message: 'Aula removida!'
        })
      })
    },
    handleUpdate (lesson) {
      this.$store.dispatch('lessons/getOne', {lesson})
      window.jQuery('#modal2').modal('open')
    }
  },
  created () {
    this.initialize()
  },
  components: {
    'Inputs-Create': InputsCreate,
    'Inputs-Update': InputsUpdate
  }
}
```

Além de termos economizado 7 linhas, o código ficou MUITO mais legível.

Tenha como comparação nossa função `handleDelete()`:

### Antes

```js
handleDelete (lesson) {
  this.resource.delete({id: lesson.id}).then((response) => {
    this.$emit('deleted')
  })
}

this.$on('deleted', () => {
  this.eventDeleted()
})

eventDeleted () {
  this.initialize()
  // notifica o usuário
  let message = 'Aula removida!'
  alert(message)
}
```
### Depois

```js
handleDelete (lesson) {
  this.$store.dispatch('lessons/deleteOne', lesson).then(() => {
    this.$store.dispatch('lessons/getAll')
    this.$store.dispatch('alerts/createOne', {
      message: 'Aula removida!'
    })
  })
}
```

![](https://media.giphy.com/media/Unhhb0W1IaDHG/giphy.gif)

Mesmo sem conhecer previamente o resto do código, ao você ler o código do **Depois**, fica muito claro o que está acontecendo:

- Despacha uma ação para deletar a aula selecionada;
- Caso seja bem sucedida, despacha uma ação para carregar todas as aulas (para atualizá-las);
- Despacha uma ação para criar um novo alerta.

Não precisamos mais de eventos e funções extras. Tudo acontece de forma transparente e legível.

E o mais importante, o componente de **View** deixa de fazer outras coisas que vão além de sua responsabilidade, e ele faz somente a parte dele, que é **Visualizar**.

# Conclusão

Este é um assunto muito polêmico, que tornou-se em alta junto com a popularidade do [React](https://facebook.github.io/react/).

Eu mesmo fui aprender sobre recentemente pela necessidade de mexer em um projeto que usufrui desta arquitetura.

Depois que aprendi, eu percebi que em outros projetos no passado, eu havia feito diversas implementações que se assemelham na lógica descrita pela pattern. Obviamente, minhas implementações eram cheias de possíveis falhas pois eu não conseguia depurar todas as situações possíveis.

Esta é a vantagem de se utilizar um **design pattern**: Você resolve problemas de antemão, sem precisar vivenciá-los diretamente para saber que a maneira X ou Y deve ser evitada ou feita de outra maneira.

Espero que este artigo ajude os mais novatos a entender este assunto tão complexo mas que depois que você absorve, percebe a beleza e simplicidade que tem por trás do mesmo.

## Obrigado por ler até aqui! :-)

#### Referências
- [https://vuejs.org/v2/guide/](https://vuejs.org/v2/guide/)
- [https://vuex.vuejs.org/en/](https://vuex.vuejs.org/en/)
- [https://facebook.github.io/flux/](https://facebook.github.io/flux/)
