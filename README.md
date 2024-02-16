# CI Workflows

В этом репозитории находится файл `.github/workflows/docker.yaml`, который
используется для автоматической сборки и публикации образа Docker.

## Использование

Необходимо в целевой репозиторий добавить файл `.github/workflows/docker.yaml`
со следующим содержимым:

```yaml
name: Docker
on: [push, workflow_dispatch]
jobs:
  "docker":
    uses: "cmctec/ci-workflows/.github/workflows/docker.yaml@main"
```

Если нужна пересборка раз в сутки (например чтобы обновить зависимости), файл
будет выглядеть так:

```yaml
name: Docker
on:
  push:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:
jobs:
  "docker":
    uses: "cmctec/ci-workflows/.github/workflows/docker.yaml@main"
```

Пример, в котором передаются параметры в `docker build` и который срабатывает
только на ветку `main`

```yaml
name: Docker
on:
  push:
    branches: ["main"]
jobs:
  docker:
    uses: "cmctec/ci-workflows/.github/workflows/docker.yaml@main"
    with:
      "build_args": >-
        --build-arg NEXT_PUBLIC_SUPABASE_URL=https://supabase.emedosmotr.kz
        --build-arg NEXT_PUBLIC_MAIN_URL=https://emedosmotr.kz
```

Также имеется возможность отключить кеширование образов:

```yaml
name: Docker
on: [push, workflow_dispatch]
jobs:
  "docker":
    uses: "cmctec/ci-workflows/.github/workflows/docker.yaml@main"
    with:
      "cache": false
```

## Алгоритм работы

Workflow в обычном режиме запускается при git push.

Всегда собирается образ. Настройки кеширования зависят от ветки и тега.

В некоторых случаях образ публикуется в
`registry.cmctech.pro/$GITHUB_REPOSITORY:$GITHUB_REF_NAME`. Например в
`registry.cmctech.pro/cmctec/smartecg-backend:main`.

### Ветки dev, main, test

Собирается образ с тегом, соответствующим имени ветки. Проверяются новые версии
образов (передаётся флаг `--pull` в `docker build`). Образ публикуется в
репозиторий.

Пример выполняемых команд:

```sh
docker build --pull --tag='registry.cmctech.pro/cmctec/myapp:main' .
docker push 'registry.cmctech.pro/cmctec/myapp:main'
```

### Ветка с именем `1.23.x`

Собирается образ с тегом, соответствующим имени ветки. Для кеширования
используется образ с тем же именем.

Пример выполняемых команд:

```sh
docker pull 'registry.cmctech.pro/cmctec/myapp:1.23.x'
docker build --cache-from='registry.cmctech.pro/cmctec/myapp:1.23.x' --tag='registry.cmctech.pro/cmctec/myapp:1.23.x' .
docker push 'registry.cmctech.pro/cmctec/myapp:1.23.x'
```

### Тег с именем `1.23.4`

Собирается образ с тегом, соответствующим имени тега. Для кеширования
используется образ с именем `1.23.x`.

Пример выполняемых команд:

```sh
docker pull 'registry.cmctech.pro/cmctec/myapp:1.23.x'
docker build --cache-from='registry.cmctech.pro/cmctec/myapp:1.23.x' --tag='registry.cmctech.pro/cmctec/myapp:1.23.4' .
docker push 'registry.cmctech.pro/cmctec/myapp:1.23.4'
```

### Тег с именем `test-1234abc`

Собирается образ с тегом, соответствующим имени тега. Специальные настройки для
кеширования не используются. Предполагается, что `1234abc` соответствует
короткому хешу коммита, хотя это не проверяется.

Пример выполняемых команд:

```
docker build --compress --tag='registry.cmctech.pro/cmctec/myapp:test-1234abc' .
docker push 'registry.cmctech.pro/cmctec/myapp:test-1234abc'
```

## Дополнительные аргументы

В переменной `build_args` можно передать дополнительные аргументы для команды
`docker build`.

## Отключение кеширования

Если передать переменную `cache: false`, то кеширование не будет использоваться
и будут использоваться новые версии образов. Чаще всего это не нужно, т.к.
увеличивает время сборки и делает её менее предсказуемой.
