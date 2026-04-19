[README.md](https://github.com/user-attachments/files/26870380/README.md)
# PII Scanner Hackathon

Решение для хакатона по поиску файлов с персональными данными. Проект рекурсивно обходит датасет, извлекает текст из разных форматов, использует OCR как fallback для сканов и применяет rule-based detection с консервативным submission filter для итогового `result.csv`.

## Поддерживаемые форматы

- Текстовые: `txt`, `csv`, `tsv`, `log`, `json`, `xml`
- Документы: `doc`, `docx`, `rtf`, `pdf`, `html`, `htm`
- Табличные: `xls`, `xlsx`, `csv`, `tsv`, `parquet`
- Изображения: `tif`, `tiff`, `png`, `jpg`, `jpeg`, `bmp`, `gif`
- Видео: `mp4`

## Как запустить

Основной entrypoint: `main.py`.

```powershell
cd C:\Users\Berserk\Documents\Codex\hacaton2
python main.py --data-dir .\DATA --result-csv .\output\result.csv --debug-csv .\output\debug_scan.csv --submission-mode conservative
```

Для удобного запуска на Windows можно использовать wrapper:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\run_result.ps1
```

Тесты:

```powershell
python -m unittest discover -s tests -v
```

## Что получается на выходе

- `output/result.csv` — итоговый файл сабмита в формате `size,time,name`
- `output/debug_scan.csv` — полный debug-лог по всем обработанным файлам

`result.csv` пишется в `utf-8` без BOM, использует реальное имя файла без URL-encoding и дедуплицируется по `(size, time, name)`.

## Кратко про подход

1. Извлекаем текст из файла подходящим extractor'ом.
2. Если это скан или изображение, используем OCR fallback.
3. Считаем сильные и слабые признаки ПДн: ФИО, телефоны, email, даты рождения, адреса, паспорт, СНИЛС, ИНН, табличные сигналы.
4. Применяем строгую rule-based decision logic.
5. Перед записью в `result.csv` пропускаем результат через консервативный submission filter, чтобы сильнее давить false positive, особенно для HTML и шумных PDF.

## Структура проекта

```text
hacaton2/
├── DATA/
├── detectors/
├── extractors/
├── output/
├── tests/
├── utils/
├── config.py
├── main.py
├── hackathon_main.py
├── run_result.ps1
└── README.md
```

## Ограничения

- Решение опирается на эвристики и OCR, поэтому качество зависит от структуры документа и качества исходного текста.
- Публичные HTML/PDF документы специально фильтруются консервативно, чтобы уменьшать ложные срабатывания.
- Для OCR на Windows нужен установленный Tesseract.
