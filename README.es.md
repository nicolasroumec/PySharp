[English](README.md) | **Español**

# PySharp

Librería para ejecutar código Python desde aplicaciones .NET 10, embebiendo el intérprete de CPython directamente en el proceso.

Es un proyecto de aprendizaje, inspirado en [CSnakes](https://github.com/tonybaloney/CSnakes): la idea es entender cómo funciona el embebido por dentro construyéndolo desde cero.

## Cómo funciona

Python expone su intérprete como una librería nativa (`python313.dll`) con una API en C. PySharp carga esa DLL y llama a sus funciones desde C# usando P/Invoke, sin dependencias externas.

## Requisitos

- .NET SDK 10
- Python 3.13 (64 bits)

## Documentación

- [Roadmap](docs/ROADMAP.md): fases del proyecto con ejercicios
- [Glosario](docs/GLOSARIO.md): conceptos explicados en simple

## Estado

En desarrollo, todavía en las primeras etapas.
