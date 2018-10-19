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

### TASK: Создание ViewModels

Далее вы будете создавать ViewModel в проекте. Сначала мы попробуем добавить ViewModel для экрана AdvertListActivity.

1. Добавьте данные dependencies в файле `app/build.gradle`.
```gradle
dependencies {
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

### TASK: Создание LiveData

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

### TASK: Обновление данных LiveData в действии

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

## 5. Room: Локальное хранилище

Работа с классом SQLiteOpenHelper всегда состовляет мучение: создание таблицы, миграция, SQL запросы, чтение данных по Cursor и т.д. Поэтому Google предложил удобную библиотеку для хранения объектов в SQLite - **Room**

* Меньше кода шаблона по сравнению со встроенными API.
  * Не нужно использовать ContentValues или Cursors
  * Используются аннотации для связки таблиц
* Проверка выполнения SQL происходит во время компиляции
  * Неверные запросы SQL будут выявлены не во время выполнения приложения
* Наблюдение за данными можно реализовать с помощью LiveData

### Room: Аннотация

Room использует аннотацию `@` для определения структуры базы данных. Основные аннотации приведены ниже:

* **@Entity** - этот компонент определяет схему таблицы базы данных.
* **@DAO** - этот компонент представляет класс или интерфейс как объект доступа к данным (DAO)
* **@Database** - свойство базы данных. В этом классе вы определяете список объектов для базы данных и объектов доступа к данным (DAO) для базы данных.

### TASK: Добавляем Room в проект

Для добавления Room в проект, выполните следующие шаги

1. Откройте `app/build.gradle` и добавьте следующие dependencies.
```gradle
dependencies {
  ...
  implementation "androidx.room:room-runtime:$room_version"
  kapt "androidx.room:room-compiler:$room_version"
}
```
2. Нажмите кнопку для синхронизации gradle.

Данные dependencies предоставляет библиотеку Room от Architecture Components. Благодаря annotation processing (*kapt*), отмеченное как room-compiler, во время компиляции приложения у вас будут проверяться валидность компонентов Room.

### TASK: Создание сущности

При работе с базами данных будет удобно, если вы предвадительно оформите таблицы, которые будете создавать в вашем приложении. В случае с копием приложения Колёса у нас будут такая таблица:

| id | title | price | specification | text | date | parameters     | photos  | phones  |
|----|-------|-------|---------------|------|------|----------------|---------|---------|
| 1  | BMW   | 10000 | foo           | foo  | 1234 | Руль:слева,... | url,... | 1234,...|  

Вы ранее работали с реляционными базами данных, такие как SQL, вы должны понимать схему этой таблицы.
Такая таблица виде POJO выглядело бы вот так:

```kotlin
package kz.kolesa.devfest.data.room
...

data class RoomAdvertisement(
        val id: Long,
        val title: String,
        val price: Long,
        val specification: String,
        val text: String,
        val date: Date,
        val parameters: List<RoomParameter>,
        val photos: List<RoomPhoto>,
        val phones: List<RoomPhone>
) {

    data class RoomPhoto(
            val url: String
    )

    data class RoomParameter(
            val label: String,
            val value: String
    )

    data class RoomPhone(
            val number: String
    )
}
```

Теперь попробуйте добавить аннотацию Entity для POJO класса RoomAdvertisement

1. Добавьте аннотацию **@Entity** с название таблицы **advertisements**.
```kotlin
@Entity(tableName = "advertisements")
data class RoomAdvertisement(
  ...
)
```
2. Укажите **primary key** в аргументе аннотации **@Entity**. В вашем случае это будет **id**.
```kotlin
@Entity(
  tableName = "advertisements",
  primaryKeys = ["id"]
)
```

Благодаря языку программирования Kotlin, процесс работы с создание POJO объектов упростилось гораздо проще. Все эти `data class` при копиляции создаст вам getter методы для каждой переменной в данном классе. Если захотите чтобы POJO объект был не только для чтения, а read/write, то вы можете попробовать поменять val на var.

Дополнительные компоненты для *Entity* вы можете найти [в данном документации](https://developer.android.com/training/data-storage/room/defining-data).

### Путь Дао - создаем Database Access Object

Далее вам нужно создать компонент [**@Dao**](https://developer.android.com/reference/android/arch/persistence/room/Dao.html). Dao буквально переводиться как 'Объект Доступа к Данным'. Аннотацию Dao чаще всего указываем для interface, но так же можно объявить для abstract class. В функциях которые будете создавать для реализации доступа к БД, вы можете добавлять все стандартные операции CRUD: `@Query`, `@Insert`, `@Update` и `@Delete`. Подробную документацию можете посмотреть [тут](https://developer.android.com/training/data-storage/room/accessing-data).

### TASK: Создать Dao

1. Создайте **interface** с названием **RoomAdvertisementDao** в папке **kz.kolesa.devfest.data.room**
2. Добавьте аннотациюю **@Dao** для **RoomAdvertisementDao**
3. Создайте метод **insertAll**, который принимает несколько количество **Advertisement**.
4. Добавьте аннотацию **insert** для метода **insertAll**.
5. Добавьте в аннотацию **insert** аргумент **onConflict** со значением **OnConflictStrategy.REPLACE**
6. Добавьте метод **getAllAdvertisements** с аннотацием **Query**. SQL запрос будет выглядет так `SELECT * FROM advertisements`

В итоге должен получится такой класс
```kotlin
package kz.kolesa.devfest.data.room
...

@Dao
interface RoomAdvertisementDao {

    @Query("SELECT * FROM advertisements")
    fun getAll(): List<RoomAdvertisement>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insertAll(vararg advertisement: RoomAdvertisement)
}
```

### TASK: Создать базу данных

Создание базы данных так же делается с помощью аннотации. Указав анотацию **@DataBase** к классу, наследующие **RoomDatabase**, у вас будет готово экземпляр объекта для чтения и записи SQLite базы.

Наличие нескольких экземпляров базы данных вызывает проблемы, если, например, вы пытаетесь прочитать базу данных одним экземпляром во время записи в другой экземпляр. Чтобы убедиться, что вы создаете только один экземпляр RoomDatabase, ваш класс базы данных должен быть Singleton.

1. В файле **kz.kolesa.devfest.data.room.KolesaDatabase** создайте **абстрактный класс**
2. Сделайте класс **KolesaDatabase** наследником от [RoomDatabase](https://developer.android.com/reference/android/arch/persistence/room/RoomDatabase)
3. Добавьте аннотацию `@Database` к классу
4. Укажите значение **version** и **entitiy** для аннотации `@Database`.
  * Так как мы только создаем БД, мы укажем версию 1
  * В значениях **entity** необходимо указать список классов, которые отметили с аннотацием `@Entity`
5. Добавить абстрактный метод `abstract fun advertisementDao(): AdvertisementDao` в классе
```kotlin
@Database(
        entities = [RoomAdvertisement::class],
        version = 1
)
@TypeConverters(DateConverter::class, AdvertisementConverter::class)
abstract class KolesaDatabase : RoomDatabase() {
  ...
}
```
6. Сделайте KolesaDatabase как Singleton.
```kotlin
companion object {

        private const val KOLESA_DATABASE_NAME = "kolesa"
        private lateinit var instance: KolesaDatabase

        fun get(): KolesaDatabase = instance

        fun initialize(context: Context) {
            instance = create(context)
        }

        private fun create(context: Context): KolesaDatabase {
            TODO()
        }
}
7. Создайте KolesaDatabase через `Room.databaseBuilder`.
```kotlin
private fun create(context: Context): KolesaDatabase {
        return Room.databaseBuilder(
                context.applicationContext,
                KolesaDatabase::class.java,
                KOLESA_DATABASE_NAME
        ).build()
}
```
7. Инициализируйте KolesaDatabase в классе `kz.kolesa.devfest.KolesaApplication`.
```kotlin
override fun onCreate() {
        super.onCreate()
        KolesaDatabase.initialize(this)
}
```

Теперь у вас есть БД с таблицой указанной в *entities*.

### Converter

В классе `RoomAdvertisement` есть такие объекты как `Date`, `List<RoomParameter>`, `List<RoomPhoto>`, `List<RoomPhone>`. Но вы не сможете записать их в БД так как у SQLite [нет указанные типы данных](https://sqlite.org/datatype3.html). Для этого вам необходимо серелисериализовать и десериализовать при обращение к SQL - у Room есть компонент [**@TypeConverter**](https://developer.android.com/reference/android/arch/persistence/room/TypeConverter).

Для преобразования между типами Java/Kotlin и типами данных, поддерживаемыми SQLite, вам необходимо определить методы преобразования и указать Room о них через аннотацию `@TypeConverter`. Для этого понадобится:
* Создать класс который содержит методы для конвертации из одного типа в другой тип
* Указать этим методом аннотацию `@TypeConverter`
* Добавить аннотацию `@TypeConverters` в классе, где реализован ваш БД с аннотацией `@Database`

### TASK: Добавить компоненты TypeConverter

У вас есть несколько типов, которые не поддерживаются - Date и список(*List*) с объектами. Один из вариантов записи этих данных в SQLite это:
* Date в Long и Long в Date когда вытаскиваем данные
* Объекты в List запишем виде String(Text) через разделитель *,*
* Так как у Parameter существует переменные *label* и *value*, запишем их в формате `{label}:{value}` при конвертации в String

Вы определись как будете записывать и считывать данные в БД, теперь попробуйте выполнить следующие шаги:
1. Откройте файл `kz.kolesa.devfest.data.room.RoomConverter` и откомментируйте методы, которые отмечены с аннотацием `@TypeConverter`.
2. Откройте файл `kz.kolesa.devfest.data.room.KolesaDatabase` и добавьте аннотацию `@TypeConverters`.
3. В аннотации `@TypeConverters` укажите объекты указаны в файле **RoomConverter**
```kotlin
@TypeConverters(DateConverter::class, AdvertisementConverter::class)
```

Таким образом вы завершили внедрение библиотеки Room в проект. Теперь ваши запросы в SQLite будут надежными, так как во время компиляции Room проверяет валидность ваших SQL запросов.
