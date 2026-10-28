# Результаты выполнения задачи вычисление деления.

## План работы
Напишите функцию (calculate), которая принимает на вход 2 операнда, возможно целые, возможно дробные , выполняет деление операнда 1 на операнд 2 и точность (epsilon, ключевой аргумент) по умолчанию 0.0001 (диапазон: 10**-1 < epsilon < 10**-9). 
Напишите функцию (load_params), которая считывает значение точности из конфигурационного файла settings.ini и задает точность для функции 1. 
Напишите тесты для тестирования работы функции 1 ( 1/2, epsilon = 0.1 = 0.5, 1/1000 = 0.001 epsilon = 0.001, деление на ноль) и функции 2 (виды тестов: проверить ситуацию открытия файла на чтение, epsilon входит в диапазон значений, формат числа в конф. файле). 
Про тестирование с использованием контекста и ConfigParser:  https://colab.research.google.com/drive/1NhFp0BQSIlojvgcvkGaYQafzLnfe5jrz?usp=sharing

Добавить тесты на проверку логирования информации о 2 ситуациях (уровень логирования ERROR и уровень логирования CRITICAL): 

ситуация деления на ноль;
ситуация, при которой файл не найден. 
Изучите информацию по ссылке: https://chat.deepseek.com/share/814ps427ozxhbt14ew

# Ход работы.
## Реализация модуля calculate.py
Были реализованы две основные функции. Настройка логирования вынесена на уровень модуля, чтобы избежать проблем с повторным вызовом basicConfig и обеспечить надёжную запись логов в тестах.

Ключевые фрагменты кода:

```
import configparser
import logging

logger = logging.getLogger(__name__)
handler = logging.FileHandler('myapp.log', encoding='utf-8')
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

MIN_EPSILON = 10 ** -9
MAX_EPSILON = 10 ** -1
DEFAULT_EPSILON = 0.0001


def validate_epsilon(epsilon):
    if not isinstance(epsilon, (int, float)):
        raise TypeError("epsilon должен быть числом")
    if not (MIN_EPSILON <= epsilon <= MAX_EPSILON):
        raise ValueError(
            f"epsilon должен быть в диапазоне "
            f"{MIN_EPSILON} <= epsilon <= {MAX_EPSILON}"
        )


def calculate(op1, op2, *, epsilon=DEFAULT_EPSILON):
    validate_epsilon(epsilon)
    if op2 == 0:
        error_text = "Деление на ноль"
        logger.error(error_text)
        raise ZeroDivisionError(error_text)
    result = op1 / op2
    return round(result / epsilon) * epsilon


def load_params(filename="settings.ini", section="settings", option="epsilon"):
    parser = configparser.ConfigParser()
    read_files = parser.read(filename, encoding="utf-8")
    if not read_files:
        file_not_found_error_text = f"Файл {filename!r} не найден или недоступен для чтения"
        logger.critical(file_not_found_error_text)
        raise FileNotFoundError(file_not_found_error_text)
    # ... (проверка секции, опции, парсинг float, validate_epsilon)
    if calculate.__kwdefaults__ is None:
        calculate.__kwdefaults__ = {}
    calculate.__kwdefaults__["epsilon"] = epsilon
    return epsilon
```

# Реализация тестов test_calculate.py
Тесты разделены на два класса: TestCalculate  и TestLoadParams.

Для проверки логирования использован встроенный механизм unittest - контекстный менеджер assertLogs, который перехватывает сообщения логгера и позволяет проверить их уровень и содержимое.

Ключевые тесты логирования:

```
def test_division_by_zero_logging(self):
    with self.assertLogs(logger, level='ERROR') as cm:
        with self.assertRaises(ZeroDivisionError):
            calculate(1, 0, epsilon=0.001)
    self.assertIn("Деление на ноль", cm.output[0])


def test_file_not_found_logging(self):
    missing_path = os.path.join(self.tmp.name, "missing.ini")
    with self.assertLogs(logger, level='CRITICAL') as cm:
        with self.assertRaises(FileNotFoundError):
            load_params(missing_path)
    self.assertIn("не найден", cm.output[0])
```
Ключевые тесты функций:

Тест                        Что проверяет

test_int_operands             Деление целых чисел

test_float_operands         	Деление дробных чисел

test_epsilon_0_1	            Округление при epsilon = 0.1 (1/3 ≈ 0.3)

test_epsilon_0_001	          Округление при epsilon = 0.001 (1/3 ≈ 0.333)

test_epsilon_0_5_invalid	    Выход за верхнюю границу диапазона

test_epsilon_zero_invalid	    Нулевое значение epsilon

test_division_by_zero	        Работа ZeroDivisionError

test_load_valid_epsilon	      Загрузка корректного epsilon и его применение

test_file_not_found	          Работа FileNotFoundError

test_epsilon_in_range	        Загрузка epsilon = 0.1

test_epsilon_out_of_range	    Загрузка epsilon = 0.5 (ошибка)

test_invalid_number_format	  Загрузка нечислового значения (abc)



# Устранённые проблемы
В ходе работы были выявлены и исправлены следующие недочёты:

Синтаксическая ошибка в load_params — отсутствовала закрывающая скобка в raise FileNotFoundError(...).

Некорректная настройка логирования — вызов logging.basicConfig внутри функции calculate приводил к нестабильной записи в файл. Заменён на явное создание logger и FileHandler на уровне модуля.

Отсутствие проверки логов — в исходных тестах проверялось только возбуждение исключения, но не факт записи в лог. Добавлены тесты с assertLogs.

Лишние экспериментальные ячейки в блокноте (round(1/3, 6), data['epsilon']) — удалены как не несущие смысловой нагрузки.

# Результаты тестирования
Запуск тестов командой:
```
python -m unittest test_calculate.py -v

```
Вывод консоли:
```
test_division_by_zero (test_calculate.TestCalculate.test_division_by_zero) ... ok
test_division_by_zero_logging (test_calculate.TestCalculate.test_division_by_zero_logging) ... ok
test_epsilon_0_001 (test_calculate.TestCalculate.test_epsilon_0_001) ... ok
test_epsilon_0_1 (test_calculate.TestCalculate.test_epsilon_0_1) ... ok
test_epsilon_0_5_invalid (test_calculate.TestCalculate.test_epsilon_0_5_invalid) ... ok
test_epsilon_zero_invalid (test_calculate.TestCalculate.test_epsilon_zero_invalid) ... ok
test_float_operands (test_calculate.TestCalculate.test_float_operands) ... ok
test_int_operands (test_calculate.TestCalculate.test_int_operands) ... ok
test_epsilon_in_range (test_calculate.TestLoadParams.test_epsilon_in_range) ... ok
test_epsilon_out_of_range (test_calculate.TestLoadParams.test_epsilon_out_of_range) ... ok
test_file_not_found (test_calculate.TestLoadParams.test_file_not_found) ... ok
test_file_not_found_logging (test_calculate.TestLoadParams.test_file_not_found_logging) ... ok
test_invalid_number_format (test_calculate.TestLoadParams.test_invalid_number_format) ... ok
test_load_valid_epsilon (test_calculate.TestLoadParams.test_load_valid_epsilon) ... ok

----------------------------------------------------------------------
Ran 14 tests in 0.007s

OK
```
Все 14 тестов пройдены успешно.

# Пример содержимого файла myapp.log
После выполнения тестов в файле myapp.log появляются записи следующего вида:
```
text
2025-01-01 12:00:00,123 - calculate - ERROR - Деление на ноль
2025-01-01 12:00:00,124 - calculate - CRITICAL - Файл '/tmp/.../missing.ini' не найден или недоступен для чтения
Это подтверждает, что логирование работает корректно для обоих требуемых уровней (ERROR и CRITICAL).
```
# Выводы
В ходе выполнения задания были получены следующие результаты:

Реализованы функции calculate и load_params в соответствии с техническим заданием: с валидацией epsilon, поддержкой целых и дробных операндов, загрузкой точности из settings.ini и подменой значения по умолчанию через __kwdefaults__.

Написано 14 модульных тестов, покрывающих:

корректность деления и округления при разных значениях epsilon;

граничные и недопустимые значения epsilon;

обработку деления на ноль;

работу с конфигурационным файлом (успешная загрузка, отсутствие файла, неверный формат);

логирование ошибок уровня ERROR и CRITICAL.

Все тесты проходят успешно.

Логирование настроено через именованный logger с FileHandler, что обеспечивает стабильную запись в myapp.log и позволяет проверять логи через assertLogs.
