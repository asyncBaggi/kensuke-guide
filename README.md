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
