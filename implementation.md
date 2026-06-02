
# Пример реализации

---

## Реализация класса PasswordGenerator

Создать файл:

```text
app/src/main/java/com/example/empty_project/PasswordGenerator.kt
```

Содержимое:

```kotlin
package com.example.empty_project

import kotlin.random.Random

class PasswordGenerator {

    private val lowercase = "abcdefghijklmnopqrstuvwxyz"
    private val digits = "0123456789"
    private val symbols = "!@#$%^&*()_+-=[]{}"
    private val uppercase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

    fun generate(
        length: Int,
        digits: Boolean,
        symbols: Boolean,
        uppercase: Boolean
    ): String {

        var pool = lowercase

        if (digits) {
            pool += this.digits
        }

        if (symbols) {
            pool += this.symbols
        }

        if (uppercase) {
            pool += this.uppercase
        }

        return (1..length)
            .map { pool.random() }
            .joinToString("")
    }
}
```

---

## Пример интерфейса (Jetpack Compose)

Файл:

```text
MainActivity.kt
```

Основной экран:

```kotlin
@Composable
fun PasswordGeneratorScreen() {

    var length by remember { mutableStateOf("12") }
    var digits by remember { mutableStateOf(true) }
    var symbols by remember { mutableStateOf(true) }
    var uppercase by remember { mutableStateOf(true) }
    var password by remember { mutableStateOf("") }

    val generator = PasswordGenerator()

    Column {

        OutlinedTextField(
            value = length,
            onValueChange = { length = it }
        )

        Checkbox(
            checked = digits,
            onCheckedChange = { digits = it }
        )

        Checkbox(
            checked = symbols,
            onCheckedChange = { symbols = it }
        )

        Checkbox(
            checked = uppercase,
            onCheckedChange = { uppercase = it }
        )

        Button(
            onClick = {
                password = generator.generate(
                    length.toIntOrNull() ?: 12,
                    digits,
                    symbols,
                    uppercase
                )
            }
        ) {
            Text("Сгенерировать")
        }

        Text(password)
    }
}
```

---

## Пример MainActivity

```kotlin
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        enableEdgeToEdge()

        setContent {
            EmptyprojectTheme {
                PasswordGeneratorScreen()
            }
        }
    }
}
```

---

## Пример Unit-тестов

Файл:

```text
app/src/test/java/com/example/empty_project/PasswordGeneratorTest.kt
```

---

### Тест 1 — длина пароля

```kotlin
@Test
fun password_has_correct_length() {

    val generator = PasswordGenerator()

    val password = generator.generate(
        12,
        true,
        true,
        true
    )

    assertEquals(12, password.length)
}
```

---

### Тест 2 — наличие цифр

```kotlin
@Test
fun password_contains_digits() {

    val generator = PasswordGenerator()

    val password = generator.generate(
        50,
        true,
        false,
        false
    )

    assertTrue(password.any { it.isDigit() })
}
```

---

### Тест 3 — заглавные буквы

```kotlin
@Test
fun password_contains_uppercase() {

    val generator = PasswordGenerator()

    val password = generator.generate(
        50,
        false,
        false,
        true
    )

    assertTrue(password.any { it.isUpperCase() })
}
```

---

### Тест 4 — спецсимволы

```kotlin
@Test
fun password_contains_symbols() {

    val generator = PasswordGenerator()

    val password = generator.generate(
        50,
        false,
        true,
        false
    )

    val regex = Regex("[^A-Za-z0-9]")

    assertTrue(password.any { regex.matches(it.toString()) })
}
```

---

### Тест 5 — пароль не пустой

```kotlin
@Test
fun password_is_not_empty() {

    val generator = PasswordGenerator()

    val password = generator.generate(
        12,
        false,
        false,
        false
    )

    assertTrue(password.isNotEmpty())
}
```

---

## GitHub Actions (CI/CD)

Создать файл:

```text
.github/workflows/android-ci.yml
```

---

```yaml
name: Android CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: Grant permissions
        run: chmod +x gradlew

      - name: Run lint
        run: ./gradlew lint

      - name: Run tests
        run: ./gradlew test

      - name: Build APK
        run: ./gradlew assembleDebug

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/app-debug.apk
```

---

## Итоговая структура проекта

```text
app/
 ├── src/
 │   ├── main/
 │   │   ├── java/com/example/empty_project/
 │   │   │   ├── MainActivity.kt
 │   │   │   └── PasswordGenerator.kt
 │   │   └── res/
 │   ├── test/
 │   │   └── PasswordGeneratorTest.kt
│
.github/
 └── workflows/
     └── android-ci.yml
```

---

## Результат выполнения работы

После push в репозиторий:

```text
✔ Checkout
✔ Setup Java
✔ Lint
✔ Test
✔ Build APK
✔ Upload Artifact
```
