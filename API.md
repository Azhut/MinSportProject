
### 1. **Frontend -> DWH** (Загрузка файлов)

**Запрос:**
- **Метод:** `POST /api/upload`
- **Описание:** Загружает Excel-файл на сервер для обработки и сохранения в MongoDB.
- **Параметры:**
  - `file` — Excel-файл (формат `.xlsx`), загружается через форму.
  
**Пример запроса:**
```bash
POST /api/upload HTTP/1.1
Content-Type: multipart/form-data

{
  "file": "<binary-data>"
}
```

**Ответ:**
- **Успешный ответ:**
  - **Код:** `200 OK`
  - **Тело:**
    ```json
    {
      "message": "File uploaded successfully",
      "fileId": "abc123"
    }
    ```

- **Ошибка:**
  - **Код:** `400 Bad Request`
  - **Тело:**
    ```json
    {
      "message": "Invalid file format",
      "details": "The file should be in .xlsx format"
    }
    ```

### 2. **DWH -> MongoDB** (Валидация и сохранение данных)

**Запрос:**
- **Метод:** `POST /api/dwh/save`
- **Описание:** Получает файл от Frontend, валидирует на дубликаты, проверяет формат данных и сохраняет в MongoDB.
- **Параметры:** 
  - `fileId` — уникальный идентификатор файла, загруженного через Frontend.

**Пример запроса:**
```bash
POST /api/dwh/save HTTP/1.1
Content-Type: application/json

{
  "fileId": "abc123"
}
```

**Ответ:**
- **Успешный ответ:**
  - **Код:** `200 OK`
  - **Тело:**
    ```json
    {
      "message": "Data successfully saved to MongoDB",
      "mongoId": "xyz789"
    }
    ```

- **Ошибка:**
  - **Код:** `400 Bad Request`
  - **Тело:**
    ```json
    {
      "message": "Duplicate data found",
      "details": "Duplicate entries detected in section 3"
    }
    ```

### 3. **Feature-store -> DWH** (Получение данных для обработки)

**Запрос:**
- **Метод:** `GET /api/dwh/data`
- **Описание:** Запрашивает данные из MongoDB для дальнейшей обработки в Feature-store.
- **Параметры:**
  - `mongoId` — уникальный идентификатор данных, сохраненных в MongoDB.

**Пример запроса:**
```bash
GET /api/dwh/data?mongoId=xyz789 HTTP/1.1
Content-Type: application/json
```

**Ответ:**
- **Успешный ответ:**
  - **Код:** `200 OK`
  - **Тело:**
    ```json
    {
      "message": "Data fetched successfully",
      "data": {
        "section1": {...},
        "section2": {...},
        "section3": {...}
      }
    }
    ```

- **Ошибка:**
  - **Код:** `404 Not Found`
  - **Тело:**
    ```json
    {
      "message": "Data not found in MongoDB"
    }
    ```

### 4. **Feature-store -> PostgreSQL** (Сохранение обработанных данных)

**Запрос:**
- **Метод:** `POST /api/feature-store/save`
- **Описание:** Обрабатывает данные, полученные из MongoDB, и сохраняет в PostgreSQL.
- **Параметры:**
  - `mongoId` — уникальный идентификатор данных, полученных из DWH.
  - `processedData` — массив обработанных данных, который будет сохранен.

**Пример запроса:**
```bash
POST /api/feature-store/save HTTP/1.1
Content-Type: application/json

{
  "mongoId": "xyz789",
  "processedData": {
    "section1": {...},
    "section2": {...},
    "aggregated": {...}
  }
}
```

**Ответ:**
- **Успешный ответ:**
  - **Код:** `200 OK`
  - **Тело:**
    ```json
    {
      "message": "Data successfully saved to PostgreSQL",
      "postgresId": "psql123"
    }
    ```

- **Ошибка:**
  - **Код:** `500 Internal Server Error`
  - **Тело:**
    ```json
    {
      "message": "Error saving to PostgreSQL",
      "details": "Duplicate key violates unique constraint"
    }
    ```

### 5. **Frontend -> Feature-store** (Запрос обработанных данных для визуализации)

**Запрос:**
- **Метод:** `GET /api/feature-store/data`
- **Описание:** Запрашивает обработанные данные из PostgreSQL для визуализации на фронтенде.
- **Параметры:**
  - `postgresId` — уникальный идентификатор данных, сохраненных в PostgreSQL.

**Пример запроса:**
```bash
GET /api/feature-store/data?postgresId=psql123 HTTP/1.1
Content-Type: application/json
```

**Ответ:**
- **Успешный ответ:**
  - **Код:** `200 OK`
  - **Тело:**
    ```json
    {
      "message": "Processed data fetched successfully",
      "data": {
        "chartData": {
          "bar": [...],
          "pie": [...],
          "line": [...]
        }
      }
    }
    ```

- **Ошибка:**
  - **Код:** `404 Not Found`
  - **Тело:**
    ```json
    {
      "message": "Processed data not found"
    }
    ```

### Взаимодействие между сервисами:
1. **Загрузка файлов**: Frontend отправляет файл на DWH для валидации и сохранения в MongoDB.
2. **Обработка данных**: Feature-store получает данные из MongoDB, обрабатывает их и сохраняет в PostgreSQL.
3. **Визуализация**: Frontend запрашивает обработанные данные из PostgreSQL через Feature-store для построения диаграмм и графиков.





