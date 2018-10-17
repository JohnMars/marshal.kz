---
layout: post
title: "Code Labs: Разработка простого приложения, используя Architecture Components"
date: 2018-10-12 10:00:00 +0600
categories: codelabs
---

> DevFest, 21 Окт. 2018
> Маршал Жанибек. Ведущий разработчик команды Android
> компании Колёса | Крыша | Маркет

## 1. Android Architecture Components

[Android Architecture Components](https://www.youtube.com/watch?time_continue=5&v=vOJCrbr144o) представляют собой набор Android-библиотек для структурирования вашего приложения таким образом, чтобы они были надежными, проверяемыми и поддерживаемыми. В дополнение к библиотекам также есть [руководство по архитектуре приложений](https://developer.android.com/topic/libraries/architecture/guide.html), в котором описан подход к архитектуре приложения для Android с использованием библиотек Android Architecture Components.

![Android Architecture Components](/assets/codelabs/achitecture-components.png)

Список представленных коллекции библиотек:
* Lifecycle Owner - Обработчик жизненного цикла на Android
* LiveData - Предоставляет объекты данных для подписчиков
* ViewModel - Сохраняет данные относящиеся к View при поворотах
* Room - Удобный инструмент для работы с SQLite БД
* Navigation
* WorkManager
* DataBinding

## 2. Что вы будете создавать

Вы будете использовать различные компоненты, чтобы сделать приложение Колёса,
которое извлекает данные из удаленного источника, сохраняет его локально и отображает его пользователю.

### Этапы создания приложения

1. Исследуйте принципы в руководстве по архитектуре приложений
2. Используйте библиотеку Lifecycle, которая включает LiveData и ViewModel
3. Используйте библиотеку сохранения данных Room

Запросы в API и UI часть будет заранее доступно в проекте

### Что вам понадобиться

1. Android Studio 3.0 или новее
2. Знакомство со зданием Android-приложений и жизненным циклом активности Android
3. Базовое знание SQLite
  * Например, вы должны иметь возможность писать операторы SELECT с предложением WHERE.
  * Базовое знание потоковой обработкой и обработкой асинхронных задач в Android

### Скачиваем исходный код

1. `git clone https://stash.kolesa-team.org/projects/MAPG/repos/android-devfest-aar/browse`
2. Открыть в Android Studio

Всю историю commit можно увидеть в master ветке

## 3. Архитектура с использованием AAC

![Architecture](/assets/codelabs/architecture.png)

**UI Controller** - это действия или фрагменты. Единственное задача UI контроллера - это знать, как отображать данные и передавать события UI, такие как нажитие кнопки. UI контроллеры не содержат данных UI, а также не работает с данными напрямую.

**ViewModels и LiveData** - эти классы представляют все данные, необходимые для отображения UI.

**Repository** - этот класс является единственным источником для всех данных нашего приложения и действует как чистый API для взаимодействия с UI. ViewModels просто запрашивает данные из Repository. Нам не нужно беспокоиться о том, должен ли Repository загружаться из базы данных или сети или как и когда сохранять данные. Repository управляет всем этим. В рамках этой задачи Repository является посредником между различными источниками данных.

**Remote Data Source** - управляет данными из удаленного источника данных, такого как Интернет.

**Model** - управляет локальными данными, хранящимися в базе данных.

### Архитектура приложения копии Kolesa

Как в итоге должен выглядеть приложение, которое вы будете разрабатывать

![AdvertListActivity and AdvertDetailsActivity](/assets/codelabs/dev-fest-app.png)

Будет 2 Activity (AdvertListActivity и AdvertDetailsActivity) с двумя ViewModel (AdvertListViewModel и AdvertDetailsActivity). Они будут обращаться к классу Repository (AdvertisementRepository), который будет управлять связью между базой данных SQLite и сетевым источником данных. В вашем располежении будет готовый AdvertisementNetworkDataSource в проекте с готовыми данными. AdvertisementNetworkDataSource будет отправлять запросы чтобы получить данные объявления [с замоканного сервера](https://stash.kolesa-team.org/projects/MAPG/repos/sample/). В нем содержится урезанные данные с сайта колёс.

```json
[
  {
    "id": 1,
    "title": "Toyota Camry 2018 года",
    "price": 8300000,
    "created_at": 1539705559,
    "specification": "литые диски, ксенон, кожа, USB, ГУР, ABS, SRS, бортовой компьютер, свежепригнан, налог уплачен, вложений не требует",
    "text": "Автомобиль в замечательном состоянии, без вложений. Все вопросы в телефон режиме! Торг!",
    "parameters": [
      {
        "label": "Город",
        "value": "Алматы"
      }
    ],
    "photos": [
      {
        "url": "https://photos-b-kl.kcdn.kz/68/689db250-d36d-4bf5-9897-1daf97939a1f/1-full.jpg"
      }
    ],
    "phones": [
      {
          "number": "+77071234567"
      }
    ]
  }
]
```

В данном code labs вы будете разрабатывать классы обрисованные зелеными линиями.
Каркасы этих классов уже заготовлены в прокте, который вы скачали ранее.

![Dev Fest Kolesa Architecture](/assets/codelabs/kolesa-architecture.png)

## 4. ViewModel и LiveData

Класс ViewModel  предназначен для содержания и управления данными UI в рамках жизненного цикла. Это позволяет хранить данные во время изменении конфигурации такие как повороты экрана. Разделив данные UI от UI контроллеров, вы можете разделить ответственность в слое Presentation:

* ViewModel будет отвественным за предостовления данных, управление им и хранение состояние UI
* UI контроллеры будет отвечать за отображение состояние данных.

ViewModel привязываются к одному UI контроллеру, которым они должны предоставлять данные. Чтобы создать ViewModel требуется классы LifecycleOwner и Lifecycle, который содержится в них:

* Lifecycle - объект который определяет состояние жизненного цикла Android
* LifecycleOwner - объект который содержит, такие как Activity и Fragment.

Когда вы создаете ViewModel, вам нужно предоставить компонент с LifecycleOwner. Это обычно Activity или Fragment. Предоставляя LifecycleOwner, вы установливаете связь между ViewModel и LifecycleOwner.

### Жизненный цикл ViewModel

Чтобы **ViewModel** мог держать данные вне зависимо от UI контроллера, жизненный цикл у ViewModel отличаются от **LifeCycleOwner**. Ниже приведен разница между жизненным циклом Activity и ViewModel.

![ViewModel lifecycle](/assets/codelabs/viewmodel-lifecycle.png)

Как мы видим, ViewModel существует до тех пор, пока Activity полностью не уничтожиться.
Для детального изучения ViewModel, вы можете ознакомиться [с данным статьей](https://medium.com/androiddevelopers/viewmodels-a-simple-example-ed5ac416317e).

### Создание ViewModels

Далее вы будете создавать ViewModel в проекте. Сначала мы попробуем добавить ViewModel для экрана AdvertListActivity.

1. Добавьте данные dependencies в файле `app/build.gradle`.
```gradle
dependencies {
  implementation "androidx.room:room-runtime:$lifecycle_version"
  implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
  kapt "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
}
```
2. Синхронизируем gradle.
3. Откройте `kz.kolesa.devfest.advertlist.AdvertListViewModel` и добавьте данный код
```kotlin
class AdvertListViewModel() : ViewModel() {
  fun getAdvertisements(): List<Advertisement> = emptyList()
}
```
4. Откройте `kz.kolesa.devfest.advertlist.AdvertListActivity` добавьте переменное AdvertListViewModel.
```kotlin
private lateinit var advertListViewModel: AdvertListViewModel
```
5. Создайте объект в `AdvertListActivity#onCreate`
```kotlin
advertListViewModel = ViewModelProviders
        .of(this)
        .get(AdvertListViewModel::class.java)
```

Когда первый раз создается *AdvertListActivity*, метод `ViewModelProviders.of` вызывается в `onCreate`. Данный метод создаст *AdvertListViewModel*, если еще не был создан. В дальнейшем, когда измененяет конфигурация и пересоздается Activity, *ViewModelProviders.of* вытащит объект, который был создан ранее в *AdvertListActivity*.

### LiveData

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata) - это класс держателя данных, который умеет следить за жизненным циклом. Он сохраняет значение и позволяет наблюдать за этим значением. Под капотом класса LiveData реализован [Observable pattern](https://refactoring.guru/ru/design-patterns/observer).

![Observer pattern](/assets/codelabs/Observer.png)
> Картинка взята с сайта [Refactoring.guru](https://refactoring.guru/ru/design-patterns/observer)

Если брать Observable pattern, LiveData это наш *Subject*, а *наблюдатели* это объекты которые наследуются от [Observer](https://developer.android.com/reference/android/arch/lifecycle/Observer.html). Чтобы подписаться на LiveData требуется предоставить `LifecycleOwner` - LiveData следит за его жизненным циклом и сам отпишет подписчиков при его уничтожение. Вы можете подписываться на LiveData, не предоставляя LifeCycleOwner, но в таком случае вам нужно отписываться от него вручную в нужный момент.

### Создание LiveData

*AdvertDetailsActivity* будет следить за LiveData который будет содержаться в AdvertListViewModel. Обычный LiveData не имеет возможность устанавливать новое значение. Вам понадобиться использовать [MutableLiveData](https://developer.android.com/reference/android/arch/lifecycle/MutableLiveData) чтобы могли уведомлять новыми значениями при получения данных от источника данных.

Выполните следующие шаги чтобы создать LiveData и наблюдать за ним:

1. В *AdvertListViewModel* добавьте LiveData, который будет содержать список объявлении.
```kotlin
private val advertListLiveData = MutableLiveData<List<Advertisement>>()
```
2. Поменяйте метод *AdvertListViewModel#getAdvertisements* на LiveData
```kotlin
fun getAdvertisements(): LiveData<List<Advertisement>> = advertListLiveData
```
3. В *AdvertListActivity* подпишитесь на LiveData с момента, где у вас создается *AdvertListViewModel*.
```kotlin
advertListViewModel.getAdvertisements.observe(this, Observer { advertisements ->
            // Обновляем UI
        })
```
4. По умолчанию значение у LiveData стоит **null**, поэтому вам нужно будет проверить его прежде чем обновлять UI.
```kotlin
// обновляем UI
advertisements?.let { updateView(it) }
```

Таким образом UI контроллер будет получать актуальные данные даже при изменении конфигурации и пересоздания Activity/Fragment. При вызове метода `MutableLiveData#setValue`, LiveData будет уведомлять всех подписчиков с данным значением. В вашем случае за списком объявлении следит *AdvertListActivity*, таким образом мы сможем отобразить данные в UI.

> Важно отметить что у `MutableLiveData` содержаться 2 метода для указания значении в LiveData: [setValue()](https://developer.android.com/reference/android/arch/lifecycle/MutableLiveData.html#setValue(T)) и [postValue()](https://developer.android.com/reference/android/arch/lifecycle/MutableLiveData.html#postValue(T)). Разница в том, что *setValue* можно вызывать только в главном потоке(*UI thread*). А для *postValuу()* можно вызывать вне зависимости от контекста текущего потока.

### Обновление данных LiveData в действии

Далее вы узнаете как сходить за данными в фоновом потоке и обновить LiveData на главном потоке. Для асинхронной задачи в проекте предоставлена библиотека [Kotlin Coroutines](https://github.com/Kotlin/kotlinx.coroutines). Вдаваться в эту библиотеку вы не будете, но если вам интересно, можете поизучить [по подробнее](https://kotlinlang.org/docs/reference/coroutines-overview.html).

1. Добавляем сервис для загрузки данных из внешнего источника. Объект **ADVERTISEMENT_SERVICE** уже доступен в проекте по singleton.
```kotlin
class AdvertListViewModel(
        private val advertisementService: AdvertisementService = ADVERTISEMENT_SERVICE
)
```
2. Данные нужно загружать только в том случае, если у *advertListLiveData* значение пустое.
```kotlin
fun getAdvertisements(): LiveData<List<Advertisement>> = advertListLiveData.apply {
        if (value == null) {
            // Загружаем данные
        }
}
```
3. Воспользуйтесь функциями Kotlin Coroutines *launch* и *withContext* чтобы выполнить асинхронную задачу для загрузки данных.
```kotlin
// Загружаем данные
requestAdvertisements()
...
private fun requestAdvertisements() {
        launch(UI) {
            val advertisements = withContext(DefaultDispatcher) {
                advertisementRepository.searchAdvertisement()
            }
            // Уведомляем LiveData о новом значении
        }
}
```
4. Данные получим в главном потоке. Об этом позаботиться Coroutine, который реализован в методе requestAdvertisements(). Поэтому мы можем вызвать метод `setValue()`.
```kotlin
// Уведомляем LiveData о новом значении
advertListLiveData.value = advertisements
// или postValue() если захотите обновить данные вне главного потока
advertListLiveData.postValue(advertisements)
```

Таким образом LiveData будет уведомлять всех подписчиков(*AdvertListActivity*), когда успешно вернется результат запроса во внешний источник данных. Даже при изменении конфигурации, объект ViewModel останется прежними и продолжит ожидать ответ по запросу. Попробуйте запустить приложение и проверить это на деле.
