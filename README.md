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

Для обучения Whisper требуется датасет формата:
|Аудиозапись (<= 30 сек, 16кГЦ, моноканальный)|Транскрипция|
|-|--------|

### Описание "желаемого" датасета
1) **Объём.** <br>
Выбирая объём, важно уделять внимание и качеству данных, ориентироваться можно на 1000-5000 часов (далее увидим, что существуют самые разные датасеты от 73 ч. до 20000 ч.). Возможно, стоит скомпилировать открытые датасеты и обучить модель на одном большом наборе данных. 
   
2) **Источник данных.** <br>
Источник данных должен максимально приближаться к тем условиям, которых будет записываться наша целевая речь. Для распознавания конференций подойдут данные из публичных выступлений, телефонных разговоров, стоит сделать акцент на разговорной речи (не чтение текста).

3) **Стиль речи.** <br>
Стиль речи — разговорная и/или официальная и/или спонтанная.

4) **Стиль транскрипции.** <br>
Исходя из конечного продукта — готовой расшифровки речи спикеров, нам нужна транскрипция с сохранением знаков пунктуации.
   
### Существующие варианты

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

Хорошие датасеты на *английском* языке: 
[LibriSpeech](https://datasets.activeloop.ai/docs/ml/datasets/librispeech-dataset/),
[RAVDESS](https://datasets.activeloop.ai/docs/ml/datasets/ravdess-dataset/),
[ATIS](https://datasets.activeloop.ai/docs/ml/datasets/atis-dataset/).

## 2. Мощности
Для модели large оптимально: 2 графических процессора A100 с 40 ГБ.
Для модели small: подойдет Google Collab (5 часов на GPU T4, около 10 кредитов в Google Colab Pro) ([источник](https://astanahub.com/ru/blog/obuchaem-whisper-small-dlia-raspoznavaniia-kazakhskoi-rechi)).

## 3. Примеры других дообученных моделей Whisper
Модели от более репрезентативных авторов:
|Модель|Язык| Параметры/оценка модели|Комментарий|
|-|--------|---|---|
|[nyrahealth/CrisperWhisper](https://huggingface.co/nyrahealth/CrisperWhisper)|Английский|Average WER = 6.66|Усовершенствованная модель Whisper Large v3, с более четкой разметкой, не пропускает фальстарт, паузы, заполнители. Авторство: мед.компания Nyra Health |
|[syvai/hviske-v2](https://huggingface.co/syvai/hviske-v2)|Датский|CoRal CER = 4.7% ± 0.07% <br> CoRal WER = 11.8% ± 0.3%|Модель обучена на Whisper v2 и датасете CoRal. Авторство: небольшая консалтинговая компания Seven.ai|
|[]()|||Авторство: |

Далее приводим модели, которые чуть менее репрезентативны (часто это модели в разработке, без подробностей дообучения):

`whisper-large-v3`

* [Qwzerty/whisper-large-v3-ru](https://huggingface.co/Qwzerty/whisper-large-v3-ru)
   <details>
   <summary>Настройки обучения модели</summary>
   
   ```learning_rate: 5e-05
   train_batch_size: 96
   eval_batch_size: 32
   seed: 42
   distributed_type: multi-GPU
   num_devices: 4
   total_train_batch_size: 384
   total_eval_batch_size: 128
   optimizer: Use adamw_torch with betas=(0.9,0.999) and epsilon=1e-08 and optimizer_args=No additional optimizer arguments
   lr_scheduler_type: linear
   lr_scheduler_warmup_steps: 50
   training_steps: 250
   mixed_precision_training: Native AMP
   ```
     * Дообучалась на данных Common Voice 11.0.
     * Нет данных про WER.
   </details>


* [primeline/whisper-large-v3-german](https://huggingface.co/primeline/whisper-large-v3-german)
   <details>
   <summary>Настройки обучения модели</summary>
   
   ```Batch size: 1024
   Epochs: 2
   Learning rate: 1e-5
   Data augmentation: No
   ```
     
     * Нет данных про WER.
   </details>
  
* [Watarungurunnn/whisper-large-v3-ja](https://huggingface.co/Watarungurunnn/whisper-large-v3-ja)
   <details>
   <summary>Настройки обучения модели</summary>
   
   ```learning_rate: 1e-05
   train_batch_size: 16
   eval_batch_size: 16
   seed: 42
   gradient_accumulation_steps: 2
   total_train_batch_size: 32
   optimizer: Adam with betas=(0.9,0.999) and epsilon=1e-08
   lr_scheduler_type: linear
   lr_scheduler_warmup_steps: 500
   training_steps: 4000
   mixed_precision_training: Native AMP
   ```
   
     * Дообучалась на данных Common Voice 16.0.
     * Loss: 0.4210
     * Wer: 14.6965. 🚨
   </details>


`whisper-large-v2`
* [vumichien/whisper-large-v2-jp](https://huggingface.co/vumichien/whisper-large-v2-jp)
  <details>
   <summary>Настройки обучения модели</summary>
   
   ```learning_rate: 1e-05
   train_batch_size: 8
   eval_batch_size: 8
   seed: 42
   gradient_accumulation_steps: 2
   total_train_batch_size: 16
   optimizer: Adam with betas=(0.9,0.999) and epsilon=1e-08
   lr_scheduler_type: linear
   lr_scheduler_warmup_steps: 500
   training_steps: 10000
   mixed_precision_training: Native AMP
   ```
   
     * Дообучалась на данных Common Voice 11.0.
     * Loss: 0.2352
     * Wer: 8.1166
     * Cer: 5.0032
   </details>

`whisper-large-v3-turbo`
