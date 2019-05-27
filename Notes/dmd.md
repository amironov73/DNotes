### Компилятор DMD

"Digital Mars D" -- рефренсный компилятор.

Скачивается со страницы `https://dlang.org/download.html`.
Установка на примере CentOS:

```
yum install dmd-2.086.0-0.fedora.x86_64.rpm
```

Также возможна установка через snap:

```
snap install go --classic
```

Компилятор в Linux проживает в файле `/usr/bin/dmd` (при штатной установке) или `/snap/dmd/current/bin/dmd` (при установке через snap), в Windows -- в файле `C:\D\dmd2\windows\bin\dmd.exe`.

Командная строка dmd выглядит так:

```
dmd файлы... -опции...
```

Здесь файлы:

| Расширение | Тип файла |
|------------|-----------|
| нет        | Исходные коды на языке D |
| .d         | Исходные коды на языке D |
| .dd        | Исходные коды для ddoc   |
| .di        | Интерфейсные файлы D     |
| .obj       | Объектные файлы (влинковываются в программу)  |
| .lib       | Библиотеки (для поиска объектного кода) |
| .exe       | Выходной исполняемый файл |
| .def       | Файл определения модуля |
| .res       | Ресурсные файлы |

`@cmdfile` -- либо переменная окружения, их которой беруются аргументы и опции, либо имя файла с аргументами и опциями. Файл может содержать комментарии, начинающиеся с `#`.

`-allinst` -- генерировать код для всех инстанцирований шаблонов.

`-betterC` -- переключает компилятор в режим "лучшего C".

`-boundscheck`=\[on|safeonly|off\] -- включить/выключить проверку попадания индекса в разрешенный диапазон.

`-c` -- только компиляция, без линковки.

`-check`=\[assert|bounds|in|invariant|out|switch|h|help|? \]\[=\[on|off\]\] -- включить/выключить указанные проверки.

`-checkaction`=\[D|C|halt|context|h|help|?\] -- реакция на нарушение при проверке.

`-color` -- включить цветной вывод информации.

`-conf`=filename -- взять конфигурацию из файла.

`-cov` -- провести анализ покрытия.

```
dmd -cov -unittest myprog.d
```

`-Dd`directory -- поместить документацию в директорию.

`-Df`filename -- поместить документацию в файл.

`-d` -- молча разрешить отменённые возможности и использовать символы с атрибутом отмены.

`-dw` -- то же, но с выводом предупреждения.

`-de` -- выдать ошибку при использовании отменённых возможностей.

`-debug` -- компилировать отладочный вариант программы.

`-debug`=level -- установить уровень отладки.

`-debug`=ident -- установить отладочный символ.

`-debuglib`=name -- линковать с библиотекой `name` вместо `phobos.lib`.

`-deps` -- респечатывать зависимости модуля (import/file/version/debug/lib).

`-deps`=filename -- распечатывать import-зависимости в файл.

`-extern-std`=standard -- внеший стандарт C++: C++98, C++1, C++14, C++17.

`-fPIC` -- генерировать перемещаемый (position independent) код.

`-g` -- добавлять отладочную информацию CodeView с расширениями D.

`-gf` -- выдавать отладочную информацию обо всех типах, на которые есть ссылки.

`-gs` -- всегда создавать стековый фрейм.

`-gx` -- генерировать "вытаптыватель" стека, т. е. код, затирающий содержимое стекового фрейма при выходе из функции.

`-H` -- генерировать интерфейсный файл.

`-Hd`=directory -- записать интерфейсный файл в директорию.

`-Hf`=filename -- записать интерфейсный файл в файл.

`--help` -- распечатать подсказку и выйти.

`-I=`directory -- искать импорты в директории.

`-i=`pattern -- добавить ссылку на пакет (`-i=foo`) или удалить ссылку на пакет (`-i=-foo.bar`).

`-ignore` -- игнорировать неподдерживаемые pragma.

`-J`=directory -- директория для поиска импорта.

`-L`=flag -- передать флаг линкёру.

`-lib` -- генерировать библиотечный файл.

`-lowmem` -- разрешить использование сборщика мусора в компиляторе, чтобы понизить потребление памяти.

`-m32` -- создавать 32-битный исполняемый файл (по умолчанию). Генерируемые объектные файлы имеют формат OMF и предназначены для применения совместно с Digital Mars C/C++.

`-m32mscoff` -- создавать 32-битный исполняемый файл. Генерируемые объектные файлы имеют формат MS-COFF.

`-m64` -- создавать 64-битный исполняемый файл. Генерируемые объектные файлы имеют формат MS-COFF и предназначены для совместного применения c компилятором Microsoft Visual Studio 10 или более поздним. Также создаются файлы .exp и .lib.

`-main` -- добавлять функцию main() по умолчанию. Полезно для запуска unit-тестов.

`-man` -- запустить браузер по умолчанию со страницей https://dlang.org/dmd-windows.html

`-map` -- генерировать файл .map

`-mcpu`=id -- установить целевую архитектуру процессора.

`-mixin`=file -- раскрыть и сохранить миксины в указанном файле.

`-mscrtlib`=name -- при использовании формата MS-COFF встраивать ссылку на C runtime от Microsoft. По умолчанию libcmt (релизная версия со статической линковкой). Также возможны варианты: libcmtd, msvcrt и msvcrtd. Если имя не указывать, никакой ссылки на C runtime не гененрируется.

`-mv`-package.module=path/filename -- использовать path/filename как исходник для package.module.

`-O` -- оптимизировать генерируемый код. Для наибольшей скорости используйте
 
 ```
 -O -release -inline -boundscheck=off
 ``` 
`-o-` -- подавить создание объектных-файлов. Полезно в сочетании с `-D` или `-H`.

`-od`=directory -- помещать объектные файлы в указанную директорию.

`-of`=filename -- помещать результат компиляции в указанный файл. Это может быть объектный файл, исполняемый файл или библиотека.

`-op` -- обычно в объектный файл помещается информация только об имени исходного файла. Данная опция заставляет dmd помещать также путь к файлу.

`-preview`=id -- включить возможности языка, находящиеся на стадии предварительной оценки.

`-preview`=? -- вывести список таких возможностей, поддерживаемых текущим компилятором.

```
Upcoming language changes listed by -preview=name:
  =all              list information on all upcoming language changes
  =dip25            implement https://github.com/dlang/DIPs/blob/master/DIPs/archive/DIP25.md (Sealed references)
  =dip1000          implement https://github.com/dlang/DIPs/blob/master/DIPs/DIP1000.md (Scoped Pointers)
  =dip1008          implement https://github.com/dlang/DIPs/blob/master/DIPs/DIP1008.md (@nogc Throwable)
  =fieldwise        use fieldwise comparisons for struct equality
  =markdown         enable Markdown replacements in Ddoc
  =fixAliasThis     when a symbol is resolved, check alias this scope before going to upper scopes
  =intpromote       fix integral promotions for unary + - ~ operators
  =dtorfields       destruct fields of partially constructed objects
```

`-profile` -- профилировать производительность генерируемого кода.

`-profile`=gc -- профилировать использование сборщика мусора.

`-release` -- генерировать релизный код (т. е. не генерировать проверки контрактов и утверждений).

`-revert`=id -- отменить указанное изменение языка.

`-revert`=? -- выдать список изменений языка, которые могут быть отменены в текущем компиляторе.

```
Revertable language changes listed by -revert=name:
  =all              list information on all revertable language changes
  =dip25            revert DIP25 changes https://github.com/dlang/DIPs/blob/master/DIPs/archive/DIP25.md
  =import           revert to single phase name lookup
```

`-run` srcfile -- компилировать, линковать и запустить указанный файл. Последующие аргументы передаются в программу как её аргументы командной строки. Объектные файлы не сохраняются.

`-shared` -- создать DLL.

`-transition`=id -- показать дополнительную информацию об изменениях в языке.

`-transition`=? -- выдать список всех изменений в языке.

`-unittest` -- компилировать исходный код unit-тестов, задать идентификатор `unittest` для `version`.

`-v` -- вывод подробной информации о компиляции.

`-vcolumnst` -- выводить номер колонки в диагностических сообщениях.

`-verrors`=num -- задать предел количества ошибок (0 означает бесконечность).

`-verrors`=context -- выводить контекст строки с ошибкой.

`--version` -- выдать номер версии компилятора и завершиться.

`--version`=ident -- компилировать с версией компилятора большей или равной указанной.

`-vgc` -- выдать список всех аллокаций GC, включая скрытые.

```
irbis.d(279): vgc: array literal may cause a GC allocation
irbis.d(300): vgc: operator `~=` may cause a GC allocation
irbis.d(324): vgc: `new` causes a GC allocation
```

`-vtls` -- выдать список всех переменных, помещенных в Thread Local Storage.

`-w` -- разрешить предупреждения.

`-wi` -- разрешить информирующие предупреждения.

`-X` -- генерировать JSON-файл (пример ниже сильно сокращен).

```json
[
 {
  "kind" : "module",
  "file" : "example1.d",
  "members" : [
   {
    "name" : "std.stdio",
    "kind" : "import",
    "line" : 1,
    "char" : 8,
    "protection" : "private"
   },
   {
    "name" : "irbis",
    "kind" : "import",
    "line" : 2,
    "char" : 8,
    "protection" : "private"
   },
   {
    "name" : "main",
    "kind" : "function",
    "line" : 4,
    "char" : 6,
    "deco" : "FZv",
    "endline" : 60,
    "endchar" : 1
   }
  ]
 },
 {
  "kind" : "module",
  "file" : "irbis.d",
  "members" : [
   {
    "name" : "std.algorithm",
    "kind" : "import",
    "line" : 9,
    "char" : 8,
    "protection" : "private",
    "selective" : [
     "canFind",
     "remove"
    ]
   },
   {
    "name" : "std.array",
    "kind" : "import",
    "line" : 10,
    "char" : 8,
    "protection" : "private"
   }
  ]
 }
]
```

`-Xf`=filename -- записать JSON в указанный файл.
