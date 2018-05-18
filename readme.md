# Criando o projeto e deixando pronto para as validações
[Link para a playlist com os screencasts no YouTube](https://www.youtube.com/watch?v=2SGlXksD2gA&list=PLIFOx3X8xDuvT4otwlbFM7ca8uCRrY9C_)

## Criar o banco de dados
- Crie o banco de dados *validation* em seu SGBD preferido
    - Eu uso MariaDB/MySQL nos screencasts

## Criar o projeto Laravel
```bash
composer create-project laravel/laravel validation
cd validation
```

## Editar os dados da conexão com o banco de dados
- No arquivo .env, coloque os dados da sua conexão com o banco de dados
```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=validation
DB_USERNAME=user
DB_PASSWORD=user
```

## Criar o model
```bash
php artisan make:model Livro -mcr
```

## Editar o arquivo de migração da tabela livros
- No arquivo database/migrations/****_**_**_******_create_livros_table
```php
public function up()
{   
    Schema::create('livros', function (Blueprint $table) {
        $table->increments('id');
        $table->string('titulo');
        $table->string('autor');
        $table->string('edicao');
        $table->string('isbn');
        $table->timestamps();
    }); 
}
```

## Migrar o banco de dados
- Se você usa o XAMPP (ou outro pacote que use o MariaDB) ou
- Se você usa o MariaDB ou versões antigas do MySQL (abaixo da 5.7.7), edite o arquivo app/Providers/AppServiceProvider.php
```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema; /* MariaDB fix */

class AppServiceProvider extends ServiceProvider
{
    /** 
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {   
        Schema::defaultStringLength(191); /* MariaDB fix */
    }

    /** 
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {   
        //  
    }   
}
```
- Rode a migration
```bash
php artisan migrate
```

## Crie dados fictícios
- Crie a Factory com o comando:
```php artisan make:factory LivroFactory --model="App\\Livro"```

- No arquivo database/factories/LivroFactory.php
```php
<?php

use Faker\Generator as Faker;

$factory->define(App\Livro::class, function (Faker $faker) {
    return [
        'titulo' => $faker->sentence,
        'autor' => $faker->name,
        'edicao' => $faker->numberBetween(1, 10),
        'isbn' => $faker->unique()->randomNumber(8),
    ];
});

```
- No arquivo database/seeds/DatabaseSeeder.php
```php
<?php

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /** 
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {   
        echo "Criando 85 livros...\n";
        factory(App\Livro::class, 85)->create();
    }   
}
```
### Rode o seeder para popular o banco de dados
```bash
php artisan db:seed
```
### Use o Tinker (ou seu cliente de banco de dados)
Verificando se os dados foram gravados no banco:
*Se não quiser usar o Tinker, use seu programa cliente de banco de dados favorito*
```bash
php artisan tinker
```
```php
>>> Livro::count()
>>> Livro::all()
```

## Ajustando o Controller
Ajustar os métodos para lidar com os dados do banco e dos formulários
- No arquivo app/Http/Copntrollers/LivroController.php
```php
<?php

namespace App\Http\Controllers;

use App\Livro;
use Illuminate\Http\Request;

class LivroController extends Controller
{
    public function index()
    {
        $livros = Livro::all();
        return view('livros.index', compact('livros'));
    }

    public function create()
    {
        return view('livros.create');
    }

    public function store(Request $request)
    {
        $livro = new Livro;
        $livro->titulo = $request->titulo;
        $livro->autor = $request->autor;
        $livro->edicao = $request->edicao;
        $livro->isbn = $request->isbn;

        $livro->save();

        $request->session()->flash('alert-success', 'Livro adicionado com sucesso!');
        return redirect()->route('livros.index');
    }

    public function show(Livro $livro)
    {
        return view('livros.show', compact('livro'));
    }

    public function edit(Livro $livro)
    {
        $action = action('LivroController@update', $livro->id);
        return view('livros.edit', compact('livro', "action"));
    }

    public function update(Request $request, Livro $livro)
    {
        $livro->titulo = $request->titulo;
        $livro->autor = $request->autor;
        $livro->edicao = $request->edicao;
        $livro->isbn = $request->isbn;

        $livro->save();

        $request->session()->flash('alert-success', 'Livro alterado com sucesso!');
        return redirect('livros');
    }

    public function destroy(Request $request, Livro $livro)
    {
        $livro->delete();
        $request->session()->flash('alert-success', 'livro apagado com sucesso!');
        return redirect()->back();
    }
}
```

## Criar as rotas para nosso resource
- No arquivo routes/web.php
```php
Route::resource('livros', 'LivroController');
```

## Criar as views
- Dentro da pasta resources/views, crie a pasta layouts, dentro dela o arquivo app.blade.php
```php
<!DOCTYPE html>
<html lang="{{ app()->getLocale() }}">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ config('app.name', 'Laravel') }}</title>

    <!-- Styles -->
    <link href="{{ asset('css/app.css') }}" rel="stylesheet">
</head>
<body>
    <div id="app">
        <nav class="navbar navbar-expand-md navbar-light navbar-laravel">
            <div class="container">
                <a class="navbar-brand" href="{{ url('/') }}">
                    {{ config('app.name', 'Laravel') }}
                </a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>

                <div class="collapse navbar-collapse" id="navbarSupportedContent">
                    <!-- Left Side Of Navbar -->
                    <ul class="navbar-nav mr-auto">

                    </ul>
                </div>
            </div>
        </nav>

        <main class="py-4">
            @yield('content')
        </main>
    </div>

    <!-- Scripts -->
    <script src="{{ asset('js/app.js') }}"></script>
</body>
</html>
```

- No arquivo resources/views/livros/index.blade.php
    - Você deve criar a pasta livros e o arquivo index.blade.php
```php
@extends('layouts.app')
@section('content')
    <div class="row justify-content-center">
        <div class="col-md-8">
            <h2>Livros</h2>
            <div class="flash-message">
                @foreach (['danger', 'warning', 'success', 'info'] as $msg)
                    @if(Session::has('alert-' . $msg))

                    <p class="alert alert-{{ $msg }}">{{ Session::get('alert-' . $msg) }} <a href="#" class="close" data-dismiss="alert" aria-label="fechar">&times;</a></p>
                    @endif
                @endforeach
            </div>
            <div class="table-responsive">
                <table class="table table-striped">
                    <thead>
                        <tr>
                            <th>Livro</th>
                            <th>Autor</th>
                            <th>Edição</th>
                            <th>ISBN</th>
                            <th colspan="2">Ações</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach($livros as $livro)
                        <tr>
                            <td><a href="livros/{{ $livro->id }}">{{ $livro->titulo }}</a></td>
                            <td>{{ $livro->autor }}</td>
                            <td>{{ $livro->edicao }}</td>
                            <td>{{ $livro->isbn }}</td>
                            <td>
                                <a href="{{action('LivroController@edit', $livro->id)}}" class="btn btn-warning">Editar</a>
                            </td>
                            <td>
                                <form action="{{action('LivroController@destroy', $livro->id)}}" method="post">
                                  {{csrf_field()}} {{ method_field('delete') }}
                                  <button class="delete-item btn btn-danger" type="submit">Deletar</button>
                              </form>
                            </td>
                        </tr>
                        @endforeach
                    </tbody>
                </table>
            </div>
        </div>
    </div>
@endsection
```
- No arquivo resources/views/livros/show.blade.php
    - Você deve criar o arquivo show.blade.php
```php
@extends('layouts.app')
@section('content')
    <div class="row justify-content-center">
        <div class="col-md-8">
            <h2>Livros</h2>
            <div class="flash-message">
                @foreach (['danger', 'warning', 'success', 'info'] as $msg)
                    @if(Session::has('alert-' . $msg))

                    <p class="alert alert-{{ $msg }}">{{ Session::get('alert-' . $msg) }} <a href="#" class="close" data-dismiss="alert" aria-label="fechar">&times;</a></p>
                    @endif
                @endforeach
            </div>
            <h3>{{ $livro->titulo }} </h3>

            <div class="card">
              <ul class="list-group list-group-flush">
                  <li class="list-group-item">Autor: <strong>{{ $livro->autor }}</strong></li>
                <li class="list-group-item"><b>Edição:</b> {{ $livro->edicao }}</li>
                <li class="list-group-item"><b>ISBN:</b> {{ $livro->isbn }}</li>
              </ul>
            </div>
            <hr>
            <a href="{{ url()->previous() }}" class="btn btn-primary">Voltar</a>
        </div>
    </div>
@endsection
```
- No arquivo resources/views/livros/create.blade.php
    - Você deve criar o arquivo create.blade.php
```php
@include('forms.livro')
```
- No arquivo resources/views/livros/edit.blade.php
    - - Você deve criar o arquivo edit.blade.php
```php
@include('forms.livro')
```
- No arquivo resources/views/forms/livro.blade.php
    - Você deve criar a pasta forms e o arquivo livro.blade.php
```php
@extends('layouts.app')
@section('content')
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card card-default">
                <div class="card-header">
                    <h3>Livro</h3>
                </div>
                <div class="card-body">
                    <form method="post" action="{{ $action or url('livros') }}">
                        {{csrf_field()}}
                        @isset($livro) {{method_field('patch')}} @endisset
                        <div class="form-group row">
                            <label for="titulo" class="col-md-4 col-form-label text-md-right">Título</label>
                            <div class="col-md-6">
                                <input id="titulo" class="form-control" name="titulo" type="text" value="{{ $livro->titulo or ''}}">
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="titulo" class="col-md-4 col-form-label text-md-right">Autor</label>
                            <div class="col-md-6">
                                <input id="autor" class="form-control" name="autor" type="text" value="{{ $livro->autor or ''}}">
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="edicao" class="col-md-4 col-form-label text-md-right">Edição</label>
                            <div class="col-md-6">
                                <input id="edicao" class="form-control" name="edicao" type="text" value="{{ $livro->edicao or ''}}">
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="isbn" class="col-md-4 col-form-label text-md-right">ISBN</label>
                            <div class="col-md-6">
                                <input id="isbn" class="form-control" name="isbn" type="text" value="{{ $livro->isbn or ''}}">
                            </div>
                        </div>
                        <div class="form-group row mb-0">
                            <div class="col-md-6 offset-md-4">
                                <button type="submit" class="btn btn-primary">
                                    Save
                                </button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
@endsection
```
