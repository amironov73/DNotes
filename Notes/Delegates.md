### Шаблоны

#### Функция как аргумент

```d
void doSomething(int function(int, int) doer) 
{
    doer(5,5);
}

int add(int left, int right) 
{
    return left + right;
}

void main() 
{
    doSomething(&add);
}
```

#### Локальная функция с контекстом

```d
import std.stdio;

void foo() 
{
    void local() 
    {
        writeln("local");
    }
    auto f = &local; // Тип f - delegate()
}
```

#### Анонимная функция

```d
void doSomething(int function(int, int) doer) 
{
    doer(5,5);
}

void main()
{
   auto f = (int left, int right)
   {
        return left + right;
   };
   
   doSomething(f);
}
```

#### Лямбда (однострочник)

```d
import std.stdio;

void doSomething(int function (int, int) doer) 
{
    doer(5,5);
}

void main() 
{
    doSomething((int left, int right) => left + right);
}
```

Удобно использовать с алгоритмами:

```d
import std.stdio;
import std.algorithm;

void main() 
{
	auto r1 = [1, 2, 3].reduce!`a + b`;
	writeln(r1);
}
```
