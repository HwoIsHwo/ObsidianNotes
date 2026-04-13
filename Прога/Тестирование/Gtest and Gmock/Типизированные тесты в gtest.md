Типизированные тесты в gtest используются, когда необходимо проверить одно и то же поведение для разных типов данных. Это актуально для шаблонного (template) кода, где логика одинакова, но типы могут отличаться.
В отличие от параметризованных тестов, где меняются значения, здесь меняется сам тип. Это позволяет проверять корректность обобщённого кода без дублирования тестов.
Базовая идея: определяется шаблонный test fixture, затем задаётся список типов, и один тест выполняется для каждого типа из этого списка.

---
## Базовый пример
Определяется шаблонный fixture:
```cpp
#include <gtest/gtest.h>

template <typename T>
class TypedTest : public ::testing::Test {
public:
    T value;
};
```
Далее задаётся список типов:
```cpp
using MyTypes = ::testing::Types<int, double, float>;
```
Регистрация набора типов:
```cpp
TYPED_TEST_SUITE(TypedTest, MyTypes);
```
После этого можно писать тесты:
```cpp
TYPED_TEST(TypedTest, DefaultValue) {
    TypeParam v{};
    EXPECT_EQ(v, TypeParam{});
}
```
`TypeParam` — это специальное имя, обозначающее текущий тип из списка. gtest подставляет его автоматически для каждого инстанцирования.

В результате будет создано несколько тестов:
- для `int`
- для `double`
- для `float`
Каждый тест выполняется независимо.

---
## Работа с методами и состоянием
Если тест требует общего состояния, оно размещается в fixture:
```cpp
template <typename T>
class VectorTest : public ::testing::Test {
protected:
    std::vector<T> data;
};
```
Использование:
```cpp
TYPED_TEST(VectorTest, InitiallyEmpty) {
    EXPECT_TRUE(this->data.empty());
}
```
Важно: доступ к членам класса осуществляется через `this->`, так как используется шаблон.


## Проверка шаблонных функций
Типизированные тесты особенно полезны для проверки обобщённых алгоритмов:
```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}

template <typename T>
class AddTest : public ::testing::Test {};

using Types = ::testing::Types<int, double>;
TYPED_TEST_SUITE(AddTest, Types);

TYPED_TEST(AddTest, Works) {
    EXPECT_EQ(add(TypeParam{1}, TypeParam{2}), TypeParam{3});
}
```
Это позволяет гарантировать, что функция корректно работает для всех заявленных типов.

---
## Typed vs Type-Parameterized Tests
В gtest есть два похожих механизма:
- Typed Tests
- Type-Parameterized Tests

Различие в том, что Typed Tests требуют заранее заданного списка типов, а Type-Parameterized Tests позволяют переиспользовать один и тот же набор тестов в разных местах с разными типами.

Пример Type-Parameterized Tests:
```cpp
template <typename T>
class MyTest : public ::testing::Test {};

TYPED_TEST_SUITE_P(MyTest);

TYPED_TEST_P(MyTest, Works) {
    TypeParam v{};
    SUCCEED();
}

REGISTER_TYPED_TEST_SUITE_P(MyTest, Works);

using MyTypes = ::testing::Types<int, double>;

INSTANTIATE_TYPED_TEST_SUITE_P(MyGroup, MyTest, MyTypes);
```
Это более сложный механизм, но он позволяет отделить объявление тестов от их инстанцирования.

---
## Ограничения
Типизированные тесты требуют, чтобы:
- типы поддерживали используемые операции
- типы были корректно выводимы в сообщения (`operator<<`)
- код был действительно обобщённым

Если типы сильно различаются по поведению, использование typed tests может усложнить тесты вместо упрощения.

---
## Типичные ошибки
Часто встречаются следующие проблемы:
- забывают использовать `this->` при доступе к членам fixture
- используют типы, не поддерживающие нужные операции
- смешивают typed tests с параметризованными без необходимости
- усложняют тесты из-за чрезмерного обобщения

Теги: #Прога 
Ссылки:
[[Параметризированные тесты gtest]]
[[Синтаксис gtest]]