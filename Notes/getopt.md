### Модуль std.getopt

Модуль std.getopt реализует функцию знаменитую getopt, которая разбирает параметры командной строки согласно стандарта POSIX. Кроме того, поддерживаются расширения GNU (длинные опции с двойным минусом в начале). Доступно связывание параметров командной строки с переменными в программе.

Синтаксис функции getopt:

```d
GetoptResult getopt(T...)(ref string[] args, T opts); 
```
Пример использования:

```d
import std.getopt;

string data = "file.dat";
int length = 24;
bool verbose;
enum Color { no, yes };
Color color;

void main(string[] args) {
  auto result = getopt(
    args,
    "length",  &length,    // numeric
    "file",    &data,      // string
    "verbose", &verbose,   // flag
    "color", "Information about this color", &color);    // enum

  if (result.helpWanted)
  {
    defaultGetoptPrinter("Some information about the program.",
      helpInformation.options);
  }
}
```
