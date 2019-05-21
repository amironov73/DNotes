### Работа с файлами

В целом работа с файлами в D выглядит довольно предсказуемо для человека, программировавшего на C. Единственная загвоздка — запомнить, какие пакеты за что отвечают.

```d
import std.stdio, std.exception, std.string;
import core.stdc.errno;
 
void main() {
    try {
        auto file = File("wisdom.txt", "r");
        scope(exit) file.close;
 
        string line;
        while ((line = file.readln()) !is null) {
            writeln("LINE: ", strip(line));
        }
    }
    catch (ErrnoException ex) {
        switch(ex.errno) {
            case EPERM:
            case EACCES:
                writeln("Access error");
                break;
 
            case ENOENT:
                writeln("No such file");
                break;
 
            default:
                writeln("Something incomprehensible happened");
                break;
        }
    }
}
```

Обратите внимание: строка считывается вместе с "\n" в конце! Поскольку вышеупомянутый вызов `file.readln` каждый раз выделяет память под строку (это создаёт излишний трафик памяти), можно уговорить D использовать один и тот же буфер:

```d
auto file = File("test.txt", "r");
char[] buffer; // Обратите внимание: не string!
  
while (file.readln(buffer)) {
    writeln("LINE: ", strip(line));
}
```

Строки можно считывать и в цикле `foreach`, результат будет тот же:

```d
foreach (line; file.byLine) {
    writeln("LINE: ", strip(line));
}
```

Прочитать весь файл как одну строку:

```d
// в кодировке UTF-8
auto utf8Data  = readText("test.txt");
  
// в кодировке UTF-16
auto utf16Data = readText!(wstring)("test.txt");
  
// в кодировке UTF-32
auto utf32Data = readText!(dstring)("test.txt");
```

Позиционирование по файлу:

```d
import std.exception, std.stdio;
...
try {
    auto file = File("test.txt", "r");
  
    // 10 байт после начала файла
    file.seek(10, SEEK_SET);
  
    // 2 байта до текущей позиции
    file.seek(-2, SEEK_CUR);
  
    // 4 байта после конца файла
    file.seek(-4, SEEK_END);
  
    // определение текущей позиции в файле
    auto pos = file.tell();
  
    // возврат к началу файла
    file.rewind();
}
catch (ErrnoException ex) {
    // обработка ошибки
}
```

Запись байтов в файл:

```d
import std.exception;
import std.stdio;
  
void main() {
    try {
        byte[] data = [0x68, 0x65, 0x6c, 0x6c, 0x6f];
        auto file = File("test.bin", "w");
        file.rawWrite(data);
    }
    catch (ErrnoException ex) {
        // обработка ошибки
    }
}
```

Можно короче:

```d
import std.file;
 
void main() {
    try {
        write("test.txt", [0x68, 0x65, 0x6c, 0x6c, 0x6f]);
    }
    catch (FileException ex) {
         // обработка ошибки
    }
}
```

Запись строк в файл:

```d
import std.exception;
import std.stdio;
  
void main() {
    try {
        auto file = File("test.txt", "w");
  
        // Write a string.
        file.write("1: Lorem ipsum\n");
  
        // Write a string followed by a newline character.
        file.writeln("2: Lorem ipsum");
  
        // Write a formatted string.
        file.writef("3: %s", "Lorem ipsum\n");
  
        // Write a formatted string followed by a newline character.
        file.writefln("4: %s", "Lorem ipsum");
    }
    catch (ErrnoException ex) {
        // Handle errors
    }
}
```

Быстрое чтение всех байтов из файла:

```d
import std.file, std.stdio;
  
void main() {
    try{
        auto data = cast(byte[]) read("test.txt");
        writeln(data);
    }
    catch (FileException ex) {
        // обработка ошибки
    }
}
```

Чтение слайса байтов:

```d
import std.file, std.stdio;
  
void main() {
    try {
        auto data = cast(byte[]) read("test.txt", 5);
        writeln(data);
    }
    catch (FileException ex) {
        // обработка ошибки
    }
}
```

Чтение файла чанками определённой длины:

```d
import std.exception, std.stdio;
  
void main() {
    try {
        auto file = File("test.txt", "r");
  
        foreach (buffer; file.byChunk(1024)) {
            writeln(buffer);
        }
    }
    catch (ErrnoException ex) {
        // обработка ошибки
    }
}
```

Создание файла (если файл есть, он обрезается до нулевой длины):

```d
try {
    File("test.txt", "w");
}
catch (ErrnoException ex) {
    // обработка ошибки
}
```

Проверка, существует ли файл:

```d
import std.file;
...
  
if (exists("test.txt")) {
    // делаем что-нибудь
}
```

Переименование/перемещение файла:

```d
import std.file;
  
try {
    rename("source.txt", "destination.txt");
}
catch (FileException ex) {
    // обработка ошибки
}
```

Копирование файла:

```d
import std.file;
  
try {
    copy("source.txt", "destination.txt");
}
catch (FileException ex) {
    // обработка ошибки
}
```

Удаление файла:

```d
import std.file;
  
try {
    remove("test.txt");
}
catch (FileException ex) {
    // Handle error
}
```

Получение информации о файле:

```d
import std.file;
import std.stdio;
  
try {
    auto file = DirEntry("test.txt");
  
    writeln("File name: ", file.name);
    writeln("Is directory: ", file.isDir);
    writeln("Is file: ", file.isFile);
    writeln("Is symlink: ", file.isSymlink);
    writeln("Size in bytes: ", file.size);
    writeln("Time last accessed: ", file.timeLastAccessed);
    writeln("Time last modified: ", file.timeLastModified);
    writeln("Attributes: ", file.attributes);
}
catch (FileException ex) {
    // обработка ошибки
}
```

Создание ZIP-архива:

```d
import std.file, std.outbuffer, std.string, std.zip;
 
void main() {
    try {
        auto file = new ArchiveMember();
        file.name = "wisdom.txt";
 
        auto data = new OutBuffer();
        data.write("Lorem ipsum");
        file.expandedData = data.toBytes();
 
        auto zip = new ZipArchive();
        zip.addMember(file);
 
        write("wisdom.zip", zip.build());
    }
    catch (ZipException ex) {
        // обработка ошибки
    }
}
```

Чтение архива, распаковка данных из него:

```d
import std.stdio, std.file, std.zip;
 
void main() {
    try {
        auto zip = new ZipArchive(read("wisdom.zip"));
 
        foreach (filename, member; zip.directory) {
            auto data = zip.expand(member);
            writeln(filename);
            writeln(cast(string)(data));
        }
    }
    catch (ZipException ex) {
        // обработка ошибки
    }
}
```

Запись в файл сжатых данных:

```d
import std.file, std.zlib;
  
void main() {
    try {
        auto data = compress("Lorem ipsum dolor sit amet");
        write("test.dat", data);
    }
    catch (ZlibException ex) {
        // обработка ошибки
    }
}
```

Чтение из файла сжатых данных:

```d
import std.file, std.zlib, std.stdio;
  
void main() {
    try {
        auto data = uncompress(read("test.dat"));
        writeln(data);
    }
    catch (ZlibException ex) {
        // обработка ошибки
    }
}
```