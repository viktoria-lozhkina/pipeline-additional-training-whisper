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
