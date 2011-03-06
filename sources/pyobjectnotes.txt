Заметки об объектной системе языка 1
================================

(origin: http://habrahabr.ru/blogs/python/114576/)

Несколько заметок об объектной системе python'a. Рассчитаны на тех, кто уже умеет программировать на python. Речь идет только о новых классах (new-style classes) в python 2.3 и выше. В этой статье рассказывается, что такое объекты и как происходит поиск атрибутов.

Объекты
------------

Все данные в питоне — это объекты. Каждый объект имеет 2 специальных атрибута __class__ и __dict__.
__class__ — определяет класс или тип, экзмепляром которого является объект. Тип (или класс объекта) определяет его поведение; он есть у всех объектов, в том числе и встроенных. Тип и класс — это разные названия одного и того же. x.__class__ <==> type(x).
__dict__ словарь, дающий доступ к внутреннему пространству имен, он есть почти у всех объектов, у многих встроенных типов его нет.
Примеры.

>>> def foo(): pass
... 
>>> foo.__class__
<type 'function'>
>>> foo.__dict__
{}
>>> (42).__dict__
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: 'int' object has no attribute '__dict__'
>>> (42).__class__
<type 'int'>
>>> class A(object):
...     qux = 'A'
...     def __init__(self, name):
...         self.name=name
...     def foo(self):
...         print 'foo'
... 
>>> a = A('a')

У a тоже есть __dict__ и __class__:

>>> a.__dict__   {'name': 'a'}
>>> a.__class__  
<class '__main__.A'>
>>> type(a)
<class '__main__.A'>
>>> a.__class__ is type(a)
True

Класс и тип — это одно и то же.

>>> a.__class__ is type(a) is A
True

a.__dict__ — это словарь, в котором находятся внутренние (или специфичные для объекта) атрибуты, в данном случае 'name'. А в a.__class__ класс (тип).

И, например, в методах класса присваивание self.foo = bar практически идентично self.__dict__['foo'] = bar или сводится к аналогичному вызову.

В __dict__ объекта нет методов класса, дескрипторов, классовых переменных, свойств, статических методов класса, все они определяются динамически с помощью класса из __class__ атрибута, и являются специфичными именно для класса (типа) объекта, а не для самого объекта.

Пример. Переопределим класс объекта a:

>>> class B(object):
...     qux = 'B'
...     def __init__(self):
...         self.name = 'B object'
...     def bar(self):
 ...         print 'bar'
... 
>>> a.__dict__
{'name': 'a'}
>>> a.foo()
foo
>>> a.__class__
<class '__main__.A'>
>>> a.__class__ = B
>>> a.__class__
<class '__main__.B'>

Смотрим, что поменялось.

Значение a.name осталось прежним, т.е. __init__ не вызывался при смене класса.

>>> a.__dict__
{'name': 'a'}
Доступ к классовым переменным и методам «прошлого» класса A пропал:
>>> a.foo()
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: 'B' object has no attribute 'foo'
А вот классовые переменные и методы класса B доступы:
>>> a.bar()
bar
>>> a.qux
'B'

Работа с атрибутам объекта: установка, удаление и поиск, равносильна вызову встроенных функций settattr, delattr, getattr:

a.x = 1 <==> setattr(a, 'x', 1)
del a.x <==> delattr(a, 'x')
a.x <==> getattr(a, 'x')

При этом стоит стоит понимать, что setattr и delattr влияют и изменяют только сам объект (точнее a.__dict__), и не изменяют класс объекта.

qux — является классовой переменной, т.е. она «принадлежит» классу B, а не объекту a:

>>> a.qux
'B'
>>> a.__dict__
{'name': 'a'}

Если мы попытаемся удалить этот атрибут, то получим ошибку, т.к. delattr будет пытаться удалить атрибут из a.__dict__

>>> delattr(a, 'qux')
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: qux
>>> del a.qux
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: qux
>>> a.qux
'B'
>>>

Далее, если мы попытаемся изменить (установить) атрибут, setattr поместит его в __dict__, специфичный для данного, конкретного объекта.

>>> b = B()
>>> b.qux
'B'
>>> a.qux = 'myB'
>>> a.qux
'myB'
>>> a.__dict__
{'qux': 'myB', 'name': 'a'}
>>> b.qux
'B'
>>> 

Ну и раз есть 'qux' в __dict__ объекта, его можно удалить с помощью delattr:

>>> del a.qux

После удаления, a.qux будет возвращать значение классовой переменной:

>>> a.qux
'B'
>>> a.__dict__
{'name': 'a'}

Итак:
класс для объекта — это значение специального атрибута __class__ и его можно менять. (Хотя в официальной документации говорится, что никаких гарантий нет, но на самом деле можно)
почти каждый объект имеет свое пространство имен (атрибутов), доступ (не всегда полный), к которому осуществляется с помощью специального атрибута __dict__
класс фактичеки влияет только на поиск атрибутов, которых нет в __dict__, как-то: методы класса, дескрипторы, магические методы, классовые переменные и прочее.

Объекты и классы
------------


Классы — это объекты, и у них тоже есть специальные атрибуты __class__ и __dict__.

>>> class A(object):
...     pass
... 

У класса тип type.

>>> A.__class__
<type 'type'>

Правда __dict__ у классов не совсем словарь

>>> A.__dict__
<dictproxy object at 0x1111e88>

Но __dict__ ответственен за доступ к внутреннему пространству имен, в котором хранятся методы, дескрипторы, переменные, свойства и прочее:

>>> dict(A.__dict__)
{'__module__': '__main__', 'qux': 'A', '__dict__': <attribute '__dict__' of 'A' objects>, 'foo': <function foo at 0x7f7797a25c08>, '__weakref__': <attribute '__weakref__' of 'A' objects>, '__doc__': None}
>>> A.__dict__.keys()
['__module__', 'qux', '__dict__', 'foo', '__weakref__', '__doc__']<

В классах помимо __class__ и __dict__, имеется еще несколько специальных атрибутов: __bases__ — список прямых родителей, __name__ — имя класса. [1]

Классы можно считать эдакими расширениями обычных объектов, которые реализуют интерфейс типа. Множество всех классов (или типов) принадлежат множеству всех объектов, а точнее является его подмножеством. Иначе говоря, любой класс является объектом, но не всякий объект является классом. Договоримся называть обычными объектами(regular objects) те объекты, которые классами не являются.

Небольшая демонстрация, которая станет лучше понятна чуть позже.
Класс является объектом.
>>> class A(object):
...     pass
... 

>>> isinstance(A, object)
True

Число — это тоже объект.

>>> isinstance(42, object)
True

Класс — это класс (т.е. тип).

>>> isinstance(A, type)
True

А вот число классом (типом) не является. (Что такое type будет пояснено позже)

>>> isinstance(42, type)
False
>>>

Ну и a — тоже обычный объект.

>>> a = A()
>>> isinstance(a, A)
True
>>> isinstance(a, object)
True
>>> isinstance(a, type)
False

И у A всего один прямой родительский класс — object.

>>> A.__bases__
(<type 'object'>,)

Часть специальных параметров можно даже менять:

>>> A.__name__
'A'
>>> A.__name__ = 'B'
>>> A
<class '__main__.B'>

С помощью getattr получаем доступ к атрибутам класса:

>>> A.qux
'A'
>>> A.foo
<unbound method A.foo>
>>> 

Поиск атрибутов в обычном объекте
------------

В первом приближении алгоритм поиска выглядит так: сначала ищется в __dict__ объекта, потом идет поиск по __dict__ словарям класса объекта (который определяется с помощью __class__) и __dict__ его базовых классов в рекурсивном порядке.

Пример.

>>> class A(object):
...     qux = 'A'
...     def __init__(self, name):
...         self.name=name
...     def foo(self):
...         print 'foo'
... 
>>> a = A()
>>> b = A()

Т.к. в обычных объектах a и b нет в __dict__ атрибута 'qux', то поиск продолжается во внутреннем словаре __dict__ их типа (класса), а потом по __dict__ словарям родителей в определенном порядке:

>>> b.qux
'A'
>>> A.qux
'A'

Меняем атрибут qux у класса A. И соответственно должны поменяться значения, которые возвращают экземпляры класса A — a и b:

>>> A.qux='B'
>>> a.qux
'B'
>>> b.qux
'B'
>>> 

Точно так же в рантайме к классу можно добавить метод:

>>> A.quux = lambda self: 'i have quux method'
>>> A.__dict__['quux']
<function <lambda> at 0x7f7797a25b90>
>>> A.quux
<unbound method A.<lambda>>

И доступ к нему появится у экземпляров:

>>> a.quux()
'i have quux method'

Точно так же как и с любыми другими объектами, можно удалить атрибут класса, например, классовую переменную qux:

>>> del A.qux

Она удалиться из __dict__

>>> A.__dict__['qux']
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
KeyError: 'qux'

И доступ у экземляров пропадет.

>>> a.qux
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: 'A' object has no attribute 'qux'
>>> 

У классов почти такой же поиск атрибутов, как и у обычных объектов, но есть отличия: поиск начинается с собственного __dict__ словаря, а потом идет поиск по __dict__ словарям суперклассов (которые хранятся в __bases__) по опредленному алгоритму, а затем по классу в __class__ и его суперклассах. (Подробнее об этом позже).

Cсылки
------------

* Unifying types and classes in Python — главный документ, объясняющий что, как и зачем в новых классах. http://www.python.org/download/releases/2.2.3/descrintro/#references
* Making Types Look More Like Classes — PEP 252, описывающий отличие старых классов от новых. http://www.python.org/dev/peps/pep-0252/
* Built-in functions — детальное описание работы всех встроенных функций. http://docs.python.org/library/functions.html
* Data model — детальное описание модели данных python'а. http://docs.python.org/reference/datamodel.html
* Python types and objects — объяснение объектной модели python на простых примерах с картинками. http://www.cafepy.com/article/python_types_and_objects/python_types_and_objects.html


Примечания
-----------

[1] О __module__ и __doc__ для простоты изложения пока забудем. Полный список атрибутов класса можно посмотреть в документации http://docs.python.org/reference/datamodel.html#the-standard-type-hierarchy


Заметки об объектной системе языка 2
================================

(origin: http://habrahabr.ru/blogs/python/114585/)

Вторая часть заметок об объектной системе python'a (первая часть тут). В этой статье рассказывается, что такое классы, метаклассы, type, object и как происходит поиск атрибутов в классе. 

Классы
-----------

Классы (типы) — это объектные фабрики. Их главная задача — создавать объекты, обладающие определенным поведением.

Классы определяют поведение объектов с помощью своих атрибутов (которые хранятся в __dict__ класса): методов, свойств, классовых переменные, дескрипторов, а также с помощью атрибутов, унаследованных от родительских классов.

Инстанцирование обычного объекта происходит в 2 этапа: сначала его создание, потом инициализация. Соответственно, сначала запускается метод класса __new__, который возвращает объект данного класса, потом выполняется метод класса __init__, который инициализирует уже созданный объект.

def __new__(cls, ...) — статический метод (но его можно таковым не объявлять), который создает объект класса cls.

def __init__(self, ...) — метод класса, который инициализирует созданный объект.

Например, объявим класс:

>>> class A(object):
...     pass
... 
>>>

Для класса A не определены ни __new__, ни __init__. В соответствии с алгоритмом поиска атрибутов для класса (типа), который не стоит путать с алгоритмом поиска атрибутов для обычных объектов, когда класс не найдет их в своем__dict__, он будет искать эти методы в __dict__ своих базовых (родительских) классах.

Класс А имеет в качестве родителя встроенный класс object. Таким образом он будет их искать в object.__dict__

И найдет:

>>> object.__dict__['__init__']
<slot wrapper '__init__' of 'object' objects>
>>> object.__dict__['__new__']
<built-in method __new__ of type object at 0x82e780>
>>>

Раз есть такие методы, значит, получается, что a = A() аналогичен последовательности вызовов:

a = object.__new__(A)
object.__init__(a)

В общем виде, используя super, который как раз и реализует алгоритм поиска атрибутов по родительским классам [1]:

a = super(A, A).__new__(A)
super(A, A).__init__(a)

Пример.

>>> class A(object):
...     def __new__(cls):
...         obj = super(A, cls).__new__(cls)
...         print 'created object', obj
...         return obj
...     def __init__(self):
...         print 'initing object', self
...
>>> A()
created object <__main__.A object at 0x1620ed0>
initing object <__main__.A object at 0x1620ed0>
<__main__.A object at 0x1620ed0>
>>> 

Singleton v.1
-----------

Понимая, как происходит создание объекта, можно написать реализацию паттерна одиночка.

Мы должны гарантировать, что у класса есть только один экземпляр. Т.е. при вызове конструктора класса, всегда возвращаем один и тот же экземпляр класса.

А это значит, что при вызов метода __new__ должен возвращать каждый раз один и тот же объект. Хранить сам объект можно, например, в классовой переменной instance.

В результате получаем:

>>> class C(object):
...     instance = None
...     def __new__(cls):
...         if cls.instance is None:
...             cls.instance = super(C, cls).__new__(cls)
...         return cls.instance
... 
>>> C() is C()
True
>>> C().x = 1
>>> c = C()
>>> d = C()
>>> c.x
1
>>> d.x
1
>>> c.x=2
>>> d.x
2
>>> c.x
2

Классы и метаклассы
-----------

Для класса (типа), так же как и для обычного объекта, существует класс (тип), который создает классы и определяет поведение класса. Этот класс называется метаклассом. 

Создание класса, как и обычного объекта происходит с помощью вызова конструктора, но т.к. в классе есть несколько дополнительных специальных атрибутов, которые должны быть инициализированы, в конструктор передаются и соответствующие обязательные параметры. 

XClass = XMetaClass(name, bases, attrs)

Тогда, сразу после создания 
XClass.__name__ равно name,
XClass.__bases__ равен bases, 
XClass.__dict__ равен attrs, а
XClass.__class__ равен XMetaClass

По умолчанию для всех определяемых классов метаклассом является type.

Таким образом.

>>> class A(object):
...     pass
... 

Эквивалентно, по аналогии с обычными объектами:

>>> type('A', (object,), {})
<class '__main__.A'>

А это: 

>>> class B(A):
...     def foo(self):
...         42
...

эквивалентно

>>> type('B', (A,), {'foo': lambda self: 42})

При определении класса, можно задать свой метакласс с помощью
классовой переменной __metaclass__:

>>> class A(object):
...     __metaclass__ = Meta
...
>>>

Что равносильно: A = Meta('A', (object,), {})

О type и object
-----------

Прежде всего type и object — это объекты. И, как у всех порядочных объектов, у них есть специальные атрибуты __class__ и __dict__:

>>> object.__class__
<type 'type'>
>>> type.__class__
<type 'type'>

>>> object.__dict__
<dictproxy object at 0x7f7797a1cf30>
>>> type.__dict__
<dictproxy object at 0x7f7797a1cfa0>

Более того object, и type — это объекты типа (классы), и у них тоже есть специальные атрибуты __name__, __bases___:

>>> object.__name__
'object'
>>> type.__name__
'type'
>>> object.__bases__
()
>>> type.__bases__
(<type 'object'>,)
>>> 

Экземпляры типа или класса object — это объекты (любые). Т.е. любой объект — экземпляр класса object:

>>> isinstance(1, object)
True
>>> isinstance(setattr, object)
True
>>> isinstance("foo", object)
True
>>> isinstance(A, object)
True

Даже функция является объектом:
>>> def bar():
...     pass
... 
>>> isinstance(bar, object)
True

Кроме того, класс object сам является своим экземпляром:

>>> isinstance(object, object)
True

type тоже является его экземпляром:

>>> isinstance(type, object)
True

Инстанцирование — object() возвращает самый простой и общий объект:

>>> o = object()

У которого даже __dict__ нет, есть только __class__.

>>> o.__dict__
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: 'object' object has no attribute '__dict__'
>>> o.__class__
<type 'object'>
>>>

Экземпляры класса или типа type — это только другие классы или другие типы:

Число — это не класс

>>> isinstance(1, type)
False

Строка тоже

>>> isinstance("foo", type)
False

Встроенная функция setattr тоже не класс.

>>> isinstance(setattr, type)
False

Класс — это класс. 

>>> isinstance(A, type)
True

Тип строки — это класс.

>>> isinstance("foo".__class__, type)
True

Т.к. object и type — тоже классы, то они являются экземплярами класса type:

>>> isinstance(object, type)
True
>>> isinstance(type, type)
True
>>> 

Т.к. множество классов (типов) являются подмножеством множества объектов, то логично предположить, что type является подклассом object, т.е.

>>> issubclass(type, object)
True
>>> issubclass(object, type)
False

type — это просто класс, экземплярами которого являются другие классы. (т.е. метакласс). А сами классы можно считать расширением простых, обычных объектов.

Таким образом, когда мы наследуем класс от object, этот класс автоматически наследует поведение класса object, т.е. при инстанцировании он будет возвращать обычный объект. А когда мы наследуем от класса type, мы также автоматически наследуем поведение класса type, т.е. при инстацированни будет создаваться класс. А класс, который создает класс, называется метаклассом.

Значит, чтобы определить просто класс, нужно наследовать его от object, чтобы определить метакласс — наследуем его от type.

И еще: не нужно путать type(a) и type(name, bases, attrs).
type(a) — вызов с одним аргументом, возвращает тип объекта,
a type(name, bases, attrs) — вызов с тремя аргументами — это вызов конструктора класса.

О поиске атрибутов в классе

Как уже было отмечено, алгоритм поиска атрибутов в обычном объекте, но есть некоторые тонкости, т.к. у типов (классов) есть __bases__ — родительские классы (типы).

Если атрибут есть в __dict__ возвращается он, затем идет поиск по базовым классам из __bases__, а потом идет обращение к __dict__ __class__'а (т.е. фактически метакласса) и его (метакласса) родительских классов (метаклассов).

Небольшой пример:

>>> class Ameta(type):
...     def foo(cls):
...         print 'Ameta.foo'
...
>>> class A(object):
...     __metaclass__ = Ameta
... 
>>> A.foo()
Ameta.foo

Все что определяется в метаклассе доступно для класса, но не доступно для экзмепляров класса — обычных объектов, т.к. поиск атрибутов в обычном объекте ведется только по __dict__ словарям класса. 

>>> a = A()
>>> a.foo
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'A' object has no attribute 'foo'

В A.__dict__ 'foo' нет:

>>> A.__dict__['foo']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'foo'

Зато он есть в метаклассе, поэтому:

>>> A.foo()
Ameta.foo

>>> class B(A):
...     @classmethod
...     def foo(cls):
...         print 'B.foo'
... 
>>> B.foo   # т.к. foo есть B.__dict__ вернется значение B.__dict__['foo']
<bound method Ameta.foo of <class '__main__.B'>>
>>> B.foo()
B.foo

>>> class C(B):
...     pass
... 
>>> C.foo()  # вернет значение из базового класса B.
B.foo

Экземпляр класса C также вызовет метод foo из класса B.

>>> c= C()
>>> c.foo()
B.foo

>>> class D(A): 
...     pass
... 
>>> D.foo()
Ameta.foo

А экземпляр D не найдет:

>>> d = D()
>>> d.foo()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'D' object has no attribute 'foo'

Метаклассы
-----------

Метаклассы являются фабриками классов (или типов). Инстанцирование класса тоже проходит в 2 этапа — создание объекта типа (класса) и его инициализация. Это также делается с помощью двух методов метакласса. Сначала вызывается метод __new__ метакласса с параметрами, необходимыми для создания класса — name, bases, attrs, а потом __init__ с теми же параметрами и уже созданным классом.

Пример.

>>> class Meta(type):
...     pass
... 
>>> Meta('A', (object,), {})
<class '__main__.A'>

В начале метакласс Meta ищет метод __new__ у себя в словаре __dict__, не находит его там и начинает искать в __dict__ своих родительских классах (т.е. метаклассах, в данном случае type), т.е. происходит обычный поиск атрибута в классе. В результате исполнения __new__ с соответствующими параметрами получает новый класс, который потом инициализируется вызовом __init__ метода метакласса.

В совсем развернутом виде получается:

cls = type.__dict__['__new__'](Meta, 'A', (object,), {})
type.__dict__['__init__'](cls, 'A', (object,), {})

Или с помощью super

cls = super(Meta, Meta).__new__(Meta, 'A', (object,), {})
super(Meta, Meta).__init__(cls, 'A', (object,), {})

Стоит отметить, что в отличие от инстанцирования обычных объектов, используется не object.__new__ и object.__init__, а type.__new__ и type.__init__. У object.__new__ и type.__new__ разные сигнатуры, и object.__new__ возвращает обычный объект (regular object), а type.__new__ — объект типа (typeobject), т.е. класс.

Посмотрим, как это все работает на примере.

>>> class Meta(type):
...     def __new__(mcls, name, bases, attrs):
...         print 'creating new class', name
...         return super(Meta, mcls).__new__(mcls, name, bases, attrs)
...     def __init__(cls, name, bases, attrs):
...         print 'initing new class', name
...         
... 
>>> class A(object):
...     __metaclass__ = Meta
... 
creating new class A
initing new class A

Во время инстанцирования просто объекта, никаких надписей не выводится.

>>> a = A()
>>> 

Кроме того, соответственно, во методах __new__ и __init__ метакласса можно менять все: имя, список суперклассов, атрибуты.

Cсылки
-----------

* Unifying types and classes in Python — главный документ, объясняющий что, как и зачем в новых классах. http://www.python.org/download/releases/2.2.3/descrintro/#references
* Making Types Look More Like Classes — PEP 252, описывающий отличие старых классов от новых. http://www.python.org/dev/peps/pep-0252/
* Built-in functions — детальное описание работы всех встроенных функций. http://docs.python.org/library/functions.html
* Data model — детальное описание модели данных python'а. http://docs.python.org/reference/datamodel.html
* Python types and objects — объяснение объектной модели python на простых примерах с картинками. http://www.cafepy.com/article/python_types_and_objects/python_types_and_objects.html


Примечания
-----------

[1] Более подробно про super — тут. http://docs.python.org/library/functions.html#super


Заметки об объектной системе языка 3
================================

(origin: http://habrahabr.ru/blogs/python/114587/)


Третья часть заметок об объектной системе python'a (первая и вторая части). В статье рассказывается о том, почему c.__call__() не то же самое, что и c(), как реализовать singleton с помощью метаклассов, что такое name mangling и как оно работает.



c.__call__ vs c(), c.__setattr__ vs setattr

Легко убедиться, что x(arg1, arg2) не равносильно x.__call__(arg1, arg2) для новых классов, хотя для старых это справедливо.

>>> class C(object):
...     pass
... 
>>> c = C() 
>>> c.__call__ = lambda: 42
>>> c() 
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'C' object is not callable
>>> C.__call__ = lambda self: 42
>>> c() 
42

На самом деле правильно:

c() <=> type(c).__call__(с)

Абсолютно такая же ситуация с __setattr__/setattr и многими другими магическими (и специальными) методами и соответствующими встроенными функциями, которые определены для всех объектов, в том числе и для объектов типа — классов.

Зачем это было сделано можно рассмотреть на примере setattr [1].
В начале убедимся, что setattr(a, 'x', 1)  <==> type(a).__setattr__(a, 'x', 1).

a.x = 1 <=> setattr(a, 'x', 1)

>>> class A(object): pass
... 
>>> a = A() 
>>> a.x = 1
>>> a
<__main__.A object at 0x7fafa9b26f90>
>>> setattr(a, 'y', 2)
>>> a.__dict__
{'y': 2, 'x': 1}

Устанавливаем с помощью метода __setattr__ новый атрибут, который пойдет в __dict__

>>> a.__setattr__('z', 3)

вроде бы все правильно:

>>> a.__dict__
{'y': 2, 'x': 1, 'z': 3}

Однако:

Установим в a.__setattr__ заведомо неправильный метод:

>>> a.__setattr__ = lambda self: 42

Вызов, которого приводит к ошибке:

>>> a.__setattr__('z', 4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: <lambda>() takes exactly 1 argument (2 given)

Однако, несмотря на это, setattr работает:

>>> setattr(a, 'foo', 'bar')
>>> a.__dict__
{'y': 2, 'x': 1, '__setattr__': <function <lambda> at 0x7fafa9b3a140>, 'z': 3, 'foo': 'bar'}

А вот если переопределить метод класса:

>>> A.__setattr__ = lambda self: 42

то setattr для экземпляра класса выдаст ошибку:

>>> setattr(a, 'baz', 'quux')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: <lambda>() takes exactly 1 argument (3 given)

Зачем это было сделано?
Пусть setattr(a, 'x',1) тоже самое, что a.__setattr__('x', 1), тогда

>>> class A(object):
...     def __setattr__(self, attr, value):
...         print 'for instances', attr, value
...         object.__setattr__(self, attr, value)
... 
>>> a = A()

Установим новый атрибут для a. a.x = 1 <==> a.__setattr__('x', 1)
Все нормально:

>>> a.__setattr__('x', 1)
for instances x 1
>>> a.__dict__
{'x': 1}

А теперь попробуем установить новый атрибут для самого класса, он же ведь тоже является объектом: A.foo = 'bar' <==> A.__setattr__('foo', 'bar')

>>> A.__setattr__('foo', 'bar')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unbound method __setattr__() must be called with A instance as first argument (got str instance instead)

Все логично, согласно алгоритму поиска атрибутов в классах (типах), сначала атрибут ищется в __dict__ класса (типа):

>>> A.__dict__['__setattr__']
<function __setattr__ at 0x7f699d22fa28>

Но дело в том, что он предназначен для экземпляров класса, а не для самого класса. Поэтому вызов A.__setattr__('foo', 'bar') будет неправильным. И именно поэтому setattr() должен делать явный поиск в классе (типе) объекта. Собственно, по этой же причине это сделано и для других магических методов __add__, __len__, __getattr__ и т.д.



Класс, как вызываемый (callable) тип
----------------------------

Класс (тип) — это вызываемый (callable) тип, и его вызов — это конструктор объекта.

>>> class C(object):
...     pass
...
>>> С()
<__main__.C object at 0x1121e10>

Эквивалентно:

>>> type(C).__call__(C)
<__main__.C object at 0x1121ed0>

Т.к. C — обычный класс, то его метаклассом является type, поэтому будет использован вызов type(C).__call__(С) <==> type.__call__(С). Внутри type.__call__(C) уже происходит вызов C.__new__(cls, ...) и C.__init__(self, ...).

Важно то, что и __new__ и __init__ ищутся с помощью обычного алгоритма поиска атрибутов в классе. И при отсутствии их в C.__dict__, будут вызваны методы из родительского класса object: object.__new__ и object.__init__, в то время как метод __call__ — это метод класса (типа) объекта — type: type.__call__(C).


Singleton v.2
----------------------------

Зная это, создадим метаклассную реализацию синглтона.

Что нам нужно от синглтона? Чтобы вызов A() возвращал один и тот же объект.

A() <=> type(A).__call__(A)

Значит, нам нужно изменить поведение метода __call__, который определяется в метаклассе. Сделаем это, не забывая, что в общем случае в __call__ могут передаваться любые параметры.

>>> class SingletonMeta(type):
...     def __call__(cls, *args, **kw):
...         return super(SingletonMeta, cls).__call__(*args, **kw)
... 
>>> 

Заглушка готова.
Пусть единственный объект будет храниться в классовом атрибуте instance. Для этого инициализируем в cls.instance в __init__.

>>> class SingletonMeta(type):
...     def __init__(cls, *args, **kw):
...         cls.instance = None
...     def __call__(cls, *args, **kw):
...         return super(SingletonMeta, cls).__call__(*args, **kw)
... 
>>> 


И вставим проверку в __call__:

>>> class SingletonMeta(type):
...     def __init__(cls, *args, **kw):
...         cls.instance = None
...     def __call__(cls, *args, **kw):
...         if cls.instance is None:
...             cls.instance = super(SingletonMeta, cls).__call__(*args, **kw)
...         return cls.instance
... 
>>> class C(object):
...     __metaclass__ = SingletonMeta
... 

Проверяем, что все работает как надо.

>>> C() is C()
True
>>> a = C()
>>> b = C()
>>> a.x = 42
>>> b.x
42
>>> 

Вызываемый (callable) тип в качестве метакласса

Метаклассом может быть не только объект типа type, но и вообще любой вызываемый (callable) тип.

Достаточно просто создать функцию, в которой создается класс с помощью метакласса type.

>>> def mymeta(name, bases, attrs):
...     attrs['foo'] = 'bar'
...     return type(name, bases, attrs)
...
>>> class D(object):
...     __metaclass__ = mymeta
... 
>>> D() 
<__main__.D object at 0x7fafa9abc090>
>>> d = D() 
>>> d.foo
'bar'
>>> d.__dict__
{}
>>> D.__dict__
<dictproxy object at 0x7fafa9b297f8>
>>> dict(D.__dict__)
{'__module__': '__main__', '__metaclass__': <function mymeta at 0x7fafa9b3a9b0>, '__dict__': <attribute '__dict__' of 'D' objects>, 'foo': 'bar', '__weakref__': <attribute '__weakref__' of 'D' objects>, '__doc__': None}

Определения класса
----------------------------

Конструкция (statement) определения класса — это просто конструкция. Также как и любое statement оно может появляться где угодно в коде программы.

>>> if True:
...     class A(object):
...         def foo(self):
...             print 42
... 
>>> A
<class '__main__.A'>
>>> A().foo() 
42
>>> 

В конструкции 'class' любые определенные «внутри» переменные, функции, классы, накапливаются в __dict__. А в определении можно использовать любые другие конструкции — циклы, if'ы:.

Поэтому можно делать так:

>>> class A(object):
...     if 1 > 2:
...         def foo(self):
...             print '1>2'
...     else:
...         def bar(self):
...             print 'else'
... 
>>> 
>>> A()
<__main__.A object at 0x7fafa9abc150>
>>> A().foo()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'A' object has no attribute 'foo'
>>> A().bar() 
else

или так
>>> class A(object):
...     if 1 > 2: 
...         x = 1
...         def foo(self):
...             print 'if'
...     else:
...         y = 1
...         def bar(self):
...             print 'else'
... 
>>> A.x
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: type object 'A' has no attribute 'x'
>>> A.y
1
>>> A.foo
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: type object 'A' has no attribute 'foo'
>>> A.bar
<unbound method A.bar>
>>> A.bar()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unbound method bar() must be called with A instance as first argument (got nothing instead)
>>> A().bar()
else
>>> 

Можно вкладывать одно определение в другое.

>>> class A(object):
...     class B(object):
...         pass
...     
... 
>>> A()
<__main__.A object at 0x7fafa9abc2d0>
>>> A.__dict__
<dictproxy object at 0x7fafa9b340f8>
>>> dict(A.__dict__)
{'__dict__': <attribute '__dict__' of 'A' objects>, '__module__': '__main__', 'B': <class '__main__.B'>, '__weakref__': <attribute '__weakref__' of 'A' objects>, '__doc__': None}
>>> A.B()
<__main__.B object at 0x7fafa9abc310>


Или же динамически создавать методы класса:

>>> FIELDS=['a', 'b', 'c']
>>> class A(object):
...     for f in FIELDS:
...         locals()[f] = lambda self: 42
... 
>>> a = A() 
>>> a.a()
42
>>> a.b()
42
>>> a.c()
42
>>> a.d()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'A' object has no attribute 'd'
>>>

Хотя конечно такое наверняка крайне не рекомендуется делать в обычной практике, и лучше воспользоваться более идиоматичными средствами. 

Name mangling

И еще про определения класса. Про name mangling.

Любой атрибут внутри определения класса classname вида ".__{attr}" (attr при этом имеет не более одного _ в конце) подменяется на "_{classname}__{attr}". Таким образом, внутри классов можно иметь «скрытые» приватные атрибуты, которые не «видны» наследникам и экземплярам класса.

>>> class A(object):
...     __private_foo=1
...
>>> A.__private_foo
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: type object 'A' has no attribute '__private_foo'

Увидеть переменную можно так:

>>> A._A__private_foo
1

Ну и храниться она в __dict__ класса:

>>> dict(A.__dict__)
{'__dict__': <attribute '__dict__' of 'A' objects>, '_A__private_foo': 1, '__module__': '__main__', '__weakref__': <attribute '__weakref__' of 'A' objects>, '__doc__': None}
>>> 

Наследники доступа не имеют:

>>> class B(A):
...     def foo(self):
...         print self.__private_foo
... 
>>> B().foo()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in foo
AttributeError: 'B' object has no attribute '_B__private_foo'

В принципе обеспечить доступ внешний доступ к атрибутам типа __{attr} внутри определения класса, т.е. обойти name_mangling, можно с помощью __dict__. 

>>> class C(object):
...     def __init__(self):
...         self.__dict__['__value'] = 1
... 
>>> C().__value
1
>>> 

Однако, такие вещи крайне не рекомендуется делать из-за того, что доступ к таким атрибутам будет невозможен внутри определения любого другого класса из-за подмены ".__{attr}" на "._{classname}__{attr}" вне зависимости к какому объекту или классу они относятся, т.е.

>>> class D(object):
...     def __init__(self):
...         self.c = C().__value
... 
>>> D() 
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in __init__
AttributeError: 'C' object has no attribute '_D__value'
>>> C().__value
1
>>>

Хотя С().__value прекрасно отработает вне определения класса. Чтобы обойти также придется использовать __dict__['__value'].

Cсылки
------------------

* Unifying types and classes in Python — главный документ, объясняющий что, как и зачем в новых классах. http://www.python.org/download/releases/2.2.3/descrintro/#references
* Making Types Look More Like Classes — PEP 252, описывающий отличие старых классов от новых. http://www.python.org/dev/peps/pep-0252/
* Built-in functions — детальное описание работы всех встроенных функций. http://docs.python.org/library/functions.html
* Data model — детальное описание модели данных python'а. http://docs.python.org/reference/datamodel.html
* Python types and objects — объяснение объектной модели python на простых примерах с картинками. http://www.cafepy.com/article/python_types_and_objects/python_types_and_objects.html

Примечания
------------------

[1] В официальной документации приводится пример с __len__/len и __hash__/hash. http://docs.python.org/reference/datamodel.html#special-method-lookup-for-new-style-classes


