# Szanlony Blade

- [Wprowadzenie](#introduction)
- [Dziedziczenie szabkonów](#template-inheritance)
    - [Definiowanie Layoutu](#defining-a-layout)
    - [Rozbudowa Layoutu](#extending-a-layout)
- [Komponenty i Gniazda](#components-and-slots)
- [Wyświetlanie danych](#displaying-data)
    - [Blade i Frameworki JavaScript ](#blade-and-javascript-frameworks)
- [Struktury Kontrolne](#control-structures)
    - [If Statements](#if-statements)
    - [Switch Statements](#switch-statements)
    - [Loops](#loops)
    - [The Loop Variable](#the-loop-variable)
    - [Comments](#comments)
    - [PHP](#php)
- [Including Sub-Views](#including-sub-views)
    - [Rendering Views For Collections](#rendering-views-for-collections)
- [Stacks](#stacks)
- [Service Injection](#service-injection)
- [Extending Blade](#extending-blade)
    - [Custom If Statements](#custom-if-statements)

<a name="introduction"></a>
## Wprowadzenie

Blade jest prostym lecz potężnym narzędziem przetwarzania szablonów w Laravelu. W przeciwieństwie do innych popularnych systemów przetwarzania szablonówx, Blade nie ogranicza możliwości używania czystego kodu PHP w widokach. W rzeczywistości wszystkie widoki Blade są skompilowane do czytego kodu PHP i przechowywane w pamięci podręcznej, dopóki nie zostaną zmodyfikowane, co oznacza, że Blade nie dodaje dodatkowego narzutu (zero overhead) Twojej aplikacji. Bladowe pliki widoków używają rozszerzenia `.blade.php` i są zazwyczaj umieszczane w katalogu `resources/views`.

<a name="template-inheritance"></a>
## Dziedziczenie szabkonów

<a name="defining-a-layout"></a>
### Definiowanie Layoutu

Dwoma głównymi zaletami stosowania Blade są  _template inheritance_ and _sections_. Aby zacząć, przyjrzyjmy się prostemu przykładowi. Najpierw zbadamy układ strony głównej.
W przypadku gdy większość stron naszej aplikacji oparta jest o taki sam charakterystyczny układ, na różnych podstronach, wygodnie jest zdefiniowanie tego układu jako pojedynczego widoku blade:

    <!-- Stored in resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

Jak widzisz, ten plik zawiera typową składnię HTML. Należy jednak pamiętać o dyrektywach @section i @yield. Dyrektywa @section, jak sama nazwa wskazuje, definiuje sekcję zawartości, podczas gdy dyrektywa @yield jest używana do wyświetlania zawartości takiej sekcji.

Teraz, gdy mam zdefiniowany układ dla naszej aplikacji, określmy stronę podrzędną, która odziedziczy ten układ.

<a name="extending-a-layout"></a>
### Rozbudowa Layoutów

Podczas tworzenia widoku podrzędnego użyj dyrektywy Blade @extends, aby określić, jaki układ w tworzonym widoku ma zostać "odziedziczony". Widoki, podrzedne które rozszerzają głowny układ, mogą do niego wstrzyknąc zawierać swoich sekcji opisanych dyrektywami `@section`. Jak pamietasz z powyższego przykładu, zawartość tych sekcji zostanie wyświetlona w szablonie w miejscach oznaczonych dyrektywami @yield:

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

W tym przykładzie sekcja `sidebar` używa dyrektywy `@parent` aby dołączyć (bez nadpisania) swoją zawartość do sekcji sidebara szablonu. W miejscu gdzie jest zadeklarowana dyrektywa `@parent` zostanie umieszczona zawartość sekcji z głownego szablonu w chwili gdy szablon będzie renderowany.

> {wskazówka} W przeciwieństwie do uwcześniejszego przykładu, w widoku podrzędnym sekcja `sidebar` kończy się symbolem `@endsection` zamiast `@show` jak było to w szblnie nadrzędnym. Dyrektywa `@endsection` definiuje tylko sekcję, podczas gdy dyrektywa `@show` będzie definiować zawartość sekcji i jednocześnie ją wyświtlać (***coś jak `@endsection` i `@yield` w jednym***). 


Widoki Blade mogą być zwracaje w routingu z pomoca globalnego helpera `view`:

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## Komponenty i Gniazda

Komponenty i Gniazda zapewniają podobną funkcjonalność do sekcji i szablonów; jednakże ktoś może poszukiwać przykładów zastosowania komponentów i gniazd dla lepszego ich zrozumienia. Na wstępie, wyobraźmy sobie powtarzalny element "alert", który chcielibyśmy wielokrotnie wykorzystać na przestrzni całej naszej aplikacji:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

Zmienna `{{ $slot }}` będzie zawierać tę zawartość, którą będziemy chcieli wstrzyknąć do komponentu w chwili jego osadzania. Teraz utwórzmy w kodzie miejsce na ten komponent używajac dyrektywy`@component`:

    @component('alert')
        <strong>Whoops!</strong> Something went wrong!
    @endcomponent

Czasami bywa pomocne zdefiniowanie kilku odrębnych gniazd w komponencie. Zmodyfikujmy nasz kod alertu, aby umożliwić wstawienie "tytułu". Ponazywane gniazda mogą być prosto wyświetlane przez zmiene odwołujące się do ich nazw:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

Teraz możemy zadeklarować wstrzykiwaną zawartość do uprzednio przygotowanego gniazda, używajac dyrektywy `@slot` opatrzonej  nazwą. Natomiast każda zawartość, która nie znajdzie się w dyrektywie `@slot` będzie przekazwyana wprost do gniazda określanego zmienną `$slot`:

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        You are not allowed to access this resource!
    @endcomponent

#### Przekazywanie Dodatkowych Danych do Komponentów

Czasami możesz potrzbowac przekazac dodakowe dane do komponnetu. W tym celu możesz przekazać tablicę jako drugi argument dyrektywy @component. Wszystkie dane zostaną będą dostępne w szablonu komponentów jako zmienne:

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

<a name="displaying-data"></a>
## Wyświetlanie Danych

Możesz wyświetlić dane przekazywane fo twojego widoku Blade przez owinięcie zmiennej w nawiasy kwadratowe. Dla przykładu, weźmy nastepujacy routing:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

Możesz wyświetlić zawartość zmiennej `name` w taki sposób:

    Hello, {{ $name }}.

Oczywiście, nie jesteś ogranicznay nie jest ograniczony do wyświetlania jedynie zawartości zmiennych przekazanych do widoku. Możesz również wyświetlać wynik dowolnej funkcji PHP. W rzeczywistości można umieścić dowolny kod PHP jaki zechcesz, osadzajac go w szablonu Blade w deklaracji `echo` oznaczanej podwójnymi nawiasami klamrowymi `{{ }}`:

    The current UNIX timestamp is {{ time() }}.

> {wskazówka} W Blade zawartość `{{ }}` jest automatycnzie przekazywana przez PHP-owa funkcję `htmlspecialchars` aby zabespieczyć cię przed atakami XSS.

#### Wyświetlanie Nieprzefiltrowanych Danych

Domyłśnie, w Blade zawartość `{{ }}`  jest automatycnzie przekazywana przez PHP-owa funkcję `htmlspecialchars` aby zabespieczyć się przed atakami XSS. jeśłi jednak nie chcesz swoich danych wyjściowych filtrować, mozesz użyć następujacej składi:

    Hello, {!! $name !!}.

> {wskazówka} Zachowaj ostrożność gdy wyświetlasz treści przekazywane przez użytkowników. Wtedy zawsze używaj wyników filrowanych otrzymywanych w wyniku użycia notacji z podwójnymi nawiasami klamrowymi.

<a name="blade-and-javascript-frameworks"></a>
### Blade i Frameworki JavaScript 

Ponieważ wiele frameworków JavaScript używa klamrowych nawiasów, do identyfikacji dynamicznych wyrażeń przetwarzanych przez przeglądarki, możesz użyć symbolu @, aby poinformować silnik renderujący Blade, że wyrażenie to powinno pozostać nieprzetworzone. Na przykład:

    <h1>Laravel</h1>

    Hello, @{{ name }}.

W tym przykładzie, stymbol `@` będzie usuniety przez parser Blade, jednak samo wyrażenie `{{ name }}` pozostanie bez zmian, pozwalając na jego przetworzenie przez framework JavaScriptu.

#### Dyrektywa `@verbatim` 

Jeśli musisz wyświetlić większu fragment JavaScriptu w obszarze twojego szablonu, możesz objąć cały tej wycinek HTML w dyrektywę `@verbatim`, co zwolni cię z konieczności poprzedzania każdej zmiennej w nawiasach klamrowych symbolem: `@`:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## Struktury Kontrolne

Oprócz dziedziczenia szablonów i wyświetlania danych, Blade zapewnia także wygodne skróty dla podstawowych struktur kontroli PHP, takich jak instrukcje warunkowe i pętle. Te skróty zapewniają bardzo czysty, zwiezły sposób pracy z strukturami kontroli znanymi z PHP.

<a name="if-statements"></a>
### Instrukcja If 

Możesz zbudować instrukcję warunkową `if` używając dyrektyw `@if`, `@elseif`, `@else`, i `@endif`. Te dyrektywy działają identycznie jak ich odpowiedniki PHP:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

Dla wygody, Blade oferuje ponadto dyrektywę `@unless`:

    @unless (Auth::check())
        You are not signed in.
    @endunless

Oprócz wymienionych dyrektyw warunkowych, dyrektywy `@isset` i `@empty` można używać jako wygodne skróty dla ich  odpoiwdników w PHP:

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### Authentication Shortcuts

Dyrektywy `@auth` i `@guest` mogą być użyte do szybkiego określenia czy obecny użytkowik jest autoryzowany czy pozostaje  tylko gościem:

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

<a name="switch-statements"></a>
### Instrukcje Switch

Instrukcje switch mogą być konstruowane w oparciu o dyrektywy: `@switch`, `@case`, `@break`, `@default` i `@endswitch`:

    @switch($i)
        @case(1)
            First case...
            @break

        @case(2)
            Second case...
            @break

        @default
            Default case...
    @endswitch

<a name="loops"></a>
### Pętle

Oprócz instrukcji warunkowegych, Blade oferuje proste instrukcje do pracy z strukturami pętli znanymi z PHP. Ponownie, każda z tych dyrektyw działa identycznie jak ich odpowiednik w PHP:

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

> {wskazówka} Podczas wykonywania pętli możesz użyć [predefiniowanych zmiennych pęti](#the-loop-variable) aby uzyskać dodatkowe informacje, np. czy pętla znajduje się w pierwszej lub ostatniej itercji.

Kiedy używasz petki możesz również przerwać proces lub przeskoczyć bieżącą iterację:

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

Możesz również wprowadzić warunki określające kiedy dana dyrektywa ma zastosowanie:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### Zmienne Pętli

Kiedy przetwarzasz petlę, zmienna`$loop` będzie  dostepna wewnatrz twojej pętli. Ta zmienan pozwala na dostęp do kilkku użytecznych informacji takich jak obecny index iteracji, i czy jest to pierwsza lub ostatnia itercja tej pętli:

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

Jeżeli wykonujesz pętlę zagnieżdżoną, możesz mieć dostep do nadrzednej zemiennej `$loop` przez właściwość `parent`:

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach

Zmienna `$loop` może przekazywać różne użyteczne własciwości:

Właściwość  | Opis
------------- | -------------
`$loop->index`  |  Index obecnej iteracji (poczatek od 0).
`$loop->iteration`  |  Obecna interacja (poczatek od 1).
`$loop->remaining`  |  Ilość iteracji pozostających do dokończenia pętli.
`$loop->count`  |  Ilość wszytkich elementó tablicy.
`$loop->first`  |  Informacja czy jest to pierwsza iteracja w pętli.
`$loop->last`  |  Informacja czy jest to ostania iteracja w pętli.
`$loop->depth`  |  Poziom zagnieżdżenia obecnej pętli.
`$loop->parent`  |  Dostęp do zmiennych z pętli nadrzędnej z pozycji  pętli zagieżdzonej.

<a name="comments"></a>
### Comments

W szablonach Blade masz również możlwiość tworzenia komentarzy. Jednak w odrózniwniu od tradycyjnych komentarzy w dokumentach HTML, komentarze Blade nie dą dołączane do wynikowgo kodu HTML twojej apliakcji:

    {{-- This comment will not be present in the rendered HTML --}}

<a name="php"></a>
### PHP

W niektórych sytuacjach istnieje potrzeba umieszczenia kodu PHP w widokach. Możesz użyć dyrektywy `@php` do wykonania zwykłego bloku PHP w twoim szblonie:

    @php
        //
    @endphp

> {wskazówka} Chociaż Blade udostępnia tę funkcję, używanie jej często może być sygnałem, że zbyt dużo logiki umieszczasz w szablonie.

<a name="including-sub-views"></a>
## Dołączanie Sub-Widoków

Dyrektywa `@include` pozwala ci na  dołączenie widoku w innym widoku. Wszystkie zmienne, które są dostępne dla widoku nadrzędnego, zostaną udostępnione do dołączonego widoku:

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

Mimo, że widok włączony dziedziczy wszystkie dane dostępne w widoku nadrzędnym, można do niego przekazać również szereg dodatkowych danych w postaci tablicy:

    @include('view.name', ['some' => 'data'])

Oczywiście, jeśli próbujesz dołączyć dyrektywą `@include` widok, który nie istnieje, Laravel zgłosi błąd. Jeśli istnieje potrzba uwzględnienia widoku, który może nie być dostępny, powinieneś użyć dyrektywy warunkowej `@includeIf`:

    @includeIf('view.name', ['some' => 'data'])

Jeśli chcesz, dołaczyć dyrektywą `@include` widok w zależności od jakiegoś warunku, możesz użyć dyrektywy @includeWhen:

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

> {uwaga} Powinieneś unikać stałych `__DIR__` i `__FILE__` w widokach Blade, ponieważ odnoszą się one do lokalizacji buforowanego, skompilowanego widoku. 

<a name="rendering-views-for-collections"></a>
### Rendering Views For Collections

You may combine loops and includes into one line with Blade's `@each` directive:

    @each('view.name', $jobs, 'job')

The first argument is the view partial to render for each element in the array or collection. The second argument is the array or collection you wish to iterate over, while the third argument is the variable name that will be assigned to the current iteration within the view. So, for example, if you are iterating over an array of `jobs`, typically you will want to access each job as a `job` variable within your view partial. The key for the current iteration will be available as the `key` variable within your view partial.

You may also pass a fourth argument to the `@each` directive. This argument determines the view that will be rendered if the given array is empty.

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} Views rendered via `@each` do not inherit the variables from the parent view. If the child view requires these variables, you should use `@foreach` and `@include` instead.

<a name="stacks"></a>
## Stacks

Blade allows you to push to named stacks which can be rendered somewhere else in another view or layout. This can be particularly useful for specifying any JavaScript libraries required by your child views:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

You may push to a stack as many times as needed. To render the complete stack contents, pass the name of the stack to the `@stack` directive:

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## Service Injection

The `@inject` directive may be used to retrieve a service from the Laravel [service container](/docs/{{version}}/container). The first argument passed to `@inject` is the name of the variable the service will be placed into, while the second argument is the class or interface name of the service you wish to resolve:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## Extending Blade

Blade allows you to define your own custom directives using the `directive` method. When the Blade compiler encounters the custom directive, it will call the provided callback with the expression that the directive contains.

The following example creates a `@datetime($var)` directive which formats a given `$var`, which should be an instance of `DateTime`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

As you can see, we will chain the `format` method onto whatever expression is passed into the directive. So, in this example, the final PHP generated by this directive will be:

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} After updating the logic of a Blade directive, you will need to delete all of the cached Blade views. The cached Blade views may be removed using the `view:clear` Artisan command.

<a name="custom-if-statements"></a>
### Custom If Statements

Programming a custom directive is sometimes more complex than necessary when defining simple, custom conditional statements. For that reason, Blade provides a `Blade::if` method which allows you to quickly define custom conditional directives using Closures. For example, let's define a custom conditional that checks the current application environment. We may do this in the `boot` method of our `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

Once the custom conditional has been defined, we can easily use it on our templates:

    @env('local')
        // The application is in the local environment...
    @else
        // The application is not in the local environment...
    @endenv
