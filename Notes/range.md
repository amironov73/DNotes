### Range

Здесь "range" переводится как "диапазон".

#### Модуль std.range

**chain**

Объединяет несколько диапазонов в один.

```d
import std.algorithm.comparison : equal;

int[] arr1 = [ 1, 2, 3, 4 ];
int[] arr2 = [ 5, 6 ];
int[] arr3 = [ 7 ];
auto s = chain(arr1, arr2, arr3);
writeln(s.length); // 7
writeln(s[5]); // 6
assert(equal(s, [1, 2, 3, 4, 5, 6, 7][]));
```

```d
import std.algorithm.comparison : equal;
import std.algorithm.sorting : sort;

int[] arr1 = [5, 2, 8];
int[] arr2 = [3, 7, 9];
int[] arr3 = [1, 4, 6];

// in-place sorting across all of the arrays
auto s = arr1.chain(arr2, arr3).sort;

assert(s.equal([1, 2, 3, 4, 5, 6, 7, 8, 9]));
assert(arr1.equal([1, 2, 3]));
assert(arr2.equal([4, 5, 6]));
assert(arr3.equal([7, 8, 9]));
```

**choose**

Выбирает один из двух диапазонов на основе логического условия.

```d
import std.algorithm.comparison : equal;
import std.algorithm.iteration : filter, map;

auto data1 = only(1, 2, 3, 4).filter!(a => a != 3);
auto data2 = only(5, 6, 7, 8).map!(a => a + 1);

// choose() is primarily useful when you need to select one of two ranges
// with different types at runtime.
static assert(!is(typeof(data1) == typeof(data2)));

auto chooseRange(bool pickFirst)
{
    // The returned range is a common wrapper type that can be used for
    // returning or storing either range without running into a type error.
    return choose(pickFirst, data1, data2);

    // Simply returning the chosen range without using choose() does not
    // work, because map() and filter() return different types.
    //return pickFirst ? data1 : data2; // does not compile
}

auto result = chooseRange(true);
assert(result.equal(only(1, 2, 4)));

result = chooseRange(false);
assert(result.equal(only(6, 7, 8, 9)));
```

**chooseAmong**

Выбирает один из нескольких диапазонов на основе индекса.

```d
auto test()
{
    import std.algorithm.comparison : equal;

    int[4] sarr1 = [1, 2, 3, 4];
    int[2] sarr2 = [5, 6];
    int[1] sarr3 = [7];
    auto arr1 = sarr1[];
    auto arr2 = sarr2[];
    auto arr3 = sarr3[];

    {
        auto s = chooseAmong(0, arr1, arr2, arr3);
        auto t = s.save;
        writeln(s.length); // 4
        writeln(s[2]); // 3
        s.popFront();
        assert(equal(t, only(1, 2, 3, 4)));
    }
    {
        auto s = chooseAmong(1, arr1, arr2, arr3);
        writeln(s.length); // 2
        s.front = 8;
        assert(equal(s, only(8, 6)));
    }
    {
        auto s = chooseAmong(1, arr1, arr2, arr3);
        writeln(s.length); // 2
        s[1] = 9;
        assert(equal(s, only(8, 9)));
    }
    {
        auto s = chooseAmong(1, arr2, arr1, arr3)[1 .. 3];
        writeln(s.length); // 2
        assert(equal(s, only(2, 3)));
    }
    {
        auto s = chooseAmong(0, arr1, arr2, arr3);
        writeln(s.length); // 4
        writeln(s.back); // 4
        s.popBack();
        s.back = 5;
        assert(equal(s, only(1, 2, 5)));
        s.back = 3;
        assert(equal(s, only(1, 2, 3)));
    }
    {
        uint[5] foo = [1, 2, 3, 4, 5];
        uint[5] bar = [6, 7, 8, 9, 10];
        auto c = chooseAmong(1, foo[], bar[]);
        writeln(c[3]); // 9
        c[3] = 42;
        writeln(c[3]); // 42
        writeln(c.moveFront()); // 6
        writeln(c.moveBack()); // 10
        writeln(c.moveAt(4)); // 10
    }
    {
        import std.range : cycle;
        auto s = chooseAmong(0, cycle(arr2), cycle(arr3));
        assert(isInfinite!(typeof(s)));
        assert(!s.empty);
        writeln(s[100]); // 8
        writeln(s[101]); // 9
        assert(s[0 .. 3].equal(only(8, 9, 8)));
    }
    return 0;
}
// works at runtime
auto a = test();
// and at compile time
static b = test();
```

**chunks**

Создает диапазон, который возвращает фрагменты фиксированного размера исходного диапазона.

```d
import std.algorithm.comparison : equal;
auto source = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
auto chunks = chunks(source, 4);
writeln(chunks[0]); // [1, 2, 3, 4]
writeln(chunks[1]); // [5, 6, 7, 8]
writeln(chunks[2]); // [9, 10]
writeln(chunks.back); // chunks[2]
writeln(chunks.front); // chunks[0]
writeln(chunks.length); // 3
assert(equal(retro(array(chunks)), array(retro(chunks))));
```

```d
import std.algorithm.comparison : equal;

int i;

// The generator doesn't save state, so it cannot be a forward range.
auto inputRange = generate!(() => ++i).take(10);

// We can still process it in chunks, but it will be single-pass only.
auto chunked = inputRange.chunks(2);

assert(chunked.front.equal([1, 2]));
assert(chunked.front.empty); // Iterating the chunk has consumed it
chunked.popFront;
assert(chunked.front.equal([3, 4]));
```

**cycle**

Создает бесконечный диапазон, который бесконечно повторяет заданный диапазон. Хорошо для реализации кольцевых буферов.

```d
import std.algorithm.comparison : equal;
import std.range : cycle, take;

// Here we create an infinitive cyclic sequence from [1, 2]
// (i.e. get here [1, 2, 1, 2, 1, 2 and so on]) then
// take 5 elements of this sequence (so we have [1, 2, 1, 2, 1])
// and compare them with the expected values for equality.
assert(cycle([1, 2]).take(5).equal([ 1, 2, 1, 2, 1 ]));
```

**drop**

Создает диапазон, который получается в результате отбрасывания первых n элементов из данного диапазона.

```d
import std.algorithm.comparison : equal;

writeln([0, 2, 1, 5, 0, 3].drop(3)); // [5, 0, 3]
writeln("hello world".drop(6)); // "world"
assert("hello world".drop(50).empty);
assert("hello world".take(6).drop(3).equal("lo "));
```

**dropBack**

Создает диапазон, который получается в результате отбрасывания последних n элементов из данного диапазона.

**dropExactly**

Creates the range that results from discarding exactly n of the first elements from the given range.

```d
import std.algorithm.comparison : equal;
import std.algorithm.iteration : filterBidirectional;

auto a = [1, 2, 3];
writeln(a.dropExactly(2)); // [3]
writeln(a.dropBackExactly(2)); // [1]

string s = "日本語";
writeln(s.dropExactly(2)); // "語"
writeln(s.dropBackExactly(2)); // "日"

auto bd = filterBidirectional!"true"([1, 2, 3]);
assert(bd.dropExactly(2).equal([3]));
assert(bd.dropBackExactly(2).equal([1]));
```

**dropBackExactly**

Создает диапазон, который получается в результате отбрасывания ровно n последних элементов из данного диапазона.

**dropOne**

Создает диапазон, который получается в результате отбрасывания первого элемента из заданного диапазона.

```d
import std.algorithm.comparison : equal;
import std.algorithm.iteration : filterBidirectional;
import std.container.dlist : DList;

auto dl = DList!int(9, 1, 2, 3, 9);
assert(dl[].dropOne().dropBackOne().equal([1, 2, 3]));

auto a = [1, 2, 3];
writeln(a.dropOne()); // [2, 3]
writeln(a.dropBackOne()); // [1, 2]

string s = "日本語";
import std.exception : assumeWontThrow;
assert(assumeWontThrow(s.dropOne() == "本語"));
assert(assumeWontThrow(s.dropBackOne() == "日本"));

auto bd = filterBidirectional!"true"([1, 2, 3]);
assert(bd.dropOne().equal([2, 3]));
assert(bd.dropBackOne().equal([1, 2]));
```

**dropBackOne**

Создает диапазон, который получается при отбрасывании последнего элемента из заданного диапазона.

**enumerate**

Итерирует диапазон с прикрепленной индексной переменной.

```d
import std.stdio : stdin, stdout;
import std.range : enumerate;

foreach (lineNum, line; stdin.byLine().enumerate(1))
    stdout.writefln("line #%s: %s", lineNum, line);
```

**evenChunks**

Создает диапазон, который возвращает фрагменты приблизительно равной длины из исходного диапазона.

```d
import std.algorithm.comparison : equal;
auto source = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
auto chunks = evenChunks(source, 3);
writeln(chunks[0]); // [1, 2, 3, 4]
writeln(chunks[1]); // [5, 6, 7]
writeln(chunks[2]); // [8, 9, 10]
```

**frontTransversal**

Создает диапазон, который перебирает первые элементы заданных диапазонов.

**generate**

Создает диапазон путем последовательных вызовов данной функции. Это позволяет создавать диапазоны как один делегат.

```d
import std.algorithm.comparison : equal;
import std.algorithm.iteration : map;

int i = 1;
auto powersOfTwo = generate!(() => i *= 2)().take(10);
assert(equal(powersOfTwo, iota(1, 11).map!"2^^a"()));
```

```d
import std.algorithm.comparison : equal;

//Returns a run-time delegate
auto infiniteIota(T)(T low, T high)
{
    T i = high;
    return (){if (i == high) i = low; return i++;};
}
//adapted as a range.
assert(equal(generate(infiniteIota(1, 4)).take(10), [1, 2, 3, 1, 2, 3, 1, 2, 3, 1]));
```

```d
import std.format : format;
import std.random : uniform;

auto r = generate!(() => uniform(0, 6)).take(10);
format("%(%s %)", r);
```

**indexed**

Создает диапазон-представление данного диапазона, как если бы его элементы были переупорядочены в соответствии с заданным диапазоном индексов.

```d
import std.algorithm.comparison : equal;
auto source = [1, 2, 3, 4, 5];
auto indices = [4, 3, 1, 2, 0, 4];
auto ind = indexed(source, indices);
assert(equal(ind, [5, 4, 2, 3, 1, 5]));
assert(equal(retro(ind), [5, 1, 3, 2, 4, 5]));
```

**iota**

Создает диапазон, состоящий из чисел между начальной и конечной точками, разнесенных на заданный интервал.

```d
void main()
{
    import std.stdio;

    // The following groups all produce the same output of:
    // 0 1 2 3 4

    foreach (i; 0 .. 5)
        writef("%s ", i);
    writeln();

    import std.range : iota;
    foreach (i; iota(0, 5))
        writef("%s ", i);
    writeln();

    writefln("%(%s %|%)", iota(0, 5));

    import std.algorithm.iteration : map;
    import std.algorithm.mutation : copy;
    import std.format;
    iota(0, 5).map!(i => format("%s ", i)).copy(stdout.lockingTextWriter());
    writeln();
}
```

```d
import std.algorithm.comparison : equal;
import std.math : approxEqual;

auto r = iota(0, 10, 1);
assert(equal(r, [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]));
assert(equal(r, [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]));
assert(3 in r);
assert(r.contains(3)); //Same as above
assert(!(10 in r));
assert(!(-8 in r));
r = iota(0, 11, 3);
assert(equal(r, [0, 3, 6, 9]));
writeln(r[2]); // 6
assert(!(2 in r));
auto rf = iota(0.0, 0.5, 0.1);
assert(approxEqual(rf, [0.0, 0.1, 0.2, 0.3, 0.4]));
```

**lockstep**

Аналогично zip, однако lockstep разработан специально для циклов foreach.

```d
auto arr1 = [1,2,3,4,5,100];
auto arr2 = [6,7,8,9,10];

foreach (ref a, b; lockstep(arr1, arr2))
{
    a += b;
}

writeln(arr1); // [7, 9, 11, 13, 15, 100]

/// Lockstep also supports iterating with an index variable:
foreach (index, a, b; lockstep(arr1, arr2))
{
    writeln(arr1[index]); // a
    writeln(arr2[index]); // b
}
```

**nullSink**

Выходной диапазон, отбрасывающий все переданные ему данные.

```d
import std.algorithm.iteration : map;
import std.algorithm.mutation : copy;
[4, 5, 6].map!(x => x * 2).copy(nullSink); // data is discarded
```

```d
import std.csv : csvNextToken;

string line = "a,b,c";

// ignore the first column
line.csvNextToken(nullSink, ',', '"');
line.popFront;

// look at the second column
Appender!string app;
line.csvNextToken(app, ',', '"');
writeln(app.data); // "b"
```

**only**

Создает диапазон, итерирующий по заданным элементам.

```d
import std.algorithm.comparison : equal;
import std.algorithm.iteration : filter, joiner, map;
import std.algorithm.searching : findSplitBefore;
import std.uni : isUpper;

assert(equal(only('♡'), "♡"));
writeln([1, 2, 3, 4].findSplitBefore(only(3))[0]); // [1, 2]

assert(only("one", "two", "three").joiner(" ").equal("one two three"));

string title = "The D Programming Language";
assert(title
    .filter!isUpper // take the upper case letters
    .map!only       // make each letter its own range
    .joiner(".")    // join the ranges together lazily
    .equal("T.D.P.L"));
```

**padLeft**

Ленивое дополнение диапазона до указанной длины с начала.

```d
import std.algorithm.comparison : equal;

assert([1, 2, 3, 4].padLeft(0, 6).equal([0, 0, 1, 2, 3, 4]));
assert([1, 2, 3, 4].padLeft(0, 3).equal([1, 2, 3, 4]));

assert("abc".padLeft('_', 6).equal("___abc"));
```

**padRight**

Ленивое дополнение диапазона до указанной длины в конце.

```d
import std.algorithm.comparison : equal;

assert([1, 2, 3, 4].padRight(0, 6).equal([1, 2, 3, 4, 0, 0]));
assert([1, 2, 3, 4].padRight(0, 4).equal([1, 2, 3, 4]));

assert("abc".padRight('_', 6).equal("abc___"));
```

**radial**

Для заданного диапазона произвольного доступа и начальной точки, создает диапазон, который попеременно возвращает следующий левый и следующий правый элемент от начальной точки.

```d
import std.algorithm.comparison : equal;
int[] a = [ 1, 2, 3, 4, 5 ];
assert(equal(radial(a), [ 3, 4, 2, 5, 1 ]));
a = [ 1, 2, 3, 4 ];
assert(equal(radial(a), [ 2, 3, 1, 4 ]));

// If the left end is reached first, the remaining elements on the right
// are concatenated in order:
a = [ 0, 1, 2, 3, 4, 5 ];
assert(equal(radial(a, 1), [ 1, 2, 0, 3, 4, 5 ]));

// If the right end is reached first, the remaining elements on the left
// are concatenated in reverse order:
assert(equal(radial(a, 4), [ 4, 5, 3, 2, 1, 0 ]));
```

**recurrence**

Создает прямой диапазон, значения которого определяются математическим отношением рекурсии.

```d
import std.algorithm.comparison : equal;

// The Fibonacci numbers, using function in string form:
// a[0] = 1, a[1] = 1, and compute a[n+1] = a[n-1] + a[n]
auto fib = recurrence!("a[n-1] + a[n-2]")(1, 1);
assert(fib.take(10).equal([1, 1, 2, 3, 5, 8, 13, 21, 34, 55]));

// The factorials, using function in lambda form:
auto fac = recurrence!((a,n) => a[n-1] * n)(1);
assert(take(fac, 10).equal([
    1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880
]));

// The triangular numbers, using function in explicit form:
static size_t genTriangular(R)(R state, size_t n)
{
    return state[n-1] + n;
}
auto tri = recurrence!genTriangular(0);
assert(take(tri, 10).equal([0, 1, 3, 6, 10, 15, 21, 28, 36, 45]));
```

**refRange**

Передает диапазон по ссылке. И исходный диапазон, и RefRange всегда будут иметь одинаковые элементы. Любая операция, выполненная с одним, повлияет на другой.

```d
import std.algorithm.searching : find;
ubyte[] buffer = [1, 9, 45, 12, 22];
auto found1 = find(buffer, 45);
writeln(found1); // [45, 12, 22]
writeln(buffer); // [1, 9, 45, 12, 22]

auto wrapped1 = refRange(&buffer);
auto found2 = find(wrapped1, 45);
writeln(*found2.ptr); // [45, 12, 22]
writeln(buffer); // [45, 12, 22]

auto found3 = find(wrapped1.save, 22);
writeln(*found3.ptr); // [22]
writeln(buffer); // [45, 12, 22]

string str = "hello world";
auto wrappedStr = refRange(&str);
writeln(str.front); // 'h'
str.popFrontN(5);
writeln(str); // " world"
writeln(wrappedStr.front); // ' '
writeln(*wrappedStr.ptr); // " world"
```

**repeat**

Создает диапазон, состоящий из одного элемента, повторяемого n раз, либо создает бесконечный диапазон, повторяющего этот элемент бесконечно.

```d
Repeat!T repeat(T)(T value);

Take!(Repeat!T) repeat(T)(T value, size_t n);
```

```d
import std.algorithm.comparison : equal;

assert(5.repeat().take(4).equal([5, 5, 5, 5]));
```

```d
import std.algorithm.comparison : equal;

assert(5.repeat(4).equal([5, 5, 5, 5]));
```

**retro**

Итерирует двунаправленный диапазон в обратном направлении.

```d
import std.algorithm.comparison : equal;
int[5] a = [ 1, 2, 3, 4, 5 ];
int[5] b = [ 5, 4, 3, 2, 1 ];
assert(equal(retro(a[]), b[]));
assert(retro(a[]).source is a[]);
assert(retro(retro(a[])) is a[]);
```

**roundRobin**

Для заданных n диапазонов создается новый диапазон, который по очереди возвращает n первых элементов каждого диапазона, затем второй элемент каждого диапазона и т. д. в циклическом порядке.

```d
import std.algorithm.comparison : equal;

int[] a = [ 1, 2, 3 ];
int[] b = [ 10, 20, 30, 40 ];
auto r = roundRobin(a, b);
assert(equal(r, [ 1, 10, 2, 20, 3, 30, 40 ]));
```

**sequence**

Аналогично recurrence, за исключением того, что создается диапазон с произвольным доступом.

```d
auto odds = sequence!("a[0] + n * a[1]")(1, 2);
writeln(odds.front); // 1
odds.popFront();
writeln(odds.front); // 3
odds.popFront();
writeln(odds.front); // 5
```

```d
auto tri = sequence!((a,n) => n*(n+1)/2)();

// Note random access
writeln(tri[0]); // 0
writeln(tri[3]); // 6
writeln(tri[1]); // 1
writeln(tri[4]); // 10
writeln(tri[2]); // 3
```

```d
import std.math : pow, round, sqrt;
static ulong computeFib(S)(S state, size_t n)
{
    // Binet's formula
    return cast(ulong)(round((pow(state[0], n+1) - pow(state[1], n+1)) /
                             state[2]));
}
auto fib = sequence!computeFib(
    (1.0 + sqrt(5.0)) / 2.0,    // Golden Ratio
    (1.0 - sqrt(5.0)) / 2.0,    // Conjugate of Golden Ratio
    sqrt(5.0));

// Note random access with [] operator
writeln(fib[1]); // 1
writeln(fib[4]); // 5
writeln(fib[3]); // 3
writeln(fib[2]); // 2
writeln(fib[9]); // 55
```

**slide**

Создает диапазон, который возвращает скользящее окно фиксированного размера поверх исходного диапазона. В отличие от chunks, он продвигает настраиваемое количество элементов за раз, а не один кусок за раз.

```d
import std.algorithm.comparison : equal;

assert([0, 1, 2, 3].slide(2).equal!equal(
    [[0, 1], [1, 2], [2, 3]]
));

assert(5.iota.slide(3).equal!equal(
    [[0, 1, 2], [1, 2, 3], [2, 3, 4]]
));
```

```d
import std.algorithm.comparison : equal;

assert(6.iota.slide(1, 2).equal!equal(
    [[0], [2], [4]]
));

assert(6.iota.slide(2, 4).equal!equal(
    [[0, 1], [4, 5]]
));

assert(iota(7).slide(2, 2).equal!equal(
    [[0, 1], [2, 3], [4, 5], [6]]
));

assert(iota(12).slide(2, 4).equal!equal(
    [[0, 1], [4, 5], [8, 9]]
));
```

**stride**

Итерирует по диапазону с шагом n.

```d
import std.algorithm.comparison : equal;

int[] a = [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 ];
assert(equal(stride(a, 3), [ 1, 4, 7, 10 ][]));
writeln(stride(stride(a, 2), 3)); // stride(a, 6)
```

**tail**

Выдает диапазон, состоящий из n последних элементов заданного диапазона.

```d
// tail -c n
writeln([1, 2, 3].tail(1)); // [3]
writeln([1, 2, 3].tail(2)); // [2, 3]
writeln([1, 2, 3].tail(3)); // [1, 2, 3]
writeln([1, 2, 3].tail(4)); // [1, 2, 3]
writeln([1, 2, 3].tail(0).length); // 0

// tail --lines=n
import std.algorithm.comparison : equal;
import std.algorithm.iteration : joiner;
import std.exception : assumeWontThrow;
import std.string : lineSplitter;
assert("one\ntwo\nthree"
    .lineSplitter
    .tail(2)
    .joiner("\n")
    .equal("two\nthree")
    .assumeWontThrow);
```

**take**

Создает поддиапазон, состоящий только из первых n элементов данного диапазона.

```d
import std.algorithm.comparison : equal;

int[] arr1 = [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ];
auto s = take(arr1, 5);
writeln(s.length); // 5
writeln(s[4]); // 5
assert(equal(s, [ 1, 2, 3, 4, 5 ][]));
```

**takeExactly**

Аналогично take, однако предполагает, что заданный диапазон реально содержит n элементов, поэтому возвращаемая длина всегда верна.

```d
import std.algorithm.comparison : equal;

auto a = [ 1, 2, 3, 4, 5 ];

auto b = takeExactly(a, 3);
assert(equal(b, [1, 2, 3]));
static assert(is(typeof(b.length) == size_t));
writeln(b.length); // 3
writeln(b.front); // 1
writeln(b.back); // 3
```

**takeNone**

Создает пустой диапазон произвольного доступа.

```d
auto range = takeNone!(int[])();
writeln(range.length); // 0
assert(range.empty);
```

**takeOne**

Создает диапазон произвольного доступа, состоящий ровно из первого элемента данного диапазона.

```d
auto s = takeOne([42, 43, 44]);
static assert(isRandomAccessRange!(typeof(s)));
writeln(s.length); // 1
assert(!s.empty);
writeln(s.front); // 42
s.front = 43;
writeln(s.front); // 43
writeln(s.back); // 43
writeln(s[0]); // 43
s.popFront();
writeln(s.length); // 0
assert(s.empty);
```

**tee**

Создает диапазон-обертку для данного диапазона, перенаправляя его элементы, а также вызывает предоставленную функцию для каждого элемента.

```d
import std.algorithm.comparison : equal;
import std.algorithm.iteration : filter, map;

// Sum values while copying
int[] values = [1, 4, 9, 16, 25];
int sum = 0;
auto newValues = values.tee!(a => sum += a).array;
assert(equal(newValues, values));
writeln(sum); // 1 + 4 + 9 + 16 + 25

// Count values that pass the first filter
int count = 0;
auto newValues4 = values.filter!(a => a < 10)
                        .tee!(a => count++)
                        .map!(a => a + 1)
                        .filter!(a => a < 10);

//Fine, equal also evaluates any lazy ranges passed to it.
//count is not 3 until equal evaluates newValues4
assert(equal(newValues4, [2, 5]));
writeln(count); // 3
```

**transposed**

Транспонирует диапазон диапазонов.

```d
import std.algorithm.comparison : equal;
int[][] ror = [
    [1, 2, 3],
    [4, 5, 6]
];
auto xp = transposed(ror);
assert(equal!"a.equal(b)"(xp, [
    [1, 4],
    [2, 5],
    [3, 6]
]));
```

```d
int[][] x = new int[][2];
x[0] = [1, 2];
x[1] = [3, 4];
auto tr = transposed(x);
int[][] witness = [ [ 1, 3 ], [ 2, 4 ] ];
uint i;

foreach (e; tr)
{
    writeln(array(e)); // witness[i++]
}
```

**transversal**

Создает диапазон, который перебирает n-ые элементы заданных диапазонов произвольного доступа.

```d
import std.algorithm.comparison : equal;
int[][] x = new int[][2];
x[0] = [1, 2];
x[1] = [3, 4];
auto ror = transversal(x, 1);
assert(equal(ror, [ 2, 4 ]));
```

```d
import std.algorithm.comparison : equal;
import std.algorithm.iteration : map;
int[][] y = [[1, 2, 3], [4, 5, 6]];
auto z = y.front.walkLength.iota.map!(i => transversal(y, i));
assert(equal!equal(z, [[1, 4], [2, 5], [3, 6]]));
```

**zip**

Из нескольких заданных диапазонов создает диапазон, который последовательно возвращает кортеж всех первых элементов, кортеж всех вторых элементов и т. д.

```d
import std.algorithm.comparison : equal;
import std.algorithm.iteration : map;

// pairwise sum
auto arr = only(0, 1, 2);
auto part1 = zip(arr, arr.dropOne).map!"a[0] + a[1]";
assert(part1.equal(only(1, 3)));
```

```d
import std.conv : to;

int[] a = [ 1, 2, 3 ];
string[] b = [ "a", "b", "c" ];
string[] result;

foreach (tup; zip(a, b))
{
    result ~= tup[0].to!string ~ tup[1];
}

writeln(result); // ["1a", "2b", "3c"]

size_t idx = 0;
// unpacking tuple elements with foreach
foreach (e1, e2; zip(a, b))
{
    writeln(e1); // a[idx]
    writeln(e2); // b[idx]
    ++idx;
}
```

```d
import std.algorithm.sorting : sort;

int[] a = [ 1, 2, 3 ];
string[] b = [ "a", "c", "b" ];
zip(a, b).sort!((t1, t2) => t1[0] > t2[0]);

writeln(a); // [3, 2, 1]
// b is sorted according to a's sorting
writeln(b); // ["b", "c", "a"]
```