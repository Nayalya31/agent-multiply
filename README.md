# FastAPI-прокси к Langflow: умножение чисел

Небольшое приложение на FastAPI. Оно принимает список чисел, собирает из них выражение вида `3.0 * 5.0 * 2.0` и отправляет его в flow Langflow. Ответ Langflow возвращается клиенту.

Подходит для локального запуска и для деплоя на Railway (`Procfile`).

## Эндпоинты

| Метод | Путь | Назначение |
| --- | --- | --- |
| `GET` | `/` | Проверка, что сервис поднят. Ответ: `{"status":"up"}` |
| `GET` | `/health` | Проверка готовности. Ответ: `{"status":"ok"}` |
| `POST` | `/multiply` | Запрос в Langflow |

Интерактивная документация: http://127.0.0.1:8000/docs

### `POST /multiply`

Тело запроса:

```json
{
  "numbers": [3, 5, 2],
  "user_id": "student_test_01",
  "session_id": "test_001"
}
```

- `numbers` — минимум два числа.
- `user_id` — обязательный идентификатор пользователя, от 1 до 128 символов. Пробелы по краям отбрасываются. Прокси передаёт его в Langflow в поле `user_id` того же запроса.
- `session_id` — необязательный идентификатор сессии Langflow. Если его нет, используется `multiply-session`.

Пример ответа:

```json
{
  "input": "3.0 * 5.0 * 2.0",
  "user_id": "student_test_01",
  "auth_used": "x-api-key",
  "result_text": "30",
  "raw": {}
}
```

Это ответ на запрос выше: `3 * 5 * 2 = 30`. `auth_used` показывает, какой заголовок сработал: `bearer` или `x-api-key`. `raw` — полный JSON от Langflow; в нём та же сессия `test_001` и текст `30`.

## Требования

- Python 3.11–3.13
- Запущенный Langflow с flow умножения

## Локальный запуск

```bash
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
```

Заполните `.env`:

| Переменная | Обязательна | Описание |
| --- | --- | --- |
| `LANGFLOW_URL` | да | Адрес Langflow. На macOS надёжнее `http://127.0.0.1:7860` |
| `LANGFLOW_FLOW_ID` | да | Идентификатор flow |
| `LANGFLOW_API_KEY` | да | Ключ API Langflow |
| `LANGFLOW_INPUT_TYPE` | нет | По умолчанию `chat` |
| `LANGFLOW_OUTPUT_TYPE` | нет | По умолчанию `chat` |
| `LOVEABLE_ORIGIN` | нет | Домен фронтенда, которому разрешён CORS |
| `CORS_ALLOW_ALL` | нет | `true` разрешает любой origin. Удобно для отладки |

Запуск:

```bash
uvicorn main:app --reload --port 8000
```

Сервер слушает http://127.0.0.1:8000.

Проверка:

```bash
curl http://127.0.0.1:8000/health
curl -X POST http://127.0.0.1:8000/multiply \
  -H "Content-Type: application/json" \
  -d '{"numbers":[3,5,2],"user_id":"student_test_01","session_id":"test_001"}'
```

## Деплой на Railway

`Procfile` запускает приложение так:

```text
web: uvicorn main:app --host 0.0.0.0 --port $PORT
```

Переменные из `.env` на Railway задаются в разделе Variables. Файл `.env` в репозиторий не входит.
