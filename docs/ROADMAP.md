# Roadmap de PySharp

Cada fase tiene:
- **Objetivo:** qué vas a poder hacer al terminarla.
- **Conceptos:** qué necesitas entender (están en el [glosario](GLOSARIO.md)).
- **Ejercicios:** lo que programas tú.
- **Terminado cuando:** cómo sabes que puedes pasar a la siguiente.

Regla de oro: **primero intenta tú, después mira cómo lo resuelven otros** (CSnakes, pythonnet). Leer su código antes de pelearte con el problema te quita el aprendizaje.

---

## Fase 0: Fundamentos (sin tocar la librería todavía)

**Objetivo:** entender qué significa "embeber Python" y saber llamar código nativo desde C#.

**Conceptos:** CPython, intérprete, embeber vs extender, código administrado vs nativo, DLL, P/Invoke, marshalling, convención de llamada, arquitectura x64.

**Ejercicios:**
- [ ] Leer [Embedding Python in Another Application](https://docs.python.org/3/extending/embedding.html), sobre todo la sección 1.1 ("Very High Level Embedding"). Es un ejemplo en C, y tu trabajo es "traducirlo" a C#.
- [ ] Encontrar dónde está `python313.dll` en tu PC. Pregunta para investigar: ¿por qué tiene que ser de 64 bits igual que tu app?
- [ ] Crear una app de consola .NET 10 que llame a una función de Windows con P/Invoke, por ejemplo `GetTickCount64` de `kernel32.dll` o `MessageBoxW` de `user32.dll`. Así aprendes P/Invoke con algo que no es Python.
- [ ] Hacer lo mismo con `[DllImport]` y con `[LibraryImport]`. Anotar las diferencias.

**Terminado cuando:** sabes explicar con tus palabras qué hace `[DllImport("kernel32.dll")]` y qué es `IntPtr`.

---

## Fase 1: Hola Python

**Objetivo:** que tu app .NET arranque el intérprete de Python y ejecute una línea de código.

**Conceptos:** `Py_Initialize`, `PyRun_SimpleString`, `Py_FinalizeEx`, `NativeLibrary`, stdlib, PYTHONHOME, macros de C.

**Ejercicios:**
- [ ] Crear el proyecto: `dotnet new classlib -n PySharp` (la librería) y `dotnet new console -n PySharp.Sandbox` (para probar).
- [ ] Declarar con P/Invoke las 3 funciones: `Py_Initialize`, `PyRun_SimpleString`, `Py_FinalizeEx`.
- [ ] Ejecutar `print('Hola desde Python')` desde C#.
- [ ] Cargar la DLL desde una ruta concreta con `NativeLibrary.Load` y `NativeLibrary.SetDllImportResolver`, en lugar de depender del PATH.
- [ ] Investigar: si Python no encuentra su biblioteca estándar (error `ModuleNotFoundError: encodings`), ¿qué es PYTHONHOME y cómo lo arreglas?

**Ojo:** algunas cosas que en la documentación de C parecen funciones son en realidad **macros**, y no existen en la DLL. Por ejemplo, `Py_INCREF` no existe en la DLL, pero `Py_IncRef` sí. Si P/Invoke tira `EntryPointNotFoundException`, sospecha de esto.

**Terminado cuando:** ves el print en tu consola y la app cierra sin errores.

---

## Fase 2: Objetos y llamadas a funciones

**Objetivo:** importar un archivo `.py` propio, llamar una función con argumentos y leer el resultado.

**Conceptos:** `PyObject*`, módulo, `sys.path`, tupla de argumentos, NULL como señal de error.

**Funciones de la API C:** `PyImport_ImportModule`, `PyObject_GetAttrString`, `PyObject_CallObject`, `PyTuple_New`, `PyTuple_SetItem`, `PyLong_FromLongLong`, `PyLong_AsLongLong`, `PyUnicode_FromString`, `PyUnicode_AsUTF8`.

**Ejercicios:**
- [ ] Crear `demo.py` con `def add(a, b): return a + b` y copiarlo a la carpeta de salida (`<None Update="demo.py" CopyToOutputDirectory="PreserveNewest" />`).
- [ ] Agregar la carpeta del ejecutable a `sys.path` para que Python encuentre `demo.py`.
- [ ] Llamar `add(2, 3)` desde C# y obtener `5` como `long`.
- [ ] Llamar una función que reciba y devuelva un string (`greet("Nicolás")`). Probar con tildes y emojis: ¿se ven bien?
- [ ] Investigar: ¿por qué `PyUnicode_AsUTF8` devuelve un puntero que **no** tienes que liberar?

**Terminado cuando:** puedes llamar cualquier función simple con int/string y leer su resultado.

---

## Fase 3: Memoria y conteo de referencias

**Objetivo:** que tu librería no pierda memoria ni se caiga por liberar objetos de más.

**Conceptos:** conteo de referencias, referencia nueva / prestada / robada, `Py_IncRef` / `Py_DecRef`, garbage collector de .NET, `IDisposable`, `using`, `SafeHandle`, finalizador.

**Ejercicios:**
- [ ] Revisar cada función que usaste en la fase 2 y anotar si devuelve una referencia **nueva** o **prestada**. La documentación lo dice en cada función ("Return value: New reference").
- [ ] Notar que `PyTuple_SetItem` **roba** la referencia. ¿Qué pasa si después le haces `Py_DecRef` a ese objeto?
- [ ] Crear una clase `PyObject` en C# que envuelva el puntero e implemente `IDisposable` (o herede de `SafeHandle`).
- [ ] Prueba de fuga: llamar `add` un millón de veces en un loop y mirar la memoria en el Administrador de tareas. ¿Crece sin parar? Arreglarlo.
- [ ] Investigar: si el garbage collector de .NET ejecuta tu finalizador, ¿desde qué hilo lo hace? ¿Puedes llamar a `Py_DecRef` desde ahí? (Pista: fase 6).

**Terminado cuando:** el loop de un millón de llamadas mantiene la memoria estable.

---

## Fase 4: Manejo de errores

**Objetivo:** que una excepción en Python se convierta en una excepción de .NET con mensaje y traceback útiles.

**Conceptos:** excepción pendiente, NULL / -1 como valor de error, traceback.

**Funciones:** `PyErr_Occurred`, `PyErr_GetRaisedException` (Python 3.12+), `PyObject_Str`, módulo `traceback`.

**Ejercicios:**
- [ ] Llamar una función que haga `raise ValueError("algo salió mal")` y ver qué devuelve la API C.
- [ ] Crear `PythonException : Exception` con el tipo de error de Python, el mensaje y el traceback.
- [ ] Hacer un helper que revise cada resultado: si es NULL, lanzar `PythonException`.
- [ ] Probar `import modulo_que_no_existe` y una división por cero.

**Terminado cuando:** ningún error de Python pasa desapercibido y el mensaje en C# te dice qué línea del `.py` falló.

---

## Fase 5: Conversión de tipos

**Objetivo:** pasar datos entre C# y Python de forma natural.

**Conceptos:** marshalling, singletons de Python (None, True, False), enteros de precisión arbitraria, protocolo de secuencia/mapeo.

**Ejercicios (cada tipo en ambas direcciones, C# → Python y Python → C#):**
- [ ] `long` ↔ `int`. ¿Qué pasa con `2**100`? (Pista: `BigInteger`, overflow).
- [ ] `double` ↔ `float`
- [ ] `bool` ↔ `bool`, y `null` ↔ `None`. Investigar `Py_GetConstant` (Python 3.13).
- [ ] `string` ↔ `str` y `byte[]` ↔ `bytes`
- [ ] `List<T>` / arrays ↔ `list`, y tuplas de C# ↔ `tuple`
- [ ] `Dictionary<K,V>` ↔ `dict`
- [ ] Decisión de diseño: ¿cómo elige tu librería a qué tipo de C# convertir un objeto de Python? (genéricos `Call<T>`, `dynamic`, un tipo `PyObject` genérico...). Escribir la decisión y por qué la tomaste.

**Terminado cuando:** puedes mandar un `Dictionary<string, List<int>>` a Python y recibirlo de vuelta igual.

---

## Fase 6: Hilos y el GIL

**Objetivo:** usar Python desde varios hilos de .NET sin que la app se caiga.

**Conceptos:** GIL, hilo del sistema operativo, estado de hilo de Python, `async/await` y cambio de hilo, Python free-threaded.

**Funciones:** `PyEval_SaveThread`, `PyEval_RestoreThread`, `PyGILState_Ensure`, `PyGILState_Release`.

**Ejercicios:**
- [ ] Llamar a Python desde `Parallel.For` **sin** manejar el GIL y ver qué pasa. Es importante verlo fallar.
- [ ] Después de `Py_Initialize`, soltar el GIL con `PyEval_SaveThread` (el hilo que inicializa lo tiene tomado).
- [ ] Crear algo como `using (Py.GIL()) { ... }` que tome y suelte el GIL automáticamente.
- [ ] Volver a la pregunta de la fase 3: arreglar el finalizador para que tome el GIL antes de `Py_DecRef`.
- [ ] Investigar: ¿por qué es peligroso hacer `await` mientras tienes el GIL tomado?
- [ ] Bonus: medir si correr Python en 4 hilos es más rápido que en 1. ¿Por qué no lo es?

**Terminado cuando:** 8 hilos llaman a Python al mismo tiempo durante un minuto sin errores.

---

## Fase 7: Encontrar Python y usar entornos virtuales

**Objetivo:** que tu librería funcione en cualquier PC, no solo en la tuya.

**Conceptos:** venv, `site-packages`, `pyvenv.cfg`, registro de Windows (PEP 514), PYTHONHOME, `PyConfig`, nombres de librerías por sistema operativo.

**Ejercicios:**
- [ ] Encontrar Python automáticamente: variable de entorno, ruta explícita, el registro de Windows (`HKCU\Software\Python\PythonCore`) o el PATH.
- [ ] Crear un venv (`python -m venv .venv`), instalar un paquete (`pip install requests`) y usarlo desde C#. Pista simple: `site.addsitedir(...)` después de inicializar.
- [ ] Instalar paquetes desde C# ejecutando `python -m pip install -r requirements.txt` con `Process`.
- [ ] Investigar: en Linux la librería se llama `libpython3.13.so` y en macOS `libpython3.13.dylib`. ¿Cómo eliges la correcta? (`OperatingSystem.IsWindows()`...)
- [ ] Bonus avanzado: inicializar con `PyConfig` y `Py_InitializeFromConfig` en lugar de `Py_Initialize`.

**Terminado cuando:** usas `requests` u otro paquete de pip desde C# usando un venv.

---

## Fase 8: Diseño de la API pública

**Objetivo:** que PySharp sea cómodo de usar para otra persona.

**Conceptos:** API pública vs interna, ciclo de vida del intérprete (se inicializa **una vez por proceso**), patrón builder/opciones.

**Ejercicios:**
- [ ] Antes de programar, escribir en el README cómo te gustaría usar la librería. Por ejemplo:
  ```csharp
  using var python = PythonEnvironment.Create(o => o.WithVenv(".venv"));
  var demo = python.Import("demo");
  int result = demo.Call<int>("add", 2, 3);
  ```
- [ ] Ocultar todo el P/Invoke detrás de clases `internal`. El usuario nunca ve un `IntPtr`.
- [ ] Investigar: ¿por qué no conviene llamar `Py_Initialize` → `Py_Finalize` → `Py_Initialize` otra vez? (Pista: numpy y otras extensiones).
- [ ] Comparar tu API con la de CSnakes y pythonnet. ¿Qué te gusta más de cada una?

**Terminado cuando:** alguien que no conoce la API C de Python puede usar tu librería leyendo solo el README.

---

## Fase 9: Tests, CI y publicación

**Objetivo:** asegurarte de que nada se rompe y poder compartir la librería.

**Conceptos:** tests unitarios (xUnit), fixture compartida, GitHub Actions, NuGet.

**Ejercicios:**
- [ ] Crear un proyecto `PySharp.Tests` con xUnit.
- [ ] Usar un solo intérprete para todos los tests (Python no se puede reiniciar). Investigar "collection fixture" en xUnit.
- [ ] Tests para cada conversión de tipos de la fase 5 (ida y vuelta).
- [ ] GitHub Actions: compilar y correr tests en cada push (`actions/setup-dotnet` + `actions/setup-python`).
- [ ] Bonus: correr la CI en Windows, Linux y macOS.
- [ ] Empaquetar con `dotnet pack` y probar el `.nupkg` desde otro proyecto.

**Terminado cuando:** la CI está en verde y puedes instalar tu paquete en un proyecto nuevo.

---

## Fase 10 (opcional, avanzada): Generador de código

**Objetivo:** que a partir de `demo.py` se genere automáticamente una clase C# tipada, como hace CSnakes.

**Conceptos:** Roslyn, source generator incremental (`IIncrementalGenerator`), `AdditionalFiles`, type hints de Python, netstandard2.0.

**Ejercicios:**
- [ ] Hacer un source generator "hola mundo" que genere una clase vacía.
- [ ] Leer archivos `.py` como `AdditionalFiles`.
- [ ] Parsear firmas con type hints: `def add(a: int, b: int) -> int:`. Empezar simple y no intentar parsear todo Python.
- [ ] Generar `public long Add(long a, long b)` que por dentro usa tu librería.

**Terminado cuando:** escribes una función en Python y aparece en el IntelliSense de C#.

---

## Fase 11 (extras): Para ir más lejos

Elegir lo que te interese:
- [ ] **Callbacks:** que Python llame a una función de C#.
- [ ] **numpy sin copias:** buffer protocol (`Py_buffer`) → `Span<T>`.
- [ ] **Corrutinas:** convertir un `async def` de Python en un `Task` de C#.
- [ ] **Rendimiento:** medir con BenchmarkDotNet y comparar con CSnakes.
- [ ] **Native AOT:** que tu librería funcione compilada con AOT (por eso `LibraryImport`).
- [ ] **Python free-threaded** (3.13t): ¿qué cambia sin GIL?

---

## Recursos

- [Python/C API Reference](https://docs.python.org/3/c-api/index.html): la referencia principal. Cada función dice si devuelve una referencia nueva o prestada.
- [Embedding Python](https://docs.python.org/3/extending/embedding.html)
- [P/Invoke en .NET](https://learn.microsoft.com/dotnet/standard/native-interop/pinvoke)
- [Source generators](https://learn.microsoft.com/dotnet/csharp/roslyn-sdk/source-generators-overview)
- Código de referencia (después de intentarlo tú): [CSnakes](https://github.com/tonybaloney/CSnakes), [pythonnet](https://github.com/pythonnet/pythonnet)
