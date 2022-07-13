# Общий синтаксис тестов

Для работы с тестируемой функцией используются макросы `TEST()`, `TEST_F()` и так далее. В них поведение функции контролируется с помощью:
- `ASSERT_*` - генерирует фатальные ошибки
- `EXCEPT_*` - генерирует не фатальные ошибки

Для вывода пользовательского сообщения об ошибке используется оператор `<<`:

```cpp
ASSERT_EQ(x.size(), y.size()) << "Vectors x and y are of unequal length";
```

Список необходимых макросов:

1. Простейшие логические
- ASSERT_TRUE(condition);
- ASSERT_FALSE(condition);

2. Сравнение
- ASSERT_EQ(expected, actual); — =
- ASSERT_NE(val1, val2); — !=
- ASSERT_LT(val1, val2); — <
- ASSERT_LE(val1, val2); — <=
- ASSERT_GT(val1, val2); — >
- ASSERT_GE(val1, val2); — >=

3. Сравнение строк
- ASSERT_STREQ(expected_str, actual_str);
- ASSERT_STRNE(str1, str2);
- ASSERT_STRCASEEQ(expected_str, actual_str); — регистронезависимо
- ASSERT_STRCASENE(str1, str2); — регистронезависимо

4. Проверка на исключения
- ASSERT_THROW(statement, exception_type);
- ASSERT_ANY_THROW(statement);
- ASSERT_NO_THROW(statement);

5. Проверка предикатов
- ASSERT_PREDN(pred, val1, val2, ..., valN); — N <= 5
- ASSERT_PRED_FORMATN(pred_format, val1, val2, ..., valN); — работает аналогично предыдущей, но позволяет контролировать вывод

6. Сравнение чисел с плавающей точкой
- ASSERT_FLOAT_EQ(expected, actual); — неточное сравнение float
- ASSERT_DOUBLE_EQ(expected, actual); — неточное сравнение double
- ASSERT_NEAR(val1, val2, abs_error); — разница между val1 и val2 не превышает погрешность abs_error

7. Вызов отказа или успеха
- SUCCEED();
- FAIL();
- ADD_FAILURE();
- ADD_FAILURE_AT(«file_path», line_number);

Название теста не должно содержать нижних подчеркивааний:

```cpp
TEST(TestSuiteName, TestName) {
  ... test body ...
}

// Tests factorial of 0.
TEST(FactorialTest, HandlesZeroInput) {
  EXPECT_EQ(Factorial(0), 1);
}

// Tests factorial of positive numbers.
TEST(FactorialTest, HandlesPositiveInput) {
  EXPECT_EQ(Factorial(1), 1);
  EXPECT_EQ(Factorial(2), 2);
  EXPECT_EQ(Factorial(3), 6);
  EXPECT_EQ(Factorial(8), 40320);
}
```

Для запуска тестов используется `RUN_ALL_TESTS()`, которая возвращает 0 в случае успеха. Возвращаемое значение этой функции должно проверяться.

# Глобальная конфигурация объектов

Если в нескольких тестах необходимо использовать одинаковые данные, то их можно инициализировать один раз. Для этого существуют `Test Fixtures`.

Для создания такой инициализации необходимо:
1. Унаследовать класс от `::testing::Test`. В своем классе занести все поля в protected.
2. Определить все необходимые объекты.
3. Конструктором является функция `SetUp()` с `override`. Деструктором - `TearDown()`

Тело теста:

```cpp
TEST_F(TestFixtureName, TestName) {
  ... test body ...
}
```

В нем можно обращаться в объектам класса созданного выше.

Пример:

```cpp
template <typename E>  // E is the element type.
class Queue {
 public:
  Queue();
  void Enqueue(const E& element);
  E* Dequeue();  // Returns NULL if the queue is empty.
  size_t size() const;
  ...
};

class QueueTest : public ::testing::Test {
 protected:
  void SetUp() override {
     q1_.Enqueue(1);
     q2_.Enqueue(2);
     q2_.Enqueue(3);
  }

  // void TearDown() override {}

  Queue<int> q0_;
  Queue<int> q1_;
  Queue<int> q2_;
};

TEST_F(QueueTest, IsEmptyInitially) {
  EXPECT_EQ(q0_.size(), 0);
}

TEST_F(QueueTest, DequeueWorks) {
  int* n = q0_.Dequeue();
  EXPECT_EQ(n, nullptr);

  n = q1_.Dequeue();
  ASSERT_NE(n, nullptr);
  EXPECT_EQ(*n, 1);
  EXPECT_EQ(q1_.size(), 0);
  delete n;

  n = q2_.Dequeue();
  ASSERT_NE(n, nullptr);
  EXPECT_EQ(*n, 2);
  EXPECT_EQ(q2_.size(), 1);
  delete n;
}
```

# Сборка через Cmake
Чтобы подколючить GoogleTests к проекту надо прописать:

    cmake_minimum_required(VERSION 3.14)
    project(my_project)

    # GoogleTest requires at least C++14
    set(CMAKE_CXX_STANDARD 14)

    include(FetchContent)
    FetchContent_Declare(
      googletest
      GIT_REPOSITORY https://github.com/google/googletest.git
      GIT_TAG release-1.12.1
    )

Пример кода тестов:

```cpp
#include <gtest/gtest.h>

// Demonstrate some basic assertions.
TEST(HelloTest, BasicAssertions) {
  // Expect two strings not to be equal.
  EXPECT_STRNE("hello", "world");
  // Expect equality.
  EXPECT_EQ(7 * 6, 42);
}
```

Создание целей для тестов в CmakeLists:

    enable_testing()

    add_executable(
      hello_test
      hello_test.cc
    )
    target_link_libraries(
      hello_test
      GTest::gtest_main #тут надо просто написать gtest_main
    )

    include(GoogleTest)
    gtest_discover_tests(hello_test)