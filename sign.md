Отлично, это как раз важный “взрослый” этап CI/CD — подпись релиза. Его стоит оформить аккуратно, потому что тут уже начинается безопасность, а не просто сборка.

Вот готовый блок документации в том же стиле, как у тебя:

---

# Этап 4. Подпись релиза Android-приложения

## Теоретические сведения

Каждое Android-приложение перед установкой на устройство должно быть подписано цифровым ключом.

Подпись используется для:

* подтверждения авторства приложения;
* защиты от подмены APK;
* обновления приложения (тот же ключ обязателен для новых версий).

---

## Типы ключей подписи

### Debug-ключ

Используется во время разработки:

```text
debug.keystore
```

Особенности:

* создаётся автоматически Android Studio;
* используется только локально;
* небезопасен для публикации;
* не подходит для Google Play.

---

### Release-ключ

Используется для финальной сборки:

```text
release.keystore
```

Особенности:

* создаётся вручную разработчиком;
* используется для публикации;
* должен храниться в защищённом месте;
* критичен для обновления приложения.

---

## Почему нельзя хранить ключи в репозитории

Файлы keystore содержат приватный ключ подписи.

Если он попадёт в публичный доступ:

* злоумышленник сможет подписывать вредоносные APK;
* приложение можно будет подменить;
* обновления могут быть скомпрометированы.

Поэтому хранение в Git запрещено.

---

## Где хранятся ключи

В CI/CD системах ключи помещаются в защищённые хранилища:

* GitHub Secrets
* GitLab CI Variables
* Jenkins Credentials
* HashiCorp Vault

---

## Использование секретов в GitHub Actions

В GitHub Actions секреты передаются через:

```text
${{ secrets.NAME }}
```

---

## Пример настроек секретов

В репозитории GitHub:

```
Settings → Secrets and variables → Actions
```

Добавляются:

* `KEYSTORE_FILE`
* `KEYSTORE_PASSWORD`
* `KEY_ALIAS`
* `KEY_ALIAS_PASSWORD`

---

## Подготовка keystore для релиза

Создание ключа:

```bash
keytool -genkeypair \
  -alias mykey \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -keystore release.keystore
```

---

## Использование в Gradle

В `app/build.gradle.kts` добавляется конфигурация:

```kotlin
android {

    signingConfigs {
        create("release") {
            storeFile = file(System.getenv("KEYSTORE_PATH"))
            storePassword = System.getenv("KEYSTORE_PASSWORD")
            keyAlias = System.getenv("KEY_ALIAS")
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }

    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

---

## Передача секретов в GitHub Actions

Добавляется шаг:

```yaml
- name: Build Release APK
  run: ./gradlew assembleRelease
  env:
    KEYSTORE_PATH: ${{ secrets.KEYSTORE_FILE }}
    KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
    KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
    KEY_PASSWORD: ${{ secrets.KEY_ALIAS_PASSWORD }}
```

