---
layout: post
title: "Code Labs: Разработка простого приложения, используя Android Architecture Components"
date: 2018-10-12 10:00:00 +0600
categories: codelabs
---

> DevFest, 20 Окт. 2018
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
которое извлекает данные из удаленного источника, сохраняет локально и отображает их пользователю.

### Этапы создания приложения

1. Исследуйте принципы в руководстве по архитектуре приложений
2. Используйте библиотеку Lifecycle, которая включает LiveData и ViewModel
3. Используйте библиотеку сохранения данных Room

Запросы в API и UI часть будут заранее доступны в проекте

### Что вам понадобится

1. Android Studio 3.0 или новее
2. Знакомство с созданием Android-приложений и жизненным циклом активности Android
3. Базовое знание SQLite
  * Например, вы должны иметь возможность писать операторы SELECT с предложением WHERE.
  * Базовое знание потоковой обработки и обработки асинхронных задач в Android

### Скачиваем исходный код

1. `git clone https://github.com/JohnMars/google-devfest-almaty-2018.git`
2. Открыть в Android Studio

Всю историю commit можно увидеть в master ветке

## 3. Архитектура с использованием AAC

![Architecture](/assets/codelabs/architecture.png)

**UI Controller** - это действия или фрагменты. Единственное задача UI контроллера - это знать, как отображать данные и передавать события UI, такие как нажитие кнопки. UI контроллеры не содержат данных UI, а также не работают с данными напрямую.

**ViewModels и LiveData** - эти классы представляют все данные, необходимые для отображения UI.

**Repository** - этот класс является единственным источником для всех данных нашего приложения и действует как чистый API для взаимодействия с UI. ViewModels просто запрашивает данные из Repository. Нам не нужно беспокоиться о том, должен ли Repository загружаться из базы данных или сети или как и когда сохранять данные. Repository управляет всем этим. В рамках этой задачи Repository является посредником между различными источниками данных.

**Remote Data Source** - управляет данными из удаленного источника данных, такого как Интернет.

**Model** - управляет локальными данными, хранящимися в базе данных.

### Архитектура приложения копии Kolesa

Как в итоге должно выглядеть приложение, которое вы будете разрабатывать

![AdvertListActivity and AdvertDetailsActivity](/assets/codelabs/dev-fest-app.png)

Будет 2 Activity (AdvertListActivity и AdvertDetailsActivity) с двумя ViewModel (AdvertListViewModel и AdvertDetailsViewModel). Они будут обращаться к классу Repository (AdvertisementRepository), который будет управлять связью между базой данных SQLite и сетевым источником данных. В вашем расположении будет готовый AdvertisementNetworkDataSource в проекте с готовыми данными. AdvertisementNetworkDataSource будет отправлять запросы чтобы получить данные объявления [с замоканного сервера](https://stash.kolesa-team.org/projects/MAPG/repos/sample/). В нем содержится урезанные данные с сайта колёс.

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
Каркасы этих классов уже заготовлены в проекте, который вы скачали ранее.

![Dev Fest Kolesa Architecture](/assets/codelabs/kolesa-architecture.png)

## 4. ViewModel и LiveData

Класс ViewModel  предназначен для содержания и управления данными UI в рамках жизненного цикла. Это позволяет хранить данные во время изменении конфигурации такие как повороты экрана. Разделив данные UI от UI контроллеров, вы можете разделить ответственность в слое Presentation:

* ViewModel будет отвественным за предоставление данных, управление ими и хранение состояния UI
* UI контроллеры будут отвечать за отображение состояния данных.

ViewModel привязываются к одному UI контроллеру, которому они должны предоставлять данные. Чтобы создать ViewModel требуется классы LifecycleOwner и Lifecycle, который содержится в них:

* Lifecycle - объект который определяет состояние жизненного цикла Android
* LifecycleOwner - объект который содержит, такие как Activity и Fragment.

Когда вы создаете ViewModel, вам нужно предоставить компонент с LifecycleOwner. Это обычно Activity или Fragment. Предоставляя LifecycleOwner, вы установливаете связь между ViewModel и LifecycleOwner.

### Жизненный цикл ViewModel

Чтобы **ViewModel** мог держать данные вне зависимо от UI контроллера, жизненный цикл у ViewModel отличается от **LifeCycleOwner**. Ниже приведена разница между жизненным циклом Activity и ViewModel.

![ViewModel lifecycle](/assets/codelabs/viewmodel-lifecycle.png)

Как мы видим, ViewModel существует до тех пор, пока Activity полностью не уничтожится.
Для детального изучения ViewModel, вы можете ознакомиться [с данной статьей](https://medium.com/androiddevelopers/viewmodels-a-simple-example-ed5ac416317e).

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
4. Откройте `kz.kolesa.devfest.advertlist.AdvertListActivity` добавьте переменную AdvertListViewModel.
```kotlin
private lateinit var advertListViewModel: AdvertListViewModel
```
5. Создайте объект в `AdvertListActivity#onCreate`
```kotlin
advertListViewModel = ViewModelProviders
        .of(this)
        .get(AdvertListViewModel::class.java)
```

Когда первый раз создается *AdvertListActivity*, метод `ViewModelProviders.of` вызывается в `onCreate`. Данный метод создаст *AdvertListViewModel*, если он еще не был создан. В дальнейшем, когда измененяется конфигурация и пересоздается Activity, *ViewModelProviders.of* вытащит объект, который был создан ранее в *AdvertListActivity*.

### LiveData

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata) - это класс держателя данных, который умеет следить за жизненным циклом. Он сохраняет значение и позволяет наблюдать за этим значением. Под капотом класса LiveData реализован [Observable pattern](https://refactoring.guru/ru/design-patterns/observer).

![Observer pattern](/assets/codelabs/observer.png)
> Картинка взята с сайта [Refactoring.guru](https://refactoring.guru/ru/design-patterns/observer)

Если брать Observable pattern, LiveData это наш *Subject*, а *наблюдатели* это объекты которые наследуются от [Observer](https://developer.android.com/reference/android/arch/lifecycle/Observer.html). Чтобы подписаться на LiveData требуется предоставить `LifecycleOwner` - LiveData следит за его жизненным циклом и сам отпишет подписчиков при его уничтожении. Вы можете подписываться на LiveData, не предоставляя LifeCycleOwner, но в таком случае вам нужно отписываться от него вручную в нужный момент.

### TASK: Создание LiveData

*AdvertDetailsActivity* будет следить за LiveData, который будет содержаться в AdvertListViewModel. Обычный LiveData не имеет возможность устанавливать новое значение. Вам понадобится использовать [MutableLiveData](https://developer.android.com/reference/android/arch/lifecycle/MutableLiveData) чтобы могли уведомлять новыми значениями при получения данных от источника данных.

Выполните следующие шаги чтобы создать LiveData и наблюдать за ним:

1. В *AdvertListViewModel* добавьте LiveData, который будет содержать список объявлений.
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

Таким образом UI контроллер будет получать актуальные данные даже при изменении конфигурации и пересоздании Activity/Fragment. При вызове метода `MutableLiveData#setValue`, LiveData будет уведомлять всех подписчиков с данным значением. В вашем случае за списком объявлений следит *AdvertListActivity*, таким образом мы сможем отобразить данные в UI.

> Важно отметить что у `MutableLiveData` содержатся 2 метода для указания значении в LiveData: [setValue()](https://developer.android.com/reference/android/arch/lifecycle/MutableLiveData.html#setValue(T)) и [postValue()](https://developer.android.com/reference/android/arch/lifecycle/MutableLiveData.html#postValue(T)). Разница в том, что *setValue* можно вызывать только в главном потоке(*UI thread*). А для *postValue()* можно вызывать вне зависимости от контекста текущего потока.

### TASK: Обновление данных LiveData в действии

Далее вы узнаете как сходить за данными в фоновом потоке и обновить LiveData на главном потоке. Для асинхронной задачи в проекте предоставлена библиотека [Kotlin Coroutines](https://github.com/Kotlin/kotlinx.coroutines). Вдаваться в эту библиотеку вы не будете, но если вам интересно, можете поизучить [по подробнее](https://kotlinlang.org/docs/reference/coroutines-overview.html).

1. Добавляем сервис для загрузки данных из внешнего источника. Объект **ADVERTISEMENT_SERVICE** уже доступен в проекте по singleton.
```kotlin
class AdvertListViewModel(
        private val advertisementService: AdvertisementService = ADVERTISEMENT_SERVICE,
        private val apiAdvertisementMapper: ApiAdvertisementMapper = ApiAdvertisementMapper()
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
                val searchResponse = advertisementService.searchAdvertisements().execute()
                searchResponse.body()?.map { apiAdvertisementMapper.map(it) }
            }
            // Уведомляем LiveData о новом значении
        }
}
```
4. Данные получим в главном потоке. Об этом позаботится Coroutine, который реализован в методе requestAdvertisements(). Поэтому мы можем вызвать метод `setValue()`.
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
| 1  | Toyota Camry 2018 года | 10000 | foo | foo  | 1234 | Руль:слева,... | https://photos-b-kl.kcdn.kz/68/689db250-d36d-4bf5-9897-1daf97939a1f/1-full.jpg,... | +77071234567,...|  

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

Далее вам нужно создать компонент [**@Dao**](https://developer.android.com/reference/android/arch/persistence/room/Dao.html). Dao буквально переводится как 'Объект Доступа к Данным'. Аннотацию Dao чаще всего указываем для interface, но так же можно объявить для abstract class. В функциях которые будете создавать для реализации доступа к БД, вы можете добавлять все стандартные операции CRUD: `@Query`, `@Insert`, `@Update` и `@Delete`. Подробную документацию можете посмотреть [тут](https://developer.android.com/training/data-storage/room/accessing-data).

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
  abstract class KolesaDatabase : RoomDatabase() {

        abstract fun advertisementDao(): RoomAdvertisementDao
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
```
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
1. Откройте файл `kz.kolesa.devfest.data.room.RoomConverter` и добавьте аннотацию `@TypeConverter` для каждого метода.
2. Откройте файл `kz.kolesa.devfest.data.room.KolesaDatabase` и добавьте аннотацию `@TypeConverters`.
3. В аннотации `@TypeConverters` укажите объекты указаны в файле **RoomConverter**
```kotlin
@TypeConverters(DateConverter::class, AdvertisementConverter::class)
```

Таким образом вы завершили внедрение библиотеки Room в проект. Теперь ваши запросы в SQLite будут надежными, так как во время компиляции Room проверяет валидность ваших SQL запросов.

## 6. Шаблон проектирования Repository

Repository отвечают за обработку данных.
* Они предоставляют чистый API для остальной части приложения для данных приложения.
* Где взять данные и какие вызовы API делать при обновлении данных.
* Они являются посредниками между различными источниками данных (Room и API)

Основная вашего Repository будет записи загруженных объявлении с **RemoteDataSource** и получить записанное объявление для просмотра детали в **AdvertDetailsActivity**.
![Repository flow](/assets/codelabs/repository.png)

В вашем случае класс Repository будет управлять обменом данными между вашим недавно созданным **AdvertisementDao**, который дает вам доступ ко всему в базе данных и к **RemoteDataSource**.

Нкито, кроме класса Repository, не будет напрямую связываться с базой данных или сетевыми пакетами, а пакеты данных и сети не будут связываться с классами за пределами их ответствуенности. Таким образом, в Repository будет API для получения данных для отображения на экранах *AdvertListActivity* и *AdvertDetailsActivity*.

Обращение к Repository будете реализовать подобной схемой:
* Ходим в RemoteDataSource за всеми объявлениями
![Repository RemoteDataSource](/assets/codelabs/Repository-flow-1.png)
* Записываем полученные объявлении в KolesaDatabase
![Repository KolesaDatabase](/assets/codelabs/Repository-flow-2.png)
* Ответ от RemoteDataSource оборвется, мы вы обратимся за объявлиями в KolesaDatabase
![Repository KolesaDatabase](/assets/codelabs/Repository-flow-no-connection.png)

### TASK: Реализация

В следующем шаге вам необходимо записать полученные объявлении от *RemoteDataSource* в *KolesaDatabase*. Так же вам необходимо передать *AdvertisementToRoomMapper*. Mapper это специальный класс для преобразования моделей между слоями, например, от модели БД к модели домена. Обычно они называются XxxMapper и имеют один метод с Map имен (или преобразованием / преобразованием). В ващем случае конвертирует *Advertisement* в *RoomAdvertisement*.

1. Откройте `kz.kolesa.devfest.data.DefaultAdvertisementRepository` и добавьте *KolesaDatabase*, *AdvertisementToRoomMapper* и *RoomToAdvertisementMapper* в конструктор.
```kotlin
class DefaultAdvertisementRepository(
        ...
        private val kolesaDatabase: KolesaDatabase = KolesaDatabase.get(),
        private val advertToRoomMapper: AdvertisementToRoomMapper = AdvertisementToRoomMapper(),
        private val roomToAdvertisementMapper: RoomToAdvertisementMapper = RoomToAdvertisementMapper()
): AdvertisementRepository
```
2. Запишите объект *Advertisement* полученное от *RemoteDataSource*.
```kotlin
  override fun searchAdvertisement(): List<Advertisement> {
      ...
      val advertisementDao = kolesaDatabase.advertisementDao()
      val advertisementList = response.body()?.map {
          val advertisement = apiAdvertisementMapper.map(it)
          val roomAdvertisement = advertToRoomMapper.map(advertisement)
          advertisementDao.insertAll(roomAdvertisement)

          advertisement
      } ?: emptyList()

      return advertisementList
  }
```
3. Если же *RemoteDataSource* не смог вытащить список объявлении по каким-то причинами, вы можете попробовать вытащить все объявлении которые записаны в *KolesaDatabase*. Для этого мы переделаем `emptyList()` на обращением за данным по методу *AdvertisementDao*.
```kotlin
override fun searchAdvertisement(): List<Advertisement> {
  ...
    ?: getLocalAdvertisements()
  ...
}    
private fun getLocalAdvertisements(): List<Advertisement> {
    return kolesaDatabase.advertisementDao().getAll().map {
        roomToAdvertisementMapper.map(it)
    }
}
```
4. Откройте `kz.kolesa.devfest.data.room.AdvertisementDao` и добавьте метод для поиска одного объекта *Advertisement*

5. Возвращаемся в Repository. Вытащите *Advertisement* из *KolesaDatabase* при вызове метода `getAdvertisement(id: Long): Advertisement?`
```kotlin
override fun getAdvertisement(id: Long): Advertisement? {
    val advertisementDao = kolesaDatabase.advertisementDao()
    val localAdvertisement = advertisementDao.find(id).firstOrNull()

    return roomToAdvertisementMapper.map(localAdvertisement)
}
```
6. Если же в *KolesaDatabase* отсутствует *RoomAdvertisement*, то вам необходимо обратиться за ним из *RemoteDataSource*.
```kotlin
    return if (localAdvertisement == null) {
        requestAdvertisement(id)?.apply {
            advertisementDao.insertAll(advertToRoomMapper.map(this))
        }
    } else {
        roomToAdvertisementMapper.map(localAdvertisement)
    }
```
7. Осталось вам добавить класс *AdvertisementRepository* в *AdvertListViewModel*. Объект *AdvertisementRepository* уже объявлен в *DefaultAdvertisementRepository* виде переменной **val DEFAULT_ADVERTISEMENT_REPOSITOR**. В *AdvertListViewModel* удалите **advertisementService** так как единственным источником истинных данных должен быть получен от Repository.
```kotlin
class AdvertListViewModel(
        private val advertisementRepository: AdvertisementRepository = DEFAULT_ADVERTISEMENT_REPOSITORY
) : ViewModel() {

    private fun requestAdvertisements() {
        launch(UI) {
            val searchResponse = withContext(DefaultDispatcher) {
                advertisementRepository.searchAdvertisements().execute()
            }
            advertListLiveData.value = searchResponse.body()
        }
    }
}
```

## 7. AdvertDetailsViewModel

Вы реализовали Repository в слое data layer. Теперь вам нужно показать детали объявления, которое будет отображаться в **AdvertDetailsActivity**. У этого *Activity* будет свой ViewModel **AdvertDetailsViewModel**. Реализацию ViewModel вы ознакомились при создание **AdvertListViewModel**, поэтому вам не должно составить труда понять его реализацию.

### TASK: Реализация AdvertDetailsViewModel

1. Откройте `kz.kolesa.devfest.advertdetails.AdvertDetailsViewModel` и создайте *LiveData* чтобы предоставить **Advertisement**.
```kotlin
class AdvertDetailsViewModel(
    ...
) : ViewModel() {
    val advertisementLiveData = MutableLiveData<Advertisement>()
}
```
2. Вытащите *Advertisement* из *AdvertisementRepository* и запишите его в **advertisementLiveData**.
```kotlin
    val advertisementLiveData = MutableLiveData<Advertisement>().apply {
        if (value == null) {
            requestAdvertisement(advertisementId)
        }
    }

    private fun requestAdvertisement(advertisementId: Long) {
        launch(UI) {
            val advertisement = withContext(DefaultDispatcher) {
                advertisementRepository.getAdvertisement(advertisementId)
            }
            advertisementLiveData.value = advertisement
        }
    }
```
3. Последним шагом будет отображение *Advertisement*, получаемое от **advertisementLiveData** в файле `kz.kolesa.devfest.advertdetails.AdvertDetailsActivity`.
```kotlin
    private fun observeLiveData() {
        ...
        advertDetailsViewModel.advertisementLiveData.observe(this, Observer { advertisement ->
            advertisement?.let { onAdvertisementUpdated(it) }
        })
    }
```

## 8. ViewModelProvider.Factory

Когда вам нужно передавать объекты во ViewModel при созданием, вам необходимо указать класс с interface [ViewModelProvider.Factory](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProvider.Factory). ViewModelProviders, который указываете в Activity, вытаскивает объект ViewModel из своего кэша либо инициализирует через рефлексию для создания ViewModel. Поэтому получается что при инициализации этот *ViewModelProviders* не знает как передать объект в конструктор ViewModel.

### TASK: Создание AdvertDetailsViewModelFactory

При нажатие на элемент в списке объявлении передается идентификатор выбранного объявления в *AdvertDetailsActivity*. При создание *AdvertDetailsViewModel* в этом Activity вы укажите **AdvertDetailsViewModelFactory**.

1. Откройте файл `kz.kolesa.devfest.advertdetails.AdvertDetailsViewModelFactory` и передайте идентифатор объявления в конструктор класса.
```kotlin
class AdvertDetailsViewModelFactory(
        private val advertisementId: Long
) : ViewModelProvider.Factory
```
2. Напишите реализацию interface **ViewModelProvider.Factory**.
```kotlin
override fun <T : ViewModel?> create(modelClass: Class<T>): T {
    @Suppress("UNCHECKED_CAST")
    if (modelClass.isAssignableFrom(AdvertDetailsViewModel::class.java)) {
        return AdvertDetailsViewModel(advertisementId) as T
    }

    throw IllegalArgumentException("Could not instantiate " +
            AdvertDetailsViewModel::class.java.simpleName)
}
```
3. Укажите **AdvertDetailsViewModelFactory** в качестве *Factory* при создание *ViewModel* в файле `kz.kolesa.devfest.advertdetails.AdvertDetailsActivity`.
```kotlin
    private fun initViewModel() {
      val advertisementId = getAdvertisementId()
      val viewModelFactory = AdvertDetailsViewModelFactory(advertisementId)
      advertDetailsViewModel = ViewModelProviders
              .of(this, viewModelFactory)
              .get(AdvertDetailsViewModel::class.java)
    }
```

## 9. Итог разработки приложения с помощью Android Architecture Components

#### Поздравляю! Вы дошли до конца и реализовали приложение базовой версии Kolesa.

В этом code labs узнали о компонентах Android Architecture: Lifecycle Owner, ViewModel, LiveData, Room. Так же вы рассмотрели подходы реализации чистой архитектуры. В чистой архитектуре очень важно абстрагировать реализации каждого класса чтобы они отвечали конкретно за одну логику.

* UI контроллеры - Activity или Fragment должен отвечать только за отображение данных, полученных от ViewModel.
* ViewModel - выполняет роль за хранением данных UI контроллера и обращение к слою data.
* LiveData - уведомляет подписчиков при получение новых данных.
* Repository - является связующем звеном для ViewModel, решает откуда достать данные.
* Room - реализует локальное хралище данных в SQLite.
* RemoteDataSource - отвечает за загрузку даных из API сервера.

### Что дальше

В приложении отображение данных реализован и навигация по экрану реализован по простому способу - передаем полученые данные от LiveData в **RecyclerView.Adapter**, а навигация происходит по Intent. Эти вещи вы бы переписать с помощию дополнительных компонентов Android Architecture Components:

* [DataBinding](https://developer.android.com/topic/libraries/data-binding/) - это библиотека, которая позволяет связывать компоненты UI в ваших XML с источниками данных в вашем приложении, используя *декларативный формат*, а не программно.
* [Navigation](https://developer.android.com/topic/libraries/architecture/navigation/) - упрощает реализацию навигации в приложении для Android. В данном библиотеке вам нужно будет заменить UI контроллер с Activity на Fragment.
* [Paging](https://developer.android.com/topic/libraries/architecture/paging/) - упрощает загрузку данных в RecyclerView, например для реализации бесконечной загрузки при листание списка.
* [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager/) - упрощает определение отложенных, асинхронных задач и их запуск. Эти API позволяют вам запускать синхронизации данных периодически и на фоне.

На этом code labs по теме Android Architecture Components завершен. Вы можете посмотреть весь этап разработки по коммитам [в данном Git Repository](https://github.com/JohnMars/google-devfest-almaty-2018/commits/master).

## 10. Использованные материалы

1. [Google Code Labs: Build an App with Architecture Components](https://codelabs.developers.google.com/codelabs/build-app-with-arch-components/index.html)
2. [Android Developers: Guide to app architecture](https://developer.android.com/jetpack/docs/guide)
3. [Florina Muntenescu: 7 Pro tips for Room](https://medium.com/androiddevelopers/7-pro-tips-for-room-fbadea4bfbd1)

> Acknowledgement: this code labs was mostly based on code labs by the Google https://codelabs.developers.google.com/codelabs/build-app-with-arch-components
