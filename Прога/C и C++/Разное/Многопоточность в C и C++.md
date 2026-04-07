В C исторически стандарт языка не содержал потоков. На практике используется POSIX Threads — библиотека `pthread` в Unix-подобных системах. Поток создаётся вызовом `pthread_create`, которому передаётся функция-старт и указатель на аргумент. ОС выделяет стек, формирует контекст и ставит поток в очередь планировщика. Синхронизация осуществляется через `pthread_mutex_t`, `pthread_cond_t`, `pthread_rwlock_t`, семафоры и барьеры. Все примитивы — тонкие обёртки над механизмами ядра (в Linux обычно через futex). Управление жизненным циклом — через `pthread_join` или `pthread_detach`.
```C
#include <pthread.h>
#include <stdio.h>

void* thread_func(void* arg)
{
    printf("Hello from thread\n");
    return NULL;
}

int main()
{
    pthread_t t;
    pthread_create(&t, NULL, thread_func, NULL);
    pthread_join(t, NULL);
    return 0;
}
```

В C++ с версии C++11 в стандартной библиотеке появилась полноценная модель многопоточности. Ключевой класс — `std::thread`. Он инкапсулирует создание и управление потоком. В отличие от C, аргументы и возвращаемые значения типобезопасны, поддерживаются лямбда-выражения и объекты.
```C++
#include <thread>
#include <iostream>

int main()
{
    std::thread t([]{
        std::cout << "Hello from thread\n";
    });

    t.join();
}
```
Важный момент: если объект `std::thread` уничтожается без вызова `join` или `detach`, программа завершится аварийно. Это сознательное решение стандарта, чтобы избежать «висячих» потоков.

Для синхронизации в C++ используются `std::mutex`, `std::recursive_mutex`, `std::timed_mutex`, `std::condition_variable`, `std::shared_mutex`. Обычно мьютекс оборачивается в RAII-объект `std::lock_guard` или `std::unique_lock`, чтобы исключить утечки блокировки при исключениях.
```C++
#include <mutex>

std::mutex m;

void f()
{
    std::lock_guard<std::mutex> lock(m);
    // критическая секция
}
```

Теги: #C/Cpp 
Ссылки:
[[Механизмы многопоточности]]
[[Лямбда-функции C++]]
[[Многопоточность и асинхронность]]
[[Мьютекс mutex C++]]
[[Мьютекс shared_mutex C++]]