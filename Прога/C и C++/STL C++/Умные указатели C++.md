В стандартной библиотеке три основных умных указателя: `std::unique_ptr`, `std::shared_ptr` и `std::weak_ptr`. Все они решают одну задачу — автоматическое управление временем жизни объекта — но делают это принципиально разными способами.


## std::unique_ptr
Эксклюзивное владение. В каждый момент времени объектом владеет ровно один `unique_ptr`. Копирование запрещено, разрешено только перемещение.
```
std::unique_ptr<int> p = std::make_unique<int>(10);

// std::unique_ptr<int> q = p;   // ошибка
std::unique_ptr<int> q = std::move(p); // владение передано
```
Внутри — просто сырой указатель. Никаких счётчиков ссылок. Размер — обычно один указатель (или два, если есть кастомный deleter).
Удаление происходит в деструкторе:
```
~unique_ptr() { delete ptr; }
```
Основные методы:
```
p.get();        // сырой указатель
p.release();    // отдать указатель и перестать владеть
p.reset();      // удалить текущий объект
p.reset(new T); // заменить объект
p.swap(other);


auto p = std::make_unique<int>(5);

int* raw = p.release(); // p теперь nullptr
delete raw;
```
- Когда объект имеет единственного владельца.
- Для членов класса (вместо raw pointer).
- Для владения ресурсами (RAII).
- Для фабрик:


## std::shared_ptr
Разделяемое владение. Объект уничтожается, когда последний `shared_ptr` перестаёт на него ссылаться.
```
auto p1 = std::make_shared<int>(10);
auto p2 = p1;  // счётчик = 2
```
Когда `p1` и `p2` уничтожаются — объект удаляется.

Используется **контрольный блок**, который хранит:
- указатель на объект,
- счётчик strong-ссылок,
- счётчик weak-ссылок,
- deleter.
Основные методы:
```
p.use_count();   // количество shared_ptr
p.unique();      // use_count() == 1
p.reset();       // уменьшить счётчик
p.get();         // сырой указатель


auto p1 = std::make_shared<int>(42);
{
    auto p2 = p1;
    std::cout << p1.use_count(); // 2
}
std::cout << p1.use_count(); // 1
```
- Когда объект логически принадлежит нескольким владельцам.
- В графах объектов.
- В системах событий.
- В плагинных архитектурах.


## std::weak_ptr
Невладеющая ссылка на объект, управляемый `shared_ptr`. Не увеличивает strong-счётчик.
```
std::weak_ptr<int> w = p;

if (auto s = w.lock()) {
    // объект ещё жив
}
```
`lock()` возвращает `shared_ptr`, если объект существует.

`weak_ptr` увеличивает только weak-счётчик в control block. Объект удаляется, когда strong-счётчик == 0. Control block удаляется, когда strong == 0 и weak == 0.
- Разрыв циклов.
- Кэш.
- Наблюдатели.
- Обратные ссылки (parent pointer в дереве).
```
struct Node {
    std::shared_ptr<Node> child;
    std::weak_ptr<Node> parent;
};
```


## Блок контроля
Типичная реализация (концептуально) включает:
- указатель на объект (`T*`)
- strong counter (количество `shared_ptr`)
- weak counter (количество `weak_ptr`)
- deleter
- allocator (опционально)
- иногда type-erasure для кастомного удаления

Примерно так:
```
struct control_block {
    std::atomic<long> strong_count;
    std::atomic<long> weak_count;
    void (*deleter)(void*);
    void* ptr;
};
```

Strong-ссылка — это владеющая ссылка.
Каждый `shared_ptr`:
- увеличивает `strong_count` при копировании,
- уменьшает при разрушении.

Weak-ссылка не владеет объектом. Когда говорят, что «ссылка владеет объектом», имеют в виду **ответственность за его уничтожение**.


Примеры работы:
```
#include <iostream>
#include <memory>
#include <string>

struct Object {
    std::string name;

    Object(std::string n) : name(std::move(n)) {
        std::cout << "ctor " << name << "\n";
    }

    ~Object() {
        std::cout << "dtor " << name << "\n";
    }

    void hello() const {
        std::cout << "Hello, " << name << "\n";
    }
};



void unique_example() {
    std::unique_ptr<Object> p = std::make_unique<Object>("A");

    // чтение поля
    std::cout << p->name << "\n";

    // запись
    p->name = "A1";

    // вызов метода
    p->hello();

    // передача владения
    std::unique_ptr<Object> p2 = std::move(p);

    if (!p) {
        std::cout << "p is empty\n";
    }

    p2->hello();
} // объект уничтожится здесь



void shared_example() {
    std::shared_ptr<Object> p1 = std::make_shared<Object>("B");

    std::cout << "use_count = " << p1.use_count() << "\n";

    {
        std::shared_ptr<Object> p2 = p1; // +1 владелец

        std::cout << "use_count = " << p1.use_count() << "\n";

        p2->name = "B1";
        p2->hello();
    }

    std::cout << "use_count = " << p1.use_count() << "\n";
} // уничтожится когда последний shared_ptr уйдёт



void weak_example() {
    std::shared_ptr<Object> p = std::make_shared<Object>("C");
    std::weak_ptr<Object> w = p;

    std::cout << "use_count = " << w.use_count() << "\n";

    if (auto locked = w.lock()) { // проверка на жизнь объекта
        locked->hello();
    }

    p.reset(); // уничтожаем объект

    if (auto locked = w.lock()) {
        locked->hello();
    } else {
        std::cout << "object expired\n";
    }
}



void print_object(const Object* obj) {
    if (obj) {
        obj->hello();
    }
}

void raw_example() {
    auto p = std::make_unique<Object>("D");

    print_object(p.get()); // передаём невладеющий указатель
}
```


Теги: #C/Cpp 
Ссылки:
[[Способы вытеснения данных из кэша]]
[[Двусвязный список list C++]]