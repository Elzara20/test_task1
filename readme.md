# Тестовое задание (Junior LLM/ML Developer)

**Задание**: выявить города при заказе цветов (данные data_given.csv)

**Решение**: NER(дообучение) или LangChain

[Дообученная модель (zip файл)](https://drive.google.com/file/d/1KgkhWqnsoV2jLh1uDExhdpk0_PiKr8JF/view?usp=sharing)

## NER(дообучение)
Кратко: проанализированы датасеты RURED и NEREL для дообучения (модель дообучена на NEREL), модель, которая использовалась, = `google-bert/bert-base-multilingual-uncased`

Итог: разпознавание есть, но существует проблема с сокращениями (таких, как НН, Екб, мск)
### Подбор датасета
Для дообучения рассмотрены 2 датасета с размеченными сущностями [RURED](https://github.com/InstituteForIndustrialEconomics/rured) и [NEREL](https://github.com/nerel-ds/NEREL). 

***Нужные сущности***: `'B-CITY', 'I-CITY'`

#### Анализ
<ol>

<li>У RURED 28 сущностей, у NEREL 29 сущностей => "занулила" (поставила сущность 'O') для не нужных сущностей и удалила предложения, в которых, после "обнуления", не было сущности город (только 'O').

Количество предложений с сущностью город:

|<table class="iksweb">
	<tbody>
		<tr>
			<td>**RURED**</td>
			<td>**NEREL**</td>
		</tr>
		<tr>
			<td>
в датасете train кол-во предложений без сущностей (только О) = 67685
итоговое кол-во предложений в train = 7555 



в датасете dev кол-во предложений без сущностей (только О) = 8438
итоговое кол-во предложений в dev = 967

в датасете test кол-во предложений без сущностей (только О) = 8397
итоговое кол-во предложений в test = 1009

</td>
			<td>
в датасете train кол-во предложений без сущностей (только О) = 7623
итоговое кол-во предложений в train = 1356

в датасете dev кол-во предложений без сущностей (только О) = 982
итоговое кол-во предложений в dev = 156

в датасете test кол-во предложений без сущностей (только О) = 1021
итоговое кол-во предложений в test = 171

</td>
		</tr>
	</tbody>
</table>
Итог: кол-во данных после удаления больше у RURED</li>
<li>Частота встречаемости городов в датасетах, их количество.

|<table class="iksweb">
	<tbody>
		<tr>
			<td>**RURED**</td>
			<td>**NEREL**</td>
		</tr>
		<tr>
			<td>
кол-во уникальных сущностей CITY = 149

города, которые встречаются часто:
[('Москвы', 771), ('Санкт-Петербурге', 892), ('Пикалево', 1097)]

города, которые встречаются редко:
[('Йоханнесбург', 1), ('Московской', 1), ('Борисполе', 1)]
</td>
			<td>
кол-во уникальных сущностей CITY = 1057

3 наиболее встречаемых:
[('Москве', 136), ('Москвы', 88), ('Нью-Йорке', 53)]

3 наименее встречаемых:
[('Ынтымак', 1), ('столицы Киргизии', 1), ('БИШКЕК', 1)]

</td>
		</tr>
	</tbody>
</table>
Итог: наиболее "разнообразен" городами NEREL</li>

<li>Проверка какие сущности есть в RURED, но нет в NEREL

```сущности, которые есть в RURED, но нет в NEREL:
['Йоханнесбург', 'Борисполе', 'Калининграде', 'Неренги', 'Междуреченск', 'минских', 'Вершина Тёи', 'Южно-Сахалинске', 'Сургуте', 'Севастополем', 'Мангейме', 'Шведт', 'Ульяновске'...]
их кол-во = 78


город=Калининградскую   кол-во=1
город=Калининградом   кол-во=1
город=Йоханнесбурге   кол-во=1
город=Калининграда   кол-во=1
```
Итог: есть 78 сущностей, которые не вошли в NEREL, однако при поиске оказалось, что данные сущности есть, но с другим окончание (не все сущности проверены, только `Йоханнесбург` и `Калининград`)</li>

<li>Проверка наличия сокращенных названий городов (Екат, мск, нск и т.д.) - в обоих датасетах

```python
short_city = ['Новосиб', 'новосиб', 'ЕКБ', 'Екб', 'Нижний']
[short_city_i in dict(city_frequency_rured).keys() for short_city_i in short_city]
[short_city_i in dict(city_frequency_nerel).keys() for short_city_i in short_city]
```
Вывод:
```python
[False, False, False, False, False]
[False, False, False, False, False]

```
</li>


</ol>
Для дальнейшего дообучения рассматривала только NEREL, только RURED, сконкатенированные датасеты (NEREL+RURED)

### Дообучение
При дообучении указала 50 эпох, однако также использовала callback функцию раннего останова

**NEREL+RURED**
```python
'eval_loss': 0.02902187407016754,
'eval_precision': 0.8482490272373541,
'eval_recall': 0.8790322580645161,
'eval_f1': 0.8633663366336634,
'eval_accuracy': 0.9943942059822415,
'epoch': 4.0
```
**RURED**
```python
'eval_loss': 0.012539058923721313,
'eval_precision': 0.8649789029535865,
'eval_recall': 0.8266129032258065,
'eval_f1': 0.845360824742268,
'eval_accuracy': 0.9963883940554918,
'epoch': 4.0
```
**NEREL**

```python
'eval_loss': 0.03175358846783638,
'eval_precision': 0.9145129224652088,
'eval_recall': 0.9274193548387096,
'eval_f1': 0.9209209209209209,
'eval_accuracy': 0.9906376518218624,
'epoch': 4.0
```
Итог: точность для каждой модели (относительно одинакова и )высока, потери низки при обучении на RURED, однако наиболее высокий показатель F1 имеет модель, обученная на NEREL => для распознавания использовать **модель, обученную на NEREL**

### Inference
Проблема распознавания сокращений городов (не всех сокращений)

Пример предложений (модель город не распознала):
```python
text = 2. Здравствуйте, мне нужно срочно доставить цветы в НН. Как скоро вы сможете это сделать?
text = ЕКБ - город красивый, но надоело уже скучную серую рутину. Хочу подарить себе радость и заказать цветы. Какие у вас варианты для доставки?
```

Пример предложений (модель город распознала):
```python
text = Здарова, магаз, дай мне штуку цветов в Новосиб. Срочно!
токен: новосиб.
слово: Новосиб.
сущность: B-CITY
точность: 0.9579000473022461
расположение в тексте: [39:47]

text = Добрый день, хочу заказать розы для своей мамы в СПб. Как быстро можно осуществить доставку?
токен: спб.
слово: СПб.
сущность: B-CITY
точность: 0.9939614534378052
расположение в тексте: [49:53]

text = Добрый день, сколько будет стоить доставка цветов в Екатеринбурге на следующей неделе?#
токен: екатеринбурге
слово: Екатеринбурге
сущность: B-CITY
точность: 0.9983174800872803
расположение в тексте: [52:65]
```


# LangChain
Идея: использовать ретривер и LLM для вывода города (данные для хранилища - [data_chain.csv](https://github.com/Elzara20/test_task1/blob/main/data_chain.csv)

Итог: (1) проблема при создании векторной БД FAISS (скорее всего проблема с эмбеддингами openai) 

=> 

использовать другое хранилище Pinecone = (2) проблема с наличием аттрибута (`AttributeError: type object 'Pinecone' has no attribute 'from_documents'`) [одно из решений установка библиотеки `!pip install -U -q langchain-pinecone` - не помогло]

# Заметки
>Мои ключи openai и pinecone в блокноте удалены
>
>Есть ячейки в блокноте, которые обозначены как черновик - они не влияют на дообучение в целом, однако оставлены для пояснения.

# Источники
[информация по подготовке данных NEREL](https://huggingface.co/datasets/surdan/nerel_short/blob/main/Prepare_original_data.ipynb())

[туториал дообучения для NER](https://github.com/huggingface/notebooks/blob/main/examples/token_classification.ipynb)

[туториал LangChain (Pinecone)](https://www.machinelearningnuggets.com/how-to-build-llm-applications-with-langchain-and-openai/)

[туториал LangChain (FAISS)](https://habr.com/ru/articles/729664/)
