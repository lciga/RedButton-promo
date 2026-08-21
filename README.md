# RedButton promo

Промостраница с терминальной заставкой и выбором между Telegram и ВКонтакте.

## Локальный запуск

```bash
npm ci
VITE_TELEGRAM_URL=https://t.me/example VITE_VK_URL=https://vk.com/example npm run dev
```

Если переменные не заданы, кнопки ведут на главные страницы соответствующих сервисов.

## Docker

Создайте `.env` (файл исключён из Git):

```dotenv
VITE_TELEGRAM_URL=https://t.me/example
VITE_VK_URL=https://vk.com/example
```

Затем выполните `docker compose up --build`.
