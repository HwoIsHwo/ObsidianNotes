# Введение в gtest/gmock

## Общая идея unit-тестирования
Unit-тестирование — это проверка отдельных модулей (функций, классов) в изоляции от остальной системы. Цель — гарантировать корректность поведения и упростить отладку. В C++ для этого широко используется связка **Google Test (gtest)** и **Google Mock (gmock)**.
Фреймворк тестирования реализует подход xUnit: тесты описываются декларативно и выполняются автоматически, с генерацией отчётов о прохождении или падении тестов ([TutorialsPoint](https://www.tutorialspoint.com/gtest/gtest-quick-guide.htm?utm_source=chatgpt.com "GoogleTest - Quick Guide")).


---
## Google Test (gtest)
gtest — это фреймворк для написания unit-тестов на C++. Он предоставляет:
- инфраструктуру запуска тестов
- набор макросов для проверок (assertions)
- средства группировки и организации тестов
Основной принцип — тестировать код в изоляции, проверяя ожидаемое поведение.

### Базовая структура теста
```cpp
#include <gtest/gtest.h>

// Тестовый набор (Test Suite) + конкретный тест (Test Case)
TEST(FactorialTest, HandlesPositiveInput) {
    EXPECT_EQ(Factorial(3), 6);  // проверка результата
}
```
Точка входа:
```cpp
int main(int argc, char** argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS(); // запуск всех тестов
}
```

### Assertions
gtest предоставляет два типа проверок:
- `EXPECT_*` — non-fatal (тест продолжается)
- `ASSERT_*` — fatal (тест немедленно прерывается)

Примеры:
```cpp
EXPECT_EQ(a, b);   // равенство
EXPECT_NE(a, b);   // неравенство
ASSERT_TRUE(ptr);  // критическая проверка
```
Разница принципиальная: `ASSERT_*` имеет смысл использовать, когда дальнейшее выполнение теста невозможно (например, null pointer).

### Организация тестов
gtest вводит несколько уровней абстракции:
- Test Suite — группа тестов
- Test Case — отдельный тест
- Test Fixture — общий контекст (setup/teardown)

Пример fixture:
```cpp
class MyTest : public ::testing::Test {
protected:
    void SetUp() override {
        // инициализация
    }

    void TearDown() override {
        // очистка
    }

    int value = 0;
};

TEST_F(MyTest, Example) {
    value = 10;
    EXPECT_EQ(value, 10);
}
```


---
## Google Mock (gmock)
gmock — это библиотека для создания mock-объектов, которая используется совместно с gtest.
Ключевая идея: заменить реальные зависимости (БД, сеть, железо) на управляемые объекты, чтобы тестировать модуль изолированно ([Stack Overflow](https://stackoverflow.com/questions/13696376/what-is-the-difference-between-gtest-and-gmock?utm_source=chatgpt.com "c++ - What is the difference between gtest and gmock? - Stack Overflow")).
Mock-объект:
- реализует тот же интерфейс, что и реальный объект
- позволяет задать ожидаемое поведение
- проверяет, как код взаимодействует с зависимостями ([Google](https://google.github.io/googletest/gmock_for_dummies.html?utm_source=chatgpt.com "gMock for Dummies | GoogleTest"))

### Когда нужен gmock
Используется, если:
- зависимость медленная (БД, сеть)
- зависимость нестабильная
- нужно проверить взаимодействие (а не только результат)
- система ещё не полностью реализована

### Пример mock-класса
```cpp
#include <gmock/gmock.h>

class IDataProvider {
public:
    virtual ~IDataProvider() = default;
    virtual int GetValue() = 0;
};

class MockDataProvider : public IDataProvider {
public:
    MOCK_METHOD(int, GetValue, (), (override));
};
```

### Использование mock
```cpp
TEST(MyTest, UsesMock) {
    MockDataProvider mock;

    // задаём поведение
    EXPECT_CALL(mock, GetValue())
        .Times(1)
        .WillOnce(::testing::Return(42));

    int result = mock.GetValue();

    EXPECT_EQ(result, 42);
}
```
Здесь проверяется не только результат, но и факт вызова метода.

## Отличие gtest и gmock
- gtest — тестовый фреймворк (запуск, assertions)
- gmock — инструмент для имитации зависимостей
gmock не является самостоятельным тестовым фреймворком, а расширяет возможности gtest ([Stack Overflow](https://stackoverflow.com/questions/13696376/what-is-the-difference-between-gtest-and-gmock?utm_source=chatgpt.com "c++ - What is the difference between gtest and gmock? - Stack Overflow")).


---
## Концептуальные моменты
### 1. Изоляция
Каждый тест должен работать независимо. Mock-объекты позволяют убрать внешние зависимости.

### 2. Проверка поведения vs состояния
- gtest → проверка состояния (результат функции)
- gmock → проверка поведения (как вызывались методы)

### 3. Test-Driven Development (TDD)
gtest/gmock хорошо вписываются в TDD:
1. написать тест
2. получить падение
3. реализовать код
4. добиться прохождения


---
## Минимальный workflow
1. Подключить gtest/gmock
2. Написать тесты через `TEST`
3. Добавить assertions
4. При необходимости — создать mock
5. Собрать и запустить бинарник тестов

Теги: #Прога 
Ссылки:
