# Crear y configurar un microservicio

Esta sección reúne los comandos necesarios para crear un microservicio bajo el enfoque de Clean Architecture, incluyendo la configuración inicial y única del entorno de trabajo.

## Contenido

| Documento | Descripción |
|---|---|
| [Crear la solución general](1-solucion-general.md) | Configuración inicial y única: crea la solución que contiene todos los microservicios |
| [Crear un microservicio](2-microservicio.md) | Comandos para crear un microservicio por capa y referenciarlos entre sí |
| [Gestión de paquetes NuGet](3-paquetes-nuget.md) | Instalar, eliminar y listar paquetes; paquetes esenciales recomendados por capa |
| [Conexión a la base de datos](4-conexion-bd.md) | Configuración mínima del `DbContext`, `DependencyInjection` y la cadena de conexión en `Infrastructure` |
| [Registro de MediatR](5-registro-mediatr.md) | Configuración **única por microservicio** de la capa `Application` registrar **MediatR** y **AddAutoMapper** |

[⬅](./../README.md) Volver al índice principal | Empezar [➡](1-solucion-general.md)
