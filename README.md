# pipeline-additional-training-whisper
Пайплайн дообучения Whisper

### Структура репозитория
1. `add-training-whisper-large-v3.ipynb`<br>
   Пошаговый код для дообучения Whisper с комментариями и примером обработки данных. Для примера дообучения использовался открытый датасет `mozilla-foundation/common_voice_17_0` ([ссылка](https://huggingface.co/datasets/mozilla-foundation/common_voice_17_0)).
3. `pipeline-additional-training-whisper.xlsx` <br>
   Пайплайн процесса дообучения в формате `.xlsx` (разбивка по шагам привязана к коду).
5. `run-speech-recognition` <br>
Примеры запуска процесса обучения модели обработки речи, использующей Python, библиотеку Hugging Face Transformers, контрольную точку `openai/whisper-large-v3` и дообучающуюся на данных `mozilla-foundation/common_voice_17_0`. Первая команда запускается на одном графическом процессоре (GPU), а вторая — на двух. ([Источник: Automatic Speech Recognition Examples](https://github.com/huggingface/transformers/tree/main/examples/pytorch/speech-recognition#single-gpu-whisper-training:~:text=Single%20GPU%20Whisper%20Training))
6. `whisper-large-v3-russian-description.ipynb` <br>
Описание конфигурации чужой модели `antony66/whisper-large-v3-russian`. В файле содержится информация о параметрах дообучения и обработке аудиоданных с комментариями.

# Алгоритм дообучения модели Whisper
## 1. Данные
>Существует не так много готовых качественных датасетов на русском языке, подходящих под задачу компании. Однако некий материал всё же удалось собрать. Ниже приводится описание "желаемого" датасета — если планируется сбор данных вручную, а также существующие варианты, которые можно использовать.

#### Описание "желаемого" датасета
1) **Объём.** <br>
Выбирая объём, важно уделять внимание и качеству данных, ориентироваться можно на 1000-5000 часов (далее увидим, что существуют самые разные датасеты от 73 ч. до 20000 ч.). Возможно, стоит скомпилировать открытые датасеты и обучить модель на одном большом наборе данных. 
   
2) **Источник данных.** <br>
Источник данных должен максимально приближаться к тем условиям, которых будет записываться наша целевая речь. Для распознавания конференций подойдут данные из публичных выступлений, телефонных разговоров, стоит сделать акцент на разговорной речи (не чтение текста).

3) **Стиль речи.** <br>
Стиль речи — разговорная и/или официальная и/или спонтанная.

4) **Стиль транскрипции.** <br>
Исходя из конечного продукта — готовой расшифровки речи спикеров, нам нужна транскрипция с сохранением знаков пунктуации.
   
#### Существующие варианты

Ниже приведены датасеты на **русском языке**, которые могут подойти для дообучения.

[Коллаб с примером загрузки датасетов](https://colab.research.google.com/drive/1M8x_3dbMID7lqp0kMxGcFVluHvNJwBsg?usp=sharing)

|Датасет|Объем|Комментарий|
|-|--------|---|
|[mozilla-foundation/common_voice_17_0](https://huggingface.co/datasets/mozilla-foundation/common_voice_17_0)|train: 26377 строк <br> val: 10203 строки <br> test: 10203 строки|Наиболее полный открытый датасет, по сравнению с предыдущими версиями увеличилось число часов и доступных языков|
|[WueNLP/sib-fleurs](https://huggingface.co/datasets/WueNLP/sib-fleurs)|train: 733 строки <br> val: 71	строка <br> test: 173 строки |Датасет собран из озвученных статей wikibooks. Темы: наука, технологии, путешествия, политика, спорт, здоровье, развлечения и география|
|[Russian Open Speech To Text (STT/ASR) Dataset](https://github.com/snakers4/open_stt)|~20 000 часов|Аудиозаписи речи на радио, аудиокниги, Ютуб-выступления, телефонные звонки|
|[facebook/2M-Belebele](https://huggingface.co/datasets/facebook/2M-Belebele)| 819 строк | 2M-Belebele speech is composed of recordings gathered by **Meta**(запрещена в РФ) as well as existing recordings from the Fleurs dataset |
|[SeraDreams/Enigma-Dataset](https://huggingface.co/datasets/SeraDreams/Enigma-Dataset)| Датасет содержит ~2000 часов русской речи, состоящий из приблизительно 1,526,360 аудиообразцов. | Создавался для задач TTS, но и для наших целей может подойти |
|[google/fleurs](https://huggingface.co/datasets/google/fleurs)|train: 2562 строки <br> val: 356 строк <br> test: 775 строк| Записи чтения текста вслух |
|[FBK-MT/Speech-MASSIVE](https://huggingface.co/datasets/FBK-MT/Speech-MASSIVE)| train: 115 строк <br> val: 2033 строки <br> test: 2974 строки | Есть спикеры с акцентом|
|[PyAra: Russian bona fide and spoofed speech](https://www.kaggle.com/datasets/alep079/pyara)| |Набор аудиозаписей реального и синтезированного голоса. Он содержит аудиозаписи на русском языке продолжительностью от 3 до 10 секунд в формате WAV|
|[Russian Single Speaker Speech Dataset](https://www.kaggle.com/datasets/bryanpark/russian-single-speaker-speech-dataset)| |CSS10 — набор данных для распознавания речи одного говорящего на десяти языках. Он состоит из коротких аудиозаписей аудиокниг LibriVox и их текстов|
|[VoxLingua107](https://cs.taltech.ee/staff/tanel.alumae/data/voxlingua107/)|73 часа|Минус: датасет не содержит расшифровки, только аудиозаписи|
