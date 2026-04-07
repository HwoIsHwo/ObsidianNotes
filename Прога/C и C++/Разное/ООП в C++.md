ООП в C++ — это способ управлять сложностью через типы, инварианты и полиморфизм. Классические принципы: инкапсуляция, абстракция, наследование, полиморфизм. Плюс на практике почти всегда добавляют композицию и правило SOLID.


## 1. Инкапсуляция
Сокрытие внутреннего состояния и контроль доступа к нему.
Идея: объект сам гарантирует корректность своего состояния.
```
class Counter {
public:
    void increment() {
        ++value_;
    }

    int value() const {
        return value_;
    }

private:
    int value_ = 0;
};
```
Почему это важно:
- нельзя напрямую изменить `value_`
- можно централизованно добавить проверки
- инварианты не ломаются извне


## 2. Абстракция
Выделение существенного поведения и сокрытие деталей реализации.
Обычно через интерфейс:
```
class IStorage {
public:
    virtual ~IStorage() = default;
    virtual void save(const std::string& data) = 0;
};
```
Конкретная реализация:
```
class FileStorage : public IStorage {
public:
    void save(const std::string& data) override {
        // запись в файл
    }
};
```
Код работает через абстракцию:
```
void process(IStorage& storage) {
    storage.save("data");
}
```
`process` не знает, куда пишутся данные — файл, БД, сеть. Это снижает связанность.


## 3. Наследование
Механизм повторного использования интерфейса или реализации.
```
class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;
};

class Circle : public Shape {
public:
    explicit Circle(double r) : r_(r) {}

    double area() const override {
        return 3.14159 * r_ * r_;
    }

private:
    double r_;
};
```
Наследование — это отношение «is-a».  
`Circle` — это `Shape`.
Важно: использовать наследование для расширения поведения, а не просто ради повторного использования кода. Для этого чаще подходит композиция.


## 4. Полиморфизм
Способность работать с разными типами через общий интерфейс.
```
void print_area(const Shape& shape) {
    std::cout << shape.area() << "\n";
}
```
Использование:
```
Circle c(10.0);
print_area(c);
```
Вызов `area()` определяется во время выполнения (динамический полиморфизм через virtual).

Есть также статический полиморфизм — шаблоны:
```
template<typename T>
void print_area(const T& shape) {
    std::cout << shape.area() << "\n";
}
```
Здесь выбор происходит во время компиляции.


## 5. Композиция (часто важнее наследования)
Объект содержит другие объекты.
```
class Engine {
public:
    void start() {}
};

class Car {
public:
    void start() {
        engine_.start();
    }

private:
    Engine engine_;
};
```
`Car` не «является» `Engine`, он его содержит.
В современном C++ композиция предпочтительнее наследования, если нет реального отношения "is-a".


## 6. Принцип подстановки (LSP)
Объект производного класса должен корректно использоваться вместо базового.
Плохой пример:
```
class Rectangle {
public:
    virtual void set_width(int w) { width_ = w; }
    virtual void set_height(int h) { height_ = h; }

protected:
    int width_, height_;
};

class Square : public Rectangle {
public:
    void set_width(int w) override {
        width_ = height_ = w;
    }

    void set_height(int h) override {
        width_ = height_ = h;
    }
};
```
Теперь код, ожидающий `Rectangle`, может работать некорректно с `Square`.  
Это нарушение LSP. Здесь лучше отказаться от такого наследования.

Теги: #C/Cpp 
Ссылки:
[[Виртуальные функции и классы-интерфейсы]]
[[Приведение типов C++]]