---
title: Autenticação no Laravel 5 com models diferentes (Admin/User/etc)
date: 2016-08-04 10:50:59
tags: ["autenticação", "laravel", "admin"]
cover: cover.jpg
share_cover: /2016/08/04/Autenticacao-no-Laravel-5-com-models-diferentes-Admin-User-etc/cover.jpg
---

Passei recentemente por um problema que eu tinha certeza que era simples de se resolver mas nenhuma informação que eu encontrei na internet 
disponível conseguia explicar claramente como resolvê-la. Nem mesmo na documentação oficial do Laravel.

Então resolvi escrever sobre.

# O cenário

Temos uma aplicação em Laravel 5 onde ela serve como **Gerenciador de Conteúdo (CMS)** e **API (WebService)** para a parte do front-end.

Se tratando de rotas, seria:

```
/admin/
/admin/post
/admin/post/<id>/comment

/api/
/api/post
/api/post/<id>comment
```

# Primeiros passos

Você já rodou este comando abaixo na sua app? Se não, faça-o:

```
php artisan make:auth
```

Este comando irá gerar rotas, views e controllers para a autenticação.

# Configurando a autenticação

Abra o **config/auth.app**. Vamos trabalhar aqui em 2 índices: **'guards'** e **'providers'**.

Em **'guards'** vamos deixar assim:

```php
// Antigo
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'api' => [
        'driver' => 'token',
        'provider' => 'users',
    ],
],
// Novo
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'admin' => [
        'driver' => 'session',
        'provider' => 'admins',
    ],
    'api' => [
        'driver' => 'token',
        'provider' => 'users',
    ],
],
```

Em **'providers'** vamos deixar assim:

```php
// Antigo
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\User::class,
    ],
],
// Novo
'providers' => [
    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Admin::class,
    ],
    'users' => [
        'driver' => 'eloquent',
        'model' => App\User::class,
    ],
],
```

No caso, apesar de termos um guard para API feita já pelo Laravel, não vamos usá-la.

Ela já vem com uma pré-configuração onde se faz necessário uma coluna "api_token" na tabela do model a ser autenticado.

É uma implementação fora do padrão do **JWT** por exemplo.

Então criamos um novo guard chamado admin (e consequentemente um novo provider para servir ao nosso model de Admin), que será usado pelas rotas do admin e a API usará o guard padrão que é o web.

# Adaptando os controllers
Após fazer os passos acima, vamos mexer nos controllers.

Abra o controller **AuthController.php** que o artisan gerou para nós no primeiro passo.

Ele se encontra em **app/Http/Controllers/Auth/AuthController.php**.

Aqui iremos indicar qual deve ser o guard usado por esta autenticação.

Aproximadamente na linha 31, embaixo do $redirectTo, vamos definir o guard.

```php
    protected $redirectTo = '/';

    // Aqui nossa nova linha
    protected $guard = 'admin';
```

**(Opcional)** Se você for usar a página de registro, deve alterar o model de User para Admin no método create() também:

```php
    protected function create(array $data)
    {
        return Admin::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
        ]);
    }
```

Agora, temos que indicar ao middleware do Auth, que queremos usar o nosso guard admin.

Para isto, vamos editar 2 controllers:

**HomeController.php** - gerado pelo nosso primeiro comando.
**AdminController.php** - Este por padrão não existe, mas recomendo que você crie e use-o como classe pai para as suas páginas do Admin. Assim ele vai herdar os middlewares que nós configurarmos.

**app/Http/Controllers/HomeController.php:**

```php
// Antigo
public function __construct()
{
    $this->middleware('auth');
}
// Novo
public function __construct()
{
    $this->middleware('auth:admin');
}
```

**app/Http/Controllers/Admin/AdminController.php:**

```php
// Antigo
public function __construct()
{
    $this->middleware('auth');
}
// Novo
public function __construct()
{
    $this->middleware('auth:admin');
}
```

# Corrigindo as views do admin

Se você abrir o layout que é gerado pelo Artisan, verá algumas linhas fazendo referência ao Auth desta forma:

```php
@if (Auth::guest())
@endif
```

Neste caso, você deverá trocar para o seguinte:

```php
@if (Auth::guard('admin')->guest())
@endif
```

Isso significa que ele vai analisar se há alguma sessão especificamente no guard de admin.

PRONTO!

## Conclusão

Se você tentar abrir a página de login do seu administrador agora e tentar efetuar o login, vai ver que ele vai verificar a tabela de Admins.

Se a sua API estiver configurada corretamente e você tentar requisitar algum endpoint protegido, ele usará por padrão o guard 'web', que no caso vai acionar a tabela de Users.

Espero que gostem. Eu quebrei a cabeça com este assunto, encontrei diversas soluções na internet, mas que envolviam geração de middlewares customizados, etc.

Coisas que eu acredito não serem necessárias já que o Laravel tem um bom suporte a camada de Autenticação por padrão, bastamos apenas nós extendermos ela.


## Referências

Posts que eu li que me ajudaram a abstrair esta ideia:
- https://scotch.io/tutorials/role-based-authentication-in-laravel-with-jwt
- http://imrealashu.in/code/laravel/multi-auth-with-laravel-5-2-2/
- https://laravel.com/docs/5.2/authentication