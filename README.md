# SciAudit Student — шаблон для разметки научных статей

Этот репозиторий содержит **структуру файлов, JSON-схемы и синтетические примеры**, но **не содержит готового кода**. Все `.py` (включая `run.py` и `scripts/*.py`) специально созданы пустыми. Реализация — ваша задача.

## Задача

Для каждого научного утверждения (`claim`) получить **полный текст статьи** и самостоятельно найти подтверждающие, ограничивающие или опровергающие фрагменты (`evidence`). Затем определить один из вердиктов `warranted`, `overclaimed`, `contradicted`, `insufficient`, обосновать его и сохранить ответ в JSONL.

Сравните независимую ручную разметку, **один вызов обычной LLM** и ваш **многошаговый агент**. Для ответов LLM и агента сохраняйте неизменённую и исправленную человеком версии. Итоговую разметку независимым образом проверяет другой студент. Отдельно исследуйте устойчивость агента при смыслосохраняющих изменениях входа.

## Что заполнять

- `src/paper_loader.py` — загрузка статьи, представленной полным набором текстовых и табличных блоков.
- `src/retrieval.py` — самостоятельный поиск релевантных блоков статьи, **без готового evidence на входе**.
- `src/baseline.py` — один вызов LLM с полной статьёй и `claim`.
- `src/agent.py` — отдельные этапы retrieval → verdict → self-check с сохранением промежуточных результатов.
- `src/validation.py` — контроль формата данных и существования ссылок на блоки `bid`.
- `run.py` — единый CLI-запуск baseline/agent.
- `scripts/validate.py`, `scripts/check_submission.py`, `scripts/score.py` — проверка данных, комплектности и метрик.
- `requirements.txt` — зависимости ваших реализаций.
- `reports/report.md`, `reports/contributions.md` — исследовательский отчёт и личный вклад участников.
- `tests/` — собственные тесты команды.

Все эти файлы сейчас **пустые**. Команды запуска в следующем разделе — **контракт, который нужно реализовать**, а не уже работающая программа.

## Формат данных

- `schemas/paper.schema.json`: статья в JSON с упорядоченными `blocks` и уникальными внутри статьи `bid`.
- `schemas/claim_input.schema.json`: JSONL-вход `instance_id`, `paper_id`, `article_path`, `claim`. Во входе **нет** правильной метки или предварительно найденного evidence.
- `schemas/prediction.schema.json`: результат `verdict`, `confidence`, `predicted_eids`, `issue_tags`, `rationale_short`. **`predicted_eids` содержит `bid` блоков статьи** — это сохранённое для единообразия название поля, не отдельный набор `eid`.
- `schemas/student_annotation.schema.json`: разметка человека, исправления и согласованная версия (`supporting_eids` также содержит `bid`).
- `schemas/review.schema.json`: независимая проверка другим участником.

Формат `.jsonl`: один JSON-объект в одной строке, UTF-8. Все строки, относящиеся к одному утверждению, имеют один `instance_id`. `article_path` указывает на путь от корня проекта. Для реальных статей заполните `data/papers/`, а материалы для сдачи — `data/submission/`. Не копируйте синтетические примеры как реальные результаты.

В `data/examples/` лежат синтетические (вымышленные) данные: статья, вход, ручная разметка, ответы моделей, их исправления, финальная разметка и иллюстрация проверки устойчивости. Все файлы в `data/submission/` и `data/robustness/` пока **пусты**, чтобы вы их заполняли сами.

## Ожидаемые команды запуска (реализуют студенты)

```bash
python -m pip install -r requirements.txt
python run.py --method baseline --input data/submission/inputs.jsonl --output data/submission/llm_raw.jsonl
python run.py --method agent --input data/submission/inputs.jsonl --output data/submission/agent_raw.jsonl
python scripts/validate.py --kind input --path data/submission/inputs.jsonl
python scripts/check_submission.py --data-dir data/submission
python scripts/score.py --gold data/submission/final_annotations.jsonl --pred data/submission/agent_raw.jsonl
```

Эти команды **не будут работать до написания соответствующего Python-кода**.

Не коммитьте API-ключи, секреты и полные тексты статей без права на распространение.
