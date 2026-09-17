# Microservicios con .NET

Guía de estudio y consulta rápida para el diseño, construcción y despliegue de microservicios en **.NET** siguiendo **Clean Architecture**, con ejemplos de comandos de **.NET CLI**, **Entity Framework Core** y despliegue en **Azure App Service**.

---

## Índice

### Arquitectura
- Introducción a la arquitectura [➡](docs/arquitectura/README.md)
- Microservicios [➡](docs/arquitectura/microservicios.md)
- Clean Architecture y arquitectura por capas [➡](docs/arquitectura/clean-architecture.md)
- Estructura de carpetas por capa [➡](docs/arquitectura/estructura-carpetas.md)

### .NET
- Introducción [➡](docs/dotnet/README.md)
- Estructura de un microservicio [➡](docs/dotnet/estructura/README.md)
- Interacción con la base de datos [➡](docs/dotnet/database/README.md)
- Preparar proyecto [➡](docs/dotnet/preparar-proyecto.md)
- Crear una entidad (ejemplo Persona) [➡](docs/dotnet/crear-entidad.md)
- Publicar un microservicio [➡](docs/dotnet/publicar-proyecto.md)

### Azure
- Introducción [➡](docs/azure/README.md)
- Despliegue en Azure App Service [➡](docs/azure/app-service-deployment.md)

---

1. **Principios de Clean Architecture** y distribución de las responsabilidades entre capas.
2. **Crear la solución y los proyectos** de cada microservicio con `dotnet CLI`.
3. Configurar las **dependencias (NuGet)** necesarias por capa.
4. Configurar la **conexión a la BD**.
5. Trabajar con **Entity Framework Core** para generar y aplicar migraciones.
6. Crear un microservicio.
7. **Publicar y desplegar**.
