gmock (Google Mock) — расширение к gtest, предназначенное для создания mock-объектов и проверки взаимодействия между компонентами. В отличие от gtest, который проверяет состояние и результаты, gmock позволяет описывать ожидаемые вызовы методов и их поведение.

Основная идея — заменить зависимость объектом, который:
1. задаёт ожидаемые вызовы
2. возвращает контролируемые значения
3. проверяет корректность использования

## Объявление mock-класса
Mock создаётся на основе интерфейса (или виртуального класса). Используется макрос `MOCK_METHOD`.
```cpp
#include <gmock/gmock.h>

class IDataBase {
public:
    virtual ~IDataBase() = default;
    virtual int get(int id) = 0;
};

class MockDataBase : public IDataBase {
public:
    MOCK_METHOD(int, get, (int id), (override));
};
```

Синтаксис:
```cpp
MOCK_METHOD(return_type, method_name, (args...), (specifiers));
```

Где:
- `return_type` — тип возвращаемого значения
- `args` — список аргументов в скобках
- `specifiers` — `override`, `const`, `noexcept` и т.д.

Для const-методов:
```cpp
MOCK_METHOD(int, get, (int id), (const, override));
```

---
## EXPECT_CALL — задание ожиданий
Основной механизм — `EXPECT_CALL`. Он описывает, какие вызовы ожидаются.
```cpp
TEST(ServiceTest, CallsDatabase) {
    MockDataBase mock;

    EXPECT_CALL(mock, get(10));

    mock.get(10);
}
```
Если вызов не произойдёт или параметры не совпадут — тест упадёт.


---
## Управление количеством вызовов
```cpp
EXPECT_CALL(mock, get(10))
    .Times(1);

EXPECT_CALL(mock, get(10))
    .Times(0); // запрещён вызов
```

Допустимые варианты:
- `Times(1)` — ровно один раз
- `Times(n)` — n раз
- `Times(::testing::AnyNumber())`
- `Times(::testing::AtLeast(n))`
- `Times(::testing::AtMost(n))`


---
## Возвращаемые значения
Задаются через `WillOnce` и `WillRepeatedly`.
```cpp
EXPECT_CALL(mock, get(10))
    .WillOnce(::testing::Return(5));
```
Несколько вызовов:
```cpp
EXPECT_CALL(mock, get(10))
    .WillOnce(::testing::Return(1))
    .WillOnce(::testing::Return(2));
```
Поведение по умолчанию:
```cpp
EXPECT_CALL(mock, get(::testing::_))
    .WillRepeatedly(::testing::Return(0));
```


---
## Matchers (сопоставление аргументов)
Аргументы можно проверять не только на равенство, но и через matchers:
```cpp
using ::testing::_;

EXPECT_CALL(mock, get(_)); // любой аргумент
```
Более сложные:
```cpp
using ::testing::Gt;
using ::testing::Lt;

EXPECT_CALL(mock, get(Gt(5)));  // > 5
EXPECT_CALL(mock, get(Lt(10))); // < 10
```
Комбинирование:
```cpp
using ::testing::AllOf;

EXPECT_CALL(mock, get(AllOf(Gt(0), Lt(10))));
```


---
## Действия (Actions)
Помимо `Return`, можно выполнять произвольный код.
```cpp
EXPECT_CALL(mock, get(10))
    .WillOnce(::testing::Invoke([](int id) {
        return id * 2;
    }));
```
Или:
```cpp
EXPECT_CALL(mock, get(::testing::_))
    .WillRepeatedly(::testing::Invoke([](int id) {
        return id + 1;
    }));
```


---
## ON_CALL — поведение по умолчанию
`ON_CALL` задаёт поведение без строгих ожиданий.
```cpp
ON_CALL(mock, get(::testing::_))
    .WillByDefault(::testing::Return(42));
```
Разница:
- `EXPECT_CALL` — проверяет, что вызов был
- `ON_CALL` — только задаёт поведение

Часто используется вместе:
```cpp
ON_CALL(mock, get(::testing::_))
    .WillByDefault(::testing::Return(0));

EXPECT_CALL(mock, get(10))
    .Times(1)
    .WillOnce(::testing::Return(5));
```


---
## Последовательность вызовов
Можно задать порядок:
```cpp
using ::testing::InSequence;

TEST(Test, OrderMatters) {
    InSequence seq;

    EXPECT_CALL(mock, get(1));
    EXPECT_CALL(mock, get(2));
}
```
Теперь `get(1)` должен быть вызван до `get(2)`.


---
## Проверка отсутствия вызовов
```cpp
EXPECT_CALL(mock, get(::testing::_))
    .Times(0);
```
Это используется для проверки, что зависимость не была задействована.


---
## NiceMock и StrictMock
По умолчанию неожиданные вызовы не приводят к падению теста (только warning). Это можно изменить.
```cpp
::testing::StrictMock<MockDataBase> mock;
```
Теперь любой неожиданный вызов — ошибка.
```cpp
::testing::NiceMock<MockDataBase> mock;
```
Игнорирует неожиданные вызовы.


---
## Частые ошибки
Основные проблемы при работе с gmock:
- проверка внутренней реализации вместо поведения
- избыточное количество `EXPECT_CALL`
- жёсткая привязка к порядку вызовов без необходимости
- отсутствие `WillRepeatedly`, что приводит к падению при дополнительных вызовах
- смешивание `ON_CALL` и `EXPECT_CALL` без понимания приоритетов

Теги: #Прога 
Ссылки:
[[Использование gmock]]
[[Mock и Fake объекты в тестировании]]