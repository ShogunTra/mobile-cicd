# Практическая работа

# Настройка CI/CD для Android-приложения «Генератор паролей»

---

## Цель работы

Изучить полный цикл непрерывной интеграции и доставки (CI/CD) на примере Android-приложения.

В ходе работы необходимо:

* разработать простое Android-приложение на Kotlin;
* реализовать бизнес-логику генерации паролей;
* создать Git-репозиторий;
* написать unit-тесты;
* настроить CI-конвейер (GitHub Actions);
* автоматизировать сборку APK;
* публиковать артефакты сборки.

---

# Теоретические сведения

В современной разработке процесс поставки приложения включает этапы:

1. Написание кода
2. Контроль версий (Git)
3. Автоматическая сборка
4. Автоматическое тестирование
5. Проверка качества кода (lint)
6. Формирование APK
7. Публикация артефактов

CI/CD позволяет автоматизировать этот процесс и быстро находить ошибки.

---

# Постановка задачи

Необходимо разработать Android-приложение «Генератор паролей».

Приложение должно позволять пользователю:

* задавать длину пароля;
* включать/выключать цифры;
* включать/выключать специальные символы;
* включать/выключать заглавные буквы;
* генерировать случайный пароль.

---

# Требования к приложению

## Архитектура

Приложение должно содержать:

* UI на Jetpack Compose
* бизнес-логику в отдельном классе `PasswordGenerator`
* unit-тесты для проверки логики

---

## UI (MainActivity / Compose)

Экран содержит:

* поле ввода длины пароля
* чекбокс "Цифры"
* чекбокс "Спецсимволы"
* чекбокс "Заглавные буквы"
* кнопку "Сгенерировать"
* поле результата (пароль)

Пример результата:

```
Ab3#Rt9!Lp2Q
```

---

## Логика генерации пароля

Создать класс:

```kotlin
PasswordGenerator
```

Метод:

```kotlin
generate(
    length: Int,
    digits: Boolean,
    symbols: Boolean,
    uppercase: Boolean
): String
```

### Описание:

* всегда используется латиница a-z
* при включении digits добавляются цифры
* при включении symbols добавляются спецсимволы
* при включении uppercase добавляются A-Z
* результат формируется случайным образом

---

# Этап 1. Создание проекта

1. Создать Android-проект (Compose)
2. Kotlin как основной язык
3. Минимальный SDK: API 24+
4. Проверить сборку:

```bash
./gradlew assembleDebug
```

---

# Этап 2. Реализация PasswordGenerator

Реализовать класс:

```kotlin
class PasswordGenerator {

    fun generate(
        length: Int,
        digits: Boolean,
        symbols: Boolean,
        uppercase: Boolean
    ): String
}
```

---

## Логика:

* базовый набор: `a-z`
* если digits → добавить `0-9`
* если symbols → добавить `!@#$%^&*()_+-=[]{}`
* если uppercase → добавить `A-Z`

---

# Этап 3. Unit-тесты

Создать:

```kotlin
PasswordGeneratorTest
```

---

## Тест 1 — проверка длины

```kotlin
assertEquals(12, password.length)
```

---

## Тест 2 — наличие цифр

```kotlin
assertTrue(password.any { it.isDigit() })
```

---

## Тест 3 — наличие заглавных букв

```kotlin
assertTrue(password.any { it.isUpperCase() })
```

---

## Тест 4 — наличие спецсимволов

```kotlin
val regex = Regex("[^A-Za-z0-9]")
assertTrue(password.any { regex.matches(it.toString()) })
```

---

## Тест 5 — пароль не пустой

```kotlin
assertTrue(password.isNotEmpty())
```

---

# Этап 4. Локальная проверка

```bash
./gradlew test
```

Ожидаемый результат:

```
BUILD SUCCESSFUL
```

---

```bash
./gradlew assembleDebug
```

APK:

```
app/build/outputs/apk/debug/
```

---

# Этап 5. Git-репозиторий

Инициализация:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <URL>
git push -u origin main
```

---

# Этап 6. Настройка GitHub Actions

Создать файл:

```
.github/workflows/android-ci.yml
```

---

## Pipeline:

### Checkout

Клонирование проекта

### Setup Java

Установка JDK 17

### Grant permissions

```bash
chmod +x gradlew
```

### Lint

```bash
./gradlew lint
```

### Test

```bash
./gradlew test
```

### Build

```bash
./gradlew assembleDebug
```

### Upload artifact

Загрузка APK из:

```
app/build/outputs/apk/debug/app-debug.apk
```

---

# Этап 7. Проверка CI

```bash
git add .
git commit -m "Add CI pipeline"
git push
```

В GitHub:

```
Actions → workflow run
```

Ожидается:

```
✔ Checkout
✔ Setup Java
✔ Lint
✔ Test
✔ Build
✔ Upload artifact
```

---

# Этап 8. Проверка отказоустойчивости CI

## Шаг 1 — сломать тест

Например:

```kotlin
assertEquals(10, password.length)
```

(ожидание не совпадает с реальностью)

---

## Шаг 2 — push

```bash
git push
```

---

## Ожидаемый результат

* CI падает на этапе `Test`
* build НЕ проходит дальше
* artifact НЕ публикуется

---
