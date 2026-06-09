# Datasets

Этот репозиторий хранит результаты парсинга PDF-документов.

## pdf-docling-hp-proliant-dl360-g6

Источник: `HP ProLiant DL360 G6 Server Maintenance and Service Guide.pdf`

Файлы:

- `parsed_english_text_tables.jsonl` — структурированные записи, извлеченные Docling: заголовки, текст и таблицы.
- `parsed_english_text_tables.debug.md` — человекочитаемый Markdown для проверки результата парсинга.
- `chunked_english_text_tables.jsonl` — чанки с overlap для следующего этапа embeddings/RAG.

Параметры чанкования:

- размер чанка: 180 слов
- overlap: 40 слов
- таблицы включены как Markdown-текст

Проверка последнего прогона:

- parsed records: 534
- chunks: 92
- max words per chunk: 180
- exact 40-word overlap pairs: 71
