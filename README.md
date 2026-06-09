# Parsing PDF Docs

Проект извлекает из PDF англоязычный текст, заголовки и таблицы через Docling, а затем готовит overlapped JSONL-чанки для следующего этапа embeddings/RAG.

## Что получается на выходе

Первый этап, `parce_docling_english.py`, создает структурированные Docling-записи:

- `out/parsed_english_text_tables.jsonl`
- `out/parsed_english_text_tables.debug.md`

Второй этап, `chunk_docling_jsonl.py`, читает JSONL первого этапа и создает чанки с overlap:

- `out/chunked_english_text_tables.jsonl`

## Установка

Нужен Python 3.10+.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Запуск парсера PDF

Обычный запуск для PDF-файлов из папки `data/pdfs`:

```bash
python parce_docling_english.py --input data/pdfs --out out
```

Для одного PDF:

```bash
python parce_docling_english.py --input "data/pdfs/HP ProLiant DL360 G6 Server Maintenance and Service Guide.pdf" --out out
```

Если PDF сканированный, включите OCR:

```bash
python parce_docling_english.py --input data/pdfs --out out --ocr
```

## Запуск на Mac GPU

Для Apple Silicon можно попробовать MPS:

```bash
python parce_docling_english.py --input data/pdfs --out out --device mps
```

Если PyTorch в текущем окружении не видит Apple GPU, скрипт предупредит об этом, и Docling может вернуться к CPU. Универсальный режим по умолчанию:

```bash
python parce_docling_english.py --input data/pdfs --out out --device auto
```

## Чанкование с overlap

После парсинга запустите второй этап:

```bash
python chunk_docling_jsonl.py --input out/parsed_english_text_tables.jsonl --out out/chunked_english_text_tables.jsonl
```

По умолчанию:

- размер чанка: `180` слов
- overlap: `40` слов
- таблицы включены в чанки как Markdown-текст

Настроить размер чанков:

```bash
python chunk_docling_jsonl.py \
  --input out/parsed_english_text_tables.jsonl \
  --out out/chunked_english_text_tables.jsonl \
  --chunk-words 220 \
  --overlap-words 50
```

Исключить таблицы:

```bash
python chunk_docling_jsonl.py --no-tables
```

## Формат parsed JSONL

Каждая запись после Docling содержит метаданные:

```json
{
  "doc_id": "...",
  "source_file": "...",
  "content_type": "text",
  "page_start": 5,
  "page_end": 5,
  "section_path": ["customer self repair"],
  "text": "...",
  "table_markdown": null,
  "table_json": null
}
```

Для таблиц `content_type` будет `table`, а `table_markdown` и `table_json` будут заполнены.

## Формат chunked JSONL

Каждый чанк содержит текст для embeddings и ссылку на исходные записи:

```json
{
  "chunk_id": "...",
  "doc_id": "...",
  "source_file": "...",
  "chunk_index": 0,
  "page_start": 5,
  "page_end": 8,
  "section_path": ["customer self repair"],
  "text": "...",
  "word_count": 180,
  "source_record_indices": [6, 7, 8],
  "source_content_types": ["text"]
}
```

`text` в чанках уже готов для следующего шага: расчета embeddings и загрузки в PostgreSQL/pgvector или другую векторную базу.
