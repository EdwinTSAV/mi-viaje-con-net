# Crear la solución general con .NET CLI

Configuración inicial y única para preparar el espacio de trabajo: crea la solución que actuará como **contenedor de todos los microservicios del sistema**.

```bash
mkdir SolucionGeneral && cd SolucionGeneral
dotnet new sln -n SolucionGeneral
```

Esto genera el archivo `SolucionGeneral.slnx` en la raíz del espacio de trabajo. A partir de este punto, todos los microservicios y sus referencias se agregan y gestionan desde aquí (ver [Crear la estructura de un microservicio](2-crear-microservicio.md)).

> Este paso se ejecuta **una sola vez** por sistema, antes de crear cualquier microservicio individual.

⬅ [Volver al índice](README.md) | [Siguiente paso](2-crear-microservicio.md) ➡
