# Постановка задачи 

Команда машинного обучения пользовательского контента занимается внедрением ML-сервисов для модерации контента и генерации нового контента на карточках товара. Мы работаем с текстовыми данными и используем различные методы, начиная от rule-based подходов до обучения крупных языковых моделей (LLM).

## Задача

Вам предлагается решить задачу множественной классификации текстов для определения всех классов, к которым можно отнести каждый экземпляр. Множественная классификация отличается от многоклассовой тем, что экземпляр может одновременно относиться к нескольким классам. В задании представлены ответы на опрос, состоящий из части с выбором ответа и расширенного комментария. Необходимо для каждого ответа выбрать все затронутые 50 классов.

## Входные данные

- `train.csv`: ID + текст из ответа + теги из ответа + таргет по каждому классу.
- `test.csv`: ID + текст из ответа + теги из ответа.
- `sample_submission.csv`: формат ответа: ID - id продукта, предсказания по каждому из 50 классов.
- `baseline.ipynb`: ноутбук с простым решением.
- `trends_description.csv`: файл с полным описанием меток класса.

## Решение

Структура корневой папки:
- `data`: входные данные и файлы аугментации (начинаются с `aug_df`).
- `runs`: кривые отслеживания метрик во время обучения.
- `rubert-base-cased-sentiment`: предобученная модель.
- `text_classificator.ipynb`: код обучения и инференса моделей.
- `fine_tuned_model_*`: веса одной из топ-моделей.
- `sub.csv`: файл с предсказаниями трендов.
- `environment.yml`: файл с фиксированным окружением.

## Подготовка данных

1. Заполнение пропусков пустыми строками.
2. Перевод тегов и добавление в конец текста.
3. Нормализация пробелов, перевод эмодзи в текст, удаление повторений символов более 3 раз подряд, перевод текста в нижний регистр (кроме спец токенов).
4. Аугментация данных не проводилась, так как выбранные методы не дали хороших результатов.

## Валидация

Данные были поделены на трейн и валидационную выборку (80% и 20% соответственно).

## Использованные модели
- `DeepPavlov/rubert-base-cased` (лучшие результаты) max_length=512.
- `cointegrated/LaBSE-en-ru` (лучшие результаты) max_length=64.
- `blanchefort/rubert-base-cased-sentiment` (посредственные результаты).
- `cointegrated/rubert-tiny` (посредственные результаты).
- `cointegrated/rubert-tiny2` (посредственные результаты).

## Параметры обучения

- Размер батча: 32.
- Скорость обучения: 2e-5.
- Обучение на видеокарте (8 Гб).
- Оптимизатор: AdamW.
- Критерий: самописный FocalLoss (alpha 0.1, 0.75, 0.25, gamma 3).
- Порог: 0.5.
- Эпохи: 10, 17, 19.
- Использование LoRA на указанных моделях не дало хороших результатов
- Замораживание лишь некоторых весов не дало хороших результатов
- Использование Шедуллера не дало хороших результатов

Комментарии для воспроизведения обучения аналогичных моделей находятся в разделах variables и training файла `text_classificator.ipynb`.
Для обучения нужно раскомментить ячейку 21.

## Воспроизведение окружения

Чтобы воспроизвести окружение, используйте команду:

```bash
conda env create -f environment.yml
conda activate DLS
```

## Запуск

Для получения кривых обучения в корне проекта в командной строке выполните:

```bash
tensorboard --logdir=runs --port=6007
```

## Точки роста
- Использование других моделей
- Более тонкая настройка LoRA (пока результаты с ней были неприемлемы)
- Подбор других методик аугментации (может с помощью промпт-инжиниринга)
- Фиксация моделей средствами pytorch (параметров обучения)
