### Шаблоны

```d
import std.stdio;

auto add(T)(T lhs, T rhs) {
    return lhs + rhs;
}

void main()
{
    writeln(add!int(2, 3));
    writeln(add!float(2.0f, 3.0f));
    writeln(add(2, 3)); // тип выведется автоматически
}
```

Типы `struct`, `class` и `interface` также могут быть определены как шаблонные:

```d
import std.stdio;

struct S(T)
{
	T value;
	string name;

	this(string name, T value)
	{
		this.name = name;
		this.value = value;
	}

	void describe()
	{
		writeln(name, " [", typeid(T), "]", ": ", value);
	}
}

void main()
{
	auto s1 = S!int("length", 5);
	s1.describe;
	auto s2 = S!float("weight", 2.5);
	s2.describe;
}
```