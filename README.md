# API Protos

Общий репозиторий **Protobuf-контрактов и сгенерированного Go-кода** для сервисов проекта `one_big_app`.

Каждый сервис хранит свои `.proto`-контракты в отдельной директории. Репозиторий позволяет использовать согласованные API и общие типы в нескольких Go-сервисах.

## Структура

```text
api-protos/
├── proto/
│   └── appointment-service/
│       └── appointment-service.proto
├── gen/go/
│   └── appointment-service/
├── buf.yaml
├── buf.gen.yaml
├── buf.lock
├── go.mod
├── go.sum
├── Taskfile.yaml
└── README.md
```

`appointment-service` — первый подключённый сервис; по мере развития проекта здесь будут появляться контракты других сервисов.

## Инструменты

- [Protocol Buffers](https://protobuf.dev/) — описание контрактов.
- [gRPC](https://grpc.io/) — взаимодействие сервисов.
- [Buf](https://buf.build/docs/) — управление зависимостями, проверка и генерация кода.
- [Task](https://taskfile.dev/) — запуск повторяющихся команд.
- [Protovalidate](https://github.com/bufbuild/protovalidate) — декларативные правила валидации сообщений.

## Генерация Go-кода

Команды выполняются **из корня `api-protos`**.

При первоначальной настройке или изменении зависимостей в `buf.yaml`:

```bash
task proto:deps
```

После изменения `.proto`-файлов:

```bash
task proto
```

Задача `proto` запускает `buf build`, `buf lint`, `buf generate` и `go mod tidy`.

Сгенерированный Go-код хранится в `gen/go/` и коммитится в репозиторий, чтобы другие сервисы могли импортировать его как обычную Go-зависимость.

## Использование в Go-сервисах

Подключить модуль:

```bash
go get github.com/veronikaiav/api-protos@latest
```

Пример импорта контракта сервиса записи на приём:

```go
import appointmentv1 "github.com/veronikaiav/api-protos/gen/go/appointment-service"
```

Для других сервисов используются их соответствующие сгенерированные пакеты.

## Валидация

Контракты могут содержать правила [Protovalidate](https://github.com/bufbuild/protovalidate), например для проверки UUID, обязательных полей и дат.

**Правила в `.proto` сами по себе не выполняются:** принимающий сервис должен вызвать Protovalidate или подключить соответствующий gRPC interceptor. Проверки бизнес-логики остаются в реализации сервиса.

## Совместимость

При изменении опубликованных контрактов следует учитывать обратную совместимость. Версии API указываются в именах Protobuf-пакетов (например, `appointment.v1`). Для проверки несовместимых изменений можно использовать `buf breaking`.