# Glosario

Términos que aparecen en el proyecto, explicados en simple. **Agrega los tuyos** a medida que aprendes: explicar algo con tus propias palabras es la mejor forma de saber si lo entendiste.

Formato: **Término** (fase donde aparece): explicación.

---

## Python por dentro

**CPython** (0): la implementación "oficial" de Python, escrita en C. Es la que instalas desde python.org. Hay otras (PyPy, Jython), pero las librerías de pip están hechas para CPython.

**Intérprete** (0): el programa que lee y ejecuta código Python. En CPython, el intérprete es una librería (`python313.dll`) y `python.exe` es apenas un programa chico que la usa. Nosotros hacemos lo mismo que `python.exe`, pero desde C#.

**Embeber vs extender** (0): *extender* es escribir código en C para que Python lo use (un módulo nativo). *Embeber* es lo inverso: un programa en otro lenguaje que contiene a Python adentro. PySharp embebe.

**API C de Python** (0): el conjunto de funciones en C que expone la DLL de Python (`Py_Initialize`, `PyLong_FromLongLong`...). Es la única forma de "hablar" con el intérprete desde afuera.

**PyObject\*** (2): en CPython **todo** es un `PyObject` (números, strings, funciones, módulos, clases). Desde C# lo ves como un puntero (`IntPtr`) a ese objeto en memoria.

**Módulo** (2): un archivo `.py` importado. Para la API C es un `PyObject*` más, del que sacas funciones con `PyObject_GetAttrString`.

**sys.path** (2): la lista de carpetas donde Python busca módulos cuando haces `import`. Si tu `.py` no está en ninguna, el import falla.

**stdlib (biblioteca estándar)** (1): los módulos que vienen con Python (`os`, `json`, `encodings`...). Están en la carpeta `Lib` de la instalación, y el intérprete los necesita para arrancar.

**Singletons: None, True, False** (5): objetos que existen una sola vez en todo el intérprete. Todo `None` es el mismo objeto, por eso se compara con `is`.

**Traceback** (4): el detalle de dónde ocurrió un error: archivo, línea y cadena de llamadas.

**Excepción pendiente** (4): cuando algo falla en la API C, la excepción no "salta" como en C#. Queda guardada en el estado del intérprete, y la función devuelve NULL o -1. Tú tienes que revisarlo y recogerla.

---

## Memoria

**Conteo de referencias** (3): la forma principal en que Python sabe cuándo liberar un objeto. Cada objeto tiene un contador de cuántos lo están usando. Cuando llega a 0, se libera.

**Py_IncRef / Py_DecRef** (3): suben o bajan el contador de un objeto. Si te olvidas de hacer `DecRef`, pierdes memoria (fuga). Si haces uno de más, el objeto se libera mientras alguien lo usa y la app se cae.

**Referencia nueva** (3): la función te da un objeto y **tú eres responsable** de hacerle `DecRef` cuando termines. La documentación dice "Return value: New reference".

**Referencia prestada (borrowed)** (3): te dan acceso al objeto, pero no es tuyo. **No** le hagas `DecRef`. Si lo quieres guardar, hazle `IncRef`.

**Referencia robada (stolen)** (3): le das un objeto a una función y ella se queda con tu referencia, por ejemplo `PyTuple_SetItem`. Después de eso ya no le hagas `DecRef`.

**Garbage collector (GC) de .NET** (3): libera automáticamente los objetos de C# que nadie usa. **No sabe nada** de los objetos de Python: para él, un `IntPtr` es solo un número.

**IDisposable / using** (3): la forma de .NET de decir "este objeto tiene un recurso que hay que liberar a mano". `using` llama a `Dispose()` automáticamente al salir del bloque.

**SafeHandle** (3): clase de .NET pensada para envolver recursos nativos (punteros, handles) y liberarlos de forma segura, incluso si te olvidas de llamar a `Dispose`.

**Finalizador** (3): método que el GC ejecuta antes de borrar un objeto, en **otro hilo** que el GC elige. Por eso hay que tener cuidado si ahí adentro llamas a Python (ver GIL).

---

## Interop: .NET ↔ código nativo

**Código administrado vs nativo** (0): *administrado* es el código que corre dentro de .NET, con GC y seguridad de tipos. *Nativo* es código compilado directo a instrucciones de máquina, como la DLL de Python, que fue escrita en C. Pasar de uno al otro es "interop".

**DLL** (0): librería nativa de Windows (*Dynamic Link Library*): un archivo con funciones compiladas que otros programas pueden cargar. En Linux son `.so` y en macOS `.dylib`.

**Arquitectura (x64 / x86 / ARM64)** (0): el tipo de procesador para el que se compiló algo. Tu app y la DLL de Python tienen que ser de la misma arquitectura, o falla con `BadImageFormatException`.

**P/Invoke** (0): *Platform Invoke*, el mecanismo de .NET para llamar funciones de librerías nativas. Declaras la función en C# con un atributo y .NET se encarga del resto.

**DllImport** (0): el atributo clásico de P/Invoke. Hace la conversión de datos en tiempo de ejecución.

**LibraryImport** (0): el atributo moderno (.NET 7+). Genera el código de conversión al compilar, así que es más rápido y compatible con AOT. Necesita `partial` y a veces `AllowUnsafeBlocks`.

**Marshalling** (0): convertir datos entre el formato de .NET y el formato nativo. Por ejemplo, un `string` de C# (UTF-16) a un `char*` de C (UTF-8 en el caso de Python).

**IntPtr / nint** (0): un entero del tamaño de un puntero (8 bytes en x64). Es como C# guarda la dirección de memoria de algo nativo, por ejemplo un `PyObject*`.

**NULL** (2): un puntero con valor 0, que significa "no apunta a nada". En la API C de Python, que una función devuelva NULL casi siempre significa "hubo un error".

**Convención de llamada (cdecl, stdcall)** (0): reglas de cómo se pasan los argumentos entre funciones nativas. Python usa `cdecl`. En x64 hay una sola convención, así que casi no te afecta.

**EntryPointNotFoundException** (1): .NET encontró la DLL pero no la función con ese nombre. Revisa que el nombre esté bien escrito y que no sea una macro.

**Macro (en C)** (1): algo que parece una función pero el compilador de C lo reemplaza por código antes de compilar. **No existe en la DLL**, así que no puedes llamarlo con P/Invoke. Ejemplos: `Py_INCREF` (usa `Py_IncRef`), `PyTuple_GET_ITEM` (usa `PyTuple_GetItem`).

**NativeLibrary** (1): clase de .NET para cargar una DLL a mano desde una ruta (`NativeLibrary.Load`) y decidir de dónde se cargan las DLL de tus `[DllImport]` (`SetDllImportResolver`).

**UTF-8 / UTF-16** (2): formas de guardar texto en bytes. C# usa UTF-16 por dentro y la API C de Python usa UTF-8. Si los mezclas mal, las tildes y los emojis salen rotos.

---

## Hilos

**Hilo (thread)** (6): una línea de ejecución dentro de tu programa. .NET puede tener muchos corriendo a la vez (`Task`, `Parallel.For`, el hilo del finalizador...).

**GIL** (6): *Global Interpreter Lock*. Un candado que garantiza que **solo un hilo** ejecute código Python a la vez. Antes de tocar cualquier cosa de Python desde un hilo, hay que tomarlo, y soltarlo al terminar.

**Estado de hilo (PyThreadState)** (6): la información que Python guarda de cada hilo que lo usa. `PyGILState_Ensure` lo crea si hace falta y toma el GIL.

**async/await y cambio de hilo** (6): después de un `await`, tu código puede seguir en **otro hilo**. Como el GIL pertenece a un hilo concreto, hacer `await` con el GIL tomado puede romper todo.

**Python free-threaded** (6): versión experimental de Python 3.13 (`python3.13t`) sin GIL. Interesante para investigar, pero cambia muchas reglas.

---

## Entorno de Python

**venv (entorno virtual)** (7): una carpeta con su propia lista de paquetes de pip, separada del Python global. Así cada proyecto tiene sus versiones de paquetes.

**site-packages** (7): la carpeta donde pip instala los paquetes. Para usar paquetes de un venv, esa carpeta tiene que estar en `sys.path`.

**pyvenv.cfg** (7): archivo dentro de un venv que dice de qué instalación de Python se creó.

**PYTHONHOME** (1, 7): variable de entorno que le dice a Python dónde está su instalación (y por lo tanto su stdlib).

**PyConfig** (7): la forma moderna y detallada de configurar el intérprete antes de iniciarlo (rutas, opciones). Se usa con `Py_InitializeFromConfig`.

**PEP 514** (7): el estándar que define dónde registra Python sus instalaciones en el registro de Windows.

**PEP** (general): *Python Enhancement Proposal*, un documento que propone o describe un cambio o estándar de Python.

---

## .NET avanzado y publicación

**Roslyn** (10): el compilador de C#. Se puede "enchufar" código propio que corre mientras compilas.

**Source generator** (10): código que corre dentro de Roslyn y **genera más código C#** al compilar. CSnakes lo usa para leer tus `.py` y crear clases C# automáticamente.

**AdditionalFiles** (10): archivos que no son C# (como tus `.py`) que le pasas al compilador para que un source generator los lea.

**netstandard2.0** (10): un "denominador común" de .NET. Los source generators tienen que compilarse para él, porque Visual Studio y otras herramientas los cargan en versiones distintas de .NET.

**Type hints** (10): las anotaciones de tipos en Python (`def add(a: int) -> int`). Python no las usa al ejecutar, pero un generador puede leerlas para saber qué tipos de C# usar.

**NuGet** (9): el gestor de paquetes de .NET, el equivalente a pip. `dotnet pack` crea un `.nupkg`.

**Native AOT** (11): compilar una app .NET directo a código nativo, sin JIT. Arranca más rápido, pero no permite cierto código dinámico (por eso conviene `LibraryImport`).

**Buffer protocol** (11): mecanismo de Python para compartir memoria cruda (por ejemplo, el array de numpy) sin copiarla.

---

## Mis términos

_(Agrega acá lo que vayas aprendiendo)_
