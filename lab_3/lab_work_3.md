# Лабораторная работа №3

## Дано

1. Три текста на английском, немецком и неизвестном языках
2. Необходимо определить, на каком языке написан последний текст.

## Терминология
В рамках данной лабораторной работы вы будете работать с N-граммами.

N-грамма – это последовательность из n элементов,
в настоящей работе – последовательность из n **букв**.

Один элемент – униграмма.

Последовательность из двух элементов – биграмма. 

Последовательность из трёх элементов – триграмма.

Последовательность из четырех и более элементов обозначается как N-грамма,
где N заменяется на количество последовательных элементов.

Например, `word = 'sunny'`, `unigrams = ['s', 'u', 'n', 'n', 'y']`,
`bigrams = ['su', 'un', 'nn', 'ny']`, `trigrams = ['sun', 'unn', 'nny']`

## Что надо сделать

В рамках лабораторной работы №3 требуется разработать алгоритм извлечения N-грамм из текста
и использовать их для определения языка произвольного текста.


### Шаг 1. Разбиение текста на предложения и токены

Функция разбивает заданный текст на предложения и возвращает кортеж предложений с токенами.

В качестве разделителя используется завершающий пунктуационный знак, пробел и факт того, что следующее слово
начинается с большой буквы. 

Каждое предложение токенизируется и разбивается по буквам.
Все строки (буквы) в нижнем регистре, не содержат знаков пунктуации, в том числе и завершающий
знак препинания.

Каждый токен (например,`she` в примере ниже) также содержит 
специальные символы начала (`'_'`) и конца слова (`'_'`). Вставка этих символов
обязательна для корректной работы алгоритма в дальнейшем.


В немецком тексте буквы с умлаутами ö, ü, ä уже заменены на oe, ue, ae; ß заменено на ss.

Если на вход подается некорректное значение, возвращается пустой кортеж.

Например,
```py
text = 'She is happy. He is happy.'
result = (
         (('_', 's', 'h', 'e', '_'), ('_', 'i', 's', '_'), ('_', 'h', 'a', 'p', 'p', 'y', '_')),
         (('_', 'h', 'e', '_'), ('_', 'i', 's', '_'), ('_', 'h', 'a', 'p', 'p', 'y', '_'))
         )
```

**Интерфейс**:

```py
def tokenize_by_sentence(text: str) -> tuple:
  pass
```

### Шаг 2. Создание хранилища соответствий буква-число

Каждой букве из заданного текста присваивается некоторый уникальный идентификатор
`id`. Это требуется для того, чтобы работать не со строками напрямую, а с числами,
которые их представляют.

Например, букве `'a'` ставим в соответствие некоторое уникальное 
число - `12345`. Следующей букве `'c'` - `12346` и так далее.
Эти буквы необходимо поместить в поле `storage` класса `LetterStorage`,
ключом хранилища является буква, а значением – её идентификатор.
Буквы, указанные выше, будут храниться следующим образом:

```py
self.storage = {..., 'a': 12345, 'c': 12346, ...}
```

Для хранения букв и их идентификаторов необходимо создать поле класса - `storage`.  

**Интерфейс**:

```py
class LetterStorage:
    pass
```

### Шаг 2.1. Добавление буквы в хранилище

Функция принимает на вход букву и добавляет её в хранилище `storage`.

Выбор правила для присваивания 
идентификатора (счетчик, начинающийся с нуля или с заданного значения) - произвольный - на ваш
выбор. Основное условие - для одной и той же буквы существует ровно один идентификатор.
Одинаковых идентификаторов у двух разных букв быть не может.

Если буква была добавлена в хранилище или уже существовала в нем, возвращается `0`.
При этом если буква уже существовала в хранилище, идентификатор остается прежним.

Если на вход подается некорректное значение, возвращается `1`.
Пустая строка считается **некорректным** значением.

**Интерфейс**:

```py
class LetterStorage:
  ...
  def _put_letter(self, letter: str) -> int:
    pass
```

### Шаг 2.2. Получения идентификатора буквы

Функция принимает на вход букву и возвращает её идентификатор.

Если на вход подается неизвестная буква, возвращается `-1`.

**Интерфейс**:

```py
class LetterStorage:
  ...
  def get_id_by_letter(self, letter: str) -> int:
    pass
```

### Шаг 2.3. Заполнение хранилища буквами из корпуса предложений. Выполнение Шагов 1-2.3 соответствует 4 баллам

Функция заполняет хранилище `storage` класса `LetterStorage` буквами из кортежа предложений.

Например,
```py
corpus = (
         (('_', 's', 'h', 'e', '_'), ('_', 'i', 's', '_'), ('_', 'h', 'a', 'p', 'p', 'y', '_')),
         (('_', 'h', 'e', '_'), ('_', 'i', 's', '_'), ('_', 'h', 'a', 'p', 'p', 'y', '_'))
         )
```

Если добавление прошло успешно, возвращается `0`.
Если на вход подается некорректное значение, возвращается `1`.

**Дополнительные требования:**

1. В данной функции ОБЯЗАТЕЛЬНО использовать метод `_put_letter` (см. Шаг 2.1).

**Интерфейс**:

```py
class LetterStorage:
  ...
  def update(self, corpus: tuple) -> int:
    pass
```

### Шаг 3. Кодирование корпуса предложений

Функция кодирует корпус предложений, используя заполненный экземпляр класса `LetterStorage`,
и возвращает кортеж закодированных предложений.
Кодирование заключается в замене букв на соответствующие идентификаторы.

Например, 
```py
corpus = (
         (('_', 's', 'h', 'e', '_'), ('_', 'i', 's', '_'), ('_', 'h', 'a', 'p', 'p', 'y', '_')),
         (('_', 'h', 'e', '_'), ('_', 'i', 's', '_'), ('_', 'h', 'a', 'p', 'p', 'y', '_'))
         )
         
encoded_corpus = (
                  ((1, 2, 3, 4, 1), (1, 5, 2, 1), (1, 3, 6, 7, 7, 8, 1)),
                  ((1, 3, 4, 1), (1, 5, 2, 1), (1, 3, 6, 7, 7, 8, 1))
                  )
```

Если на вход подаются некорректные значения, возвращается пустой кортеж.

**Дополнительные требования:**

1. В данной функции ОБЯЗАТЕЛЬНО использовать метод `get_id_by_letter` (см. Шаг 2.2).

**Интерфейс**:

```py
def encode_corpus(storage: LetterStorage, corpus: tuple) -> tuple:
  pass
```

### Шаг 4. Создание структуры для хранения N-грамм

Класс `NGramTrie` позволяет собрать N-граммы из заданного предложения.

> Это не опечатка в названии класса. Такое название выбрано намеренно (`trie` является отраслевым
> термином).

Допустим, `text = 'he is happy'`

1. Необходимо заполнить `NGramTrie` с n, равным 2.
Получаем следующие биграммы:
`(('_', 'h'), ('h', 'e'), ('e', '_'), ('_', 'i'), ('i', 's'), ('s', '_'),
  ('_', 'h'),('h', 'a'), ('a', 'p'), ('p', 'p'), ('p', 'y'), ('y', '_'))`

2. Необходимо заполнить `NGramTrie` с n, равным 3. Получаем следующие триграммы:
`(('_', 'h', 'e'), ('h', 'e', '_'), ('_', 'i', 's'), ('i', 's', '_'), ('_', 'h', 'a'),
  ('h', 'a', 'p'), ('a', 'p', 'p'), ('p', 'p', 'y'), ('p', 'y', '_'))`

> Символ `_` обозначает конец и начало токена. Этот символ должен оказаться частью
> хранилища букв как самостоятельная буква. Попробуйте догадаться сами, зачем это необходимо и
> как это правильно сделать.

### Шаг 4.1. Объявление сущности языковой модели

Создадим класс:

```py
class NGramTrie:
  pass
```

Конструктор принимает на вход размер N-граммы `n`. Это значение сохраняется
в собственном поле экземпляра класса - `size`.

N–граммы хранятся в поле экземпляра класса – `n_grams`.
```py
text = (((1, 2, 3, 1), (1, 19, 1)),)
self.size = 2
self.n_grams = ((((1, 2), (2, 3), (3, 1)), ((1, 19), (19, 1))),)
```

N-граммы и их частоты хранятся в поле экземпляра класса - `n_gram_frequencies`.
Обязательно использовать для хранения словарь, где ключами выступают кортежи из чисел (идентификаторов), а
значения - частота появления соответствующего кортежа в тексте.

Пусть есть некоторая биграмма `(1, 2)`, которая встречалась в тексте 10 раз. Тогда
в классе она хранится следующим образом:

```py
self.n_gram_frequencies[(1,2)] = 10
```

N-граммы и их лог-вероятности хранятся в поле экземпляра класса - `n_gram_log_probabilities`
Обязательно использовать для хранения словарь, где ключами выступают кортежи из чисел (идентификаторов), а
значения - лог-вероятность появления соответствующего кортежа в тексте.

```py
self.n_gram_log_probabilities[(1,2)] = -7.072576102542302
```

### Шаг 4.2. Извлечение N-грамм из заданного корпуса предложений

Функция принимает на вход закодированный текст, извлекает N-граммы
(размер N-грамм хранится в поле `size`) и заполняет поле `n_grams` 

> Напомним, что на самом деле на данном этапе вы уже работаете с закодироваными
> предложениями, полученными на Шаге №3.

> Для студентов, желающих получить оценку 6, требуется реализовать биграммы.

> Для студентов, желающих получить оценку 8, требуется реализовать триграммы.

> Реализация N-грамм требуется только для студентов, желающих получить оценку 10.

Если добавление прошло успешно, возвращается `0`.
Если на вход подается некорректное значение, возвращается `1`.

Например,
```py
text = (((1, 2, 3, 1), (1, 19, 1)),)
self.size = 2
self.n_grams = ((((1, 2), (2, 3), (3, 1)), ((1, 19), (19, 1))),)
```

**Интерфейс**:
```py
class NGramTrie:
  ...
  def fill_n_grams(self, encoded_text: tuple) -> int:
      pass
```

### Шаг 4.3. Получение частот N-грамм

Функция получает частоты N-грамм используя поле `n_grams`.
Метод заполняет поле `n_gram_frequencies`.

Если добавление прошло успешно, возвращается `0`.
Если на вход подается некорректное значение, возвращается `1`.

Например,
```py
self.n_grams = ((((1, 2), (2, 3), (3, 1)), ((1, 19), (19, 3), (3, 1))),)
self.n_gram_frequencies = {(1, 2): 1, (2, 3): 1, (3, 1): 2, (1, 19): 1, (19, 3): 1}
```

**Интерфейс**:
```py
class NGramTrie:
  ...
  def calculate_n_grams_frequencies(self) -> int:
      pass
```

### Шаг 4.4. Расчет лог-вероятностей для каждой N-граммы

Функция рассчитывает лог-вероятности N-грамм используя поле `n_grams_frequencies`.
Метод заполняет поле `n_gram_log_probabilities`.

Формула расчета вероятности для биграмм: 

<img src="https://latex.codecogs.com/gif.latex?P(w_{n}|w_{n-1})&space;=&space;\frac{C(w_{n-1},w_{n})}{\sum_{w}C(w_{n-1},&space;w)}" title="P(w_{n}|w_{n-1}) = \frac{C(w_{n-1},w_{n})}{\sum_{w}C(w_{n-1}, w)}" />

<img src="https://latex.codecogs.com/gif.latex?C(w_{n}|w_{n-1})" title="C(w_{n}|w_{n-1})" /> - это количество появлений кортежа 

<img src="https://latex.codecogs.com/gif.latex?(w_{n-1},&space;w_{n})" title="(w_{n-1}, w_{n})" /> 
в заданном тексте (частота).

<img src="https://latex.codecogs.com/gif.latex?\sum_{w}{C(w_{n-1},&space;w)}" title="\sum_{w}{C(w_{n-1}, w)}" /> - это 
количество появлений кортежей вида 

<img src="https://latex.codecogs.com/gif.latex?(w_{n-1}&space;w)" title="(w_{n-1} w)" />
для всех `w` в заданном корпусе.

Идея проста: насколько часто биграмма, начинающаяся с буквы
<img src="https://latex.codecogs.com/gif.latex?w_{n-1}" title="w_{n-1}" />
будет заканчиваться интересующей
нас буквой 
<img src="https://latex.codecogs.com/gif.latex?w_{n}" title="w_{n}" />.

Формула расчета вероятности для N-грамм:

<img src="https://latex.codecogs.com/gif.latex?P(w_{n}|w_{n-1},w_{n-2},...,w_{n-N&plus;1})&space;=&space;\frac{C(w_{n-N&plus;1},&space;...,&space;w_{n-1},&space;w_{n})}{\sum_{w}C(w_{n-N&plus;1},&space;...,&space;w_{n-1},&space;w)}" title="P(w_{n}|w_{n-1},w_{n-2},...,w_{n-N+1}) = \frac{C(w_{n-N+1}, ..., w_{n-1}, w_{n})}{\sum_{w}C(w_{n-N+1}, ..., w_{n-1}, w)}" />



После получения относительных вероятностей (описаны выше), необходимо взять логарифм по 
основанию `e` (натуральный логарифм)
от полученного отношения. В рамках текущей работы, это просто требование - оперировать лог-вероятностями.

Все значения вероятностей хранятся в поле `n_gram_log_probabilities`.

Если добавление прошло успешно, возвращается `0`.
Если на вход подается некорректное значение, возвращается `1`

Например,
```py
self.n_gram_log_probabilities[(1,2)] = -7.072576102542302
```

**Дополнительные требования:**

1. В данной функции ОБЯЗАТЕЛЬНО использовать поле `n_gram_frequencies` (см. Шаг 4.3).

**Интерфейс**:

```py
class NGramTrie:
  ...
  def calculate_log_probabilities(self) -> int:
    pass
```

### Шаг 4.5. Получение самых частотных N-грамм. Выполнение Шагов 1-4.5 соответствует 6 баллам

Функция принимает на вход k – число самых частотных n-грамм.
Возвращает список из k самых частотных N-грамм.

Чем больше лог-вероятность, тем больше частота слова.

Если на вход подается некорректное значение, возвращается пустой кортеж.

**Дополнительные требования:**

1. В данной функции ОБЯЗАТЕЛЬНО использовать поле `n_gram_log_probabilities` (см. Шаг 4.1).


**Интерфейс**:
```py
class NGramTrie:
  ...
  def top_n_grams(self, k: int) -> tuple:
      pass
```

### Шаг 5. Создание структуры классификатора языков

Класс `LanguageDetector` позволяет определить язык произвольного текста на основе текстов на известных языках.

Конструктор принимает на вход `trie_levels`
и `top_k`, где `top_k` – это число самых частотных n-грамм.

`trie_levels` – это кортеж, который задаёт сколько нужно построить моделей NGram и какой размерности.
Для студентов, выполняющих работу на оценку 8, сюда передаётся кортеж из одного элемента `(3,)`, который означает, что
нужно построить одну модель NGram на основе триграм.
Для выполнения работы на 10, необходимо реализовать возможность построения нескольких моделей NGram с разной
размерностью, например, `(2, 5, 7)` будучи переданным при создании детектора означает использование ансамбля моделей
NGram на основе 2-, 5- и 7-грам соответственно. 

`trie_levels` сохраняется в поле класса `self.trie_levels`, `top_k` - в поле класса `self.top_k`.

Например,
```py
trie_levels = (2, 3)
top_k = 10
self.trie_levels = trie_levels
self.top_k = top_k
```

Также необходимо создать поле `self.n_gram_storages`, где будут храниться заполненные `NGramTrie` для разных языков.
Значение по умолчанию – пустой словарь.


### Шаг 5.1. Заполнение хранилища `NGramTrie` для конкретного языка

Функция принимает на вход закодированный текст и название языка,
заполняет `NGramTrie` для конкретного языка и добавляет в поле `self.n_gram_storages`.
В NGramTrie заполнены поля `self.size`, `self.n_grams`, `self.n_gram_frequencies`, `self.n_gram_log_probabilities`.

Обязательно использовать для хранения словарь, где ключами выступают названия языков,
значения - словари, где ключами выступают размеры N-грамм из поля `trie_levels`, а значениями – заполненные `NGramTrie`.

Например, после обработки английского языка с `trie_levels = (2, 3)`:
```py
self.n_gram_storages['English'] = {2: NGramTrie(2),
                                   3: NGramTrie(3)}
```

Если добавление прошло успешно, возвращается `0`.
Если на вход подаются некорректные значения, возвращается `1`

**Интерфейс**:
```py
class LanguageDetector:
  ...
  def new_language(self, encoded_text: tuple, language_name: str) -> int:
      pass
```


### Шаг 5.2. Подсчет расстояния между наборами N-грамм

Функция сравнивает наборы N-грамм.

Правило подсчёта расстояния:
* Для каждой N-граммы из первого кортежа находится соответствующая N-грамма
  во втором кортеже.
* Расстояние между ними рассчитывается как модуль разности индексов.
* Если N-грамма отсутствует во втором кортеже, расстояние равняется длине второго кортежа.
* Все полученные расстояния суммируются.

Например, первый набор - `first_n_grams = ((1, 2), (4, 5), (2, 3))`,
второй набор – `second_n_grams = ((1, 2), (2, 3), (4, 5))`. Расстояние для `(1, 2)` равно `0`,
так как индекс в первом наборе – `0`, во втором – `0`, `|0 – 0| = 0`. Расстояние для `(4, 5)` равно `1`,
расстояние для `(2, 3)` равно `1`. Соответственно расстояние между наборами равно `2`.

Если на вход подаются некорректные значения, возвращается -1.

**Интерфейс**:
```py
class LanguageDetector:
  ...
   def _calculate_distance(self, first_n_grams: tuple, second_n_grams: tuple) -> int:
       pass
```

### Шаг 5.3. Определение языка текста. Выполнение Шагов 1-5.3 необходимо для получения оценки 8

Функция принимает на вход закодированный текст на неизвестном языке.
Функция возвращает словарь, где ключом является язык, значением – расстояние.

**Обратите внимание, что текст на неизвестном языке должен быть закодирован при помощи `letter_storage`,
который использовался для кодирования текстов на известных языках.**

Язык определяется по расстоянию между `top_k` N-грамм.
Чем меньше расстояние, тем вероятнее тот или иной язык.

Находятся расстояния между `top_k` N-грамм для известных языков из заполненных `n_gram_storages`
и `top_k` N-грамм из текста на неизвестном языке.
Для каждого языка находится усредненное расстояние, например,
для английского языка расстояние между`top-k` N-грамм из текста на неизвестном языке и `top-k` N-грамм из `NgramTrie(2)` равно `10`,
и `top-k` N-грамм из `NgramTrie(3)` равно `14`. Тогда конечное расстояние для английского языка равно `12`.

И тогда, если расстояние между `top_k` N-грамм из английского и неизвестного текста равно `12`,
а расстояние между `top_k` N-грамм из немецкого и неизвестного текста равно `7`,
функция вернет `{'English': 12, 'German': 7}`.

Если на вход подаются некорректные значения, возвращается пустой словарь.

**Дополнительные требования:**

1. В данной функции ОБЯЗАТЕЛЬНО использовать функцию `_calculate_distance` (см. Шаг 5.2.).
2. В данной функции ОБЯЗАТЕЛЬНО использовать поле `n_gram_storages` (см. Шаг 5.1).


**Интерфейс**:
```py
class LanguageDetector:
  ...
  def detect_language(self, encoded_text: tuple) -> dict:
      pass
```

### Шаг 6. Создание дополнительной структуры классификатора языков

Класс `ProbabilityLanguageDetector` наследуется от класса `LanguageDetector`
и позволяет определить язык произвольного предложения на основе вероятности того,
что данное предложение написано на том или ином языке.


### Шаг 6.1. Расчет вероятности предложения на одном языке

Функция принимает на вход заполненный экземпляр класса `NgramTrie` на известном языке и
предложение с N-граммами на неизвестном языке. В экземпляре класса `NgramTrie` заполнены все поля.

Функция возвращает вероятность того, что данное предложение написано на данном языке.

Вероятность предложения рассчитывается как сумма лог-вероятностей входящих в него N-грамм.
Для получения вероятностей N-грамм используется поле `n_gram_log_probabilities`.

Если на вход подаются некорректные значения, возвращается `-1.0`.

**Дополнительные требования:**

1. В данной функции ОБЯЗАТЕЛЬНО использовать поле `n_gram_log_probabilities` (см. Шаг 4.4.).

**Интерфейс**:

```py
class ProbabilityLanguageDetector:
  ... 
  def _calculate_sentence_probability(self, n_gram_storage: NGramTrie, sentence_n_grams: tuple) -> float:
      pass
```


### Шаг 6.2. Определение языка предложения по вероятности. Выполнение Шагов 1-6.2. необходимо для получения оценки 10

Функция принимает на вход закодированный текст на неизвестном языке. Текст состоит из одного предложения.
Функция возвращает словарь, где ключом является язык, значением – вероятность.

**Обратите внимание, что предложение на неизвестном языке должно быть закодировано при помощи `letter_storage`,
который использовался для кодирования текстов на известных языках.**

Пример словаря:
```py
{
 'English': -7.072576102542302,
 'German': -12.012676864542572
}
```

Функция подсчитывает вероятность принадлежности предложения к языку
на основании вероятности входящих в это предложение N-грамм из данного языка.
Для подсчета вероятности используется метод `_calculate_sentence_probability`.

Вероятности подсчитываются на основе всех `n_gram_storages` (см. Шаг 5.1.), то есть для всех размеров N из поля `trie_levels` соответственно.
Имея кортеж `self.trie_levels = (2, 3)`, подсчитываются две вероятности принадлежности предложения к каждому языку – на основе `NGramTrie(2)` (биграмм) и `NGramTrie(3)` (триграмм).

Например, вероятность принадлежности предложения к английскому языку на основе `NGramTrie(2)` равно `0.56`, `NGramTrie(3)` –  `0.64`.
Вероятность принадлежности предложения к немецкому языку на основе `NGramTrie(2)` равно `0.13`, `NGramTrie(3)` –  ` 0.23`.

Полученные вероятности усредняются (`0.6` для английского, `0.18` для немецкого).
И функция возвращает `{ 'English': 0.6, 'German': 0.18}`.

Если на вход подаются некорректные значения, возвращается пустой словарь.


**Дополнительные требования:**

1. В данной функции ОБЯЗАТЕЛЬНО использовать функцию `_calculate_sentence_probability` (см. Шаг 8).


**Интерфейс**:

```py
class ProbabilityLanguageDetector:
  ... 
  def detect_language(self, encoded_sentence: tuple) -> dict:
      pass
```

### Ссылки

* [Полезный ресурс для пытливых умов](https://web.stanford.edu/~jurafsky/slp3/3.pdf)
* [Что такое лог-вероятности и зачем они нужны](https://en.wikipedia.org/wiki/Log_probability)