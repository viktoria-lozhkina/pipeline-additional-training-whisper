# pipeline-additional-training-whisper
Пайплайн дообучения Whisper

### Структура репозитория
1. `add-training-whisper-large-v3.ipynb`<br>
   Пошаговый код для дообучения Whisper с комментариями и примером обработки данных. Для примера дообучения использовался открытый датасет `mozilla-foundation/common_voice_17_0` ([ссылка](https://huggingface.co/datasets/mozilla-foundation/common_voice_17_0)).
3. `pipeline-additional-training-whisper.xlsx` <br>
   Пайплайн процесса дообучения в формате `.xlsx` (разбивка по шагам привязана к коду).
5. `run-speech-recognition` <br>
Примеры запуска процесса обучения модели обработки речи, использующей Python, библиотеку Hugging Face Transformers, контрольную точку `openai/whisper-large-v3` и дообучающуюся на данных `mozilla-foundation/common_voice_17_0`. Первая команда запускается на одном графическом процессоре (GPU), а вторая — на двух.
6. `whisper-large-v3-russian-description.ipynb` <br>
Описание конфигурации чужой модели `antony66/whisper-large-v3-russian`. В файле содержится информация о параметрах дообучения и обработке аудиоданных с комментариями.

# Алгоритм дообучения модели Whisper
## 1. Данные
>Существует не так много готовых качественных датасетов на русском языке, подходящих под задачу компании. Однако некий материал всё же удалось собрать. Ниже приводится описание "идеального" датасета — если планируется сбор данных вручную, а также существующие варианты, которые можно использовать.

#### Описание "идеального" датасета
1) Объём.
1000 примеров https://huggingface.co/datasets/WueNLP/sib-fleurs
   1000 часов - 5000 часов
   
3) Источник данных.
   
4) Стиль речи.
Стиль речи — разговорная / официальная/ спонтанная.
5) Стиль транскрипции.
   
#### Существующие варианты

Ниже приведены датасеты на **русском языке**, которые могут подойти для дообучения.

|Датасет|Объем|Комментарий|
|-|--------|---|
|[mozilla-foundation/common_voice_17_0](https://huggingface.co/datasets/mozilla-foundation/common_voice_17_0)|||
|[WueNLP/sib-fleurs](https://huggingface.co/datasets/WueNLP/sib-fleurs)| | |
|[facebook/2M-Belebele](https://huggingface.co/datasets/facebook/2M-Belebele)| | |
|[SeraDreams/Enigma-Dataset](https://huggingface.co/datasets/SeraDreams/Enigma-Dataset)| | |
|[google/fleurs](https://huggingface.co/datasets/google/fleurs)| | |
|[FBK-MT/Speech-MASSIVE](https://huggingface.co/datasets/FBK-MT/Speech-MASSIVE)| | есть спикеры с акцентом|
|[PyAra: Russian bona fide and spoofed speech](https://www.kaggle.com/datasets/alep079/pyara)| | |
|[Russian Single Speaker Speech Dataset](https://www.kaggle.com/datasets/bryanpark/russian-single-speaker-speech-dataset)| | |

