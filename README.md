# Гайд по Kensuke(stats-service)
![screenshot](https://ibb.co/KsSKM1Y)

## Что такое Kensuke?
[Kensuke](https://github.com/delfikpro/kensuke) или stats-service - хранилище данных(логично да?). Позволяет максимально просто связать данные игроков между серверов(победы,убийства и другие данные).

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

# Kotlin(by [me](https://github.com/BaggiYT)):
```kotlin
lateinit var kensuke: Kensuke
    val statScope = Scope("topgame", UserData::class.java)
    val userManager = BukkitUserManager(
        listOf(statScope),
        { session, context -> User(session, context.getData(statScope)) },
        { user, context -> context.store(statScope, user.stat) }

    )
    override fun onEnable() {
        kensuke = BukkitKensuke.setup(this)
        kensuke.addGlobalUserManager(userManager)
        kensuke.globalRealm = "REALM-1"
        userManager.isOptional = true

        fun getUser(uuid: UUID): User? = userManager.getUser(uuid)
        fun getUser(player: Player) = getUser(player.uniqueId)
    }
```
# Java(by [DelfikPro](https://github.com/delfikpro)):
```java
public static final Scope<SomeGameStats> statsScope = new PlayerScope<>("somegame", SomeGameStats.class);

	private UserPool<SomeGameUser> userManager;

	@Override
	public void onEnable() {
		IKensuke statService = new Kensuke(IServerPlatform.get(), KensukeConnectionData.fromEnvironment());
		CoreApi.get().registerService(IKensuke.class, statService);

		statService.useScopes(statsScope);
		statService.setDataRequired(false);

		this.userManager = statService.registerUserManager(
				reader -> new SomeGameUser(reader.getUuid(), reader.getName(), reader.getData(statsScope)),
				SomeGameUser::save);
```

(Надеюсь все понятно, каждую строчку распишу позже)

Далее создаем 2 класса `User` и `UserData`:

# Kotlin `User`(by [me](https://github.com/BaggiYT)):
```kotlin
class User(session: KensukeSession, stat: UserData?) : IBukkitKensukeUser {

    var stat: UserData?
    var wins = 0
    private var session: KensukeSession

    private var player: Player? = null
    override fun getSession(): KensukeSession { return session }
    override fun getPlayer(): Player? = player
    override fun setPlayer(p0: Player?) { player = p0 }

    init {
        this.stat = stat ?: UserData(
            UUID.fromString(session.userId),
            0,
        )
        this.session = session
    }
}
```
# Kotlin `UserData`(by [me](https://github.com/BaggiYT)):
```kotlin
data class UserData(
    var uuid: UUID,
    var wins: Int,
)
```

# Java `User`(by [DelfikPro](https://github.com/delfikpro)):
```java
//Да да lombok
public class AnotherUser extends BukkitKensukeUser {

    @Getter
    @Delegate
    private final AnotherData data;

    public AnotherUser(Session session, AnotherData data) {
        super(session);
        this.data = data;
    }
}
```
# Java `UserData`(by [DelfikPro](https://github.com/delfikpro)):
```java
//Да да lombok
@Data
@AllArgsConstructor
public class AnotherData {

    private int wins;
    private double rating;
    private Map<String, String> complexStuff;

}
```
(Если внимательно почитать код, то понятно, позже распишу) 

Теперь мы создали данные игрока и храним их!

## Как получить данные??
Все очень просто, сейчас покажу на примере сообщения игроку:

# Kotlin(by [me](https://github.com/BaggiYT)):
```kotlin
   // app меняем на ваш главный класс
   val user = app.userManager.getUser(player.uniqueId)
   player.sendMessage("Побед: " + user.stat!!.win)
```

# Java(by [DelfikPro](https://github.com/delfikpro)):
```java
   SomeGameUser user = userManager.getUser(sender);
		sender.sendMessage("§eРейтинг: §f" + user.getRating());
		return true;
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

### Как появилось название Kensuke? (Эту историю нам рассказал [Павикс](https://github.com/ItsPVX))
`Анфаник приходит в дс к делфику и говорит переименовуй стат сервис в кенсуке это мальчик из яой недавно смотрел`
