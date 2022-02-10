# Гайд по Kensuke(stats-service)
![screenshot](https://i.ibb.co/mygNLxm/imgonline-com-ua-Blur-ln-Mu-Ny-F3kaz73vy-1.png)

## Что такое Kensuke?
[Kensuke](https://github.com/delfikpro/kensuke) или stats-service - хранилище данных(логично да?). Позволяет максимально просто связать данные игроков между серверов(победы, убийства и другие данные).

## Как установить?
Заходим в `build.gradle` и пишем следующие:
```
repositories {
  mavenLocal()
  mavenCentral()
  maven {
    url 'https://repo.implario.dev/public/'
  }
}

dependencies {
  implementation 'dev.implario:kensuke-client-bukkit:2.1.10'
}
```
И все!

## Настройка Kensuke в плагине 
Самое интересное только начинается(в хорошем смысле)!

Заходим в главный класс плагина и пишем:

# Kotlin:
```kotlin
lateinit var kensuke: Kensuke
    val statScope = Scope("topgame", UserData::class.java)
    val userManager = BukkitUserManager(
        listOf(statScope),
        { session, context -> User(session, context.getData(statScope)) },
        { user, context -> context.store(statScope, user.stat) }

    )
    override fun onEnable() {
        //Сетапим кенсуке
        kensuke = BukkitKensuke.setup(this)
	//Добавляем ифну о игроке
        kensuke.addGlobalUserManager(userManager)
        userManager.isOptional = true
	
	//Методы, чтобы брать игрока
        fun getUser(uuid: UUID): User? = userManager.getUser(uuid)
        fun getUser(player: Player) = getUser(player.uniqueId)
    }
```
# Java:
```java
public static final Scope<SomeGameStats> statsScope = new PlayerScope<>("somegame", SomeGameStats.class);

	private UserPool<SomeGameUser> userManager;

	@Override
	public void onEnable() {
	        //Регаем кенсуке
		IKensuke statService = new Kensuke(IServerPlatform.get(), KensukeConnectionData.fromEnvironment());
		CoreApi.get().registerService(IKensuke.class, statService);
		//Добавляем инфу о игроке
		statService.useScopes(statsScope);
		statService.setDataRequired(false);
		
		this.userManager = statService.registerUserManager(
				reader -> new SomeGameUser(reader.getUuid(), reader.getName(), reader.getData(statsScope)),
				SomeGameUser::save);
```


Далее создаем 2 класса `User` и `UserData`:


# Kotlin `User`:
```kotlin
class User(session: KensukeSession, stat: UserData?) : IBukkitKensukeUser {
    //Статистика
    var stat: UserData?
    var wins = 0
    
    //Методы, которые облегчат вам жизнь
    private var session: KensukeSession
    private var player: Player? = null
    override fun getSession(): KensukeSession { return session }
    override fun getPlayer(): Player? = player
    override fun setPlayer(p0: Player?) { player = p0 }

    init {
        this.stat = stat ?: UserData(
	//Указываем статистику в порядке как выше(var stat, wins).
            UUID.fromString(session.userId),
            0,
        )
        this.session = session
    }
}
```
# Kotlin `UserData`:
```kotlin
data class UserData(
    //Делаем статистику как в User(только без stat: UserData?)
    var uuid: UUID,
    var wins: Int,
)
```

# Java `User`:
```java
//Да да lombok
public class AnotherUser extends BukkitKensukeUser {

    @Getter
    @Delegate
    private final AnotherData data;

    //AnotherData == UserData 
    public AnotherUser(Session session, AnotherData data) {
        super(session);
        this.data = data;
    }
}
```
# Java `UserData`:
```java
//Да да lombok
@Data
@AllArgsConstructor
public class AnotherData {
    //Статистика
    private int wins;
    private double rating;
    private Map<String, String> complexStuff;

}
```

Теперь мы создали данные игрока и храним их!

## Как получить данные??
Все очень просто, сейчас покажу на примере сообщения игроку:

# Kotlin:
```kotlin
   // app меняем на ваш главный класс
   val user = app.userManager.getUser(player.uniqueId)
   player.sendMessage("Побед: " + user.stat!!.win)
```

# Java:
```java
   SomeGameUser user = userManager.getUser(player);
   player.sendMessage("§eРейтинг: §f" + player.getRating());
```

## Мелочь в start.sh файле
Добавляем:
```
export KENSUKE_HOST='your_host'
export KENSUKE_PORT='your_port'
export KENSUKE_LOGIN='your_login'
```
(Данные от Kensuke просите у [DelfikPro](https://vk.com/delfikpro))

## Источники
Kensuke - https://github.com/delfikpro/kensuke
Kensuke client - https://github.com/delfikpro/kensuke-client

Спасибо за помощь в изучении Kensuke - [CrazyL3gend](https://github.com/CrazyL3gend), [Func](https://github.com/funcid), [DelfikPro](https://github.com/delfikpro)

### Как появилось название Kensuke? (Эту историю нам рассказал [Павикс](https://github.com/ItsPVX))
`Анфаник приходит в дс к делфику и говорит переименовуй стат сервис в кенсуке это мальчик из яой недавно смотрел`
