###  Модуль object

Содержит определения:

* типов: `size_t`, `ptrdiff_t`, `sizediff_t`, `hash_t`, `equals_t`;
* типов: `string`, `wstring`, `dstring`;
* функций: `int __cmp(T)(scope const T[] lhs, scope const T[] rhs)`, `int __cmp(T1, T2)(T1[] s1, T2[] s2)`, `bool __equals(T1, T2)(T1[] lhs, T2[] rhs)`;
* функции `destroy`: ,
* операторов: `bool opEquals(Object lhs, Object rhs)`, `bool opEquals(const Object lhs, const Object rhs)`, 
* типа `Object` -- предка всех классов;
* структуры `Interface` -- информации об интерфейсе;
* классы `Throwable`, `Exception` и `Error` -- предки всех исключений;
* функции `dup` и `idup` позволяющие клонировать слайсы.

Этот модуль подключается неявно автоматически.

#### Функция destroy

Уничтожает указанный объект или структуру, опционально возвращает их в начальное состояние.

```d
void destroy(bool initialize = true, T)(ref T obj);
void destroy(bool initialize = true, T)(T obj);
void destroy(bool initialize = true, T : U[n], U, size_t n)(ref T obj);
void destroy(bool initialize = true, T)(ref T obj);
```

#### Object

```d
class Object
{
    /// Преобразование в строку 
    string toString() {
        return typeid(this).name;
    }
    
    /// Вычисление хеша
    size_t toHash() @trusted nothrow {
        size_t addr = cast(size_t) cast(void*) this;
        return addr ^ (addr >>> 4);
    }
    
    /// Сравнение с другим объектом
    int opCmp(Object o) {
        throw new Exception("need opCmp for class " ~ typeid(this).name);
    }
    
    /// Равенство с другим объектом
    bool opEquals(Object o) {
        return this is o;
    }
    
    // Создание объекта по имени типа
    static Object factory(string classname) {
        auto ci = TypeInfo_Class.find(classname);
        if (ci) {
            return ci.create();
        }
        return null;
    }
}
```

#### Interface

```d
struct Interface
{
    TypeInfo_Class   classinfo;  /// .classinfo for this interface (not for containing class)
    void*[]     vtbl;
    size_t      offset;     /// offset to Interface 'this' from Object 'this'
}
```