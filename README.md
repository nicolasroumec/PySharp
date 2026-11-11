**English** | [Español](README.es.md)

# PySharp

A library for running Python code from .NET 10 applications by embedding the CPython interpreter directly in the process.

This is a learning project inspired by [CSnakes](https://github.com/tonybaloney/CSnakes): the goal is to understand how embedding works under the hood by building it from scratch.

## How it works

Python exposes its interpreter as a native library (`python313.dll`) with a C API. PySharp loads that DLL and calls its functions from C# using P/Invoke, with no external dependencies.

## Requirements

- .NET SDK 10
- Python 3.13 (64-bit)

## Documentation

- [Roadmap](docs/ROADMAP.md): project phases with exercises (Spanish)
- [Glossary](docs/GLOSARIO.md): concepts explained simply (Spanish)

## Status

Work in progress, still in the early stages.
