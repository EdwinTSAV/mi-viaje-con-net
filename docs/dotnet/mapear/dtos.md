# DTOs de respuesta (ejemplo Persona)

## Tabla de contenido
- [¿Para qué sirve?](#para-qué-sirve)
- [1. Crear el DTO de respuesta](#1-crear-el-dto-de-respuesta)
- [Buenas prácticas](#buenas-prácticas)
- [Errores comunes](#errores-comunes)
- [Recursos relacionados](#recursos-relacionados)

## ¿Para qué sirve?

Hasta ahora, las Queries de [Crear un CRUD](crear-crud.md) devuelven directamente la entidad de dominio (`Persona`) al controlador. Esto funciona, pero acopla el contrato de la API a los detalles internos de persistencia: cualquier cambio en la entidad (agregar una columna interna, renombrar una propiedad por una migración) rompe automáticamente la respuesta que reciben los consumidores de la API.

Un **DTO (Data Transfer Object)** de respuesta resuelve esto: define explícitamente qué campos se exponen hacia afuera, independientemente de cómo esté modelada la entidad por dentro.

> **Nota**
> Este documento solo cubre la creación del DTO como clase. Cómo convertir la entidad en ese DTO automáticamente se documenta por separado en [Mapeo con AutoMapper](automapper.md).

> **Nota**
> No se crea un DTO de "entrada" (`Request`) para `Crear`/`Actualizar`, porque en este proyecto los propios `Command` (`CrearPersonaCommand`, `ActualizarPersonaCommand`) ya cumplen esa función: son el objeto que el controlador recibe del cliente. Solo se agrega DTO para lo que **sale** de la API (`Response`), que es donde se filtra qué expone la entidad de dominio.

## 1. Crear el DTO de respuesta

En `Application/DTOs/PersonaResponse.cs`:

```csharp
namespace MiProyecto.Application.DTOs;

public class PersonaResponse
{
    public Guid Id { get; set; }
    public string NroDni { get; set; } = string.Empty;
    public string PrimerNombre { get; set; } = string.Empty;
    public string ApellidoPaterno { get; set; } = string.Empty;
    public string ApellidoMaterno { get; set; } = string.Empty;
    public DateOnly FechaNacimiento { get; set; }
}
```

> **Tip**
> En este ejemplo `PersonaResponse` tiene los mismos campos que la entidad `Persona`, así que el beneficio no se nota a simple vista. La diferencia se hace evidente cuando la entidad crece con campos que **no** deben salir por la API (por ejemplo, un campo interno de auditoría o una clave foránea técnica): el DTO solo expone lo que tú decidas.

## Buenas prácticas

- Ubica todos los DTOs de una capa `Application` bajo una única carpeta `DTOs/`, con un archivo por cada uno (`PersonaResponse.cs`, `PedidoResponse.cs`, etc.).
- Nombra los DTOs de respuesta con el sufijo `Response` (o `Dto`, elige uno y mantenlo consistente en todo el proyecto).
- No agregues comportamiento (métodos, validaciones) dentro de un DTO: es una estructura de datos plana, sin lógica.

## Errores comunes

- ❌ Definir el DTO de respuesta dentro de `Domain` o `Infrastructure`: los DTOs son un concepto de `Application` (o incluso de `API`), nunca del dominio.
- ❌ Reutilizar el mismo DTO de respuesta como DTO de entrada para `Crear`/`Actualizar`: mezclar ambos casos suele terminar con campos opcionales que no deberían serlo (por ejemplo, `Id` en un DTO de creación).

## Recursos relacionados

- [Mapeo con AutoMapper](automapper.md)
- [Crear un CRUD](crear-crud.md)
- [Crear una entidad](crear-entidad.md)

[⬅ Volver al índice de .NET](README.md)
