---
title: "[OLD] SpotiBar: ANGULARJS"
date: 2018-04-13T11:02:00-04:00
toc: true
toc_label: "Table of Contents"
categories:
  - blog
tags:
  - coding
---

[This is a post from my old website. Outdated packages and libraries. Viewer discretion is advised ;-)]

## PART 1

Zgodnie z poprzednim wpisem, rozbierzemy SpotiBar w wersji JavaScript na czynniki pierwsze. Użyjemy AngularJS, JavaScript i API Spotify.

Jako że to pierwszy kontakt z JavaScriptem jako takim i choć budowana przez nas aplikacja pewnie nie spełnia kryteriów dobrego kodu, to przynajmniej pokazuje, że mimo [Mnogości](https://risingstars.js.org/2017/en/) JS-owych frameworków i możliwości, rozpoczęcie zabawy w developerkę nie jest trudne.

### Użyte frameworki JavaScript i narzędzia

* JavaScript
* [AngularJS](https://angularjs.org) 1.6.x (na ten moment jest to wersja 1.6.9)
* Kilka paczek Angularowych: [angularjs-slider](https://github.com/angular-slider/angularjs-slider), [checklist-model](https://vitalets.github.io/checklist-model/), [angular-spotify](https://github.com/eddiemoore/angular-spotify)
* [Konto developerskie na Spotify](https://developer.spotify.com/)
* Terminal, przeglądarka i edytor tekstu

### Ustawiamy workflow

Potrzebujemy małego serwera, żeby obsługiwać zwroty z serwera Spotify. Na Macu sprawa jest prosta, bo mamy preinstalowanego Pythona. Serwer lokalny odpalamy komendą 'python -m SimpleHTTPServer 8888', gdzie pod 8888 możemy wstawić interesujący nas port.

![image-center](/assets/images/oldblog/Screen-Shot-2018-04-13-at-08.55.48.png){: .align-center}

Alternatywne rozwiązania:

* [atom-live-server](https://atom.io/packages/atom-live-server), dla użytkowników edytora [Atom](https://atom.io/)
* [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer), dla użytkowników edytora [Visual Studio Code](https://code.visualstudio.com/) (pomimo kilkudniowych prób, nie udało mi się za jego pomocą obsłużyć callbacku od Spotify)
* [MAMP](https://www.mamp.info/en/) i podobne

Jeżeli używamy sposobu terminalowego, to trzeba pamiętać o odpaleniu komendy w katalogu naszego projektu, a nie np. w katalogu domowym.

Mając już prosty serwer możemy stworzyć folder projektu z podfolderami: `app` (w nim folder `lib`) i `views`. Struktura projektu jest konwencją, ale że ze ścieżek do plików będziemy korzystać, to warto pamiętać, gdzie jest jaki plik.

Ostatni krok to zarejestrowanie aplikacji w konsoli Spotify. Wchodzimy w [My Apps](https://developer.spotify.com/my-applications/) i po zalogowaniu tworzymy nową aplikację.

![image-center](/assets/images/oldblog/spotify_create_new_app.jpg){: .align-center}

Następnie w jej ustawieniach dodajemy adres do callbacku, który będziemy potrzebować, by obsłużyć autoryzację ze strony Spotify. (możemy wpisać adres serwera lokalnego, pamiętając jednak, by kierował on na ten port, na który mamy ustawiony serwer).

![image-center](/assets/images/oldblog/spotify_callback_url.jpg{: .align-center}

Na koniec kwestia bibliotek Angularowych. Można je ściągnąć z podlinkowanych wcześniej stron. Na repozytorium aplikacji też są wrzucone, więc równie dobrze można je pobrać [stąd](https://github.com/MiWy/spotify-recommendations-app-js/tree/gh-pages/app/lib). Obojętnie skąd je weźmiemy, mają wylądować w katalogu `app/lib`.

## PART 2

Drogą przypomnienia, mamy przygotowane biblioteki JavaScriptowe, strukturę folderów projektu(*) utworzoną aplikację w konsoli dewelopera Spotify i serwer. Odpalamy serwer w katalogu naszego projektu.

***Oprócz wspomnianych w poprzednim wpisie plików, skopiować można również [ten](https://github.com/MiWy/spotify-recommendations-app-js/blob/gh-pages/app/style.css) plik CSS. Wtedy na ekranie powinno się pojawiać dokładnie to samo :).**

### Bazowy plik index.html

Użytkownik naszej prostej aplikacji będzie de facto przemieszczać się pomiędzy dwoma stronami: na jednej będzie ustalać kryteria wyszukiwania rekomendacji, na drugiej zaś wyświetlane będą wyniki. Nie będziemy do tego celu tworzyć osobnych pełnoprawnych plików HTML, tylko skorzystamy z obecnej w AngularJS możliwości dynamicznego podmieniania elementów HTML.

Naszym bazowym plikiem będzie umieszczony w wyjściowym folderze naszego projektu plik `index.html`. Pełen kod źródłowy znajduje się [w repozytorium](https://github.com/MiWy/spotify-recommendations-app-js/blob/gh-pages/index.html), tutaj opiszę tylko kluczowe kwestie.

#### <html>

Oznaczenie Angularowe pojawia się już w pierwszym znaczniku:

```html    
    <html ng-app="spotiBar">
```

`ng-app` mówi Angularowi jaki jest najbardziej podstawowy element naszej aplikacji, rejestruje ją dla frameworku. Równie dobrze można w tym przypadku dać ten atrybut w elemencie `body`.

#### <head>

W elemencie `<head>` mamy zbiór odwołań do plików .css i .js. Niektóre z nich mają standardową strukturę, przykładowo dla elementu `script` jest podana ścieżka do pliku w określonej lokalizacji na dysku. Dla celów pokazowych, nie wszystkie pliki są tak ładowane :).  Na przykład pliki Bootstrapowe są ładowane z serwerów Bootstrapa, nie z dysku.

Ma to swoje plusy i minusy, z jednej strony jest szansa, że takie korzystanie z [CDN](https://gtmetrix.com/why-use-a-cdn.html) (_content delivery network_) pozwoli na szybsze załadowanie strony. Z drugiej strony, jak serwery, na których jest dany plik umieszczony, się posypią, to nasza strona też może ucierpieć.

Ostatnią sprawą jest znacznik `<base />`. Ułatwia on życie, pozwalając na zdefiniowanie adresu bazowego naszej strony internetowej. Pozostawienie go na `/` oznacza, że podstrony będą się ładować na zasadzie:

www.example.com/podstrona

Zaś ustawienie `<base href="/bazowylink/"/>` spowoduje, że strony będą szukane pod:

www.example.com/bazowylink/podstrona


```html   
    <head>
      <title>SpotiBar</title>
      <base href="/">
    
      <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" 
        integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm"
        crossorigin="anonymous">
      <link href="https://fonts.googleapis.com/css?family=Lobster" rel="stylesheet">
      <link href="https://fonts.googleapis.com/css?family=Cabin" rel="stylesheet">
      <link rel="stylesheet" href="app/style.css">
      <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" 
        integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN"
        crossorigin="anonymous"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" 
        integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q"
        crossorigin="anonymous"></script>
      <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" 
        integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl"
        crossorigin="anonymous"></script>
      <script src="app/lib/angular.min.js"></script>
      <script src="app/lib/angular-route.min.js"></script>
      <script src="app/lib/checklist-model.js"></script>
      <script src="app/lib/angular-spotify.min.js"></script>
      <link rel="stylesheet" type="text/css" href="app/lib/rzslider.min.css" />
      <script src="app/lib/rzslider.min.js"></script>
      <script src="app/app.js"></script>
    </head>
```

#### <body>

W tym momencie zrobimy tylko pasek nawigacyjny z przyciskiem do logowania.

Kod wygląda tak:

```html    
    <nav ng-controller="LoginController" class="navbar navbar-inverse altnavbg" role="navigation">
      <div class="container-fluid">
        <div class="navbar-header">
          <a class="navbar-brand" href="/#!/search">SpotiBar - Advanced Spotify Search</a>
        </div>
        <ul class="nav navbar-nav navbar-right">
          <button type="button" class="btn btn-default navbar-btn" ng-click="login()">Login</button>
        </ul>
      </div>
    </nav>
```

Prawie wszystkie atrybuty elementów są Bootstrapowe i nie będziemy ich w tym miejscu tłumaczyć. Dokumentacja Bootstrapa jest przyjazna użytkownikowi :). Natomiast kilka rzeczy jest Angularowych. Atrybut `ng-controller` jest dyrektywą Angulara. Określa jaki fragment kodu będzie obsługiwać wskazaną część strony.

Warta zwrócenia uwagi jest też wartość atrybutu `href` dla naszego `navbar-brand`, czyli `/#!/search`. `#!`  bierze się stąd, że delegujemy przekierowania Angularowi, a on domyślnie wykorzystuje to wyrażenie. Jak zobaczymy w kolejnej części, link ten nie tyle ładuje zupełnie nową stronę HTML, co mówi Angularowi, aby zmodyfikował DOM i pokazał opcje wyszukiwania na aktualnie wyświetlanej stronie (czyli index.html). [Co to DOM](https://www.digitalocean.com/community/tutorials/introduction-to-the-dom)?

Ostatnia sprawa to dyrektywa `ng-click`. Znowu Angular, wykorzystamy ją, by zarejestrować chęć zalogowania się przez użytkownika. O różnicy pomiędzy `ng-click` a `onclick` można poczytać na [stackoverflow](https://stackoverflow.com/a/20274848).

### Autoryzacja OAuth

Do korzystania z API Spotify użytkownik musi przejść przez proces [autoryzacji](https://developer.spotify.com/web-api/authorization-guide/). Uwierzytelnienie polega na wysłaniu żądania wraz z kluczem naszej aplikacji i odebraniu tokena zwrotnego. Token musi być następnie dołączany do wysyłanych przez użytkownika kolejnych zapytań do API.

Do komunikacji z API skorzystamy z przygotowanego pod AngularJS [angular-spotify](https://github.com/eddiemoore/angular-spotify).

#### callback.html

Potrzebujemy prostego pliku HTML, do którego Spotify będzie odsyłać token po skończonej autoryzacji. Autor angular-spotify dostarcza templatkę takowego pliku, [o tutaj](https://github.com/eddiemoore/angular-spotify/blob/master/examples/callback.html). Wystarczy nam ona w zupełności. Kopiujemy ją do głównego folderu naszego projektu.

Autoryzacja przebiega żądaniem GET, więc ten mały skrypcik w pliku `callback.html` patrzy po pasku adresowym okna i, jeżeli go znajdzie, zapisuje token w pamięci przeglądarki (`localStorage`) i zamyka okno, a jeśli nie to tylko zamyka okno.

W poprzednim wpisie ustawialiśmy **Redirect URI** dla naszej aplikacji Spotify (w [konsoli dewelopera](https://beta.developer.spotify.com/dashboard)). Jeżeli zmienimy nazwę `callback.html` na coś innego, to musimy również w ustawieniach aplikacji zmienić przekierowanie.

#### app.js

Zakładając, że w naszym pliku `index.html` mamy wszystko to, co powyżej opisane, tworzymy nowy plik: `app.js` w folderze `app`.

W pierwszej kolejności, musimy stworzyć instancję naszej aplikacji (w zasadzie - moduł) dla AngularJS. Składnia wygląda tak:

```javascript    
    const spotiBar = angular.module("spotiBar", [
      "spotify"
    ]);
```

O ile w pliku `index.html` nadaliśmy wartość `spotiBar` atrybutowi `ng-app`, to tutaj musi się pojawić ta sama nazwa.

W nawiasie kwadratowym wymieniamy paczki, z których będziemy korzystać. Póki co będziemy korzystać tylko z "spotify" właśnie :). Resztę dodamy później.

Następnie czeka nas odrobinę konfiguracji:

```javascript
    spotiBar.config([
      "SpotifyProvider",
      function(SpotifyProvider) {
        
        SpotifyProvider.setClientId("CLIENT_ID");
        SpotifyProvider.setRedirectUri("http://localhost:8080/callback.html");
        SpotifyProvider.setScope("playlist-read-private");
    
        if (localStorage.getItem("spotify-token")) {
          SpotifyProvider.setAuthToken(localStorage.getItem("spotify-token"));
          console.log("Token got from localStorage.");
        } else {
          console.log("There was no token in localStorage.");
        }
      }
    ]);
```

W nawiasie kwadratowym wymieniamy, ponownie, zależności (w tym wypadku `SpotifyProvider` z angular-spotify), a następnie implementujemy funkcję konfiguracyjną.

Pod `CLIENT_ID` podajemy client id wzięty ze strony naszej aplikacji w konsoli dewelopera Spotify (**NIE** secret, tylko "zwykły").

Pod `.setRedirectUri`wklepujemy adres do callbacku, zgodny z naszym serwerem lokalnym, nazwą pliku callbackowego, oraz zgodny z tym, który wpisaliśmy w ustawieniach aplikacji w konsoli Spotify*****.

Pod `.setScope` definiujemy jakie upoważnienia użytkownik naszej aplikacji ma nam nadać (dla celów przykładowych u nas jest to odczytanie jego prywatnych playlist). Pełny zestaw scope'ów znaleźć można [tutaj](https://developer.spotify.com/web-api/using-scopes/).

Wyrażenie warunkowe `if` sprawdza, czy w pamięci lokalnej przeglądarki jest już token. Jeśli tak, zapisujemy go do `SpotifyProvidera` i dostajemy zwrotny komunikat w konsoli. Jeśli nie, to czeka nas tylko komunikat zwrotny w konsoli ;).

Ok. Brakuje nam tylko jednej rzeczy: kodu kontrolera, odpowiedzialnego za logowanie użytkownika!

```Javascript
    spotiBar.controller("LoginController", [
      "$scope",
      "Spotify",
      function($scope, Spotify) {
        $scope.login = function() {
          Spotify.login().then(
            function(data) {
              console.log(data);
              Spotify.setAuthToken(data);
              alert("You are now logged in");
            },
            function() {
              console.log("didn't log in");
            }
          );
        };
      }
    ]);
```

Podobnie jak poprzednio, na początku nawiasu kwadratowego wymieniamy potrzebne zależności (`$scope` i `Spotify`). `$scope` to obiekt w Angular pozwalający na komunikację pomiędzy HTML a JS.

Czyli pisząc `$scope.login =`, definiujemy funkcję, do której w pliku `index.html` odwoływaliśmy się pisząc `ng-click=login()`.

Funkcja `Spotify.login()`, z kolei, to metoda z angular-spotify, wysyłająca żądanie autoryzacji do Spotify. Po zakończeniu autoryzacji, o ile zakończyła się sukcesem, zapisuje informację o tokenie w obiekcie Spotify, którym później będziemy wyszukiwać rekomendacji muzycznych. Jest też wyświetlany alert z informacją o logowaniu.


## Logujemy się


Jeśli nasz serwer nadal jest włączony to wchodzimy na nasze http://localhost:8888 (ew. inny port) i patrzymy. Zakładając, że mamy te same pliki CSS, powinno to wyglądać tak:

![image-center](/assets/images/oldblog/Screen-Shot-2018-04-14-at-17.58.32.png){: .align-center}

Gdy klikniemy w przycisk "Login", wyskoczy okienko proszące o potwierdzenie (ew. zalogowanie do Spotify, jeśli nie byliście w przeglądarce zalogowani). Po jego kliknięciu, przekierowanie pchnie token do callback.html, okienko zniknie, a nam ukaże się:

![image-center](/assets/images/oldblog/Screen-Shot-2018-04-14-at-17.58.59.png){: .align-center}

Token ma termin ważności, gdy się wyczerpie, będziemy musieli zalogować się ponownie (inne metody autoryzacji pozwalają na odświeżanie tokenu, w angular-spotify tego nie ma).

Następnym razem zajmiemy się stworzeniem widoku z kryteriami wyszukiwania :).

(*) Jeśli okienko z autoryzacją się samo nie zamyka, tylko pojawia się błąd 404 albo informacja o złym redirect_uri, to trzeba pokombinować trochę z ustawieniem adresu callbacku w konsoli dewelopera Spotify, i ustawieniami w naszej konfiguracji `SpotifyProvider`. Raz mi działa, kiedy podaję bezpośrednio z końcówką ".html", a raz bez. ... ;)

## PART 3

W tym odcinku dodamy do naszej aplikacji formularz, zbierający kryteria wyszukiwania oraz widokiem wyświetlającym rezultaty.

### index.html

Jako, że chcemy by Angular sam uzupełniał DOM o potrzebne elementy HTML, musimy oznaczyć miejsce, w którym ma to robić. Służy temu dyrektywa `ng-view`.

W pliku `index.html` pod elementem `<nav></nav>` dodajemy:

```html    
    <main ng-view>
```

### app.js (2)

By scedować zarządzanie przekierowaniami na stronie Angularowi musimy skonfigurować moduł AngularJS, który domyślnie nie jest ładowany: ngRoute.

Zmodyfikujemy w tym celu pierwszą linijkę pliku app.js, w której inicjowaliśmy naszą aplikację:

```javascript    
    const spotiBar = angular.module("spotiBar", [
      "ngRoute",
      "checklist-model",
      "rzModule",
      "spotify"
    ]);
```

Oprócz ngRoute dodaliśmy od razu [checklist-model](https://vitalets.github.io/checklist-model/) oraz [rzModule](https://github.com/angular-slider/angularjs-slider), czyli dodatkowe biblioteki, które ułatwią nam pracę z formularzami.

ngRoute trzeba stosownie skonfigurować, do naszej funkcji konfigurującej dodajemy:

```javascript    
    spotiBar.config([
      "$routeProvider",
      "SpotifyProvider",
      function($routeProvider, SpotifyProvider) {
        $routeProvider
          .when("/", {
            redirectTo: "/search"
          })
          .when("/search", {
            templateUrl: "views/search.html",
            controller: "SearchController"
          })
          .when("/results", {
            templateUrl: "views/results.html",
            controller: "ResultsController"
          });
        // Poniżej reszta z poprzednich części: SpotifyProvider.setClientId()....
        // A także sprawdzenie obecności tokena w localStorage: if(localStorage.getItem())....
      }
    ]);
```

Dzięki temu, kiedy użytkownik kliknie link odsyłający do, dajmy na to, `/search`, Angular będzie wiedział jaki fragment HTML ma dodać w miejsce naszego `ng-view` w `index.html` (będzie to fragment zlokalizowany w `views/search.html`) oraz jaki kontroler jest tam używany (`SearchController`).

Przypadek `.when("/", {})` to przypadek wejścia na stronę główną. Również wtedy Angular ma załadować to samo, co w momencie przekierowania na `/search`.

### search.html

Tworzymy plik `search.html` w folderze `/views`.

Kod formularza wyszukiwania jest dosyć długi, gdyż jest wiele kryteriów, które użytkownik może ustawić.

```html
    <div class=" row justify-content-center">
            <button type="button" class="btn-primary" ng-click="searchRecommendations()">Search recommendations</button>
    </div>
    <div class="container row">
        <div class="col-md-1"></div>
        <div class="col-md-4">
          <div class="form-group ">
            <form>
              <input type="submit" ng-click="searchSeeds()" class="btn-warning" value="Search for seeds">
                <input type="text"  ng-model="seedsQuery" placeholder="Enter seeds query"/>
              <label class="spotify-font-color">Limit (1-100):
                <input type="number" min="1" max="100" ng-model="limit" />
              </label>
              <br />
              <label class="spotify-font-color">Duration (in seconds):
                <input type="number" ng-model="duration" />
              </label>
              <br />
              <label class="spotify-font-color">Key (0 = C, 2 = D, and so on):
                <input type="number" min="0" max="11" value="0" ng-model="key" />
              </label>
              <br />
              <label  class="spotify-font-color">Major/Minus (1/0):
                <input type="number" min="0" max="1" value="0" ng-model="ismajor" />
              </label>
              <br />
              <label  class="spotify-font-color">Tempo (BPM):
                <input type="number" min="0" max="1000" value="120" ng-model="tempo" />
              </label>
              <br />
    
              <div>
                <label class="spotify-font-color">Acousticness:
                  <input type="text" ng-model="sliderAcousticness" />
                </label>
                <br/>
                <rzslider rz-slider-model="sliderAcousticness" rz-slider-options="{floor: 0, ceil: 100, step: 1, showSelectionBar: true}"></rzslider>
              </div>
              <div>
                <label class="spotify-font-color">Danceability:
                  <input type="text" min="0" max="100" ng-model="sliderDanceability" />
                </label>
                <rzslider rz-slider-model="sliderDanceability" rz-slider-options="{floor: 0, ceil: 100, step: 1, precision: 1}"></rzslider>
              </div>
              <div>
                <label class="spotify-font-color">Energy:
                  <input type="text" min="0" max="100" ng-model="sliderEnergy" />
                </label>
                <rzslider rz-slider-model="sliderEnergy" rz-slider-options="{floor: 0, ceil: 100, step: 1, precision: 1}"></rzslider>
              </div>
              <div>
                <label class="spotify-font-color">Liveness:
                  <input type="text" min="0" max="100" ng-model="sliderLiveness" />
                </label>
                <rzslider rz-slider-model="sliderLiveness" rz-slider-options="{floor: 0, ceil: 100, step: 1, precision: 1}"></rzslider>
              </div>
              <div>
                <label class="spotify-font-color">Instrumentalness:
                  <input type="text" min="0" max="100" ng-model="sliderInstrumentalness" />
                </label>
                <rzslider rz-slider-model="sliderInstrumentalness" rz-slider-options="{floor: 0, ceil: 100, step: 1, precision: 1}"></rzslider>
              </div>
              <div>
                <label class="spotify-font-color">Popularity:
                  <input type="text" min="0" max="100" ng-model="sliderPopularity" />
                </label>
                <rzslider rz-slider-model="sliderPopularity" rz-slider-options="{floor: 0, ceil: 100, step: 1, precision: 1}"></rzslider>
              </div>
              <div>
                <label class="spotify-font-color">Speechiness:
                  <input type="text" min="0" max="100" ng-model="sliderSpeechiness" />
                </label>
                <rzslider rz-slider-model="sliderSpeechiness" rz-slider-options="{floor: 0, ceil: 100, step: 1, precision: 1}"></rzslider>
              </div>
              <div>
                <label class="spotify-font-color">Valence:
                  <input type="text" min="0" max="100" ng-model="sliderValence" />
                </label>
                <rzslider rz-slider-model="sliderValence" rz-slider-options="{floor: 0, ceil: 100, step: 1, precision: 1}"></rzslider>
              </div>
            </form>
          </div>
        </div>
    
        <div class="col-md-7">
          <div class="form-group">
          </div>
          <div class="container row">
            <div class="col-md-4">
              <p class="spotify-font-color">Artists:</p>
              <div class="row" ng-repeat="artist in seedsArtists">
                <input type="checkbox" checklist-model="seedsSelection.artists" checklist-value="artist.id" /> 
                <span class="spotify-font-color">{{artist.name}}</span>
              </div>
            </div>
            <div class="col-md-6">
              <p class="spotify-font-color">Tracks:</p>
              <div class="row" ng-repeat="track in seedsTracks">
                <input type="checkbox" checklist-model="seedsSelection.tracks" checklist-value="track.id" /> 
                <span class="spotify-font-color">{{track.name}} -</span>
                <span class="spotify-font-color" ng-repeat="artist in track.artists"> &nbsp;{{artist.name}} </span>
              </div>
            </div>
            <div class="col-md-2">
              <p class="spotify-font-color">Genres:</p>
              <div class="row" ng-repeat="genre in seedsGenres">
                <input type="checkbox" checklist-model="seedsSelection.genres" checklist-value="genre" /> 
                <span class="spotify-font-color">{{genre}}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
```

Za rozpoczęcie wyszukiwania na bazie ustalonych w formularzu przez użytkownika kryteriów odpowiada obecny w 2 linijce kodu element `<button>`, z atrybutem `ng-click="searchRecommendations()"`, odwołującym się do funkcji w kontrolerze `SearchController`, którą zaraz napiszemy.

Analogicznie, w linijce 9, mamy `input` z` ng-click="searchSeeds()"`. Jednym z kryteriów wyszukiwania rekomendacji w Spotify są artyści, utwory i gatunku muzyczne. Funkcja `searchSeeds` posłuży nam do wygenerowania z nimi listy na podstawie hasła wpisanego przez użytkownika.

Wszystkie `ng-model` informują Angular, pod jaką nazwą ma trzymać dane zebrane w danym polu formularza. Wszystkie dane zebrane z poziomu HTML są w AngularJS trzymane w obiekcie `$scope`, do którego będziemy się odwoływać implementując nasz kontroler.

`rz-slider-model`, `checklist-model`,  - te atrybuty definiują, analogicznie do `ng-model,` pod jaką nazwą właściwości `$scope` Angular ma trzymać dane ze sliderów i checkboxów (i pochodzą z wcześniej przez nas dodanych do naszej aplikacji bibliotek).

`rz-slider-options` definiuje dodatkowo ustawienia slidera. W kodzie ustawiony jest zakres wartości oraz stopień minimalnej zmiany wartości.

`ng-repeat` to dyrektywa pozwalająca nam na dynamiczne tworzenie iteracji danych elementów HTML, czyli po prostu w pętli. Przykładowo w linijce 93, jeśli w `seedsArtist` jest 100 artystów, to cały element `<div>` (wraz z wewnętrzną strukturą) będzie wygenerowany 100 razy.

`checklist-value` ustanawia wartość checkboxa, czyli jak użytkownik zaznaczy dany checkbox, to ten atrybut mówi naszej aplikacji, co ma trzymać w pamięci.

Z kolei zapis taki, jak w linijce 95: `{{artist.name}}`, podwójne klamry, to składnia Angulara, wiążąca dane ukryte pod `artist.name` (siedzące naturalnie w obiekcie `$scope`) tak, że na naszej stronie widzimy, w tym wypadku, imię danego wykonawcy, a nie dosłowny string "{{artist.name}}".

### app.js SearchController

Powracając do pliku app.js, zaczynamy pisać kod kontrolera:

```javascript    
    spotiBar.controller("SearchController", [
      "$scope",
      "Spotify",
      function($scope, Spotify) {
        // Tutaj będą funkcje kontrolera
      }
    ]);
```

Jak widzimy, będziemy się odwoływać do obiektu $scope, pozwalającemu nam pobrać ustawione przez użytkownika kryteria, a w dalszym planie także wyświetlić użytkownikowi wyniki. Pierwsza funkcja, którą napiszemy, to obecna w search.html i przywołana już funkcja searchSeeds.

W obrębie naszego kontrolera:

```javascript    
    let seedsSelection = {};
    
    $scope.searchSeeds = function() {
      $scope.seedsSelection = seedsSelection;
      Spotify.getAvailableGenreSeeds().then(
        function(data) {
          $scope.seedsGenres = data.data.genres;
        },
        function(error) {
          alert("The access token expired. Please login again.");
        }
      );
      Spotify.search($scope.seedsQuery, "artist").then(function(data) {
        $scope.seedsArtists = data.data.artists.items;
      });
      Spotify.search($scope.seedsQuery, "track").then(function(data) {
        $scope.seedsTracks = data.data.tracks.items;
      });
    };
```

Tworzymy obiekt seedsSelection, będziemy w nim przetrzymywać wybrane przez użytkownika nazwy artystów/utworów/gatunków muzycznych. Wiążemy go z obecnym w $scope seedsSelection.

Spotify API posiada szereg rodzajów zapytań do bazy. Jednym z nich jest żądanie zwrócenia listy dostępnych gatunków muzycznych. Do tej metody odwołujemy się w linijce 5. Jeśli zakończy się ona sukcesem, to zwrócona zostanie lista gatunków w formacie JSON w obiekcie `data`. Przypisujemy te dane do `$scope.seedsGenres`, a kod w `search.html` w linijkach 108-111 wyświetli je użytkownikowi. Dodatkowo kontrolujemy ewentualne wystąpienie błędu połączenia, będzie to oznaczać, że token uzyskany przez użytkownika w trakcie autoryzacji przestał być ważny i trzeba się zalogować na nowo. Analogicznie z zapytaniami o artystów i utwory, ale tutaj już tego sprawdzenia błędu nie obsługujemy, jako że wszystkie metody są wykonywane mniej więcej w tym samym czasie.

Następną funkcję, dokonanie właściwego wyszukania rekomendacji, podzielimy na szereg funkcji:

```javascript
    $scope.searchRecommendations = function() {
         if (checkIfSeedsInLimit()) {
           Spotify.getRecommendations(prepCriteria()).then(
             function(data) {
               //Zrób coś z wynikami wyszukania, np. wypisz je w konsoli przeglądarki.
               console.log(data);
               // Przekieruj okno przeglądarki pod widok z wynikami.
               document.location.href = "/#!/results";
             },
             function(error) {
               alert("The access token expired. Please login again.");
             }
           );
         } else {
           alert("You can select up to 5 different seeds.");
         }
       };
    
       checkIfSeedsInLimit = function() {
         if (
           $scope.seedsSelection == undefined ||
           ($scope.seedsSelection.artists == undefined &&
             $scope.seedsSelection.tracks == undefined &&
             $scope.seedsSelection.genres == undefined)
         ) {
           alert("You did not define any seeds");
           return false;
         }
    
         let numberOfSeeds = 0;
         if (!($scope.seedsSelection.artists == undefined)) {
           numberOfSeeds =
             numberOfSeeds + Object.keys($scope.seedsSelection.artists).length;
         }
         if (!($scope.seedsSelection.genres == undefined)) {
           numberOfSeeds =
             numberOfSeeds + Object.keys($scope.seedsSelection.genres).length;
         }
         if (!($scope.seedsSelection.tracks == undefined)) {
           numberOfSeeds =
             numberOfSeeds + Object.keys($scope.seedsSelection.tracks).length;
         }
         if (0 < numberOfSeeds && 5 >= numberOfSeeds) {
           return true;
         } else {
           return false;
         }
       };
    
       prepCriteria = function() {
         let criteria = {};
         if ($scope.seedsSelection.artists != undefined) {
           criteria.seed_artists = $scope.seedsSelection.artists.join();
         }
         if ($scope.seedsSelection.tracks != undefined) {
           criteria.seed_tracks = $scope.seedsSelection.tracks.join();
         }
         if ($scope.seedsSelection.genres != undefined) {
           criteria.seed_genres = $scope.seedsSelection.genres.join();
         }
         if ($scope.limit != null) {
           criteria.limit = $scope.limit;
         }
         if ($scope.duration != null) {
           criteria.target_duration_ms = $scope.duration;
         }
         if ($scope.key != null) {
           criteria.target_key = $scope.key;
         }
         if ($scope.ismajor != null) {
           criteria.target_mode = $scope.ismajor;
         }
         if ($scope.tempo != null) {
           criteria.target_tempo = $scope.tempo;
         }
         if (!isNaN($scope.sliderAcousticness)) {
           criteria.target_acousticness = $scope.sliderAcousticness / 100;
         }
         if (!isNaN($scope.sliderDanceability)) {
           criteria.target_danceability = $scope.sliderDanceability / 100;
         }
         if (!isNaN($scope.sliderEnergy)) {
           criteria.target_energy = $scope.sliderEnergy / 100;
         }
         if (!isNaN($scope.sliderInstrumentalness)) {
           criteria.target_instrumentalness = $scope.sliderInstrumentalness / 100;
         }
         if (!isNaN($scope.sliderLiveness)) {
           criteria.target_liveness = $scope.sliderLiveness / 100;
         }
         if (!isNaN($scope.sliderPopularity)) {
           criteria.target_popularity = $scope.sliderPopularity / 100;
         }
         if (!isNaN($scope.sliderSpeechiness)) {
           criteria.target_speechiness = $scope.sliderSpeechiness / 100;
         }
         if (!isNaN($scope.sliderValence)) {
           criteria.target_valence = $scope.sliderValence / 100;
         }
         return criteria;
       };
```



Funkcja `searchRecommendations` rozpoczyna się od sprawdzenia poprawności wybranych przez użytkownika seedów (czyli utworów, artystów i gatunków): funkcja `checkIfSeedsInLimit`. Według ograniczeń Spotify API użytkownik musi wybrać ich w przedziale 1-5 (zawsze musi wybrać coś, ale nie więcej niż 5 łącznie). Jeżeli będzie ich za dużo albo nie będzie ich w ogóle, użytkownikowi wyświetli się mały alert, okienko z informacją.

Po przejściu przez warunek wysyłamy zapytanie przez API, gdzie obiekt z naszymi kryteriami jest przygotowywany w funkcji `prepCriteria`. Różne warunki (`null`, `undefined` lub `NaN`) wynikają z tego, jak dane pole w formularzu jest widziane przez JS w momencie, kiedy jest puste. Wartości ze sliderów dzielimy przez 100, bo API akceptuje w wypadku tych kryteriów zakres 0.00-1.00, a nie 0-100.

W linijce 5 mamy sytuację, w której otrzymaliśmy wyniki ze Spotify w obiekcie `data`. Należałoby z nimi coś zrobić, ale że wyświetlamy wyniki w osobnym widoku (pod przekierowaniem `/results`), do którego przyporządkowaliśmy w konfiguracji `ngRoute` kontroler `ResultsController`, to musimy jakoś skomunikować się między kontrolerami.

### ResultsController i SharingResultsService

W AngularJS do tego celu służą Service (serwisy/usługi? [Tutaj](https://www.p-programowanie.pl/angularjs/service-factory-provider-odchudzenie-kontrolera/) o nich więcej). Pozwalają one na komunikację pomiędzy kontrolerami. Na nasze potrzeby wystarczy bardzo prosta implementacja:

```javascript    
    // Service
    spotiBar.factory("SharingResultsService", function() {
      let sharingRecommendationResults = {
        data: {}
      };
      return sharingRecommendationResults;
    });
```

Kontrolery, które będą korzystać z `SharingResultsService` będą miały do dyspozycji obiekt `sharingRecommendationResults`. Na poziomie `SearchControllera` zapiszemy do tego obiektu wyniki wyszukiwania, zaś na poziomie `ResultsController` się do tych wyników odwołamy. Musimy zmodyfikować nieco nasz `SearchController`:

```javascript    
    spotiBar.controller("SearchController", [
      "$scope",
      "Spotify",
      "SharingResultsService",
      function($scope, Spotify, SharingResultsService) {
        // $scope.searchSeeds = function() ....
    
        $scope.searchRecommendations = function() {
          if (checkIfSeedsInLimit()) {
            Spotify.getRecommendations(prepCriteria()).then(
              function(data) {
                SharingResultsService.data = data.data;
                console.log(data);
                document.location.href = "/#!/results";
              },
              function(error) {
                alert("The access token expired. Please login again.");
              }
            );
          } else {
            alert("You can select up to 5 different seeds.");
          }
        };
        // reszta kodu kontrolera
    }]);
```    


Względem poprzedniej wersji dodaliśmy `SharingResultsService` jako zależność oraz zapisaliśmy (linijka 12) wyniki.

`ResultsController` będzie bardzo krótki. W obiekcie data nasze wyniki są pod tracks:

```javascript    
    spotiBar.controller("ResultsController", [
      "$scope",
      "SharingResultsService",
      function($scope, SharingResultsService) {
        $scope.recommendedTracks = SharingResultsService.data.tracks;
      }
    ]);
```

W naszym widoku wyników musimy ustawić odwołanie jako `recommendedTracks`.

### Results.html

Jako że jeszcze go nie stworzyliśmy, to zrobimy to teraz. Tworzymy plik `results.html` w folderze `/views`:

```html    
    <div class="container">
        <h1 class="spotify-font-artist-name">Results:</h1>
    
        <div class="justify-content-center row">
            <div class="col-lg-2 col-md-3 col-sm-6 col-xs-12 text-center" ng-repeat="track in recommendedTracks" style="padding-left: 5px;padding-right: 5px;">
    
                <a href="{{track.album.uri}}"><img ng-src="{{track.album.images[1].url}}" alt="" style="width:170px;height:170px;"></a>
                <br/>
                <div class="text-center">
                    <p style="margin-bottom: 0px;margin-top: 5px;">
                        <a class="spotify-font-artist-name" ng-repeat="artist in track.artists" href="{{artist.uri}}">{{artist.name}} </a>
                    </p>
                    <p class="text-warning" style="margin-bottom: 0px;"><a class="text-warning" href="{{track.uri}}">{{track.name}}</a></p>
                </div>
    
            </div>
        </div>
    </div>
```

Pomijając klasy Bootstrapowe (sprawiające, że nasze wyniki będą wyświetlane w kilku responsywnych kolumnach), ustawiliśmy `ng-repeat` na iteracje utworów w `recommendedTracks`. Wyświetlana jest okładka albumu, nazwa utworu i wykonawcy.

I to tyle!

Po zapisaniu i odpaleniu na serwerze lokalnym powinniśmy widzieć coś takiego:

Strona z kryteriami wyszukiwania:

![image-center](/assets/images/oldblog/Screen-Shot-2018-04-20-at-21.56.05-1.png){: .align-center}

Strona z wynikami:

![image-center](/assets/images/oldblog/Screen-Shot-2018-04-20-at-21.56.16.png){: .align-center}

I już! W następnej części poprawimy estetykę naszej aplikacji i dodamy instrukcję obsługi (mogliśmy od niej zacząć, właśnie sobie uświadomiłem, że użytkownik nie musi wiedzieć na czym polega wyszukiwanie rekomendacji w Spotify API ;)).

## PART 4

W tym odcinku poprawimy nieco estetykę naszej aplikacji, dodamy instrukcję i ułatwimy UI.

### Estetyka

Skorzystamy z darmowej wersji [Shards](https://designrevision.com/demo/shards/), upiększającej nieco domyślnego Bootstrapa.

Tak było:

![image-center](/assets/images/oldblog/SpotiBar01.png){: .align-center}

A tak będzie:

![image-center](/assets/images/oldblog/SpotiBar02.png){: .align-center}

W samym wpisie wymienimy tylko najważniejsze zmiany, małe zmiany CSSowe pominę. Wszystko i tak jest na GitHub w nowszej wersji. Jest to bardziej wpis sprawozdawczy, niż poradnikowy. Zawsze można prześledzić historię repozytorium ;-).

#### Search.html

Zrezygnujemy z paczki rz-slider, na rzecz... niczego :). Slidery są mało wygodne, było ich za dużo i odciągały uwagę. Wyniki wyszukiwania z API też nie powalają precyzją, jeśli chodzi o kryteria, do których były suwaki... Zamiast tego są elementy select z kilkoma predefiniowanymi wartościami.

Prawą połowę ekranu wyszukiwania zajmie instrukcja, która będzie wyświetlana tak długo, jak długo użytkownik nie wyszuka seedów. Odpowiadają za to dyrektywy ng-switch i ng-switch-when.

Pełen kod dla search.html znajduje się poniżej:

```html    
    <div class="container row mt-5 ml-5">
    
      <div class="col-md-6">
    
        <label class="col-form-label-lg">1. Choose up to five tracks/genres/artists: </label>
    
        <form class="form-inline">
          <div class="input-group input-group-lg mx-sm-3 mb-2">
            <div class="input-group-prepend">
              <span class="input-group-text">Seeds</span>
            </div>
            <input type="text" class="form-control" id="form-seeds" placeholder="Type in seeds query" ng-model="seedsQuery">
            <div class="input-group-append">
              <button type="button" class="btn btn-outline-secondary" ng-click="searchSeeds()">Find seeds</button>
            </div>
          </div>
        </form>
    
        <label class="col-form-label-lg mr-5">2. Search!</label>
    
        <button type="button" class="btn btn-success btn-lg btn-block" ng-click="searchRecommendations()">Find tracks</button>
    
        <label class="col-form-label-lg">* Optional criteria</label>
    
        <form>
          <div class="form-row">
            <div class="form-group col-md-6">
              <label for="inputDuration" class="col-form-label-sm">Duration: </label>
              <input id="inputDuration" type="number" class="form-control input-group-sm" ng-model="durationInput" placeholder="Duration in ms (>0)">
            </div>
            <div class="form-group col-md-6">
              <label for="inputLimit" class="col-form-label-sm">Limit: </label>
              <input id="inputLimit" type="number" class="form-control input-group-sm" ng-model="limitInput" placeholder="1-100 results">
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-row mb-3">
            <div class="form-group col-md-6">
              <label for="inputTempo" class="col-form-label-sm">Tempo (BPM): </label>
              <input id="inputTempo" type="number" class="form-control input-group-sm" ng-model="tempoInput" placeholder="Beats per minute (>0)">
            </div>
            <div class="form-group col-md-6">
              <label for="inputKey" class="col-form-label-sm">Key: </label>
              <input id="inputKey" type="number" class="form-control input-group-sm" ng-model="keyInput" placeholder="0 - C, 1 - C#, up to 11">
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputMajor" class="col-sm-5 col-form-label">Major/Minor: </label>
            <div class="col-sm-7">
              <select id="inputMajor" class="custom-select" ng-model="isMajorSelect">
                <option value="">Select....</option>
                <option value="1">Major</option>
                <option value="0">Minor</option>
              </select>
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputAcoustic" class="col-sm-5 col-form-label">Acousticness: </label>
            <div class="col-sm-7">
              <select id="inputAcoustic" class="custom-select" ng-model="acousticnessSelect" ng-options="x.level for x in criteriaOptions">
                <option value="">Select...</option>
              </select>
            </div>
          </div>
    
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputDanceability" class="col-sm-5 col-form-label">Danceability: </label>
            <div class="col-sm-7">
              <select id="inputDanceability" class="custom-select" ng-model="danceabilitySelect" ng-options="x.level for x in criteriaOptions">
                <option value="">Select...</option>
              </select>
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputEnergy" class="col-sm-5 col-form-label">Energy: </label>
            <div class="col-sm-7">
              <select id="inputEnergy" class="custom-select" ng-model="energySelect" ng-options="x.level for x in criteriaOptions">
                <option value="">Select...</option>
              </select>
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputLiveness" class="col-sm-5 col-form-label">Liveness: </label>
            <div class="col-sm-7">
              <select id="inputLiveness" class="custom-select" ng-model="livenessSelect" ng-options="x.level for x in criteriaOptions">
                <option value="">Select...</option>
              </select>
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputInstrument" class="col-sm-5 col-form-label">Instrumentalness: </label>
            <div class="col-sm-7">
              <select id="inputInstrument" class="custom-select" ng-model="instrumentalnessSelect" ng-options="x.level for x in criteriaOptions">
                <option value="">Select...</option>
              </select>
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputPopularity" class="col-sm-5 col-form-label">Popularity: </label>
            <div class="col-sm-7">
              <select id="inputPopularity" class="custom-select" ng-model="popularitySelect" ng-options="x.level for x in criteriaOptions">
                <option value="">Select...</option>
              </select>
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputSpeechiness" class="col-sm-5 col-form-label">Speechiness: </label>
            <div class="col-sm-7">
              <select id="inputSpeechiness" class="custom-select" ng-model="speechinessSelect" ng-options="x.level for x in criteriaOptions">
                <option value="">Select...</option>
              </select>
            </div>
          </div>
        </form>
    
        <form>
          <div class="form-group row">
            <label for="inputValence" class="col-sm-5 col-form-label">Valence: </label>
            <div class="col-sm-7">
              <select id="inputValence" class="custom-select" ng-model="valenceSelect" ng-options="x.level for x in criteriaOptions">
                <option value="">Select...</option>
              </select>
            </div>
          </div>
        </form>
    
      </div>
    
      <div class="col-md-6" ng-switch="seedOrInstruction">
        <div ng-switch-when="seeds">
          <div class="row">
            <div class="col-md-4">
              <label class="col-form-label-lg">Artists:</label>
              <div class="row" ng-repeat="artist in seedsArtists">
                <label>
                  <input type="checkbox" checklist-model="seedsSelection.artists" checklist-value="artist.id" />
                  <span>{{artist.name}}</span>
                </label>
              </div>
            </div>
            <div class="col-md-8">
              <label class="col-form-label-lg">Tracks:</label>
              <div class="row" ng-repeat="track in seedsTracks">
                <label>
                  <input type="checkbox" checklist-model="seedsSelection.tracks" checklist-value="track.id" />
                  <span>{{track.name}} -</span>
                  <span ng-repeat="artist in track.artists"> &nbsp;{{artist.name}} </span>
                </label>
              </div>
            </div>
          </div>
          <div class="row">
            <div class="col-md-12">
              <label class="col-form-label-lg">Genres:</label>
              <br>
              <div class="form-check-inline" ng-repeat="genre in seedsGenres">
                <label>
                  <input type="checkbox" class="" checklist-model="seedsSelection.genres" checklist-value="genre" /> {{genre}}
                </label>
              </div>
            </div>
          </div>
    
        </div>
        <div ng-switch-when="instruction">
      
          <div class="jumbotron  jumbotron-fluid">
            <div class="container">
            <h4 class="display-4">How to</h2>
              <p class="lead">
                <span class="font-weight-bold">SpotiBar</span> allows you to search Spotify recommendations based on several criteria. In order to use it, you
                should make sure that:</p>
                <ol class="list-group mb-4">
                  <li class="list-group-item">You are signed in to your Spotify account. To do so, click the button in the top-right corner</li>
                  <li class="list-group-item">You've selected
                    <span class="font-weight-bold">at least 1 and not more than 5</span> seeds (artists/tracks/genres). You can search for them by quering
                    field on the left</li>
                  <li class="list-group-item">(optional) You can also set optional criteria, such as Duration, Key and others to get more specific recommendations</li>
                </ol>
                <hr class="my-4">
                <p class="lead">Some optional criteria have ranges given in placeholders. Search mechanisms will ignore such criteria that are not within range.
              </p>
            </div>
        </div>
        </div>
      </div>
    </div>
```

#### app.js (3)

Jako że zrezygnowaliśmy z rz-slider, to paczka musi zniknąć również w naszej deklaracji modułu:

```javascript    
    const spotiBar = angular.module("spotiBar", [
      "ngRoute",
      "checklist-model",
      "spotify",
    ]);
```

Oprócz tego zmienia się również nasz SearchController, tak aby uwzględnić rezygnację ze sliderów. Definiujemy między innymi criteriaOptions, do których są odwołania w search.html w elementach select. Mamy też inną formę walidacji kryteriów (nadal strasznie leniwą):

```javascript    
    spotiBar.controller("SearchController", [
      "$scope",
      "Spotify",
      "SharingResultsService",
      function($scope, Spotify, SharingResultsService) {
        let seedsSelection = {};
    
        $scope.seedOrInstruction = "instruction";
    
        $scope.criteriaOptions = [
          { level: "Very low", value: "0.00" },
          { level: "Low", value: "0.25" },
          { level: "Medium", value: "0.50" },
          { level: "High", value: "0.75" },
          { level: "Very high", value: "1.00" }
        ];
    
        $scope.searchSeeds = function() {
          $scope.seedsSelection = seedsSelection;
          Spotify.getAvailableGenreSeeds().then(
            function(data) {
              $scope.seedOrInstruction = "seeds";
              $scope.seedsGenres = data.data.genres;
            },
            function(error) {
              alert("The access token expired. Please login again.");
            }
          );
          Spotify.search($scope.seedsQuery, "artist").then(function(data) {
            $scope.seedsArtists = data.data.artists.items;
          });
          Spotify.search($scope.seedsQuery, "track").then(function(data) {
            $scope.seedsTracks = data.data.tracks.items;
          });
        };
    
        $scope.searchRecommendations = function() {
          if (checkIfSeedsInLimit()) {
            Spotify.getRecommendations(prepCriteria()).then(
              function(data) {
                SharingResultsService.data = data.data;
                console.log(data);
                document.location.href = "/spotify-recommendations-app-js/#!/results";
              },
              function(error) {
                alert("The access token expired. Please login again.");
              }
            );
          } else {
            alert("You can select up to 5 different seeds.");
          }
        };
    
        checkIfSeedsInLimit = function() {
          if (
            $scope.seedsSelection == undefined ||
            ($scope.seedsSelection.artists == undefined &&
              $scope.seedsSelection.tracks == undefined &&
              $scope.seedsSelection.genres == undefined)
          ) {
            alert("You did not define any seeds");
            return false;
          }
    
          let numberOfSeeds = 0;
          if (!($scope.seedsSelection.artists == undefined)) {
            numberOfSeeds =
              numberOfSeeds + Object.keys($scope.seedsSelection.artists).length;
          }
          if (!($scope.seedsSelection.genres == undefined)) {
            numberOfSeeds =
              numberOfSeeds + Object.keys($scope.seedsSelection.genres).length;
          }
          if (!($scope.seedsSelection.tracks == undefined)) {
            numberOfSeeds =
              numberOfSeeds + Object.keys($scope.seedsSelection.tracks).length;
          }
          if (0 < numberOfSeeds && 5 >= numberOfSeeds) {
            return true;
          } else {
            return false;
          }
        };
    
        prepCriteria = function() {
          let criteria = {};
          if ($scope.seedsSelection.artists != undefined) {
            criteria.seed_artists = $scope.seedsSelection.artists.join();
          }
          if ($scope.seedsSelection.tracks != undefined) {
            criteria.seed_tracks = $scope.seedsSelection.tracks.join();
          }
          if ($scope.seedsSelection.genres != undefined) {
            criteria.seed_genres = $scope.seedsSelection.genres.join();
          }
          if ($scope.limitInput != null) {
            if ($scope.limitInput > 0 && $scope.limitInput < 101) {
              criteria.limit = $scope.limitInput;
            }
          }
          if ($scope.durationInput != null) {
            if ($scope.durationInput > 0) {
              criteria.target_duration_ms = $scope.durationInput;
            }
          }
          if ($scope.keyInput != null) {
            if ($scope.keyInput > -1 && $scope.keyInput < 12) {
              criteria.target_key = $scope.keyInput;
            }
          }
          if ($scope.isMajorSelect != null) {
            criteria.target_mode = $scope.isMajorSelect;
          }
          if ($scope.tempoInput != null) {
            if ($scope.tempoInput > 0) {
              criteria.target_tempo = $scope.tempoInput;
            }
          }
          if ($scope.acousticnessSelect != null) {
            criteria.target_acousticness = $scope.acousticnessSelect.value;
          }
          if ($scope.danceabilitySelect != null) {
            criteria.target_danceability = $scope.danceabilitySelect.value;
          }
          if ($scope.energySelect != null) {
            criteria.target_energy = $scope.energySelect.value;
          }
          if ($scope.livenessSelect != null) {
            criteria.target_liveness = $scope.livenessSelect.value;
          }
          if ($scope.instrumentalnessSelect != null) {
            criteria.target_instrumentalness = $scope.instrumentalnessSelect.value;
          }
          if ($scope.popularitySelect != null) {
            criteria.target_popularity = $scope.popularitySelect.value;
          }
          if ($scope.speechinessSelect != null) {
            criteria.target_speechiness = $scope.speechinessSelect.value;
          }
          if ($scope.valenceSelect != null) {
            criteria.target_valence = $scope.valenceSelect.value;
          }
          return criteria;
        };
      }
    ]);
```

I to w zasadzie tyle... Pozostaje oczyszczenie elementu head w index.html ze zbędnych już bibliotek i dodanie Shards. Plus pomniejsze zmiany w CSS i atrybutach klas dla poszczególnych elementów strony :).

#### Czego nie ma, co można zmienić

* dodać prawdziwą walidację pól opcjonalnych, tak aby użytkownik nie mógł wyszukać rekomendacji, jeśli łamie zakres danego pola
* dodać obsługę alertów, a nie polegać na wbudowanych w JS :)
* polepszyć obsługę błędów
* * zwiększyć czytelność JS, np. podzielić nasz plik JSowy
* przesunąć Stringi do osobnego pliku i je wczytywać podług potrzeb
