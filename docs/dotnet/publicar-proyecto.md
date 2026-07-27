# Publicar un microservicio (`dotnet publish`)

## Tabla de contenido
- [¿Qué es?](#qué-es)
- [¿Para qué sirve?](#para-qué-sirve)
- [Pasos para generar el artefacto de publicación](#pasos-para-generar-el-artefacto-de-publicación)
- [Limpiar artefactos anteriores](#limpiar-artefactos-anteriores)
- [Buenas prácticas](#buenas-prácticas)
- [Errores comunes](#errores-comunes)
- [Recursos relacionados](#recursos-relacionados)

## ¿Qué es?

`dotnet publish` es el comando que genera una versión **compilada y lista para desplegar** de un proyecto .NET, incluyendo todos los binarios y dependencias necesarias para ejecutarlo en un servidor, sin necesidad de tener instalado el SDK completo de .NET.

## ¿Para qué sirve?

Es el paso previo obligatorio antes de subir un microservicio a cualquier entorno de hosting, como **Azure App Service**, un contenedor Docker, o un servidor IIS.

## Pasos para generar el artefacto de publicación

### 1. Ubicarse en el proyecto de la API

```bash
cd ./MicroservicioUsuarios/API
```

### 2. Limpiar, compilar y publicar en orden

```bash
dotnet clean
dotnet build
dotnet publish -c Release -o ./publish
```

| Comando | Propósito |
|---|---|
| `dotnet clean` | Elimina los artefactos de compilaciones anteriores |
| `dotnet build` | Compila el proyecto y valida que no existan errores |
| `dotnet publish -c Release -o ./publish` | Genera el paquete final en configuración `Release`, en la carpeta `./publish` |

### 3. Comprimir la carpeta generada

```bash
# En PowerShell
Compress-Archive -Path ./publish/* -DestinationPath publish.zip

# En bash/Linux
cd publish && zip -r ../publish.zip . && cd ..
```

El archivo `.zip` resultante es el que se sube al servicio de hosting (ver [Despliegue en Azure App Service](../azure/app-service-deployment.md)).

## Limpiar artefactos anteriores

Antes de generar una nueva publicación, es recomendable eliminar la carpeta `publish` previa:

```bash
# Linux/macOS
rm -rf ./publish

# PowerShell
Remove-Item -Recurse -Force ./publish -ErrorAction SilentlyContinue
```

## Buenas prácticas

- Publica siempre en configuración `Release`, nunca `Debug`, para entornos productivos.
- Automatiza este flujo (`clean` → `build` → `publish` → `compress`) en un script o en un pipeline de CI/CD para evitar errores manuales.
- Verifica el contenido del `.zip` antes de subirlo, confirmando que incluya el archivo `.dll` principal y el `appsettings.json` correspondiente al entorno.

## Errores comunes

- ❌ Comprimir la carpeta `publish` completa en lugar de su **contenido** (`./publish/*`), lo que genera una carpeta anidada extra dentro del `.zip` y puede romper el despliegue.
- ❌ Olvidar ejecutar `dotnet clean` antes de publicar, arrastrando artefactos de una versión anterior.
- ❌ Publicar en configuración `Debug` en un entorno productivo.

## Recursos relacionados

- [Despliegue en Azure App Service](../azure/app-service-deployment.md)
- [Entity Framework Core: migraciones](entity-framework-migraciones.md)

[⬅ Volver al índice de .NET](README.md)
