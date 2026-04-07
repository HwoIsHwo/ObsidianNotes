Асинхронность в C++ можно рассмотреть на трёх уровнях: задачи на потоках (`future/async`), пул потоков с очередью задач и корутины C++20. Во всех случаях важно понимать: «асинхронность» — это неблокирующая организация работы, но реальное выполнение всё равно происходит либо в потоках, либо в рамках событийного цикла.

Начнём с `std::async`. Это самый простой способ запустить задачу «в фоне» и получить результат позже.
```C++
#include <future>
#include <iostream>
#include <thread>
#include <chrono>

int heavy_compute(int x)
{
    std::this_thread::sleep_for(std::chrono::seconds(2));
    return x * x;
}

int main()
{
    auto fut = std::async(std::launch::async, heavy_compute, 10);

    std::cout << "Работаем дальше...\n";

    int result = fut.get(); // блокируется до завершения
    std::cout << "Result = " << result << "\n";
}
```

`std::launch::async` гарантирует запуск в отдельном потоке. Без него реализация может отложить выполнение до `get()`. Это важно: `std::async` не является полноценным планировщиком задач. Если вызвать его тысячу раз, можно создать тысячу потоков.

Более контролируемый вариант — собственный пул потоков. Идея: создать фиксированное число worker’ов и очередь задач. Это базовая модель большинства серверов.

Минимальная схема:
```C++
#include <thread>
#include <vector>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>

class ThreadPool {
public:
    ThreadPool(size_t n)
    {
        for (size_t i = 0; i < n; ++i)
            workers.emplace_back([this] { worker_loop(); });
    }

    ~ThreadPool()
    {
        {
            std::unique_lock<std::mutex> lock(m);
            stop = true;
        }
        cv.notify_all();
        for (auto& t : workers)
            t.join();
    }

    void submit(std::function<void()> task)
    {
        {
            std::unique_lock<std::mutex> lock(m);
            tasks.push(std::move(task));
        }
        cv.notify_one();
    }

private:
    void worker_loop()
    {
        while (true)
        {
            std::function<void()> task;
            {
                std::unique_lock<std::mutex> lock(m);
                cv.wait(lock, [this] { return stop || !tasks.empty(); });

                if (stop && tasks.empty())
                    return;

                task = std::move(tasks.front());
                tasks.pop();
            }
            task();
        }
    }

    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex m;
    std::condition_variable cv;
    bool stop = false;
};
```
Теперь можно отправлять задачи:
```C++
ThreadPool pool(4);
pool.submit([] { /* асинхронная работа */ });
```
Это уже реальная масштабируемая модель: число потоков фиксировано, задачи выполняются по мере готовности.

Далее — асинхронность без блокирования потока через корутины (C++20). Корутина — это функция, которая может приостанавливать своё выполнение и возобновляться позже. Компилятор превращает её в конечный автомат.

Простейший пример «task»-типа:
```C++
#include <coroutine>
#include <iostream>

struct Task {
    struct promise_type {
        Task get_return_object() {
            return Task{ std::coroutine_handle<promise_type>::from_promise(*this) };
        }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;
    Task(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Task() { if (handle) handle.destroy(); }
};

Task simple()
{
    std::cout << "Start\n";
    co_return;
}
```
Это минимальный пример, который не делает реальной асинхронности, но показывает механизм. Настоящая асинхронность появляется при использовании `co_await`.

Пример awaitable-объекта:
```C++
#include <coroutine>
#include <thread>
#include <chrono>

struct SleepAwaiter {
    int ms;

    bool await_ready() const noexcept { return false; }

    void await_suspend(std::coroutine_handle<> h) const {
        std::thread([h, t = ms] {
            std::this_thread::sleep_for(std::chrono::milliseconds(t));
            h.resume();
        }).detach();
    }

    void await_resume() const noexcept {}
};

SleepAwaiter async_sleep(int ms)
{
    return SleepAwaiter{ms};
}
```

Использование:
```C++
Task example()
{
    std::cout << "Before sleep\n";
    co_await async_sleep(1000);
    std::cout << "After sleep\n";
}
```

Здесь корутина приостанавливается на `co_await`. Поток не блокируется — вместо этого создаётся вспомогательный поток, который через секунду вызывает `resume`. В реальной системе вместо создания нового потока использовался бы event loop или пул.

Ключевой момент: корутины не создают потоков автоматически. Они лишь сохраняют состояние и позволяют возобновить выполнение позже. Механизм планирования полностью определяется библиотекой или инфраструктурой.

Если сравнивать подходы:

`std::async` — простой способ параллельного запуска, но без контроля масштабируемости.  
Пул потоков — управляемая модель для CPU-bound задач.  
Корутины — инструмент для неблокирующей логики и I/O-bound систем, особенно при наличии event loop.

Инженерно важно выбирать модель под тип нагрузки. Если задача — интенсивные вычисления, нужен контролируемый пул потоков. Если задача — сеть, ожидание диска, таймеры — корутины дают более чистую структуру и масштабируются лучше, чем тысячи потоков.

Теги: #C/Cpp 
Ссылки:
[[Многопоточность и асинхронность]]
[[Прерывания]]
[[Volatile в C]]
[[Многопоточность в C и C++]]
[[Механизмы многопоточности]]