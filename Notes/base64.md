### base64 в D

Поддержка base64 проживает в модуле `std.base64` и содержит две реализации. Первая `Base64` работает строго по стандарту, вторая - `Base64URL` модифицирована для удобного встраивания в имена файлов и URL.

Применение интуитивно понятно:

```d
ubyte[] data = [0x14, 0xfb, 0x9c, 0x03, 0xd9, 0x7e];

const(char)[] encoded = Base64.encode(data);
assert(encoded == "FPucA9l+");

ubyte[] decoded = Base64.decode("FPucA9l+");
assert(decoded == [0x14, 0xfb, 0x9c, 0x03, 0xd9, 0x7e]);
```

Также доступно Range API:

```d
File f = File("./text.txt", "r");
scope(exit) f.close();

Appender!string mime64 = appender!string;

foreach (encoded; Base64.encoder(f.byChunk(57))) {
    mime64.put(encoded);
    mime64.put("\r\n");
}

writeln(mime64.data);
```
