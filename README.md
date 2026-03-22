<div align="center">

<img src="figures/banner.svg" alt="Generator for Reasoning Banner" width="100%">

# 🧠 Generator for Reasoning

_«Чтобы научить модель думать — сначала нужно создать мысли.»_

**Автоматическая генерация русскоязычного reasoning-датасета с адаптивными стратегиями рассуждения через Cerebras API**

[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-818CF8?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Qwen-3-235B](https://img.shields.io/badge/Qwen--3-235B--A22B-C4B5FD?style=for-the-badge&logo=huggingface&logoColor=white)](https://huggingface.co/Qwen)
[![Cerebras](https://img.shields.io/badge/Cerebras-Cloud_API-6366F1?style=for-the-badge&logo=cloud&logoColor=white)](https://cerebras.ai)
[![Dataset](https://img.shields.io/badge/Dataset-7_785_examples-818CF8?style=for-the-badge&logo=huggingface&logoColor=white)](https://huggingface.co/Siesher)
[![License: MIT](https://img.shields.io/badge/License-MIT-C4B5FD?style=for-the-badge)](LICENSE)

[О проекте](#-о-проекте) · [Стратегии](#-стратегии-рассуждения) · [Pipeline](#-generation-pipeline) · [Быстрый старт](#-быстрый-старт) · [Данные](#-формат-данных)

---

</div>

## ✦ О проекте

Generator for Reasoning — пайплайн для автоматической генерации reasoning-датасета на русском языке. Использует модель Qwen-3-235B через Cerebras Cloud API для создания вопросов и рассуждений в трёх стилях: Chain-of-Thought, Program-of-Thought и Skip-Thinking.

Сгенерированный датасет предназначен для fine-tuning моделей на адаптивное мышление — модель учится выбирать стратегию рассуждения в зависимости от задачи.

| **Ключевые особенности** | **Описание** |
|---|---|
| 🎯 Три стратегии рассуждения | CoT, PoT, Skip-Thinking с настраиваемыми весами |
| 📚 18 тематических доменов | Математика, логика, физика, экономика, криптография и др. |
| ⚡ Параллельная генерация | ThreadPoolExecutor + ротация API-ключей |
| 📊 Аналитика датасета | Гистограмма распределения токенов |
| 🤗 Hugging Face Hub | Выгрузка готового датасета одной ячейкой |

---

## ✦ Стратегии рассуждения

| Стратегия | Вес | Описание |
|---|---|---|
| **CoT** (Chain-of-Thought) | 20% | Текстовое пошаговое рассуждение |
| **PoT** (Program-of-Thought) | 50% | Рассуждение с Python-кодом в markdown-блоках |
| **SkipThinking** | 30% | Chunk-based рассуждение с `<chunk_start>`/`<chunk_end>` тегами и маркерами `[...]` |

Каждый пример оборачивается в структуру:
```
<thought_type>CoT|PoT|SkipThinking</thought_type>
<think>
...рассуждение в выбранной стратегии...
</think>
Краткий финальный ответ
```

---

## ✦ Generation Pipeline

```
 ╔═══════════════════════════════════════════════════╗
 ║  Stage 1: Generate Questions                     ║
 ║  400 уникальных вопросов × 18 тематик            ║
 ║  Batch-генерация по 3 вопроса за запрос           ║
 ╚════════════════════╤══════════════════════════════╝
                      ▼
 ╔═══════════════════════════════════════════════════╗
 ║  Stage 2: Prepare Tasks                          ║
 ║  6 reasoning-задач на вопрос                      ║
 ║  Случайный выбор стратегии (CoT/PoT/Skip)        ║
 ╚════════════════════╤══════════════════════════════╝
                      ▼
 ╔═══════════════════════════════════════════════════╗
 ║  Stage 3: Parallel Generation                    ║
 ║  ThreadPoolExecutor (4 workers)                  ║
 ║  Multi-key ротация + retry (3 попытки/ключ)      ║
 ╚════════════════════╤══════════════════════════════╝
                      ▼
 ╔═══════════════════════════════════════════════════╗
 ║  Stage 4: Save Dataset                           ║
 ║  7 785 валидных примеров → JSONL (14.8 MB)       ║
 ╚══════════════════════════════════════════════════╝
```

---

## ✦ Тематические домены

<details>
<summary><b>18 категорий вопросов</b></summary>

| # | Домен | Примеры |
|---|---|---|
| 1 | Логические задачи | Силлогизмы, дедукция |
| 2 | Базовая математика | Арифметика, алгебра |
| 3 | Бытовая логика | Повседневные рассуждения |
| 4 | Пространственное мышление | Геометрия, ориентация |
| 5 | Физические парадоксы | Контринтуитивные задачи |
| 6 | Оценочные задачи | Ферми-задачи, приближения |
| 7 | Алгоритмические задачи | Сортировка, поиск, оптимизация |
| 8 | Экология | Экосистемы, устойчивое развитие |
| 9 | История | Причинно-следственные связи |
| 10 | Экономика | Рыночные модели, финансы |
| 11 | Теория вероятностей | Комбинаторика, случайные события |
| 12 | Естественные науки | Физика, химия |
| 13 | Геометрия | Планиметрия, стереометрия |
| 14 | Финансовая математика | Проценты, кредиты, инвестиции |
| 15 | Задачи на время | Часы, календари, расписания |
| 16 | Криптография | Шифры, кодирование |
| 17 | Анализ данных | Статистика, визуализация |
| 18 | Биология | Генетика, эволюция |

</details>

---

## ✦ Структура проекта

```
Generator_for_reasoning/
├── Make_Data_Ada_think.ipynb                  # Основной ноутбук генерации
├── qwen3_adaptive_reasoning_dataset.jsonl     # Датасет (7 785 примеров, 14.8 MB)
├── .gitignore                                 # Исключение .env
└── README.md
```

---

## ✦ Быстрый старт

### 1. Клонирование

```bash
git clone https://github.com/Siesher/Generator_for_reasoning.git
cd Generator_for_reasoning
```

### 2. Установка зависимостей

```bash
pip install cerebras-cloud-sdk tqdm matplotlib huggingface_hub tiktoken datasets
```

### 3. Настройка

Добавьте API-ключи Cerebras в переменную `API_KEYS` в ноутбуке:

```python
API_KEYS = [
    "your-cerebras-api-key-1",
    "your-cerebras-api-key-2",
]
```

### 4. Генерация

Запустите ячейки `Make_Data_Ada_think.ipynb` последовательно:
1. **Генерация вопросов** — создание уникальных вопросов по 18 темам
2. **Генерация рассуждений** — параллельная генерация CoT/PoT/SkipThinking
3. **Анализ** — гистограмма распределения длин токенов
4. **Выгрузка** — загрузка датасета на Hugging Face Hub

---

## ✦ Формат данных

Каждая запись в JSONL-файле:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Сколько существует способов разместить 5 различных книг на полке?"
    },
    {
      "role": "assistant",
      "content": "<thought_type>PoT</thought_type>\n<think>\nЗадача на перестановки...\n```python\nimport math\nresult = math.factorial(5)\nprint(result)\n```\nРезультат: 120\n</think>\nСуществует 120 способов разместить 5 различных книг на полке."
    }
  ]
}
```

---

## ✦ Конфигурация

| Параметр | Значение | Описание |
|---|---|---|
| `MODEL_NAME_TEACHER` | `qwen-3-235b-a22b` | Модель-учитель |
| `MAX_WORKERS` | 4 | Потоки параллельной генерации |
| `NUM_UNIQUE_QUESTIONS` | 400 | Количество уникальных вопросов |
| `QUESTIONS_BATCH_SIZE` | 3 | Вопросов за один API-запрос |
| `NUM_STRATEGIES_PER_Q` | 6 | Стратегий на вопрос |
| `MAX_RETRIES_PER_KEY` | 3 | Попытки на API-ключ |

---

## ✦ Связанные проекты

| Проект | Описание |
|---|---|
| [Qwen3_LoRA_pet](https://github.com/Siesher/Qwen3_LoRA_pet) | LoRA fine-tuning Qwen3-1.7B на сгенерированном датасете |
| [HuggingFace Models](https://huggingface.co/Siesher) | Обученные модели и датасеты |

---

## ✦ Научная основа

<details>
<summary>Ключевые статьи</summary>

| Метод | Paper | Применение |
|---|---|---|
| Program-of-Thought | [arXiv 2211.12588](https://arxiv.org/abs/2211.12588) | PoT-стратегия с исполняемым кодом |
| Skip-Thinking | [arXiv 2505.18642](https://arxiv.org/abs/2505.18642) | Chunk-based дистилляция рассуждений |
| Distilling Reasoning | [arXiv 2401.11864](https://arxiv.org/abs/2401.11864) | Дистилляция математического мышления |
| Small LM Reasoning | [arXiv 2502.11569](https://arxiv.org/abs/2502.11569) | Reasoning для малых моделей |

</details>

---

## ✦ Лицензия

MIT License — свободно используйте, форкайте, дорабатывайте.

---

<div align="center">

**Сухацкий Максим** · МГТУ им. Н.Э. Баумана (Калужский филиал) · 2025

[![HuggingFace](https://img.shields.io/badge/%F0%9F%A4%97_HuggingFace-Siesher-C4B5FD?style=flat-square)](https://huggingface.co/Siesher)
[![GitHub](https://img.shields.io/badge/GitHub-Siesher-818CF8?style=flat-square&logo=github)](https://github.com/Siesher)

</div>
