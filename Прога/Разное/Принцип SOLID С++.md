SOLID — это набор принципов проектирования классов и зависимостей для обеспечения устойчивости архитектуры к изменениям.


## S — Single Responsibility Principle
**У класса должна быть одна причина для изменения.**
Плохо:
```
class Report {
public:
    std::string generate() {
        return "report data";
    }

    void save_to_file(const std::string& path) {
        // запись в файл
    }
};
```
Здесь две ответственности:
- формирование данных
- сохранение
Изменение формата отчёта и изменение способа хранения — разные причины для модификации.
Лучше:
```
class Report {
public:
    std::string generate() const {
        return "report data";
    }
};

class FileSaver {
public:
    void save(const std::string& data, const std::string& path) {
        // запись
    }
};
```


## O — Open/Closed Principle
**Класс должен быть открыт для расширения, но закрыт для изменения.**
Плохо:
```
class AreaCalculator {
public:
    double area(const std::string& type) {
        if (type == "circle") return 3.14 * 10 * 10;
        if (type == "square") return 5 * 5;
        return 0;
    }
};
```
При добавлении новой фигуры нужно менять класс.
Лучше — полиморфизм:
```
class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;
};

class Circle : public Shape {
public:
    double area() const override {
        return 3.14 * r_ * r_;
    }
private:
    double r_ = 10;
};

class AreaCalculator {
public:
    double area(const Shape& shape) {
        return shape.area();
    }
};
```


## L — Liskov Substitution Principle
**Объекты производного класса должны корректно заменять базовый.**
Проблемный пример:
```
class Bird {
public:
    virtual void fly() {}
};

class Ostrich : public Bird {
public:
    void fly() override {
        throw std::logic_error("can't fly");
    }
};
```
`Ostrich` нарушает контракт `Bird`.
Корректнее разделить иерархию:
```
class Bird {};

class FlyingBird : public Bird {
public:
    virtual void fly() = 0;
};
```


## I — Interface Segregation Principle
**Лучше несколько специализированных интерфейсов, чем один общий.**
Плохо:
```
class IMachine {
public:
    virtual void print() = 0;
    virtual void scan() = 0;
};
```
Простой принтер обязан реализовывать `scan()`.
Лучше:
```
class IPrinter {
public:
    virtual void print() = 0;
};

class IScanner {
public:
    virtual void scan() = 0;
};
```


## D — Dependency Inversion Principle
**Зависеть нужно от абстракций, а не от конкретных реализаций.**
Плохо:
```
class FileLogger {
public:
    void log(const std::string& msg) {}
};

class Service {
public:
    Service() : logger_() {}

    void run() {
        logger_.log("run");
    }

private:
    FileLogger logger_;
};
```
`Service` жёстко связан с `FileLogger`.
Лучше:
```
class ILogger {
public:
    virtual ~ILogger() = default;
    virtual void log(const std::string&) = 0;
};

class Service {
public:
    Service(ILogger& logger) : logger_(logger) {}

    void run() {
        logger_.log("run");
    }

private:
    ILogger& logger_;
};
```


## Как принципы работают вместе

На практике они пересекаются:
- SRP снижает связанность
- OCP достигается через полиморфизм
- LSP гарантирует корректную расширяемость
- ISP уменьшает избыточные зависимости
- DIP разрывает жёсткие связи

Теги: #Прога 
Ссылки:
[[ООП в C++]]
[[Виртуальные функции и классы-интерфейсы]]