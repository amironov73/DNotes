### Циклы

#### for

```d
import std.stdio;

void main() {
    int[] a = [1, 2, 3];
    for (int i = 0; i < a.length; ++i) {
        writeln(a[i]);
    }
}
```

Выход из цикла:

```d
import std.stdio;

void main() {
    outer: for (int i = 0; i < 10; ++i) {
        for (int j = 0; j < 20; ++j) {
            if (i + j > 5) {
                break outer;
            }
        }
    }
}
```

Также имеется `continue`.

#### while

```d
import std.stdio;

void main() {
    int i = 1;
    while (i < 1_000_000) {
        writeln(i);
        i *= 2;
    }
}
```

#### do ... while

```d
import std.stdio;

void main() {
    int i = 1;
    do {
        writeln(i);
        i *= 2;
    } while (i < 1_000_000);
}
```

#### foreach

```d
import std.stdio;

void main() {
    int[] arr = [1, 2, 3];
    foreach (int e; arr) {
    // можно написать: foreach(e; arr)
    // в этом случае тип выводится автоматически
        writeln(e);
    }
}
```

Доступ по ссылке:

```d
foreach (ref e, arr) {
    ...
}
```

По факту `foreach (element; range)` разворачивается в

```d
for (; !range.empty; range.popFront()) {
    auto element = range.front;
}
```

Здесь `range` описывается такой стркутурой:

```d
struct Range {
    @property empty() const;
    void popFront();
    T front();
}
```

Имеется также `foreach_reverse`:

```d
foreach_reverse(e; arr) {
    ...
}
```