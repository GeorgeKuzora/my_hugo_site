---
title:       "Протоколы Python и структурная типизация"
subtitle:    "Python Protocols"
description: "Протоколы Python используют структурную типизацию для создания иерархии классов"
date:        2023-07-15T09:37:40+03:00
author:      "Георгий Кузора"
image:       "img/python-protocols.jpg"
tags:        ["Python", "Coding", "OOP", "Types"]
categories:  ["Tech"]
draft:       false
---

Система типов Python поддерживает два способа определения совместимости двух объектов как типов: номинальная типизация и структурная типизация.

## Номинальная типизация

Номинальная типизация основана на иерархии классов. Если класс `Dog` наследует класс `Animal`, он является подтипом `Animal`. Экземпляры `Dog` могут использоваться, когда ожидаются экземпляры `Animal`. Система типов Python преимущественно использует эту форму типизации. Ее легко понять и она соответствует тому, как работает проверка `isinstance` - на основе иерархии классов.

## Структурная типизация

Структурная типизация основана на операциях, которые могут быть выполнены с объектом. Класс `Dog` является структурным подтипом класса `Animal`, если первый имеет все атрибуты и методы последнего, и других совместимых типов.

Структурное типирование можно рассматривать как статический эквивалент `Duck typing`, которая хорошо известна программистам Python.

## Определение протоколов

Вы можете определить свой собственный класс протокола, унаследовав специальный класс протокола:

```python
from typing import Iterable
from typing_extensions import Protocol


class SupportsClose(Protocol):
    # Пустое тело метода обозначается '...'
    def close(self) -> None: ...


class Resource:  # Не нужно наследовать от класса SupportsClose!
    def close(self) -> None:
       self.resource.release()

    # ... прочие методы ...


def close_all(items: Iterable[SupportsClose]) -> None:
    for item in items:
        item.close()

close_all([Resource(), open('some/file')])  # Все работает
```

`Resource` — это подтип протокола `SupportsClose`, поскольку он определяет совместимый метод `close()`.  Обычные файловые объекты, возвращаемые функцией `open()`, также совместимы с этим протоколом, поскольку они поддерживают функцию `close()`.

## Определение подпротоколов и протоколов подклассов

Вы также можете определить подпротоколы.

Существующие протоколы можно расширять и объединять с помощью множественного наследования.  Пример:

```python
# ... Продолжение предыдущего примера

class SupportsRead(Protocol):
    def read(self, amount: int) -> bytes: ...


class TaggedReadableResource(SupportsClose, SupportsRead, Protocol):
    label: str


class AdvancedResource(Resource):
    def __init__(self, label: str) -> None:
        self.label = label

    def read(self, amount: int) -> bytes:

        # Имплементация метода

        ...

resource: TaggedReadableResource
resource = AdvancedResource('handle with care')  # Все в порядке
```

Обратите внимание, что наследование от существующего протокола не превращает автоматически подкласс в протокол — он просто создает обычный (непротокольный) класс или `ABC`, реализующий данный протокол (или протоколы).  Базовый класс `Protocol` должен всегда присутствовать явно, если вы определяете протокол:

```python
class NotAProtocol(SupportsClose):  # Это не протокол
    new_attr: int


class Concrete:
   new_attr: int = 0

   def close(self) -> None:
       ...


# Ошибка: по умолчанию будет использовано номинальное типирование

x: NotAProtocol = Concrete()  # Ошибка!
```

Вы также можете включить реализации методов по умолчанию в протоколы.  Если вы явно включите такой протокол в качестве базового класса, вы можете наследовать эти реализации по умолчанию.

Явное включение протокола в качестве базового класса также является способом документирования того, что ваш класс реализует конкретный протокол, и заставляет статические анализаторы проверять, действительно ли реализация вашего класса совместима с протоколом.  В частности, пропуск значения атрибута или тела метода сделает его неявно абстрактным:

```python
class SomeProto(Protocol):
    attr: int  # Обратите внимание, нет правой части присваивания
    def method(self) -> str: ...  # Можно не указывать реализацию


class ExplicitSubclass(SomeProto):
    pass

ExplicitSubclass()  # error: Cannot instantiate abstract class 'ExplicitSubclass'

                    # with abstract attributes 'attr' and 'method'
```

Точно так же явное присвоение экземпляру протокола может быть способом попросить средство проверки типов убедиться, что ваш класс реализует протокол:

```python
SomeProto = cast(ExplicitSubclass, None)
```

## Неизменность атрибутов протокола

Общая проблема с протоколами заключается в том, что атрибуты протокола неизменны.  Например:

```python
class Box(Protocol):
      content: object


class IntBox:
      content: int


def takes_box(box: Box) -> None: ...

takes_box(IntBox())  # error: Argument 1 to "takes_box" has incompatible type "IntBox"; expected "Box"

                     # note:  Following member(s) of "IntBox" have conflicts:

                     # note:      content: expected "object", got "int"
```

Это связано с тем, что Box определяет содержимое как изменяемый атрибут.  Вот почему это проблематично:

```python
def takes_box_evil(box: Box) -> None:

    box.content = "asdf"  # Это нехорошо, так как box.content будет иметь тип object


my_int_box = IntBox()
takes_box_evil(my_int_box)
my_int_box.content + 1  # Ошибка типа!
```

Это можно исправить, объявив содержимое доступным только для чтения в протоколе `Box` с помощью `@property`:

```Python
class Box(Protocol):
    @property
    def content(self) -> object: ...


class IntBox:
    content: int

def takes_box(box: Box) -> None: ...


takes_box(IntBox(42))  # Все в порядке
```

## Рекурсивные протоколы

Протоколы могут быть рекурсивными (самореферентными) и взаимно рекурсивными.  Это полезно для объявления абстрактных рекурсивных коллекций, таких как деревья и связанные списки:

```python
from typing import TypeVar, Optional
from typing_extensions import Protocol


class TreeLike(Protocol):
    value: int

    @property
    def left(self) -> Optional['TreeLike']: ...

    @property
    def right(self) -> Optional['TreeLike']: ...


class SimpleTree:
    def __init__(self, value: int) -> None:
        self.value = value
        self.left: Optional['SimpleTree'] = None
        self.right: Optional['SimpleTree'] = None


root: TreeLike = SimpleTree(0)  # Все хорошо
```

## Использование isinstance() с протоколами

Вы можете использовать класс протокола с `isinstance()`, если украсите его декоратором класса `@runtime_checkable`.  Декоратор добавляет элементарную поддержку структурных проверок во время выполнения:

```python
from typing_extensions import Protocol, runtime_checkable

@runtime_checkable
class Portable(Protocol):
    handles: int


class Mug:
    def __init__(self) -> None:
        self.handles = 1


def use(handles: int) -> None: ...


mug = Mug()
if isinstance(mug, Portable):  # Работает в рантайме!
   use(mug.handles)
```

`isinstance()` также работает с предопределенными протоколами ввода, такими как `Iterable`.

> Предупреждение
> `isinstance()` с протоколами не является полностью безопасным во время выполнения.  Например, сигнатуры методов не проверяются.  Реализация среды выполнения проверяет только существование всех членов протокола, а не их правильный тип.  `issubclass()` с протоколами будет проверять только наличие методов.

> Примечание
> `isinstance()` с протоколами также может быть на удивление медленным.  Во многих случаях лучше использовать `hasattr()` для проверки наличия атрибутов.
