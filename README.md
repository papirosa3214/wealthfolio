# Финансовый трекер

Личный учёт денег в браузере: расходы и доходы по категориям, месячные лимиты, регулярные платежи и импорт выписок банка. Интерфейс — на русском, суммы — в рублях.

Основа — форк [Wealthfolio](https://github.com/wealthfolio/wealthfolio) (React 19 + Vite + Tailwind v4 + shadcn на фронте, Axum + SQLite на сервере), урезанный под свои нужды: инвестиции и мультивалютность вырезаны.

- Что это и почему так — [`goal-and-requirements.md`](goal-and-requirements.md).
- Запускается контейнером на домашнем сервере, открывается в браузере с мака и телефона.

## Статус

Спецификация — черновик, кода ещё нет: форк апстрима не создан. Дальше — `architecture.md` и первая фича через `brainstorming`.

## Стек

| Часть | Чем |
|---|---|
| Фронт | React 19, Vite, Tailwind v4, shadcn (`@wealthfolio/ui`) |
| Сервер | Rust, Axum, SQLite (веб-издание апстрима) |
| Доставка | Docker-образ, том `/data`, порт `8088` |
| Вход | пароль (Argon2id-хэш), один пользователь |

## Запуск (веб-издание)

```bash
docker run -d --name finance-tracker \
  -p 8088:8088 \
  -v finance-data:/data \
  -e WF_LISTEN_ADDR=0.0.0.0:8088 \
  -e WF_DB_PATH=/data/wealthfolio.db \
  -e WF_SECRET_KEY="$(openssl rand -base64 32)" \
  -e WF_AUTH_PASSWORD_HASH='$argon2id$v=19$m=19456,t=2,p=1$...' \
  -e WF_CORS_ALLOW_ORIGINS=http://<адрес-сервера>:8088 \
  wealthfolio/wealthfolio:3.4.0
```

`WF_SECRET_KEY` и `WF_AUTH_PASSWORD_HASH` обязательны: без пароля сервер отказывается слушать не-loopback адрес. Хэш пароля — `printf 'пароль' | argon2 <16симв.соль> -id -e`, именно `printf`, не `echo`.

## Лицензия

Проект — производная от Wealthfolio и остаётся под **AGPL-3.0**; файлы апстрима сохраняют свои заголовки. Для личного использования это ни к чему не обязывает, но при открытии доступа другим людям изменения придётся публиковать.
