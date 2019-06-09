---
title: "[OLD] ISS, Architecture: ViewModel, LiveData, Retrofit"
date: 2018-04-29T11:02:00-04:00
toc: true
toc_label: "Table of Contents"
categories:
  - blog
tags:
  - coding
---

[This is a post from my old website. Outdated packages and libraries. Viewer discretion is advised ;-)]

W tym artykule stworzymy prostą aplikację na Android wyświetlającą aktualną pozycję Międzynarodowej Stacji Kosmicznej, wykorzystującą ViewModel, LiveData oraz bibliotekę Retrofit. Przy okazji wyjaśnimy jak działają dodane w zeszłym roku komponenty architektury Androida.

Aplikacja wyglądać będzie tak:

![image-center](/assets/images/oldblog/issfinishedappl.png){: .align-center}

## Komponenty architektury Androida

[Architecture Components](https://developer.android.com/topic/libraries/architecture/) to biblioteki dla Androida, zaprezentowane przez Google na [I/O '17](https://www.youtube.com/watch?v=FrteWKKVyzI). Mają one ułatwić proces projektowania architektury aplikacji. Dotąd zespół Androida nie rekomendował żadnego konkretnego wzorca architektury. A tych popularnych trochę jest, by wymienić same [_model-view-*_](https://medium.com/@vicky7230/android-architecture-patterns-mv-c-p-vm-4594574eeaa1).

Dodatkowe komponenty i rekomendowany przepływ danych można zobrazować tak:

![image-center](/assets/images/oldblog/AndroidComponentsFlowChart1.png){: .align-center}

Warstwa UI oddzielona jest od tej częsci kodu, która jest odpowiedzialna za zdobywanie i przygotowywanie danych przez `ViewModel`. Obiekty tej klasy nie są niszczone wskutek zmian konfiguracyjnych aplikacji (np. zmiany orientacji ekranu), są _lifecycle-aware_, więc nie ma już potrzeby pakowania multum danych w `onSaveInstanceState()`. ViewModel przetrwa, jeśli w Activity, w której był przywołany, zostanie użyte `onDestroy()`, o ile tylko nie będzie to się wiązało z wywołaniem metody `finish()` (więcej [tutaj](https://code.tutsplus.com/tutorials/android-architecture-components-lifecycle-and-livemodel--cms-29275)).

Sam `ViewModel` również nie ma być odpowiedzialny za operacje na bazie danych albo zewnętrznym API, dostaje takie gotowe dane z `Repository`. Sama klasa Repository nie wchodzi w skład bibliotek Architecture Components. Obiekty należące do tej klasy mają pośredniczyć w wymianie danych pomiędzy źródłem danych (API/SQLite/itd.) a ViewModelem.

Biblioteka `Room` to warstwa abstrakcji nad SQLite, ułatwiająca (= mniej kodu) dostęp i manipulację danymi z bazy danych. Od strony RESTfulowych usług, pobierania danych spoza naszej aplikacji przy pomocy jakiegoś API, Android nie dostarcza gotowej biblioteki. W [dokumentacji](https://developer.android.com/topic/libraries/architecture/guide) natomiast korzysta z [`Retrofit`](http://square.github.io/retrofit/), stąd gwiazdka na diagramie :).

Pozostaje kwestia [`LiveData`](https://developer.android.com/topic/libraries/architecture/livedata). Jest to _data holder_, który może być obserwowany.  Innymi słowy, jeśli nasze dane zostaną przekazane przez ViewModel do warstwy UI jako LiveData, to obserwowanie stanu danych w LiveData wystarczy, by przy zmianie w danych zmianie też uległa wyświetlana dla użytkownika treść na ekranie.

## Aplikacja WhereIsSpaceStation

W tym wpisie zajmiemy się "prawą stroną" diagramy. Nie będziemy niczego zapisywać w bazie danych, wykorzystamy LiveData, ViewModel, Repository i Retrofit by wyświetlić na ekranie aktualną pozycję ISS. Po naciśnięciu przycisku informacja na ekranie będzie aktualizowana. W przyszłości dodamy też bazę danych.

Skorzystamy z:

* [Android Studio](https://developer.android.com/studio/) (na dzień pisania wpisu jest to wersja 3.1.1)
* [OpenNotify API](http://open-notify.org/Open-Notify-API/)
* [Retrofit](http://square.github.io/retrofit/)
* Biblioteki [Architecture Components](https://developer.android.com/topic/libraries/architecture/)

### Ustawienia projektu

Tworzymy nowy projekt w Android Studio:

![image-center](/assets/images/oldblog/Screen-Shot-2018-04-29-at-10.47.55.png){: .align-center}

![image-center](/assets/images/oldblog/Screen-Shot-2018-04-29-at-10.48.07.png){: .align-center}

Resztę ustawień pozostawiamy domyślne. Kreator powinien nam stworzyć nowy projekt, z jedną Activity (wedle templatki EmptyActivity).

Nasza aplikacja będzie korzystać z połączenia z internetem, więc w `AndroidManifest.xml` musimy dodać:

```xml    
    <uses-permission android:name="android.permission.INTERNET"/>
```

Architecture Components na dzień dzisiejszy trzeba ręcznie dodać do projektu. W `build.gradle` projektu sprawdzamy, czy wśród repozytoriów znajduje się google():

```gradle    
    repositories {
           google()
           jcenter()
       }
```

A w `build.gradle` na poziomie modułu aplikacji dodajemy potrzebne biblioteki. Aktualne wersje bibliotek znaleźć można [tutaj](https://developer.android.com/topic/libraries/architecture/adding-components):

```gradle    
    dependencies {
        implementation fileTree(dir: 'libs', include: ['*.jar'])
        implementation 'com.android.support:appcompat-v7:27.1.0'
        implementation 'com.android.support.constraint:constraint-layout:1.1.0'
        testImplementation 'junit:junit:4.12'
        androidTestImplementation 'com.android.support.test:runner:1.0.1'
        androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
    
        // Javax annotation
        implementation 'org.glassfish:javax.annotation:10.0-b28'
    
        // Retrofit
        implementation 'com.squareup.retrofit2:retrofit:2.4.0'
        implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
    
        // ViewModel and LiveData
        implementation "android.arch.lifecycle:extensions:1.1.1"
        // alternatively, just ViewModel
        implementation "android.arch.lifecycle:viewmodel:1.1.1"
        // alternatively, just LiveData
        implementation "android.arch.lifecycle:livedata:1.1.1"
    
        annotationProcessor "android.arch.lifecycle:compiler:1.1.1"
    
        // Room (use 1.1.0-beta3 for latest beta)
        implementation "android.arch.persistence.room:runtime:1.0.0"
        annotationProcessor "android.arch.persistence.room:compiler:1.0.0"
    
        // Paging
        implementation "android.arch.paging:runtime:1.0.0-rc1"
    
        // Test helpers for LiveData
        testImplementation "android.arch.core:core-testing:1.1.1"
    
        // Test helpers for Room
        testImplementation "android.arch.persistence.room:testing:1.0.0"
    }
```

Synchronizujemy i budujemy projekt, sprawdzając, czy nie ma błędów kompilacji :).

### Resources

Nasza aplikacja będzie bardzo prosta, więc możemy od razu ustawić potrzebne pliki w katalogu `res`.

W strings.xml dodajemy kilka Stringów:

```xml    
    <resources>
        <string name="app_name">WhereIsSpaceStation</string>
        <string name="header_note">Where is the International Space Station now?</string>
        <string name="button_text">Check!</string>
        <string name="response_empty">It seems we did not receive any data this time.</string>
        <string name="response_failure">It seems you have some internet connection problems.</string>
    </resources>
```

A w pliku XML naszego Activity (MainActivity.xml):

```xml    
    <?xml version="1.0" encoding="utf-8"?>
    <android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">
    
        <TextView
            android:id="@+id/header"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/header_note"
            android:textSize="16sp"
            app:layout_constraintBottom_toTopOf="@id/coordinates"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
    
        <TextView
            android:id="@+id/coordinates"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text=""
            app:layout_constraintBottom_toTopOf="@id/checkButton"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toBottomOf="@id/header" />
    
    
        <Button
            android:id="@+id/checkButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/button_text"
            app:layout_constraintTop_toBottomOf="@id/coordinates"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"/>
    
    
    </android.support.constraint.ConstraintLayout>
```

Na nasze UI składać się będzie (oprócz paska aplikacji): TextView z na sztywno ustawionym tekstem ("Where is the ISS now?"), TextView, w którym wyświetlać będziemy lokalizację ISS oraz Button, którego naciśnięcie zaktualizuje dane o lokalizacji.

### Konfiguracja Retrofit

Ustawienie Retrofit jest bardzo proste. Potrzebujemy adresu, pod który wysyłane będzie żądanie, POJO, w których Retrofit będzie zapisywać dane zwrotne i jednego interfejsu.

OpenNotify, API z którego skorzystamy, jest bardzo przyjemne, bo nie wymaga żadnej autoryzacji. Adres, pod którym znajdują się potrzebne nam dane o lokalizacji ISS jest taki: [`http://api.open-notify.org/iss-now.json`](http://api.open-notify.org/iss-now.json). Jak widać, dane zwrotne są w postaci JSON, będziemy musieli poinformować o tym Retrofit przy konfiguracji.

POJO wraz z anotacjami możemy wykreować automatycznie. W Android Studio istnieje możliwość używania pluginów. Klikamy w Preferences->Plugins, wpisujemy `Json2Pojo`, instalujemy i restartujemy IDE.

![image-center](/assets/images/oldblog/Screen-Shot-2018-04-29-at-11.09.35.png){: .align-center}

Stworzymy sobie osobną paczkę do POJO. W widoku struktury projektu z lewej strony programu klikamy na główną paczkę naszego projektu prawym przyciskiem myszy, następnie New->Package. Nazywamy ją "`pojos`". Klikamy prawym na pojos, New->Create POJOs from JSON. W Root Class Name wpisujemy nazwę naszego POJO: `IssLocationJSON`, a w okienku przeklejamy JSON z [OpenNotify](http://api.open-notify.org/iss-now.json). Klikamy OK.

Plugin powinien wygenerować dwie klasy: `IssLocationJSON` oraz `IssPosition`. Dla porównania, kod dla IssLocationJSON:

```Java    
    @Generated("net.hexar.json2pojo")
    @SuppressWarnings("unused")
    public class IssLocationJSON {
    
        @SerializedName("iss_position")
        private IssPosition mIssPosition;
        @SerializedName("message")
        private String mMessage;
        @SerializedName("timestamp")
        private Long mTimestamp;
    
        public IssPosition getIssPosition() {
            return mIssPosition;
        }
    
        public void setIssPosition(IssPosition issPosition) {
            mIssPosition = issPosition;
        }
    
        public String getMessage() {
            return mMessage;
        }
    
        public void setMessage(String message) {
            mMessage = message;
        }
    
        public Long getTimestamp() {
            return mTimestamp;
        }
    
        public void setTimestamp(Long timestamp) {
            mTimestamp = timestamp;
        }
    
    }
```

Oraz dla IssPosition:

```java
    @Generated("net.hexar.json2pojo")
    @SuppressWarnings("unused")
    public class IssPosition {
    
        @SerializedName("latitude")
        private String mLatitude;
        @SerializedName("longitude")
        private String mLongitude;
    
        public String getLatitude() {
            return mLatitude;
        }
    
        public void setLatitude(String latitude) {
            mLatitude = latitude;
        }
    
        public String getLongitude() {
            return mLongitude;
        }
    
        public void setLongitude(String longitude) {
            mLongitude = longitude;
        }
    
    }
```

W głównym folderze z kodem źródłowym, czyli obok paczki pojos, tworzymy paczkę api. W niej tworzymy interfejs dla Retrofit o nazwie `OpenNotifyService`. Ma on tylko jedną metodę abstrakcyjną: `getIssLocation()`. Anotujemy ją Retrofitowym `@GET`, podając ścieżkę do interesującego nas node'u API (czyli co jest po slashu głównego adresu):

```java    
    public interface OpenNotifyService {
    
        @GET("iss-now")
        Call<IssLocationJSON> getIssLocation();
    }
``` 

Mamy już interfejs, POJO i adres.

Później zapiszemy dane zwrotne z lokalizacją w LiveData. LiveData nie ma jednak wbudowanego sposobu na radzenie sobie z żądaniami internetowymi. A co jeśli użytkownik nie będzie mieć połączenia internetowego albo serwer OpenNotify przestanie działać? Zwrócimy null? Dokumentacja sugeruje [takie](https://developer.android.com/topic/libraries/architecture/guide#addendum) rozwiązanie, ale na nasze minimalne potrzeby wystarczy dużo prostsze. Zrobimy wrapper dla naszych IssLocationJSON, posiadający pole informujące nas o stanie połączenia. Instancje tego wrappera zapiszemy dopiero jako LiveData. Wówczas będziemy mieli dostęp do informacji, czy posiadamy dane o lokalizacji ISS, czy też ich nie mamy, bez problemów z nullami.

W paczce pojos tworzymy klasę `IssLocationWrapper`:

```java    
    package whereisspacestation.com.tryouts.whereisspacestation.pojos;
    
    public class IssLocationWrapper {
    
        private IssLocationJSON mIssLocationJson;
        private int mResponseStatus;
    
    
        public IssLocationJSON getIssLocationJson() {
            return mIssLocationJson;
        }
    
        public void setIssLocationJson(IssLocationJSON mIssLocationJson) {
            this.mIssLocationJson = mIssLocationJson;
        }
    
        public int getResponseStatus() {
            return mResponseStatus;
        }
    
        public void setResponseStatus(int mResponseStatus) {
            this.mResponseStatus = mResponseStatus;
        }
    }
```  


Stworzymy też abstrakcyjną klasę `Status` ze statycznymi kodami dla statusu naszego wrappera. Tworzymy paczkę `util` i w niej klasę `Status` (pewnie bardziej elegancko byłoby użyć enumów, ale dla naszych potrzeb wystarczą zwykłe integery):

```java   
    package whereisspacestation.com.tryouts.whereisspacestation.util;
    
    public abstract class Status {
        private final static int RESPONSE_OK = 1;
        private final static int RESPONSE_EMPTY = 0;
        private final static int RESPONSE_FAILURE = -1;
    
    
        public static int getResponseOk() {
            return RESPONSE_OK;
        }
    
        public static int getResponseEmpty() {
            return RESPONSE_EMPTY;
        }
    
        public static int getResponseFailure() {
            return RESPONSE_FAILURE;
        }
    }
```   

### Repository

Możemy już stworzyć instancję klasy `Retrofit`, która będzie zczytywać dane o lokalizacji ISS z serwera. Mając w pamięci diagram przepływu danych z komponentami architektury Androida, musimy stworzyć `Repository`.

Tworzymy paczkę `repository`, a w niej klasę `IssLocationRepository`:

```java    
    public class IssLocationRepository {
    
        private static final String BASE_URL = "http://api.open-notify.org/";
        private static Retrofit mRetrofit;
        private static OpenNotifyService service;
        private MutableLiveData<IssLocationWrapper> mSpaceStationLocation;
    
        public IssLocationRepository() {
    
            mRetrofit = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create())
                    .build();
            service = mRetrofit.create(OpenNotifyService.class);
            mSpaceStationLocation = new MutableLiveData<>();
    
            setLocation();
        }
    
    
        public void setLocation() {
            final IssLocationWrapper issLocationWrapper = new IssLocationWrapper();
            service.getIssLocation().enqueue(new Callback<IssLocationJSON>() {
                @Override
                public void onResponse(Call<IssLocationJSON> call, Response<IssLocationJSON> response) {
                    if(response.isSuccessful()) {
                        IssLocationJSON location = response.body();
                        if(location != null) {
                            issLocationWrapper.setIssLocationJson(location);
                            issLocationWrapper.setResponseStatus(Status.getResponseOk());
                            mSpaceStationLocation.postValue(issLocationWrapper);
                        } else {
                            issLocationWrapper.setResponseStatus(Status.getResponseEmpty());
                            mSpaceStationLocation.postValue(issLocationWrapper);
                        } 
                    } else {
                        issLocationWrapper.setResponseStatus(Status.getResponseEmpty());
                        mSpaceStationLocation.postValue(issLocationWrapper);
                    }
                }
    
                @Override
                public void onFailure(Call<IssLocationJSON> call, Throwable t) {
                    issLocationWrapper.setResponseStatus(Status.getResponseFailure());
                    mSpaceStationLocation.postValue(issLocationWrapper);
                }
            });
        }
    
    
        public MutableLiveData<IssLocationWrapper> getLocation() {
            return mSpaceStationLocation;
        }
    }
```

Od razu korzystamy z LiveData (w tym wypadku `MutableLiveData`, które różni się tylko tym, że upublicznia nam metody do zapisu danych w LiveData), bo z metody `getLocation()` będzie korzystać ViewModel.

W konstruktorze konfigurujemy `Retrofit`: podajemy bazowe URL API, informujemy, z jakiego konwertera ma korzystać (GSON w tym wypadku, z racji na stosowany w OpenNotify JSON), no i jakie żądania ma obsługiwać (nasz OpenNotifyService).

Metoda `setLocation()` wykonuje żądanie GET asynchronicznie, stąd `.enqueue` i `CallBack`. Jeśli nie ma połączenia z internetem wywołana zostanie przeciążona metoda `onFailure()`. Metoda `onResponse()` natomiast zostanie wywołana, jeśli otrzymamy odpowiedź z serwera. Jeśli wartość `response.isSuccesful()` jest false, to najwyraźniej dostaliśmy odpowiedź 404, 500 albo inną odpowiedź błędu, ale nie dostaliśmy w odpowiedzi danych o lokalizacji ISS, na których nam zależało. Może też się zdarzyć, że nasze żądanie zostanie przesunięte pod inny adres, ale nie otrzymamy odpowiedzi o błędnym adresie, bo adres działa (nie ma pod nim żadnych istotnych dla nas danych).

We wszystkich tych przypadkach, a także kiedy po prostu otrzymujemy interesujące nas dane (hurra), stosownie konfigurujemy obiekt `MutableLiveData<IssLocationWrapper>`.

Nic więcej w naszym Repository nie musimy dodawać.

### ViewModel

Konstrukcja i struktura ViewModel w naszym przypadku jest bardzo prosta: tworzymy nową klasę, rozszerzającą klasę ViewModel o nazwie `LocationViewModel`:

```java    
    public class LocationViewModel extends ViewModel {
    
        private IssLocationRepository mRepository;
        private MutableLiveData<IssLocationWrapper> mIssLocation;
    
        public LocationViewModel() {
            super();
            mRepository = new IssLocationRepository();
            mIssLocation = mRepository.getLocation();
        }
    
        public MutableLiveData<IssLocationWrapper> getLocation() {
            return mIssLocation;
        }
    
        public void checkLocation() {
            mRepository.setLocation();
        }
    
    
    }
```

Nasz LocationViewModel jest w stanie zwrócić informację o lokalizacji ISS (`getLocation()`) oraz wysłać żądanie do repozytorium o ponowne ustalenie lokalizacji.

### UI, MainActivity

Jesteśmy gotowi by przekazać dane do warstwy UI. Przypomnijmy, że na layout `MainActivity` składają się TextView, w którym wyświetlać będziemy lokalizację ISS oraz Button do odświeżania lokalizacji.

MainActivity:

```java    
    public class MainActivity extends AppCompatActivity {
    
        private LocationViewModel mLocationViewModel;
        private TextView mLocationTextView;
        private Button mLocationCheckButton;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            mLocationTextView = findViewById(R.id.coordinates);
    
            mLocationViewModel = ViewModelProviders.of(this).get(LocationViewModel.class);
            mLocationViewModel.getLocation().observe(this, new Observer<IssLocationWrapper>() {
                        @Override
                        public void onChanged(@Nullable IssLocationWrapper issLocationWrapper) {
                            switch(issLocationWrapper.getResponseStatus()) {
                                case 1:
                                    mLocationTextView.setText(
                                        issLocationWrapper.getIssLocationJson().getIssPosition().getLatitude() + " "
                                                + issLocationWrapper.getIssLocationJson().getIssPosition().getLongitude());
                                    break;
                                case 0:
                                    mLocationTextView.setText(R.string.response_empty);
                                    break;
                                case -1:
                                    mLocationTextView.setText(R.string.response_failure);
                                    break;
                                default:
                                    mLocationTextView.setText(R.string.response_empty);
                                    break;
                            }
    
                        }
            });
    
            mLocationCheckButton = findViewById(R.id.checkButton);
            mLocationCheckButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mLocationViewModel.checkLocation();
                }
            });
    
        }
    }
```

LocationViewModel dostarcza nam metoda `get()` [`ViewModelProvidera`](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProvider) (przypisanego do Activity), nie tworzymy ViewModeli ze ''zwykłego'' konstruktora.

Następnie zaczynamy obserwować (`observe()`) dane zwracane z ViewModel w metodzie `getLocation()`. Jeśli pamiętamy sprzed chwili, zwraca ona nam `MutableLiveData<IssLocationWrapper>`, a więc nasz obiekt z informacjami o lokalizacji ISS oraz stanem połączenia.

Jeśli dane te ulegną zmianie wywołana zostanie metoda` onChanged()` obserwatora. Sprawdzany jest status połączenia i stosownie uzupełniany TextView.

Pod koniec metody `onCreate()` mamy jeszcze zdefiniowany listener dla przycisku, obsługujący naciskanie przycisku przez użytkownika. Zauważmy, że nie wywołuje on już metody pobierającej dane z ViewModelu, a jedynie metodę aktualizacją lokalizację w samym LiveData w ViewModelu. Jako, że stan tego obiektu obserwujemy, to po zmianie jego zawartości automatycznie ujrzymy zmianę na poziomie UI w TextView.

### Finisz

I to tyle, po odpaleniu aplikacji, zakładając dostęp do internetu, ujrzymy obrazek z początku artykułu:

![image-center](/assets/images/oldblog/issfinishedappl.png){: .align-center}

Jeśli zaś włączymy w emulatorze/telefonie tryb samolotowy, to ujrzymy:

![image-center](/assets/images/oldblog/issfinishedapp.png){: .align-center}

Podsumowując, zbudowaliśmy prostą aplikację, korzystającą z bibliotek Architecture Components oraz Retrofit. Warstwa UI jest odseparowana od reszty poprzez ViewModel. Wyświetlana informacja o lokalizacji ISS pochodzi z obserwowanego z poziomu UI obiektu LiveData przekazywanego przez ViewModel. ViewModel pozyskuje potrzebne informacje z Repository. W Repository wykonujemy połączenie z API przez Retrofit. UI nie wie, jak przetwarzane i skąd źródłowo pochodzą dane do wyświetlenia. ViewModel również na dobrą sprawę nie wie. Repository i ViewModel pozbawione są odwołań do klas związanych z UI. Warstwa UI natomiast pozbawiona jest odwołań do klas bezpośrednio związanych z logiką manipulacji i pozyskiwania danych.
