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

## Метаданные в parsed JSONL

Файл: `pdf-docling-hp-proliant-dl360-g6/parsed_english_text_tables.jsonl`

Каждая строка — одна запись Docling: заголовок, текстовый блок или таблица. Все текстовые значения нормализованы и приведены к нижнему регистру.

Пример текстовой записи:

```json
{
  "doc_id": "79a8dcc27ec4fece3ead995fde6ae36eaa925d5ab39ea93ff0117440cf3d2901",
  "source_file": "HP ProLiant DL360 G6 Server Maintenance and Service Guide.pdf",
  "content_type": "text",
  "page_start": 5,
  "page_end": 5,
  "section_path": ["customer self repair"],
  "text": "mandatory -parts for which customer self repair is mandatory...",
  "table_markdown": null,
  "table_json": null
}
```

Пример записи таблицы:

```json
{
  "doc_id": "79a8dcc27ec4fece3ead995fde6ae36eaa925d5ab39ea93ff0117440cf3d2901",
  "source_file": "HP ProLiant DL360 G6 Server Maintenance and Service Guide.pdf",
  "content_type": "table",
  "page_start": 16,
  "page_end": 16,
  "section_path": ["customer self repair"],
  "text": "item description spare part number customer self repair...",
  "table_markdown": "|   item | description | spare part number | customer self repair (on page 5)   |",
  "table_json": [
    {
      "item": "1",
      "description": "access panel",
      "spare part number": "532146-001",
      "customer self repair (on page 5)": "mandatory 1"
    }
  ]
}
```

Поля:

- `doc_id` — SHA-256 хеш исходного PDF. Стабильный идентификатор документа.
- `source_file` — имя PDF-файла, из которого была получена запись.
- `content_type` — тип записи: `heading`, `text` или `table`.
- `page_start` — первая страница, к которой Docling привязал элемент.
- `page_end` — последняя страница элемента.
- `section_path` — путь по заголовкам документа, где находится запись.
- `text` — основной текст записи. Для таблиц это плоское текстовое представление заголовков и ячеек.
- `table_markdown` — Markdown-представление таблицы. Заполнено только для таблиц.
- `table_json` — таблица как список словарей. Заполнено только для таблиц.

## Метаданные в chunked JSONL

Файл: `pdf-docling-hp-proliant-dl360-g6/chunked_english_text_tables.jsonl`

Каждая строка — один чанк для embeddings/RAG. Чанки собираются внутри одного документа и одного `section_path`; соседние чанки внутри секции имеют overlap.

Пример:

```json
{
  "chunk_id": "d7defb77b439d5b2f3766a039065f7afcffdbb20721a1ac588d33125e3493c8c",
  "doc_id": "79a8dcc27ec4fece3ead995fde6ae36eaa925d5ab39ea93ff0117440cf3d2901",
  "source_file": "HP ProLiant DL360 G6 Server Maintenance and Service Guide.pdf",
  "chunk_index": 1,
  "page_start": 5,
  "page_end": 24,
  "section_path": ["customer self repair"],
  "text": "customer self repair hp products are designed with many customer self repair...",
  "word_count": 180,
  "source_record_indices": [5, 6, 7, 8, 9],
  "source_content_types": ["heading", "table", "text"]
}
```

Поля:

- `chunk_id` — SHA-256 идентификатор чанка, рассчитанный из документа, номера чанка и текста.
- `doc_id` — идентификатор исходного PDF из parsed JSONL.
- `source_file` — имя исходного PDF.
- `chunk_index` — порядковый номер чанка.
- `page_start` — минимальная страница среди исходных parsed-записей, вошедших в чанк.
- `page_end` — максимальная страница среди исходных parsed-записей, вошедших в чанк.
- `section_path` — раздел документа, внутри которого собран чанк.
- `text` — итоговый текст чанка для embeddings. Может включать заголовки, текст и Markdown-представления таблиц.
- `word_count` — количество слов в чанке.
- `source_record_indices` — индексы строк из parsed JSONL, использованных при сборке чанка.
- `source_content_types` — типы исходных записей, вошедших в чанк.
