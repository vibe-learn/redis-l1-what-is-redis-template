        # redis — Что такое Redis

        Homework-шаблон для урока **l1_what_is_redis** (Что такое Redis) на платформе Vibe Learn.

        ## Что делать

        Напиши Go-программу, которая измеряет латентность GET/SET на локальном Redis:
- устанавливает N пар key/value через pipeline;
- читает их обратно случайным порядком, измеряет p50/p95/p99 латентность;
- сравнивает с baseline (PostgreSQL `pg_dump` записан в lib).
Тесты проверят: корректность pipeline, измерение latency в hist (HDR-histogram), graceful shutdown на SIGINT.

## Контекст (из transfer-задачи урока)

Тебе достался монолит на Django + PostgreSQL. Профиль нагрузки:

- **главная страница**: 5k RPS, каждый запрос → SELECT профиля юзера, SELECT 30 последних постов из ленты;
- **рейтинг постов**: пересчитывается в real-time (лайки/дизлайки), показывается на главной;
- **сессии**: хранятся в PostgreSQL-таблице sessions, читаются на каждый аутентифицированный запрос;
- **очередь email-уведомлений**: сейчас крутится Celery с broker=RabbitMQ.

## Recap из урока

- Redis — это **in-memory key-value с типизированными структурами** (string/list/hash/set/zset/stream), а не просто кэш bytes.
- Скорость идёт от трёх вещей: **RAM-only hot path**, **single-thread event loop** (нет локов), **RESP-протокол** (микросекунды на парсинг).
- Single-thread это **фича**: операция занимает ~1мкс, сеть RTT ~50мкс — добавление потоков даёт мало пользы и много блокировок.
- Все команды над одним ключом **атомарны** — `INCR`, `LPUSH`, `ZADD` не нуждаются в локах. Это второй главный выигрыш после скорости.
- Redis это **дополнение** к основной базе, не замена. Типичные кейсы: cache, session, queue, counter, leaderboard, rate-limit.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - Docker + docker-compose — `docker compose up -d` поднимает single-node Redis 7 на `localhost:6379` (с включёнными keyspace-notifications и AOF). Адрес переопределяется через env `REDIS_ADDR`.

        ## Запуск

        ```bash
        # Поднять локальный Redis
        docker compose up -d

        # Прогнать тесты (интеграционный включается через REDIS_INTEGRATION=1)
        go test ./...
        REDIS_INTEGRATION=1 go test ./...

        # Запустить main (печатает marker; замени stub на реализацию)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
