# CI Workflows

В этом репозитории находит файл `.github/workflows/docker.yaml`, который
используется для автоматической сборки и публикации образа Docker.

## Использование

Необходимо в целевой репозиторий добавить файл `.github/workflows/docker.yaml`
со следующим содержимым:

```yaml
name: Docker
on: [push, workflow_dispatch]
jobs:
  docker:
    uses: cmctec/ci-workflows/.github/workflows/docker.yaml@main
```

## Алгоритм работы

Workflow в обычном режиме запускается при git push.

Всегда собирается образ. Настройки кеширования зависят от ветки и тега.

В некоторых случаях образ публикуется в
`registry.cmctech.pro/$GITHUB_REPOSITORY:$GITHUB_REF_NAME`. Например в
`registry.cmctech.pro/cmctec/smartecg-backend:main`.

### Ветки dev, main, test

Собирается образ с тегом, соответствующим имени ветки. Проверяются новые версии
образов.

### Ветка с именем `1.23.x`

Собирается образ с тегом, соответствующим имени ветки. Для кеширования
используется образ с тем же именем.

### Тег с именем `1.23.4`

Собирается образ с тегом, соответствующим имени тега. Для кеширования
используется образ с именем `1.23.x`.

### Тег с именем `test-1234abc`

Собирается образ с тегом, соответствующим имени ветки. Специальные настройки для
кеширования не используются. Предполагается, что `1234abc` соответствует
короткому хешу коммита, хотя это не проверяется.

## Отключение кеширования

Если передать переменную `cache: false`, то кеширование не будет использоваться
и будут использоваться новые версии образов. Чаще всего это не нужно, т.к.
увеличивает время сборки и делает её менее предсказуемой.
