# Краткое введение в GitHub Actions

## Что такое GitHub Actions

GitHub Actions — встроенная в GitHub система автоматизации, позволяющая запускать различные действия после событий в репозитории.

Примеры событий:

* отправка изменений (`git push`);
* создание Pull Request;
* создание релиза;
* запуск по расписанию.

После наступления события GitHub создаёт временный сервер (Runner) и выполняет набор инструкций, описанных в специальном YAML-файле.

---

## Где хранится конфигурация

Все сценарии GitHub Actions размещаются в каталоге:

```text
.github/
└── workflows/
    └── android-ci.yml
```

Каждый файл внутри каталога `workflows` описывает отдельный конвейер.

---

## Структура workflow

Минимальный пример:

```yaml
name: Example Workflow

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Print message
        run: echo "Hello CI"
```

---

## Основные элементы

### name

Название конвейера.

```yaml
name: Android CI
```

Отображается во вкладке Actions.

---

### on

Определяет событие запуска.

Например:

```yaml
on:
  push:
```

Конвейер будет запускаться после каждого `git push`.

Можно ограничить веткой:

```yaml
on:
  push:
    branches:
      - main
```

---

### jobs

Описывает задачи конвейера.

Пример:

```yaml
jobs:
  build:
```

Обычно в учебных проектах достаточно одной задачи `build`.

---

### runs-on

Указывает операционную систему виртуального сервера.

Для Android обычно используется:

```yaml
runs-on: ubuntu-latest
```

---

### steps

Список действий, выполняемых последовательно.

```yaml
steps:
```

Каждый шаг выполняется сверху вниз.

---

## Использование готовых Actions

GitHub позволяет использовать готовые действия других разработчиков.

Например, получение кода из репозитория:

```yaml
- name: Checkout source code
  uses: actions/checkout@v4
```

---

Установка Java:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: 17
```

---

## Выполнение команд

Для запуска команд используется ключевое слово:

```yaml
run:
```

Пример:

```yaml
- name: Run tests
  run: ./gradlew test
```

---

## Загрузка артефактов

Артефакты позволяют сохранить результаты сборки.

Пример:

```yaml
- name: Upload APK
  uses: actions/upload-artifact@v4
  with:
    name: debug-apk
    path: app/build/outputs/apk/debug/app-debug.apk
```

После завершения сборки APK можно скачать через интерфейс GitHub.

---

# Пример полного конвейера для Android

Ниже приведён пример рабочего CI-конвейера.

```yaml
name: Android CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Run tests
        run: ./gradlew test

      - name: Build Debug APK
        run: ./gradlew assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/*.apk
```