# PySharp

Librería para ejecutar código Python desde aplicaciones .NET 10, embebiendo el intérprete de CPython directamente en el proceso.

Es un proyecto de aprendizaje, inspirado en [CSnakes](https://github.com/tonybaloney/CSnakes): la idea es entender cómo funciona el embebido por dentro construyéndolo desde cero.

## Cómo funciona

Python expone su intérprete como una librería nativa (`python313.dll`) con una API en C. PySharp carga esa DLL y llama a sus funciones desde C# usando P/Invoke, sin dependencias externas.

## Requisitos

- .NET SDK 10
- Python 3.13 (64 bits)

## Roadmap

- [ ] Inicializar Python y ejecutar código (`Py_Initialize`, `PyRun_SimpleString`)
- [ ] Importar un módulo `.py` y llamar a una función
- [ ] Manejo de memoria: conteo de referencias con `SafeHandle` / `IDisposable`
- [ ] Conversión de tipos entre C# y Python (int, float, string, listas, diccionarios)
- [ ] Manejo de errores: excepciones de Python a excepciones de .NET
- [ ] Hilos y el GIL
- [ ] Encontrar la instalación de Python y usar entornos virtuales (venv)
- [ ] (Opcional) Generador de código: crear clases C# tipadas a partir de archivos `.py`

## Estado

En desarrollo, todavía en las primeras etapas.
