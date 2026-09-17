# Crear la solución general

Configuración inicial y única para preparar el espacio de trabajo: crea la solución que actuará como **contenedor de todos los microservicios del sistema**.

```bash
mkdir SolucionGeneral && cd SolucionGeneral
dotnet new sln -n SolucionGeneral
```

Esto genera el archivo `SolucionGeneral.slnx` en la raíz del espacio de trabajo. A partir de este punto, todos los microservicios y sus referencias se agregan y gestionan desde aquí.

> Este paso se ejecuta **una sola vez** por sistema, antes de crear cualquier microservicio individual.

[⬅](README.md) Volver al índice | Siguiente paso [➡](2-crear-microservicio.md)
