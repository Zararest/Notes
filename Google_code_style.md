# Введение 

В этом документе представлены основные моменты Google code style.  
Полное руководство лежит здесь [Google code style](https://google.github.io/styleguide/cppguide.html).

# Headers

Каждый файл `.cpp` должен быть ассоциирован со своим хедером. Если нужен файл только для включения в другой, то используется расширение `.inc`.

Реализации шаблонных классов лежат в самом хедере, а не в отдельном `impl.h`.

Все хедеры жолжны иметь защиту в формате: `<PROJECT>_<PATH>_<FILE>_H_.`

```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

...

#endif  // FOO_BAR_BAZ_H_
```

Нельзя полагаться на включение нужного файла из другого хедера.

Порядок включения хедеров (группы разделены пустыми строками):

1. Соответствующий текушему файлу хедер
2. С библиотеки
3. С++ библиотеки
4. другие библиотеки
5. хедеры проекта

Путь до файла не должен включать в себя `.` или `..`. Только минимальный необходимый путь:

```cpp
//google-awesome-project/src/base/logging.h 
#include "base/logging.h"
```

# Scoping

Весть код за редким исключением должен находжится в namescpace. Его окончание должно быть обюозначено комментарием с его названием.

```cpp
// In the .h file
namespace mynamespace {

// All declarations are within the namespace scope.
// Notice the lack of indentation.
class MyClass {
 public:
  ...
  void Foo();
};

}  // namespace mynamespace
```

```cpp
// In the .cc file
namespace mynamespace {

using ::foo::Bar; //добавление в этот namespace функции из другого namespace через глобальный

// Definition of functions is within scope of the namespace.
void MyClass::Foo() {
  ...
}

}  // namespace mynamespace
```

## Внутренняя линуовка

Если функция не используется за пределами `.cpp` файла, ее надо пометить как `static`.  
Такое не используется в `.h` файлах.

## Локальные переменные

Локальные переменные должны быть объявлены как можно ближе к месту использования. При этом каждая их них должна быть инициализирована.

Переменные, необходимые для циклов и условий должны быть объявлены в них:

```cpp
while (const char* p = strchr(str, '/')) str = p + 1;
```

Исключением могут являются случаи наличия клнструктора у испольхуемого объекта:

```cpp
// Inefficient implementation:
for (int i = 0; i < 1000000; ++i) {
  Foo f;  // My ctor and dtor get called 1000000 times each.
  f.DoSomething(i);
}

Foo f;  // My ctor and dtor get called once each.
for (int i = 0; i < 1000000; ++i) {
  f.DoSomething(i);
}
```

## Инициализация constexpr

Надо чекнуть Владимрова.

