# Гайд по Kensuke(stats-service)
Максимально понятный гайд(вроде)

## Что такое Kensuke?
Kensuke или stats-service - хранилище данных(логично да?). Позволяет максимально просто связать данные игроков между серверов(победы,убийства и другие данные).

## Как установить?
Заходим в `build.gradle` и пишем слейдующие:
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
# Kotlin(by me):
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
# Java(by DelfikPro):
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
# Kotlin:
```kotlin
```



### Как появилось название Kensuke? (Эту историю нам рассказал @ItsPVX)
`Анфаник приходит в дс к делфику и говорит переименовуй стат сервис в кенсуке это мальчик из яой недавно смотрел`
