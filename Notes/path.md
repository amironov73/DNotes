### Модуль std.path

#### Нормализация пути

**absolutePath** -- трансформирует путь в абсолютный.

```d
pure @safe string absolutePath(string path, lazy string base = getcwd());

version (Posix)
{
    writeln(absolutePath("some/file", "/foo/bar")); // "/foo/bar/some/file"
    writeln(absolutePath("../file", "/foo/bar")); // "/foo/bar/../file"
    writeln(absolutePath("/some/file", "/foo/bar")); // "/some/file"
}

version (Windows)
{
    writeln(absolutePath(`some\file`, `c:\foo\bar`)); // `c:\foo\bar\some\file`
    writeln(absolutePath(`..\file`, `c:\foo\bar`)); // `c:\foo\bar\..\file`
    writeln(absolutePath(`c:\some\file`, `c:\foo\bar`)); // `c:\some\file`
    writeln(absolutePath(`\`, `c:\`)); // `c:\`
    writeln(absolutePath(`\some\file`, `c:\foo\bar`)); // `c:\some\file`
}
```

**asAbsolutePath** -- трансформирует путь в абсолютный, не размещает новой памяти, возвращает ленивый Range.

```d
import std.array;
writeln(asAbsolutePath(cast(string)null).array); // ""
version (Posix)
{
    writeln(asAbsolutePath("/foo").array); // "/foo"
}
version (Windows)
{
    writeln(asAbsolutePath("c:/foo").array); // "c:/foo"
}
asAbsolutePath("foo");
```

**asNormalizedPath** -- нормализует переданный путь, не размещает новой памяти, возвращает ленивый Range.

```d
import std.array;
writeln(asNormalizedPath("foo/..").array); // "."

version (Posix)
{
    writeln(asNormalizedPath("/foo/./bar/..//baz/").array); // "/foo/baz"
    writeln(asNormalizedPath("../foo/.").array); // "../foo"
    writeln(asNormalizedPath("/foo/bar/baz/").array); // "/foo/bar/baz"
    writeln(asNormalizedPath("/foo/./bar/../../baz").array); // "/baz"
}

version (Windows)
{
    writeln(asNormalizedPath(`c:\foo\.\bar/..\\baz\`).array); // `c:\foo\baz`
    writeln(asNormalizedPath(`..\foo\.`).array); // `..\foo`
    writeln(asNormalizedPath(`c:\foo\bar\baz\`).array); // `c:\foo\bar\baz`
    writeln(asNormalizedPath(`c:\foo\bar/..`).array); // `c:\foo`
    assert(asNormalizedPath(`\\server\share\foo\..\bar`).array ==
            `\\server\share\bar`);
}
```

**asRelativePath** -- трансформирует путь в относительный, не размещает новой памяти, возвращает ленивый Range.

```d
import std.array;
version (Posix)
{
    writeln(asRelativePath("foo", "/bar").array); // "foo"
    writeln(asRelativePath("/foo/bar", "/foo/bar").array); // "."
    writeln(asRelativePath("/foo/bar", "/foo/baz").array); // "../bar"
    writeln(asRelativePath("/foo/bar/baz", "/foo/woo/wee").array); // "../../bar/baz"
    writeln(asRelativePath("/foo/bar/baz", "/foo/bar").array); // "baz"
}
version (Windows)
{
    writeln(asRelativePath("foo", `c:\bar`).array); // "foo"
    writeln(asRelativePath(`c:\foo\bar`, `c:\foo\bar`).array); // "."
    writeln(asRelativePath(`c:\foo\bar`, `c:\foo\baz`).array); // `..\bar`
    writeln(asRelativePath(`c:\foo\bar\baz`, `c:\foo\woo\wee`).array); // `..\..\bar\baz`
    writeln(asRelativePath(`c:/foo/bar/baz`, `c:\foo\woo\wee`).array); // `..\..\bar\baz`
    writeln(asRelativePath(`c:\foo\bar\baz`, `c:\foo\bar`).array); // "baz"
    writeln(asRelativePath(`c:\foo\bar`, `d:\foo`).array); // `c:\foo\bar`
    writeln(asRelativePath(`\\foo\bar`, `c:\foo`).array); // `\\foo\bar`
}
```

**buildNormalizedPath** -- версия `buildPath`, нормализующая получающийся путь.

```d
writeln(buildNormalizedPath("foo", "..")); // "."

version (Posix)
{
    writeln(buildNormalizedPath("/foo/./bar/..//baz/")); // "/foo/baz"
    writeln(buildNormalizedPath("../foo/.")); // "../foo"
    writeln(buildNormalizedPath("/foo", "bar/baz/")); // "/foo/bar/baz"
    writeln(buildNormalizedPath("/foo", "/bar/..", "baz")); // "/baz"
    writeln(buildNormalizedPath("foo/./bar", "../../", "../baz")); // "../baz"
    writeln(buildNormalizedPath("/foo/./bar", "../../baz")); // "/baz"
}

version (Windows)
{
    writeln(buildNormalizedPath(`c:\foo\.\bar/..\\baz\`)); // `c:\foo\baz`
    writeln(buildNormalizedPath(`..\foo\.`)); // `..\foo`
    writeln(buildNormalizedPath(`c:\foo`, `bar\baz\`)); // `c:\foo\bar\baz`
    writeln(buildNormalizedPath(`c:\foo`, `bar/..`)); // `c:\foo`
    assert(buildNormalizedPath(`\\server\share\foo`, `..\bar`) ==
            `\\server\share\bar`);
}
```

**buildPath** -- комбинирует сегменты пути. Если какой-либо сегмент оказывается абсолютным, предшествующие сегменты отбрасываются. Всегда размещает новую память.

```d
version (Posix)
{
    writeln(buildPath("foo", "bar", "baz")); // "foo/bar/baz"
    writeln(buildPath("/foo/", "bar/baz")); // "/foo/bar/baz"
    writeln(buildPath("/foo", "/bar")); // "/bar"
}

version (Windows)
{
    writeln(buildPath("foo", "bar", "baz")); // `foo\bar\baz`
    writeln(buildPath(`c:\foo`, `bar\baz`)); // `c:\foo\bar\baz`
    writeln(buildPath("foo", `d:\bar`)); // `d:\bar`
    writeln(buildPath("foo", `\bar`)); // `\bar`
    writeln(buildPath(`c:\foo`, `\bar`)); // `c:\bar`
}
```

**chainPath** -- ленивая версия `buildPath`, не размещает новой памяти, возвращает ленивый Range.

```d
import std.array;
version (Posix)
{
    writeln(chainPath("foo", "bar", "baz").array); // "foo/bar/baz"
    writeln(chainPath("/foo/", "bar/baz").array); // "/foo/bar/baz"
    writeln(chainPath("/foo", "/bar").array); // "/bar"
}

version (Windows)
{
    writeln(chainPath("foo", "bar", "baz").array); // `foo\bar\baz`
    writeln(chainPath(`c:\foo`, `bar\baz`).array); // `c:\foo\bar\baz`
    writeln(chainPath("foo", `d:\bar`).array); // `d:\bar`
    writeln(chainPath("foo", `\bar`).array); // `\bar`
    writeln(chainPath(`c:\foo`, `\bar`).array); // `c:\bar`
}

import std.utf : byChar;
version (Posix)
{
    writeln(chainPath("foo", "bar", "baz").array); // "foo/bar/baz"
    writeln(chainPath("/foo/".byChar, "bar/baz").array); // "/foo/bar/baz"
    writeln(chainPath("/foo", "/bar".byChar).array); // "/bar"
}

version (Windows)
{
    writeln(chainPath("foo", "bar", "baz").array); // `foo\bar\baz`
    writeln(chainPath(`c:\foo`.byChar, `bar\baz`).array); // `c:\foo\bar\baz`
    writeln(chainPath("foo", `d:\bar`).array); // `d:\bar`
    writeln(chainPath("foo", `\bar`.byChar).array); // `\bar`
    writeln(chainPath(`c:\foo`, `\bar`w).array); // `c:\bar`
}
```

**expandTilde** -- расширяет тильду в POSIX-системах.

```d
version (Posix)
{
    import std.process : environment;

    auto oldHome = environment["HOME"];
    scope(exit) environment["HOME"] = oldHome;

    environment["HOME"] = "dmd/test";
    writeln(expandTilde("~/")); // "dmd/test/"
    writeln(expandTilde("~")); // "dmd/test"
}
```

#### Разбиение на компоненты

**baseName** -- извлекает полное имя файла (включая расширение), соответствует утилите UNIX basename. Опционально удаляет из имени указанный суффикс (не обязательно расширение).

```d
writeln(baseName("dir/file.ext")); // "file.ext"
writeln(baseName("dir/file.ext", ".ext")); // "file"
writeln(baseName("dir/file.ext", ".xyz")); // "file.ext"
writeln(baseName("dir/filename", "name")); // "file"
writeln(baseName("dir/subdir/")); // "subdir"
```

**dirName** -- извлекает имя родительской папки. Если не может установить имя папки, возвращает `"."`.

```d
writeln(dirName("")); // "."
writeln(dirName("file"w)); // "."
writeln(dirName("dir/"d)); // "."
writeln(dirName("dir///")); // "."
writeln(dirName("dir/file"w.dup)); // "dir"
writeln(dirName("dir///file"d.dup)); // "dir"
writeln(dirName("dir/subdir/")); // "dir"
writeln(dirName("/dir/file"w)); // "/dir"
writeln(dirName("/file"d)); // "/"
writeln(dirName("/")); // "/"
writeln(dirName("///")); // "/"
```

**dirSeparator** -- разделитель компонентов пути. В POSIX прямой слэш, в Windows обратный слэш.

**driveName** -- извлекает имя диска. Если не может установить имя диска, возвращает пустую строку.

```d
version (Posix)  assert(driveName("c:/foo").empty);
version (Windows)
{
    assert(driveName(`dir\file`).empty);
    writeln(driveName(`d:file`)); // "d:"
    writeln(driveName(`d:\file`)); // "d:"
    writeln(driveName("d:")); // "d:"
    writeln(driveName(`\\server\share\file`)); // `\\server\share`
    writeln(driveName(`\\server\share\`)); // `\\server\share`
    writeln(driveName(`\\server\share`)); // `\\server\share`

    static assert(driveName(`d:\file`) == "d:");
}
```

**pathSeparator** -- разделитель компонентов в переменной PATH. В POSIX двоеточие, в Windows точка с запятой.

**pathSplitter** -- разбирает путь на компоненты, возвращает Range.

```d
import std.algorithm.comparison : equal;
import std.conv : to;

assert(equal(pathSplitter("/"), ["/"]));
assert(equal(pathSplitter("/foo/bar"), ["/", "foo", "bar"]));
assert(equal(pathSplitter("foo/../bar//./"), ["foo", "..", "bar", "."]));

version (Posix)
{
    assert(equal(pathSplitter("//foo/bar"), ["/", "foo", "bar"]));
}

version (Windows)
{
    assert(equal(pathSplitter(`foo\..\bar\/.\`), ["foo", "..", "bar", "."]));
    assert(equal(pathSplitter("c:"), ["c:"]));
    assert(equal(pathSplitter(`c:\foo\bar`), [`c:\`, "foo", "bar"]));
    assert(equal(pathSplitter(`c:foo\bar`), ["c:foo", "bar"]));
}
```

**relativePath** -- трансформирует путь в относительный.

```d
writeln(relativePath("foo")); // "foo"

version (Posix)
{
    writeln(relativePath("foo", "/bar")); // "foo"
    writeln(relativePath("/foo/bar", "/foo/bar")); // "."
    writeln(relativePath("/foo/bar", "/foo/baz")); // "../bar"
    writeln(relativePath("/foo/bar/baz", "/foo/woo/wee")); // "../../bar/baz"
    writeln(relativePath("/foo/bar/baz", "/foo/bar")); // "baz"
}
version (Windows)
{
    writeln(relativePath("foo", `c:\bar`)); // "foo"
    writeln(relativePath(`c:\foo\bar`, `c:\foo\bar`)); // "."
    writeln(relativePath(`c:\foo\bar`, `c:\foo\baz`)); // `..\bar`
    writeln(relativePath(`c:\foo\bar\baz`, `c:\foo\woo\wee`)); // `..\..\bar\baz`
    writeln(relativePath(`c:\foo\bar\baz`, `c:\foo\bar`)); // "baz"
    writeln(relativePath(`c:\foo\bar`, `d:\foo`)); // `c:\foo\bar`
}
```

**rootName** -- извлекает имя корневой папки. Если не может установить имя папки, возвращает `null`.

```d
assert(rootName("") is null);
assert(rootName("foo") is null);
writeln(rootName("/")); // "/"
writeln(rootName("/foo/bar")); // "/"
```

**stripDrive** -- в Windows удаляет имя диска из пути, в UNIX ничего не делает.

```d
version (Windows)
{
    writeln(stripDrive(`d:\dir\file`)); // `\dir\file`
    writeln(stripDrive(`\\server\share\dir\file`)); // `\dir\file`
}
```

#### Валидация

**isAbsolute** -- проверяет, является ли путь абсолютным.

```d
version (Posix)
{
    assert(isAbsolute("/"));
    assert(isAbsolute("/foo"));
    assert(!isAbsolute("foo"));
    assert(!isAbsolute("../foo"));
}

version (Windows)
{
    assert(isAbsolute(`d:\`));
    assert(isAbsolute(`d:\foo`));
    assert(isAbsolute(`\\foo\bar`));
    assert(!isAbsolute(`\`));
    assert(!isAbsolute(`\foo`));
    assert(!isAbsolute("d:foo"));
}
```

**isDirSeparator** -- проверяет, является ли символ разделителем компонентов пути.

```d
pure nothrow @nogc @safe bool isDirSeparator(dchar c);

version (Windows)
{
    assert( '/'.isDirSeparator);
    assert( '\\'.isDirSeparator);
}
else
{
    assert( '/'.isDirSeparator);
    assert(!'\\'.isDirSeparator);
}
```

**isRooted** -- проверяет, начинается ли путь с корневой директории. Не обращается к файловой системе.

```d
version (Posix)
{
    assert( isRooted("/"));
    assert( isRooted("/foo"));
    assert(!isRooted("foo"));
    assert(!isRooted("../foo"));
}

version (Windows)
{
    assert( isRooted(`\`));
    assert( isRooted(`\foo`));
    assert( isRooted(`d:\foo`));
    assert( isRooted(`\\foo\bar`));
    assert(!isRooted("foo"));
    assert(!isRooted("d:foo"));
}
```

**isValidFilename** -- проверяет, является ли указанное имя файла допустимым. Не обращается к файловой системе.

```d
bool isValidFilename(Range)(Range filename);

import std.utf : byCodeUnit;

assert(isValidFilename("hello.exe".byCodeUnit));
```

**isValidPath** -- проверяет, является ли указанный путь допустимым.

```d
bool isValidPath(Range)(Range path);

assert(isValidPath("/foo/bar"));
assert(!isValidPath("/foo\0/bar"));
assert(isValidPath("/"));
assert(isValidPath("a"));

version (Windows)
{
    assert(isValidPath(`c:\`));
    assert(isValidPath(`c:\foo`));
    assert(isValidPath(`c:\foo\.\bar\\\..\`));
    assert(!isValidPath(`!:\foo`));
    assert(!isValidPath(`c::\foo`));
    assert(!isValidPath(`c:\foo?`));
    assert(!isValidPath(`c:\foo.`));

    assert(isValidPath(`\\server\share`));
    assert(isValidPath(`\\server\share\foo`));
    assert(isValidPath(`\\server\share\\foo`));
    assert(!isValidPath(`\\\server\share\foo`));
    assert(!isValidPath(`\\server\\share\foo`));
    assert(!isValidPath(`\\ser*er\share\foo`));
    assert(!isValidPath(`\\server\sha?e\foo`));
    assert(!isValidPath(`\\server\share\|oo`));

    assert(isValidPath(`\\?\<>:"?*|/\..\.`));
    assert(!isValidPath("\\\\?\\foo\0bar"));

    assert(!isValidPath(`\\.\PhysicalDisk1`));
    assert(!isValidPath(`\\`));
}

import std.utf : byCodeUnit;
assert(isValidPath("/foo/bar".byCodeUnit));
```

#### Работа с расширениями файлов

**defaultExtension** -- добавляет указанное расширение, если путь не содержит расширения.

```d
writeln(defaultExtension("file", "ext")); // "file.ext"
writeln(defaultExtension("file", ".ext")); // "file.ext"
writeln(defaultExtension("file.", "ext")); // "file."
writeln(defaultExtension("file.old", "new")); // "file.old"
writeln(defaultExtension("file.old", ".new")); // "file.old"
```

**extension** -- извлекает расширение. Если его нет, возвращает `null`.

```d
assert(extension("file").empty);
writeln(extension("file.")); // "."
writeln(extension("file.ext"w)); // ".ext"
writeln(extension("file.ext1.ext2"d)); // ".ext2"
assert(extension(".foo".dup).empty);
writeln(extension(".foo.ext"w.dup)); // ".ext"

static assert(extension("file").empty);
static assert(extension("file.ext") == ".ext");
```

**setExtension** -- устанавливает или заменяет расширение.

```d
writeln(setExtension("file", "ext")); // "file.ext"
writeln(setExtension("file"w, ".ext"w)); // "file.ext"
writeln(setExtension("file."d, "ext"d)); // "file.ext"
writeln(setExtension("file.", ".ext")); // "file.ext"
writeln(setExtension("file.old"w, "new"w)); // "file.new"
writeln(setExtension("file.old"d, ".new"d)); // "file.new"
```

**stripExtension** -- удаляет расширение.

```d
writeln(stripExtension("file")); // "file"
writeln(stripExtension("file.ext")); // "file"
writeln(stripExtension("file.ext1.ext2")); // "file.ext1"
writeln(stripExtension("file.")); // "file"
writeln(stripExtension(".file")); // ".file"
writeln(stripExtension(".file.ext")); // ".file"
writeln(stripExtension("dir/file.ext")); // "dir/file"
```

**withDefaultExtension** -- добавляет указанное расширение, если в файле нет расширения. Не размещает новой памяти, возвращает ленивый Range.

```d
auto withDefaultExtension(R, C)(R path, C[] ext);

import std.array;
writeln(withDefaultExtension("file", "ext").array); // "file.ext"
writeln(withDefaultExtension("file"w, ".ext").array); // "file.ext"w
writeln(withDefaultExtension("file.", "ext").array); // "file."
writeln(withDefaultExtension("file", "").array); // "file."

import std.utf : byChar, byWchar;
writeln(withDefaultExtension("file".byChar, "ext").array); // "file.ext"
writeln(withDefaultExtension("file"w.byWchar, ".ext").array); // "file.ext"w
writeln(withDefaultExtension("file.".byChar, "ext"d).array); // "file."
writeln(withDefaultExtension("file".byChar, "").array); // "file."
```

**withExtension** -- заменяет расширение на указанное. Не размещает новой памяти, возвращает ленивый Range.

```d
auto withExtension(R, C)(R path, C[] ext);

import std.array;
writeln(withExtension("file", "ext").array); // "file.ext"
writeln(withExtension("file"w, ".ext"w).array); // "file.ext"
writeln(withExtension("file.ext"w, ".").array); // "file."

import std.utf : byChar, byWchar;
writeln(withExtension("file".byChar, "ext").array); // "file.ext"
writeln(withExtension("file"w.byWchar, ".ext"w).array); // "file.ext"w
writeln(withExtension("file.ext"w.byWchar, ".").array); // "file."w
```

#### Прочее

**filenameCharCmp** -- сравнивает символы имени файла.

```d
pure nothrow @safe int filenameCharCmp(CaseSensitive cs = CaseSensitive.osDefault)(dchar a, dchar b);
```

**filenameCmp** -- сравнивает имена файлов.

```d
int filenameCmp(CaseSensitive cs = CaseSensitive.osDefault, Range1, Range2)(Range1 filename1, Range2 filename2);
```

**globMatch** -- проверяет, соответствует ли путь указанной маске.

```d
pure nothrow @safe bool globMatch(CaseSensitive cs = CaseSensitive.osDefault, C, Range)(Range path, const(C)[] pattern);

assert(globMatch("foo.bar", "*"));
assert(globMatch("foo.bar", "*.*"));
assert(globMatch(`foo/foo\bar`, "f*b*r"));
assert(globMatch("foo.bar", "f???bar"));
assert(globMatch("foo.bar", "[fg]???bar"));
assert(globMatch("foo.bar", "[!gh]*bar"));
assert(globMatch("bar.fooz", "bar.{foo,bif}z"));
assert(globMatch("bar.bifz", "bar.{foo,bif}z"));

version (Windows)
{
    // Same as calling globMatch!(CaseSensitive.no)(path, pattern)
    assert(globMatch("foo", "Foo"));
    assert(globMatch("Goo.bar", "[fg]???bar"));
}
version (linux)
{
    // Same as calling globMatch!(CaseSensitive.yes)(path, pattern)
    assert(!globMatch("foo", "Foo"));
    assert(!globMatch("Goo.bar", "[fg]???bar"));
}
```


