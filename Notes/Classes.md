### Классы

Классы - ссылочные типы!

```d
import std.stdio;

class Person {
	string name;
	int age;

	this(string name, int age) {
		this.name = name;
		this.age = age;
	}

	void say(string whatToSay) {
		writeln(name, " says: ", whatToSay);
	}
}

void main()
{
	auto person = new Person("Alexey", 45);
	person.say("Hello");
}
```