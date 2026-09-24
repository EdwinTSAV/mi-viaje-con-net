# Crear la solución general

Configuración inicial y única para preparar el espacio de trabajo: crea la solución que actuará como **contenedor de todos los microservicios del sistema**.

```bash
mkdir SolucionGeneral && cd SolucionGeneral
dotnet new sln -n SolucionGeneral
```

Esto genera el archivo `SolucionGeneral.slnx` en la raíz del espacio de trabajo. A partir de este punto, todos los microservicios y sus referencias se agregan y gestionan desde aquí.

> **Nota**
> Este paso se ejecuta **una sola vez** por sistema, antes de crear cualquier microservicio individual.

> **Nota**
> El formato del archivo depende del SDK: en .NET 10 `dotnet new sln` genera `.slnx` por defecto; en versiones anteriores genera `.sln`. Verifica tu versión con `dotnet --version`. Si tu solución quedó como `.sln`, usa esa extensión en los comandos de los siguientes documentos.

[⬅](README.md) Volver al índice | Crear un microservicio [➡](2-microservicio.md)
