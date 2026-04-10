Google Test (gtest) предоставляет декларативный API для описания тестов. Синтаксис основан на макросах, но по сути формирует обычные C++ функции, которые регистрируются во внутреннем реестре и затем выполняются рантаймом.

Базовая структура теста включает три элемента: объявление теста, assertions и точку входа.

Минимальный пример:
```cpp
#include <gtest/gtest.h>

int sum(int a, int b) {
    return a + b;
}

TEST(SumTest, HandlesPositive) {
    EXPECT_EQ(sum(2, 3), 5);
}

int main(int argc, char** argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

Макрос `TEST(TestSuiteName, TestName)` генерирует класс и функцию. `TestSuiteName` — логическая группа, `TestName` — конкретный сценарий. На уровне бинарника все тесты собираются и запускаются через `RUN_ALL_TESTS()`.

Ключевая часть — assertions. Это проверки, которые определяют результат теста. Они делятся на две категории: фатальные (`ASSERT_*`) и нефатальные (`EXPECT_*`). Фатальные прерывают выполнение текущего теста, нефатальные — продолжают.

Простейшие проверки равенства:
```cpp
EXPECT_EQ(a, b);
EXPECT_NE(a, b);

ASSERT_EQ(a, b);
ASSERT_NE(a, b);
```

Для булевых выражений:
```cpp
EXPECT_TRUE(cond);
EXPECT_FALSE(cond);
```

Для указателей:
```cpp
EXPECT_EQ(ptr, nullptr);
ASSERT_NE(ptr, nullptr);
```

Строки требуют отдельной обработки, так как `==` сравнивает указатели:
```cpp
EXPECT_STREQ(str1, str2);   // C-style строки
EXPECT_STRNE(str1, str2);
```

Для `std::string` можно использовать обычный `EXPECT_EQ`.

Сравнения:
```cpp
EXPECT_LT(a, b);  // <
EXPECT_LE(a, b);  // <=
EXPECT_GT(a, b);  // >
EXPECT_GE(a, b);  // >=
```

Проверка исключений:
```cpp
EXPECT_THROW(func(), std::runtime_error);
EXPECT_NO_THROW(func());
EXPECT_ANY_THROW(func());
```

Для чисел с плавающей точкой используется сравнение с допуском:
```cpp
EXPECT_FLOAT_EQ(a, b);   // для float
EXPECT_DOUBLE_EQ(a, b);  // для double

EXPECT_NEAR(a, b, 1e-6); // универсальный вариант
```
`EXPECT_FLOAT_EQ` и `EXPECT_DOUBLE_EQ` учитывают ULP (единицы последнего разряда), тогда как `EXPECT_NEAR` — абсолютную погрешность.

Следующий уровень — пользовательские сообщения. Любой assertion можно дополнить выводом:
```cpp
EXPECT_EQ(a, b) << "Unexpected result for input X";
```
Это полезно при параметризованных или сложных тестах.

Для явного провала:
```cpp
FAIL() << "Critical error";
ADD_FAILURE() << "Non-fatal failure";
```
`FAIL()` эквивалентен `ASSERT_*`, `ADD_FAILURE()` — `EXPECT_*`.

Структурирование тестов осуществляется через fixtures. Это классы, наследуемые от `::testing::Test`.
```cpp
class MyTest : public ::testing::Test {
protected:
    int value;

    void SetUp() override {
        value = 10;
    }

    void TearDown() override {
        // очистка
    }
};
```

Тесты используют `TEST_F`:
```cpp
TEST_F(MyTest, CheckValue) {
    EXPECT_EQ(value, 10);
}
```
Каждый тест получает новый экземпляр fixture. Жизненный цикл: конструктор → `SetUp()` → тест → `TearDown()` → деструктор.

Для общей инициализации на уровне группы:
```cpp
class HeavyTest : public ::testing::Test {
protected:
    static void SetUpTestSuite() {
        // один раз
    }

    static void TearDownTestSuite() {
        // один раз
    }
};
```

Параметризованные тесты позволяют запускать один и тот же тест с разными входными данными.
```cpp
class ParamTest : public ::testing::TestWithParam<int> {};

TEST_P(ParamTest, Works) {
    int v = GetParam();
    EXPECT_GE(v, 0);
}

INSTANTIATE_TEST_SUITE_P(
    MyGroup,
    ParamTest,
    ::testing::Values(1, 2, 3)
);
```
Это снижает дублирование и делает тесты компактнее.

Также поддерживаются типизированные тесты, когда один и тот же тест выполняется для разных типов.
```cpp
template <typename T>
class TypedTest : public ::testing::Test {};

using MyTypes = ::testing::Types<int, double>;
TYPED_TEST_SUITE(TypedTest, MyTypes);

TYPED_TEST(TypedTest, DefaultConstructible) {
    TypeParam value{};
    SUCCEED();
}
```
Фильтрация тестов выполняется через аргументы командной строки:
```bash
./test_binary --gtest_filter=SumTest.*
```

Можно отключать тесты, добавив префикс `DISABLED_`:
```cpp
TEST(SumTest, DISABLED_SkippedTest) {
    // не будет выполняться
}
```

gtest также предоставляет вспомогательные макросы для отладки:
```cpp
SUCCEED();  // явный успех
```

Отдельно стоит отметить, что assertions работают через поток вывода (`<<`), поэтому любые типы должны поддерживать `operator<<`, иначе сообщения будут неполными.

В итоге синтаксис gtest сводится к трём уровням:
* декларация тестов (`TEST`, `TEST_F`, `TEST_P`)
* проверки (assertions)
* организация (fixtures, параметры, фильтры)

Теги: #Прога 
Ссылки:
[[Введение gtest]]
[[Написание тестов gtest. Fixture]]
[[Написание unit тестов gtest]]