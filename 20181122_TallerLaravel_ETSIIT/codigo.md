# Introducción

Taller sobre Laravel en la ETSIIT de UGR el 22 de noviembre de 2018.

Organizado por la Oficina de Software libre de la Universidad de Granada.

Impartido Diego Rodero Pulido.

# Instalación

> Documentation -> Install Laravel -> Composer

Instalar composer es muy sencillo, así como la gestión de paquetes.

Como instalar composer

> Script de la web de composer

Mover composer para que se instale de forma global

    mv composer.phar /usr/local/bin/composer

Instalamos laravel con composer

    composer global require "laravel/installer"

Ponemos la ruta de composer en nuestro path
(editando .bashrc o lo que sea)

    laravel new miprimerlaravel

Si falla (algunas veces pasa), podemos lanzar el comando composer directamente:

    composer create-project --prefer-dist laravel/laravel miprimerlaravel

# Enrutado básico

    php artisan serve

Navegamos a la web: http://127.0.0.1:8000

> /routes/web.php

En /resources tenemos un directorio para las vistas

> /resources/views/welcome.blade.php

Editamos el fichero y creamos una plantilla vacía

~~~html
<html>
	<head>
		<title>Mi primera web con Laravel</title>
	</head>
	<body>
		<h1>Taller de Laravel UGR</h1>
	</body>
</html>
~~~

Creamos otras dos rutas:

~~~php
Route::get('/contacto', function() {
	return view('contacto');
});

Route::get('/sobrenosotros', function() {
	return view('sobrenosotros');
});
~~~

Creamos dos ficheros para estas vistas:

> /resources/views/contacto.blade.php

> /resources/views/sobrenosotros.blade.php

En Welcome creamos un enlace a Contacto y Sobre Nosotros, en estas también:

~~~html
<div>
        <ul>
            <li><a href="/">Inicio</a></li>
            <li><a href="/contact">Contacto</a></li>
            <li><a href="/about">Sobre nosotros</a></li>
        </ul>
</div>
~~~

# Ficheros de plantillas BLADE


Vamos a crear una plantilla para nuestro sitio:

> /resources/views/layout.blade.php

~~~php
<html>
<head>
    <title>Mi primer sitio con Laravel</title>
</head>
<body>
    @yield('contenido')
</body>
</html>
~~~

Ahora en una pagina (welcome) hacemos referencia a la sección:

~~~php
@extends('layout')

@section('contenido')
	<h1>Mi primera sitio web</h1>
@endsection
~~~

Cambiamos sobrenosotros, contacto y wellcome

Ponemos un @yield('titulo') en la plantilla ponemos una sección para mostrar el titulo

~~~php
@yield('titulo', 'Título por defecto')

@section('titulo', 'Formulario de contacto')
~~~


# Pasando datos a las vistas

> /routes/web.php

~~~php
Route::get('/', function () {

    $tareas = [
        'Ir a la tienda',
        'Hacer los deberes de inglés',
        'Dar de comer a los peces'
    ];

    $titulo = 'Tareas Principales';

    return view('welcome')->withTareas($tareas)->withTitulo('titulo');
});
~~~

> /resources/views/wellcome.blade.php

~~~php
@extends('layout')

@section('contenido')
<h1>Página principal - {{ $titulo }}</h1>

<ul>
    @foreach ($tareas as $tarea)
        <li>{{ $tarea }}</li>
    @endforeach
</ul>

@endsection
~~~

> /routes/web.php

    return view('welcome', compact('tareas','titulo'));


# Controladores

> /routes/web.php

    Route::get('/', 'PaginasController@portada');


> php artisan make:controller PaginasController

> /app/Http/Controllers/PaginasController.php

~~~php
    class PaginasController extends Controller
    {
        public function portada() {
            $tareas = [
                'Ir a la tienda',
                'Hacer los deberes de inglés',
                'Dar de comer a los peces'
            ];

            $titulo = 'Tareas Principales';

            return view('welcome', compact('tareas','titulo'));
        }
    }
~~~

Ahora creamos dos rutas nuevas y dos funciones en el controlador para las otras dos vistas:

> /routes/web.php

    Route::get('/contacto', 'PaginasController@contacto');
    Route::get('/sobrenosotros', 'PaginasController@sobrenosotros');


> /app/Http/Controllers/PaginasController.php

~~~php
    class PaginasController extends Controller
    {       
        public function contacto() {
            return view('contact');
        }

        public function sobrenosotros() {
            return view('about');
        }
    }
~~~


# Migraciones y bases de datos

Creamos una base de datos

~~~mysql  
mariadb -u root -ptallerosl

CREATE DATABASE miprimerlaravel;

CREATE USER 'laravel'@'%' IDENTIFIED BY 'laravel';

GRANT ALL PRIVILEGES ON miprimerlaravel.* TO 'laravel'@'%' WITH GRANT OPTION;
~~~

En el fichero .env:

> /.env

Ahí se configura la conexión con la base de datos (además de otras muchas cosas)

El el fichero database, tenemos los

> /config/database.php

Parámetros por defecto para las conexiones con la base de datos

Las migraciones con como control de versiones para las bases de datos.
Guardamos la creación de las tablas y las alteraciones en la esctructuras 
de estas como migraciones diferentes conforme lo vayamos necesitando.
Si necesitamos tiempo después de haber creado una tabla un nuevo campo, en lugar de modificar
el fichero de migración de la tabla, creamos un nuevo fichero de migración para alterar la tabla
creando ese nuevo campo.

Principal beneficio: Si clonas el proyecto y ejecutas las migraciones, tienes una estructura de 
base de datos clonada sin tener que lanzar copias de seguridad y restauraciones de bases de datos.

Laravel te hace trasparente el acceso a la base de datos.

    php artisan migrate

Ejecuta las migraciones

    php artisan migrate:roolback 

Deshace la última migración creada. ¡¡Mucho cuidado!! Borra las tablas y los datos.

Si al hacer el migrate nos encontramos con este error:

~~~
  Illuminate\Database\QueryException  : SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes (SQL: alter table `users` add unique `users_email_unique`(`email`))
~~~

Tenemos que editar el fichero:

> /app/Providers/AppServiceProvider.php

~~~php
use Illuminate\Support\Facades\Schema;

public function boot()
{
    Schema::defaultStringLength(191);
}
~~~

Si al lanzar de nuevo el comando de migración nos dice que las tablas ya existen, hacemos un reseteo de la base de datos:

~~~bash
php artisan migrate:fresh
~~~

Revisamos el fichero de creación de la tabla de usuarios:

> /database/migrations/2014_10_12_000000_create_users_table.php

Probamos a cambiar la columna user por username

    php artisan help make:migration

Creamos una nueva migración para la tabla de proyectos

    php artisan make:migration create_proyectos_table

> /database/migrations/2018_11_22_195739_create_proyectos_table.php

~~~php  
public function up()
{
    Schema::create('proyectos', function (Blueprint $table) {
        $table->increments('id');
        $table->string('titulo');
        $table->string('descripcion');
        $table->timestamps();
    });
}
~~~

    php artisan migrate



# Eloquent y Namespacing

Convención: El modelo suele ir en singular sobre el nombre de la tabla que va en plural

    php artisan make:model Proyecto

Probamos con "tinker", el sistema de consola de Laravel:

    php artisan tinker

~~~php
    >>> \App\Proyecto::all();

    >>> \App\Proyecto::first();

    >>> $proyecto = new \App\Proyecto;
    >>> $proyecto->titulo = 'Mi primer proyecto';
    >>> $proyecto->descripcion = 'Lorem impsum';

    >>> $proyecto;

    >>> $proyecto->save();

    >>> \App\Proyecto::first();

    >>> $proyecto = new \App\Proyecto;
    >>> $proyecto->titulo = 'Mi segundo proyecto';
    >>> $proyecto->descripcion = 'Lorem impsum';

    >>> \App\Proyecto::all();
~~~

Las colecciones de Laravel son como Arrays de PHP con esteroides.

Ya tenemos el Modelo, la Vista y el Controlador.

> /routes/web.php

~~~php
Route::get('/proyectos', 'ProyectosController@index');
~~~

    php artisan make:controller ProyectosController

> /app/Http/Controllers/ProyectosController.php

~~~php
public function index() {
    return view('proyectos.index');
}
~~~

> /resources/views/proyectos/index.blade.php

~~~html
<html>
<head>
    <title></title>
</head>
<body>
    <h1>Proyectos</h1>
</body>
</html>
~~~

    php artisan serve

Comprobamos como se carga nuestra vista a través del controlador y la ruta. 

Ahora vamos a cargar los datos:

> /app/Http/Controllers/ProyectosController.php

~~~php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use \App\Proyecto;

class ProyectosController extends Controller
{
    public function index() {
        $proyectos = Proyecto::all();

        return view('proyectos.index', compact('proyectos'));
    }
}
~~~

> /resources/views/proyectos/index.blade.php

~~~html
<html>
<head>
    <title></title>
</head>
<body>
    <h1>Proyectos</h1>

    <ul>
        @foreach ($proyectos as $proyecto)
            <li>{{ $proyecto->titulo }}</li>
        @endforeach
    </ul>
</body>
</html>
~~~


