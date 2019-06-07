### Регулярные выражения

```d
import std.regex, std.stdio;

void main() {
    auto highlander = "There can be only one";
    auto rx = regex(`\b\w{3,4}\b`);
    foreach(match; matchAll(highlander, rx))
        writeln(match.hit);
}
```