### Пакет std.exception

**assertNotThrown**

```d
auto assertNotThrown(T : Throwable = Exception, E)(lazy E expression, string msg = null, string file = __FILE__, size_t line = __LINE__); 
```

```d
import core.exception : AssertError;
import std.string;

assertNotThrown!StringException(enforce!StringException(true, "Error!"));

//Exception is the default.
assertNotThrown(enforce!StringException(true, "Error!"));

assert(collectExceptionMsg!AssertError(assertNotThrown!StringException(
           enforce!StringException(false, "Error!"))) ==
       `assertNotThrown failed: StringException was thrown: Error!`);
```

**assertThrown**

```d
void assertThrown(T : Throwable = Exception, E)(lazy E expression, string msg = null, string file = __FILE__, size_t line = __LINE__);
```

```d
import core.exception : AssertError;
import std.string;

assertThrown!StringException(enforce!StringException(false, "Error!"));

//Exception is the default.
assertThrown(enforce!StringException(false, "Error!"));

assert(collectExceptionMsg!AssertError(assertThrown!StringException(
           enforce!StringException(true, "Error!"))) ==
       `assertThrown failed: No StringException was thrown.`);
```

**enforce**

```d
template enforce(E : Throwable = Exception) if (is(typeof(new E("", string.init, size_t.init)) : Throwable) || is(typeof(new E(string.init, size_t.init)) : Throwable))

T enforce(T, Dg, string file = __FILE__, size_t line = __LINE__)(T value, scope Dg dg)
if (isSomeFunction!Dg && is(typeof(dg())) && is(typeof(() { if (!value) { } } ))); 

T enforce(T)(T value, lazy Throwable ex); 
```

```d
import core.stdc.stdlib : malloc, free;
import std.conv : ConvException, to;

// use enforce like assert
int a = 3;
enforce(a > 2, "a needs to be higher than 2.");

// enforce can throw a custom exception
enforce!ConvException(a > 2, "a needs to be higher than 2.");

// enforce will return it's input
enum size = 42;
auto memory = enforce(malloc(size), "malloc failed")[0 .. size];
scope(exit) free(memory.ptr);
```

**collectException**

```d
T collectException(T = Exception, E)(lazy E expression, ref E result);
T collectException(T : Throwable = Exception, E)(lazy E expression); 
```

```d
int b;
int foo() { throw new Exception("blah"); }
assert(collectException(foo(), b));

int[] a = new int[3];
import core.exception : RangeError;
assert(collectException!RangeError(a[4], b));
```

**collectExceptionMsg**

```d
string collectExceptionMsg(T = Exception, E)(lazy E expression);
```

**assumeUnique**

```d
pure nothrow immutable(T)[] assumeUnique(T)(T[] array);
pure nothrow immutable(T)[] assumeUnique(T)(ref T[] array); 
pure nothrow immutable(T[U]) assumeUnique(T, U)(ref T[U] array); 
```

**assumeWontThrow**

```d
nothrow T assumeWontThrow(T)(lazy T expr, string msg = null, string file = __FILE__, size_t line = __LINE__); 
```

**doesPointTo**

```d
pure nothrow @nogc @trusted bool doesPointTo(S, T, Tdummy = void)(auto ref const S source, ref const T target); 

pure nothrow @trusted bool doesPointTo(S, T)(auto ref const shared S source, ref const shared T target); 

pure nothrow @trusted bool mayPointTo(S, T, Tdummy = void)(auto ref const S source, ref const T target); 

pure nothrow @trusted bool mayPointTo(S, T)(auto ref const shared S source, ref const shared T target); 
```

**ErrnoException**

```d
class ErrnoException: object.Exception;
```

