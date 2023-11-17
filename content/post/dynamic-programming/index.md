---
title:       "Динамическое программирование"
subtitle:    ""
description: "Динамическое программирование – это методика разработки алгоритмов, которая предполагает разделение задачи на несколько этапов или шагов. После этого вычисляется решение для каждого шага по отдельности. Затем, используя результаты этих отдельных решений, мы определяем общее решение."
date:        2023-11-06T10:16:01+03:00
author:      "Георгий Кузора"
image:       "img/bkg3.png"
tags:        ["Coding", "Algorithms", "CS", "Dynamic Programming"]
categories:  ["Tech"]
draft:       false
---

## Что такое динамическое программирование

Динамическое программирование – это методика разработки алгоритмов, которая предполагает разделение задачи на несколько этапов или шагов. После этого вычисляется решение для каждого шага по отдельности. Затем, используя результаты этих отдельных решений, мы определяем общее решение. Этот процесс близок к использованию рекурсии и принципу "разделяй и властвуй".

Однако, в отличие от обычной рекурсии, динамическое программирование подразумевает сохранение результатов решения подзадач. Это делается для того, чтобы избежать повторного вычисления уже известных результатов, если они потребуются на последующих этапах. Такая оптимизация позволяет снизить временную сложность задачи с экспоненциального уровня до полиномиального.

Например, если написать простое рекурсивное решение для вычисления чисел Фибоначчи, то временная сложность будет экспоненциальной. Однако, если использовать методику динамического программирования и сохранять результаты решения подзадач, то можно снизить временную сложность до линейного уровня.

### Решение рекурсией

Сложность алгоритма: O(2^n).

```python
def fib(n: int):
    if n <= 1:
       return n
    return fib(n-1) + fib(n-2)
```

### Решение динамическим программированием

Сложность алгоритма: O(n).

```python
def fib(n: int):
    f = [0, 1]
    for i in range(2, n + 1):
       f[i] = f[i-1] + f[i-2]
    return f[n]
```

## Принципы динамического программирования

- Задача разбивается на подзадачи меньшего размера.
- Решение подзадач объединяется в общее решение исходной задачи.
- Подзадачи решаются только один раз. Результаты сохраняются и используются в дальнейшем.
- Выбирается порядок решения подзадач таким образом, чтобы использовать результаты уже решенных подзадач при решении последующих.

## Методы решения задач динамического программирования

### Мемоизация vs Табуляция

Табуляция и мемонизация - это два метода оптимизации вычислений, которые используются в динамическом программировании. Они оба направлены на то, чтобы сократить время вычислений, но имеют некоторые отличия.

||Табуляция| Мемоизация|
| -| - | -|
| Состояние | Отношение перехода состояния трудно представить | Отношение перехода состояния легко представить |
| Код| Код усложняется, при увеличении количества входных условий| Код прост и менее сложен|
| Скорость| Быстро, поскольку мы напрямую получаем доступ к предыдущим состояниям из таблицы| Медленно из-за большого количества рекурсивных вызовов и операторов возврата.|
| Решение подзадач | Если все подзадачи необходимо решить хотя бы один раз, алгоритм динамического программирования «снизу вверх» обычно превосходит мемоизированный алгоритм «сверху вниз» в постоянный коэффициент. | Если некоторые подзадачи в пространстве подзадач вообще не требуют решения, мемоизированное решение имеет то преимущество, что решает только те подзадачи, которые определенно необходимы. |
| Записи таблицы| При табуляции, все записи заполняются поочередно, начиная с первой записи.| В отличие от табуляции, при мемоизации таблица не обязательно заполняется полностью. Вносятся только записи которые опреденно необходимы. |
| Подход |Как правило, табуляция представляет собой итеративный подход.|Мемоизация — это рекурсивный подход.     |

### Мемоизация (решение сверху-вниз)
Мемоизация — это нисходящий подход, при котором мы кэшируем результаты вызовов функций. Если функция вызывается снова с теми же входными данными, то возвращаем кэшированный результат.

Такой подход используется, когда мы можем разделить проблему на подзадачи, при этом подзадачи перекрываются и используют результаты решения подзадач из предыдущего шага. Мемоизация обычно реализуется с использованием рекурсии и хорошо подходит для задач с относительно небольшим набором входных данных.

Пример решения задачи нахождения числа в ряду чисел Фибоначи при помощи мемоизации:

```python
# Создаем список в котором будем сохранять промежуточные значения.
term = [0 for i in range(n)]
 
def fib(n):
    # Базовый случай
    if n <= 1:
        return n

    # если fib(n) уже вычислен,
    # мы не выполняем дальнейших рекурсивных вызовов
    # следовательно, уменьшаем количество повторных операций.
    if term[n] != 0:
        return term[n]
    else:
     
        # сохраним вычисленное значение fib(n)
        # в члене массива с индексом n,
        # чтобы его не нужно было повторно вычислять
        term[n] = fib(n - 1) + fib(n - 2)
   return term[n]
```

Временная сложность этого алгоритма - O(n). Вычисляется n значений в ряду чисел Фибоначи. При этом не происходит повторных вычислений.

### Табуляция (решение снизу-вверх)

Таблизация — это восходящий подход, при котором мы сохраняем результаты решений подзадач в таблице и используем эти результаты для решения более крупных подзадач, пока не решим всю проблему.

Такой подход используется, когда мы можем определить проблему как последовательность подзадач, и подзадачи не перекрываются. Табуляция обычно реализуется с помощью итерации и хорошо подходит для задач с большим набором входных данных.

Пример решения задачи нахождения числа в ряду чисел Фибоначи при помощи табуляции:

```python
# Создаем список в котором будем сохранять промежуточные значения.
def fib(n):
    # Частные случаи
    if n == 0:
        return 0
    elif n == 1:
        return 1

    # Создаем таблицу в которую будем сохранять вычисленные значения.
    table = [0, 1]
    # Вычисляем значения и сохраняем их в таблицу. Финальные результат
    # строится на предыдущих.
    for i in range(2, n+1):
       table[i] = table[i-1] + table[i-2]
    # Возвращаем финальный результат.
    return table[n]
```

Временная сложность этого алгоритма - O(n). Вычисляется n значений в ряду чисел Фибоначи. При этом не происходит повторных вычислений.

## Как решить задачу динамического программирования

Шаги по решению проблемы динамического программирования:

1. Определите, является ли это проблемой динамического программирования.
2. Определите минимальное число параметров, которые отражают состояние на каждом шаге.
3. Сформулируйте то как происходит переход от одного состояния к другому.
4. Выполните табуляцию (или мемоизацию).

### 1. Как классифицировать проблему как проблему алгоритма динамического программирования?

Как правило, все проблемы, требующие максимизации или минимизации определенных величин, или задачи подсчета, требующие подсчета композиций при определенных условиях, или определенные проблемы вероятности, могут быть решены с помощью динамического программирования.

Все задачи динамического программирования удовлетворяют свойству перекрывающихся подзадач, а большинство классических задач динамического программирования также удовлетворяют свойству оптимальной подструктуры. Как только мы обнаружим эти свойства в данной задаче, будьте уверены, что ее можно решить с помощью динамического программирования.

- Перекрывающиеся подзадачи: когда решения одних и тех же подзадач необходимы повторно для решения реальной проблемы, говорят, что проблема имеет свойство перекрывающихся подзадач.
- Свойство оптимальной подструктуры: если оптимальное решение данной проблемы может быть получено с помощью оптимальных решений ее подзадач, то говорят, что проблема обладает свойством оптимальной подструктуры.

### 2. Определение состояния.

Проблемы динамического программирования в основном связаны с состоянием и его переходом. Поэтому определение состояния это самый фундаментальный этап. Он должен выполняться с особой тщательностью, поскольку переход между состояниями зависит от выбранного определения состояния.

> Состояние — это набор характеристик, которые можно использовать для описания конкретной позиции или положения в конкретной задаче. Чтобы минимизировать пространство состояний, этот набор параметров должен быть настолько компактным, насколько это возможно.

### 3. Формулирование отношений между состояниями.

Это наиболее сложный этап в решении задачи динамического программирования. Здесь нужно определить то как будет происходить переход между различными состояниями:

- Какое рекурентное отношение существует между различными состояниями.
- Как происходит переход: рекурсивно либо итеративно.
- Какие базовые случаи существуют при рекурсивном подходе.
- И тд.

### 4. Добавление мемоизации или табуляции для состояния

Cохранение решения состояния позволит нам получить к нему доступ из памяти в следующий раз, когда это состояние понадобится.

## Пример решения задачи динамического программирования

> Имея две строки `text1` и `text2`, верните длину их самой длинной общей подстроки.
> Подстрока — это новая строка, созданная из исходной строки с удаленными некоторыми символами (может быть и ни одного).
> Например, «ace» — это подстрока «abcde».

1. Эта задача является задачей динамического программирования. Ее решение нужно разбить на подзадачи по сравнению подстрок. При этом результаты сравнения подстрок позволят вычислить финальный результат.
2. Состояние описывается двумя параметрами.
   1. Позиция в первой строке.
   2. Позиция во второй строке.
3. Переход между состояниями происходит за счет увеличения индекса в первой, второй, либо обеих строках. Затем происходит сравнение значений подстрок.
4. Так как параметров описывающих состояние системы два, то для сохранения результатов понадобится двухмерная таблица либо матрица.

### Решение при помощи мемоизации

```python
def lcs(text1, text2, m, n, table):
    # Базовый случай
    if m < 0 or n < 0:
        return 0

    # Если мы раннее уже вычисляли подобное значение
    # Возвращаем кэшированный результат.
    if table[m][n] != -1:
        return table[m][n]

    # Если мы нашли совпадающие знаки
    # Добавляем данный 1 к результату
    # Сохраняем его на соответствующую позицию в таблицу
    # Рекурсивно вызываем следующую позицию
    if text1[m] == text2[n]:
        table[m][n] = 1 + lcs(text1, text2, m - 1, n - 1, table)
        return table[m][n]
    
    # В случае если знаки не совпадают
    # Вызываем рекурсивно смещение позиций в одной и другой строке
    # Возвращаем максимальное число совпадающих знаков
    table[m][n] = max(lcs(text1, text2, m, n - 1, table), lcs(text1, text2, m - 1, n, table))
    return table[m][n]

# Исходные строки
text1 = "AGGTAB"
text2 = "GXTXAYB"
# Длина исходных строк
m = len(text1)
n = len(text2)
# Таблица для сохранения результатов вычислений
table = [[-1 for i in range(n)] for j in range(m)]

# Вывод результата
result = lcs(text1, text2, m-1, n-1, table)
print(f"Length of LCS is {result}")
```

### Решение при помощи табуляции

```python
def lcs(text1, text2, m, n):
    # Создаем таблицу для хранения промежуточных вычислений
    table = [[None]*(n+1) for _ in range(m+1)]

    # Итеративно заполняем таблицу "снизу-вверх"
    for i in range(m+1):
        for j in range(n+1):
            # Первая строка и столбец заполнены нулями.
            # Это состояние до начала сравнения подстрок
            if i == 0 or j == 0:
                table[i][j] = 0

            # Сравниваем значение в подстроках
            # Если значения равны заносим в таблицу
            # суммируя со значением из предыдущей строки и столбца.
            elif text1[i-1] == text2[j-1]:
                table[i][j] = table[i-1][j-1]+1

            # Если значения в подстроках не равны,
            # записываем в таблицу максимум
            # между предыдущим столбцом и строкой.
            else:
                table[i][j] = max(table[i-1][j], table[i][j-1])

    # Последняя ячейка таблицы содержит максимальную длину
    return table[m][n]


# Исходные строки
text1 = "AGGTAB"
text2 = "GXTXAYB"
# Длина исходных строк
m = len(text1)
n = len(text2)

# Вывод результата
result = lcs(text1, text2, m, n)
print(f"Length of LCS is {result}")
```
