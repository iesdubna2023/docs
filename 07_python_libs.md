# Python: библиотеки

## Встроенные библиотеки
Это библиотеки, поставляемые вместе с интерпретатором и поэтому нет необходимости устанавливать их отдельно. Таких библиотек довольно много. Давайте рассмотрим некоторые из них.

### threading, multiprocessing, queue
Библиотеки `threading` и `multiprocessing` используются для того, чтобы организовать одновременное (не обязательно параллельное) выполнение нескольких задач. У них очень похожий программный интерфейс или как говорят API. Давайте рассмотрим на примерах.

Для начала давайте создадим имитацию "долго работающей" задачи. Для этого просто создадим функицию с циклом с паузами в 0.1 секунды на каждой итерации.
```python
import time

def long_running_task(name):
    for i in range(10):
        print(f"Name: {name} Tick: {i}")
        time.sleep(0.1)

```

Теперь давайте попробуем одновременно запустить два экземпляра этой задачи, используя библиотеку `threading`. Этот код создает два потока и вызывает в каждом потоке функцию, которую мы только что создали. Функция передается через аргумент `target`, а аргументы к этой функции передаются через аргумент `args`. После того, как мы создали поток и сказали ему, какую функцию и с какими аргументами он должен запустить, мы вызываем у потока метод  `start`, который и запускает поток. Метод `start` запускает поток, но не ждет его завершения. И наоборот, когда мы вызываем метод `join`, этот метод вернет управление, только когда поток завершится.
```python
import threading
t1 = threading.Thread(target=long_running_task, args=("thread_1",))
t2 = threading.Thread(target=long_running_task, args=("thread_2",))
t1.start()
t2.start()
t1.join()
t2.join()

```

Примерно так же будет выглядеть код, использующий `multiprocessing`.
```python
import multiprocessing
p1 = multiprocessing.Process(target=long_running_task, args=("proc_1",))
p2 = multiprocessing.Process(target=long_running_task, args=("proc_2",))
p1.start()
p2.start()
p1.join()
p2.join()

```

Потоки могут обмениваться данными во время выполнения, но не все структуры данных *безопасны* при использовании несколькими потоками одновременно. Это означает, что потоки могут одновременно попытаться изменить структуру данных, что может привести к повреждению данных. Но есть специальные структуры, предназначенные для обмена данными между потоками. Например, библиотека `queue` предоставляет простую реализацию *очереди*. Очередь - это такая структура данных, куда с одной стороны можно добавлять объекты, а с другой стороны можно эти объекты забирать, причем и добавлять и забирать могут сразу несколько потоков без риска потерять или задублировать эти объекты.

Внимательно изучите и поиграйте (запустите, помодифицируйте) с примером простейшей реализации модели `master-workers`. В этой модели мы одновременно запускаем несколько объектов типа `Worker` и передаем им очередь, откуда они могут брать задачи и выполнять их. Перед этим мы поместили ряд задач в эту очередь.
```python
import threading
from queue import Queue, Empty
from random import random
import time


class Worker:
    def __init__(self, name, task_queue):
        self.name = name
        self.task_queue = task_queue
        self.done = False

    def run(self):
        while True:
            try:
                task = self.task_queue.get(block=False)
                task(self.name)
            except Empty:
                print(f"Worker {self.name}: done")
                self.done = True
                return


def new_task(task_name):
    def task(worker_name):
        time.sleep(random())
        print(f"Worker: {worker_name} Task: {task_name}")
    return task


class Master:
    def __init__(self):
        self.task_queue = Queue()
        self.num_workers = 3
        self.num_tasks = 20
        for i in range(self.num_tasks):
            self.task_queue.put(new_task(f"task_{i}"))
        self.workers = []

    def run(self):
        for i in range(self.num_workers):
            w = Worker(f"worker_{i}", self.task_queue)
            self.workers.append(w)
            t = threading.Thread(target=w.run)
            t.start()

    def join(self):
        while True:
            if all([w.done for w in self.workers]):
                return
            time.sleep(0.1)


m = Master()
m.run()
m.join()
```

### collections
В встроенной библиотеке `collections` доступны некоторые альтернативные типы, которые расширяют функциональность базовых типов `dict`, `list`, `tuple`.

Например, очень удобным типом является `defaultdict`. Часто бывает так, что мы хотим заполнить словарь некоторыми объектами хранящими состояние. Нам приходится сначала проверять есть ли вообще данный ключ в словаре и если его нет, то заполнять ключ объектом инициализированным по умолчанию, а потом уже изменять этот объект. `defaultdict` позволяет сделать код компактнее.

Проще всего это понять на примере. В данном примере у нас есть текст и мы хотим посчитать сколько раз в этом тексте встречается то или иное слово. Мы просто итерируемся по словам в тексте. Если текущего слова в словаре еще нет, то мы сначала должны добавить это слово в словарь и установить счетчик в 1, а если текущее слово уже есть в словаре, то мы должны просто увеличить счетчик для данного слова.
```python
text = """дора дора дора ты слаще помидора"""
d = {}
for word in text.split():
    if word not in d:
        d[word] = 0
    d[word] += 1

print(d)
```
Сравните вот c этим кодом. Здесь мы просто использовали `defaultdict` и сказали ему, что если ключ явно не установлен, то по умолчанию нужно возвращать целое число. А целые числа в Python по умолчанию инициализируются в 0.
```python
from collections import defaultdict
text = """дора дора дора ты слаще помидора"""
d = defaultdict(int)
for word in text.split():
    d[word] += 1

print(dict(d))
```

### functools
Эта библиотека предоставляет различные полезные функции для модификации поведения ваших собственных функций.

Давайте рассмотрим примеры
```python
import time

# создаем функцию, которая эмулирует какие-то очень долгие тяжелые вычисления
def heavy_sum(a, b):
    time.sleep(2)
    return a + b

# несмотря на элементарную операцию сложения, данной функции потребуется пара секунд, чтобы вычислить и вернуть сумму
print(heavy_sum(1, 2))

```

Давайте модифицируем пример
```python
import time
from functools import cache

# теперь результаты будут кэшироваться
@cache
def not_so_heavy_sum(a, b):
    time.sleep(2)
    return a + b

# первый вызов этой функции также займет пару секунд
print(not_so_heavy_sum(1, 2))
# если мы снова вызовем функцию с такими же аргументами, то
# результат она вернет сразу, потому что не будет снова его вычислять,
# а вернет результат из кэша
print(not_so_heavy_sum(1, 2))
# если мы вызовем функцию уже с другими аргументами, то
# в этом случае функции снова потребуется пара секунд
print(not_so_heavy_sum(2, 3))

```

Давайте рассмотрим другой интересный пример с использованием той же библиотеки
```python
from functools import partial

# допустим у нас есть функция от трех аргументов
def f(a, b, c):
    print(f"a = {a}; b = {b}; c = {c}")

# допустим с помощью этой функции вы обрабатываете какие-то массивы данных
# и первый аргумент является параметром процедуры, то есть меняется очень редко
# а второй и третий аргументы допустим задают точки, в которых нужно что-то вычислить

# тогда разумно сделать там
procedure_parameter = "calibration"
partial_f = partial(f, procedure_parameter)

for i in range(3):
    for j in range(3):
        partial_f(i, j)

```

### Работа с файлами
Почти всегда, когда мы создаем программу сложнее, чем просто модельный пример, нам приходится иметь дело с файлами. Мы должны иметь возможность читать данные из файлов и записывать данные в файлы. В Python работать с файлами очень просто. Для этого есть встроенная функция `open`, которая возвращает *файловый объект*. Этот объект имеет методы `read` (прочитать), `write` (записать) и `close` (закрыть). После использования файловый объект обязательно нужно закрывать. Файловый объект среди прочего является контекстным менеджером, а значит его можно использовать вместе со словом `with`. В методе `__exit__`, который вызывается при выходе из блока `with`, вызывается метод `close`, поэтому лучше пользоваться именно интерфесом контекстного менеджера, потому что так вам не нужно заботиться о закрытии объекта.

Давайте рассмотрим примеры
```python
# открываем файл на запись без использования контекстного менеджера
f = open("test_file.txt", "w")
f.write("Однажды в студеную зимнюю пору...\n")
f.close() # обязательно закрываем

# открываем файл на чтение с использованием контекстного менеджера
with open("test_file.txt", "r") as f:
    text = f.read()
    # закрывать не нужно, об этом позаботится контекстный менеджер

print(text)
```

## Requests (HTTP client)
HTTP это текстовый протокол для обмена данными между сервером и клиентом. Например, самое популярное применение протокола HTTP это взаимодействие между веб-серверами и веб-браузерами. Веб-браузеры скачивают с веб-серверов HTML странички и потом отображают их пользователям. Однако есть множество других способов применения этого протокола. Нам сейчас важно знать, что в Python есть библиотека `requests`, которая реализует HTTP клиента. И вы можете используя Python легко скачивать документы с HTTP серверов.
Чтобы установить эту библиотеку в ваше виртуальное окружение используйте команду
```sh
pip install requests
```
Ниже пример использования этой библиотеки
```python
import requests

# если документ небольшой
with requests.get("http://ya.ru") as response:
    # напечатаем код ответа сервера (см. https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
    print(response.status_code)
    # сам документ (главная страница яндекс поисковика) доступен в поле text
    # мы напечатаем первые 128 символов
    print(response.text[:128])

# если документ большой, например книга или набор данных, то скачивание можно осуществлять
# по кусочкам (chunks) и записывать их в файл
link = https://www.gutenberg.org/files/1688/1688-0.txt

with requests.get(link, stream=True) as response:
    with open(path, "wb")  as c:
        for chunk in response.iter_content(chunk_size=8192):
            c.write(chunk)
```

## PyYaml (structured text)
Часто когда вы пишете программу, вам нужно передать ей некоторые конфигурационные параметры (всевозможные пути к файлам, URL-ы, таймауты, инициализационные значения каких-нибудь переменных и так далее). Существуют специальные форматы структурированного текста, которые легко разбирать (программисты говорят парсить от английского слова parse) и превращать в структурированные типы данных вроде словарей или списков произвольной вложенности. На входе у вас есть текст, на выходе у вас есть словари и списки. В свою очередь вложенные словари и списки легко сериализовать в структурированный текст.

`Yaml` - это один из таких форматов и в Python есть библиотека с одноименным названием `yaml`. Чтобы ее установить используйте команду
```sh
pip install PyYAML
```

Давайте теперь на примере рассмотрим как ее использовать
```python
import yaml
# допустим у нас имеется текст
# сейчас он задан как значение переменной прямо в программе,
# но мы можем этот текст получить, например, прочитав файл
s = """---
foo: bar
custom_list:
- Hello
- world
custom_dict:
  Alice: 22
  Bob: 25
"""
# дальше мы этот текст парсим
data = yaml.safe_load(s)

# и теперь data содержит словарь и с ним можно работать, обращаясь к ключам как обычно
print(data["foo"])
print(data["custom_dict"])
print(data["custom_list"])

# можно этот словарь сериализовать снова в текст
print(yaml.safe_dump(data))
```
