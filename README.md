Конечно! Вот **один файл**, который содержит все этапы — от простого эха-сервера до оконного клиента. Просто запусти нужную часть, закомментировав остальное:

```python
# chat_app.py

import socket
from datetime import datetime
import threading
import tkinter as tk
from tkinter import messagebox

HOST = 'localhost'
PORT = 50007

# ============== 5.1.1 Сервер с эхом ==============
def echo_server():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('', PORT))
    s.listen(1)
    print("Сервер запущен (echo)... Ожидание клиента...")
    conn, addr = s.accept()
    print('Подключен клиент:', addr)
    while True:
        data = conn.recv(1024)
        if not data:
            break
        print("Получено сообщение:", data.decode('utf-8'))
        conn.sendall(data)
    conn.close()

# ============== 5.1.2 Клиент с эхом ==============
def echo_client():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    s.sendall("Hello, world".encode())
    data = s.recv(1024)
    s.close()
    print("Сообщение сервера:", data.decode('utf-8'))

# ============== 5.2.2 Сервер общего чата ==============
def chat_server():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('', PORT))
    s.listen(5)
    print("Сервер общего чата запущен...")

    def handle_client(conn, addr):
        while True:
            data = conn.recv(1024)
            if not data:
                break
            now = datetime.now().strftime("%Y.%m.%d %H:%M:%S")
            print(f"{now} ({addr}): {data.decode('utf-8')}")
        conn.close()

    while True:
        conn, addr = s.accept()
        threading.Thread(target=handle_client, args=(conn, addr)).start()

# ============== 5.2.1 Клиент общего чата ==============
def chat_client():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    while True:
        msg = input("Введите сообщение (end - выход): ")
        if msg == 'end':
            break
        s.sendall(msg.encode())
    s.close()

# ============== 5.3.2 Сервер с логинами ==============
def login_chat_server():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('', PORT))
    s.listen(5)
    print("Сервер с логинами запущен...")

    def handle_client(conn, addr):
        login = conn.recv(1024).decode('utf-8')
        print(f"Подключился {login}")
        while True:
            data = conn.recv(1024)
            if not data:
                break
            now = datetime.now().strftime("%Y.%m.%d %H:%M:%S")
            print(f"{now} {login}: {data.decode('utf-8')}")
        conn.close()

    while True:
        conn, addr = s.accept()
        threading.Thread(target=handle_client, args=(conn, addr)).start()

# ============== 5.3.1 Клиент с логином ==============
def login_chat_client():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    login = input("Введите логин: ")
    s.sendall(login.encode())
    while True:
        msg = input("Введите сообщение (end - выход): ")
        if msg == 'end':
            break
        s.sendall(msg.encode())
    s.close()

# ============== 5.4.1 Оконный клиент ==============
def gui_client():
    def send_message():
        login = login_entry.get()
        msg = message_entry.get()
        if not login or not msg:
            messagebox.showwarning("Ошибка", "Введите логин и сообщение")
            return
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.connect((HOST, PORT))
            s.sendall(login.encode())
            s.sendall(msg.encode())
            s.close()
            message_entry.delete(0, tk.END)
        except Exception as e:
            messagebox.showerror("Ошибка", str(e))

    root = tk.Tk()
    root.title("Оконный чат-клиент")

    tk.Label(root, text="Логин:").pack()
    login_entry = tk.Entry(root)
    login_entry.pack()

    tk.Label(root, text="Сообщение:").pack()
    message_entry = tk.Entry(root, width=50)
    message_entry.pack()

    tk.Button(root, text="Отправить", command=send_message).pack(pady=10)
    root.mainloop()

# ============== Запуск нужной части ==============
if __name__ == '__main__':
    # Раскомментируй нужную функцию для запуска:

    # echo_server()
    # echo_client()
    # chat_server()
    # chat_client()
    # login_chat_server()
    # login_chat_client()
    # gui_client()
    pass
```

---

### **Инструкция по запуску:**
1. Сохрани файл как `chat_app.py`.
2. Открой два окна терминала или IDLE:
   - В первом — **раскомментируй `login_chat_server()`** и запусти.
   - Во втором — **раскомментируй `login_chat_client()`** (или `gui_client()` для окна) и запусти.

Хочешь, я могу сделать автозапуск с меню выбора прямо в консоли?


Подключение модулей C и Python возможно через API Python/C и `ctypes`. Вот примеры для двух направлений:

---

## **5.1 Подключение C-модуля к Python**

### **Шаг 1: C-файл (mycmodule.c)**
```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}
```

### **Шаг 2: Скомпилировать в общую библиотеку**

**Linux/macOS:**
```bash
gcc -shared -o mycmodule.so -fPIC mycmodule.c
```

**Windows (MinGW):**
```bash
gcc -shared -o mycmodule.dll -Wl,--out-implib,libmycmodule.a -fPIC mycmodule.c
```

### **Шаг 3: Python-скрипт**
```python
import ctypes

# Загрузить библиотеку
lib = ctypes.CDLL('./mycmodule.so')  # или './mycmodule.dll' на Windows

# Указать типы аргументов и возвращаемого значения
lib.add.argtypes = [ctypes.c_int, ctypes.c_int]
lib.add.restype = ctypes.c_int

# Вызов функции
result = lib.add(3, 5)
print("Результат из C:", result)
```

---

## **5.2 Подключение Python-модуля к программе на C**

### **Шаг 1: Python-модуль (myscript.py)**
```python
def multiply(a, b):
    return a * b
```

### **Шаг 2: C-программа (callpython.c)**

```c
#include <Python.h>

int main() {
    Py_Initialize();

    PyObject *pName = PyUnicode_DecodeFSDefault("myscript");
    PyObject *pModule = PyImport_Import(pName);
    Py_DECREF(pName);

    if (pModule != NULL) {
        PyObject *pFunc = PyObject_GetAttrString(pModule, "multiply");
        if (PyCallable_Check(pFunc)) {
            PyObject *pArgs = PyTuple_Pack(2, PyLong_FromLong(6), PyLong_FromLong(7));
            PyObject *pValue = PyObject_CallObject(pFunc, pArgs);
            Py_DECREF(pArgs);

            if (pValue != NULL) {
                printf("Результат из Python: %ld\n", PyLong_AsLong(pValue));
                Py_DECREF(pValue);
            } else {
                PyErr_Print();
            }
        }
        Py_XDECREF(pFunc);
        Py_DECREF(pModule);
    } else {
        PyErr_Print();
    }

    Py_Finalize();
    return 0;
}
```

### **Шаг 3: Компиляция**
Убедитесь, что у вас установлены Python-заголовки (на Linux — `python3-dev`).

**Linux:**
```bash
gcc -o callpython callpython.c -I/usr/include/python3.X -lpython3.X
```

**Windows:**
Компиляция под Windows сложнее и требует настройки среды (Visual Studio или MinGW + python libs).

---

Хочешь, я могу подготовить архив с этими файлами или помочь с компиляцией под твою ОС?

# threads_demo.py

import threading
import time
import _thread

# ========== 5.1 Приложение с семафорами ==========
def semaphore_example():
    sem = threading.Semaphore(2)  # максимум 2 потока одновременно

    def task(i):
        print(f"Поток {i} ожидает семафор")
        with sem:
            print(f"Поток {i} зашел в критическую секцию")
            time.sleep(2)
            print(f"Поток {i} вышел из критической секции")

    for i in range(5):
        threading.Thread(target=task, args=(i,)).start()

# ========== 5.2 Обмен сообщениями через Event ==========
def event_example():
    event = threading.Event()

    def wait_for_event():
        print("Ожидание события...")
        event.wait()
        print("Событие произошло! Поток продолжает работу.")

    def trigger_event():
        time.sleep(2)
        print("Установка события")
        event.set()

    threading.Thread(target=wait_for_event).start()
    threading.Thread(target=trigger_event).start()

# ========== 5.3 Обмен сообщениями через Condition ==========
def condition_example():
    condition = threading.Condition()
    shared_data = []

    def producer():
        with condition:
            print("Производитель добавляет данные")
            shared_data.append("сообщение")
            condition.notify()

    def consumer():
        with condition:
            print("Потребитель ждет данные")
            condition.wait()
            print("Потребитель получил:", shared_data.pop())

    threading.Thread(target=consumer).start()
    time.sleep(1)
    threading.Thread(target=producer).start()

# ========== 5.4 Низкоуровневый поток через _thread ==========
def low_level_thread_example():
    def thread_func(name, delay):
        for i in range(3):
            print(f"[{name}] Итерация {i}")
            time.sleep(delay)
        print(f"[{name}] Завершен")

    try:
        _thread.start_new_thread(thread_func, ("Низкий поток 1", 1))
        _thread.start_new_thread(thread_func, ("Низкий поток 2", 1.5))
    except Exception as e:
        print("Ошибка при запуске потока:", e)

    # Пауза основного потока, чтобы дочерние успели завершиться
    time.sleep(5)

# ========== Меню запуска ==========
if __name__ == "__main__":
    print("Выбери задание:")
    print("1 - Семафоры")
    print("2 - Event")
    print("3 - Condition")
    print("4 - _thread (низкий уровень)")

    choice = input("Ввод: ")
    if choice == "1":
        semaphore_example()
    elif choice == "2":
        event_example()
    elif choice == "3":
        condition_example()
    elif choice == "4":
        low_level_thread_example()
    else:
        print("Неверный выбор")
