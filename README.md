# Гайд по Kensuke(stats-service)
![screenshot](https://i.ibb.co/mygNLxm/imgonline-com-ua-Blur-ln-Mu-Ny-F3kaz73vy-1.png)

## Что такое Kensuke?
[Kensuke](https://github.com/delfikpro/kensuke) или stats-service - хранилище данных(логично да?). Позволяет максимально просто связать данные игроков между серверов(победы, убийства и другие данные).

## Как установить?
Заходим в `build.gradle` и пишем следующие:

#Гайд устарел!

## Настройка Kensuke в плагине 
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
private final Scope<GameStats> scope = new PlayerScope<>("game", GameStats.class);

	private UserPool<SomeGameUser> userManager;

	@Override
	public void onEnable() {
	       IStatService statService = new StatService(IServerPlatform.get(), StatServiceConnectionData.fromEnvironment());
               coreApi.registerService(IStatService.class, statService);

               statService.useScopes(scope);

               userManager = statService.registerUserManager(
                       ctx -> new User(ctx.getUuid(), ctx.getName(), ctx.getData(scope)),
                       (u, s) -> s.store(scope, u.getStats())
               );
	}
   
```


Далее создаем 2 класса `User`(кроме Java версии??) и `UserData`:


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

# Java `UserData`:
```java
//Да да lombok
@Data
public class GameStats {

    private final UUID id;
    private int blocksPlaced;
    private int wins;
    private int blocksMined;
    private int overtaking;

}
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
## Данный гайд устарел!!!

## Источники
Kensuke - https://github.com/delfikpro/kensuke
Kensuke client - https://github.com/delfikpro/kensuke-client

Спасибо за помощь в изучении Kensuke - [CrazyL3gend](https://github.com/CrazyL3gend), [Func](https://github.com/funcid), [DelfikPro](https://github.com/delfikpro)

### Как появилось название Kensuke?
`Анфаник приходит в дс к делфику и говорит переименовуй стат сервис в кенсуке это мальчик из яой недавно смотрел`
