Параметризированные тесты в gtest предназначены для устранения дублирования, когда один и тот же сценарий необходимо проверить на разных входных данных. Вместо копирования теста несколько раз используется единая реализация, которая выполняется с набором параметров.

Базовая идея: тест определяется один раз, а данные подаются извне через механизм параметров. Каждый набор параметров порождает отдельный тест-кейс на этапе выполнения.

Основой служит класс `::testing::TestWithParam<T>`, где `T` — тип параметра.

Минимальный пример:
```cpp
#include <gtest/gtest.h>

int isEven(int x) {
    return x % 2 == 0;
}

class EvenTest : public ::testing::TestWithParam<int> {};

TEST_P(EvenTest, HandlesVariousInputs) {
    int value = GetParam(); // получение параметра
    EXPECT_EQ(isEven(value), value % 2 == 0);
}
```
Здесь `TEST_P` используется вместо `TEST`. Внутри теста доступна функция `GetParam()`, возвращающая текущий параметр.

Далее необходимо задать набор значений. Это делается через `INSTANTIATE_TEST_SUITE_P`:
```cpp
INSTANTIATE_TEST_SUITE_P(
    EvenNumbers,        // имя группы инстанцирования
    EvenTest,           // класс теста
    ::testing::Values(1, 2, 3, 4, 5)
);
```
В результате будет создано 5 тестов, каждый со своим значением параметра.

Ключевой момент: параметризованный тест не будет выполняться, если не вызвать `INSTANTIATE_TEST_SUITE_P`. Это типичная ошибка — объявить `TEST_P`, но забыть про инстанцирование.

Параметры могут быть не только примитивами, но и структурами. Это позволяет передавать сложные входные данные и ожидаемые результаты.
```cpp
struct TestCase {
    int input;
    int expected;
};

class SumTest : public ::testing::TestWithParam<TestCase> {};

int add(int a, int b) {
    return a + b;
}

TEST_P(SumTest, Works) {
    TestCase tc = GetParam();
    EXPECT_EQ(add(tc.input, 10), tc.expected);
}

INSTANTIATE_TEST_SUITE_P(
    SumCases,
    SumTest,
    ::testing::Values(
        TestCase{1, 11},
        TestCase{5, 15},
        TestCase{-1, 9}
    )
);
```
В реальных задачах это основной сценарий использования: объединение входных данных и ожидаемого результата в одну структуру.

gtest также поддерживает генерацию параметров. Например, диапазоны:
```cpp
INSTANTIATE_TEST_SUITE_P(
    RangeTest,
    EvenTest,
    ::testing::Range(0, 10) // [0, 10)
);
```

Или комбинирование:
```cpp
class PairTest : public ::testing::TestWithParam<std::tuple<int, int>> {};

TEST_P(PairTest, SumPositive) {
    auto [a, b] = GetParam();
    EXPECT_GT(a + b, 0);
}

INSTANTIATE_TEST_SUITE_P(
    Pairs,
    PairTest,
    ::testing::Combine(
        ::testing::Values(1, 2),
        ::testing::Values(3, 4)
    )
);
```
`Combine` создаёт декартово произведение наборов параметров.

Отдельный аспект — читаемость вывода. По умолчанию gtest генерирует имена тестов автоматически, что не всегда удобно. Для этого можно задать пользовательскую функцию генерации имён.
```cpp
std::string ParamName(const testing::TestParamInfo<int>& info) {
    return "Value_" + std::to_string(info.param);
}

INSTANTIATE_TEST_SUITE_P(
    Named,
    EvenTest,
    ::testing::Values(1, 2, 3),
    ParamName
);
```
Это упрощает анализ падений.

Параметризованные тесты можно комбинировать с fixtures. В этом случае класс наследуется сразу от `Test` и `TestWithParam`.
```cpp
class MyTest : public ::testing::Test,
               public ::testing::WithParamInterface<int> {
protected:
    int base = 10;
};

TEST_P(MyTest, WorksWithFixture) {
    EXPECT_EQ(base + GetParam(), 10 + GetParam());
}
```
Такой подход используется, когда требуется общее состояние и параметры одновременно.

Важно учитывать ограничения. Параметры копируются, поэтому тип должен быть копируемым. Также не стоит передавать слишком сложные объекты — это ухудшает читаемость и диагностику.

Типичная ошибка — перегрузка параметризации. Если тест становится сложным из-за большого количества параметров, лучше разбить его на несколько независимых тестов. Параметризация — инструмент для устранения дублирования, а не универсальное решение.

С точки зрения архитектуры параметризованные тесты особенно полезны для:
- проверки граничных значений
- тестирования алгоритмов
- валидации таблиц соответствия

Теги: #Прога 
Ссылки:
[[Синтаксис gtest]]
[[Написание тестов gtest. Fixture]]