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
