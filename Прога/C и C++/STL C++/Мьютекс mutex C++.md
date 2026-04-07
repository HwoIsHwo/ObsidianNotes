`std::mutex` — примитив синхронизации из `<mutex>`, обеспечивающий **взаимное исключение**: в критическую секцию одновременно может войти только один поток.

## Базовая идея
Есть общий ресурс (память, контейнер, файл). Без синхронизации — data race и UB.  
`mutex` сериализует доступ.
```
#include <mutex>
#include <thread>
#include <vector>

std::mutex m;
int counter = 0;

void inc() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(m);
        ++counter;
    }
}
```
`lock_guard` — RAII-обёртка: захват в конструкторе, освобождение в деструкторе. Это базовый и правильный способ работы с mutex.

## RAII-обёртки
### 1. `std::lock_guard`
Простой и безопасный вариант.
```
{
    std::lock_guard<std::mutex> lock(m);
    // критическая секция
}
// автоматический unlock
```
Нельзя вручную разблокировать раньше конца области.

### 2. `std::unique_lock`
Более гибкий:
- можно откладывать захват
- можно вручную unlock/lock
- можно передавать владение
```
std::unique_lock<std::mutex> lock(m, std::defer_lock);

lock.lock();
// критическая секция
lock.unlock();
```
Используется с `std::condition_variable`.

### 3. `std::scoped_lock` (C++17)
Для захвата нескольких mutex без дедлока:
`std::scoped_lock lock(m1, m2);`
Использует алгоритм `std::lock`.


Condition_variable
`std::condition_variable` — примитив синхронизации для ожидания события между потоками. Используется совместно с `std::mutex`. Типичный сценарий — один поток ждёт изменения состояния, другой это состояние меняет и уведомляет.

Ключевая идея: поток **не крутится в цикле**, а засыпает до уведомления. При этом `condition_variable` не хранит состояние — оно хранится в ваших данных. Поэтому всегда используется предикат.

### Базовая схема работы
Есть общий ресурс:
```
std::mutex mtx;
std::condition_variable cv;
bool ready = false;
```
Поток-потребитель:
```
void worker() {
    std::unique_lock<std::mutex> lock(mtx);

    cv.wait(lock, [] { return ready; });  
    // поток спит, mutex освобожден
    // при пробуждении mutex снова захвачен

    std::cout << "Work started\n";
}
```
Поток-производитель:

```
void signal() {
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    } // mutex освобождён

    cv.notify_one(); // пробуждаем один поток
}
```


Теги: #C/Cpp 
Ссылки:
