## Manual Técnico: Sistema Control de Pastillas (PillTracker)

Versión: 1.0.0

Fecha: 28 Enero 2026

Autor: Senior Technical Documentation Team

1. Introducción y Stack Tecnológico

Propósito del Software

El sistema PillTracker es una solución ligera de gestión de medicación personal. Permite a los usuarios registrar tomas diarias, configurar alarmas sonoras personalizadas mediante la API de Web Audio y sincronizar el estado entre dispositivos utilizando un backend minimalista basado en el sistema de archivos.

Stack Tecnológico

El proyecto sigue una arquitectura monolítica desacoplada, donde el servidor entrega la aplicación frontend y gestiona una API RESTful básica.

Backend:

Lenguaje: Python 3.x

Framework: http.server (Standard Library) para manejo de peticiones HTTP.

Persistencia: Sistema de archivos (JSON Flat Files).

Logging: Módulo logging nativo.

Frontend:

Framework: React 18 (vía CDN, sin build-step).

Transpilación: Babel Standalone (in-browser).

Estilos: Tailwind CSS (vía CDN).

Audio: Web Audio API nativa (Osciladores para generación de alarmas).

Almacenamiento Local: localStorage para caché e identificación de usuario.

2. Arquitectura del Sistema

El sistema utiliza un patrón Cliente-Servidor. El servidor actúa como un despachador de archivos estáticos y como una API JSON. No existe una capa de base de datos relacional; la persistencia se maneja mediante serialización directa de objetos a archivos JSON en el disco del servidor.

Diagrama de Arquitectura de Alto Nivel

Este diagrama ilustra la interacción entre el cliente (navegador), el servidor de aplicaciones Python y la capa de almacenamiento.

``` mermaid
graph TD
    subgraph "Cliente (Navegador)"
        UI["Interfaz React/HTML"]
        Logic["Lógica de Negocio JS"]
        Audio["Web Audio API"]
        Cache["LocalStorage"]
    end

    subgraph "Servidor (Python Host)"
        PyServer["Python http.server"]
        Router["Enrutador Simple"]
        Logger["Sistema de Logs"]
    end

    subgraph "Persistencia (File System)"
        DataDir["Directorio /data"]
        JSONFiles["Archivos user_id.json"]
    end

    UI --> Logic
    Logic -->|Polling Fetch 5s| PyServer
    Logic -->|Play Alarm| Audio
    Logic <--> Cache
    
    PyServer --> Router
    Router -->|GET/POST| DataDir
    DataDir <--> JSONFiles
    Router -.->|Write| Logger
``` 

3. Guía de Configuración (Setup)

Requisitos Previos

Python 3.8 o superior instalado en el entorno.

Navegador web moderno con soporte para ES6 y Web Audio API.

Conexión a internet (para cargar librerías CDN: React, Tailwind, Babel).

Instalación y Ejecución

Clonar el repositorio:

git clone <url-repositorio>
cd pill-tracker


Estructura de Directorios:
Asegúrese de que server.py e index.html estén en la raíz.

/
├── index.html
├── server.py
└── data/ (Se creará automáticamente)


Iniciar el Servidor:
Ejecute el script de Python. Esto levantará el servicio en el puerto 8000 por defecto.

python server.py


Salida esperada:
💊 Servidor Python corriendo en http://localhost:8000

Acceso:
Abra http://localhost:8000 en su navegador.

4. Documentación de la API

La API no sigue estrictamente REST, pero utiliza verbos HTTP para operaciones CRUD sobre el archivo JSON del usuario.

Endpoints

1. Obtener Datos del Usuario

Recupera el estado de las tomas y configuración del día actual.

Método: GET

URL: /api/pastillas

Parámetros Query:

id (string, requerido): Identificador único del usuario.

Respuesta Exitosa (200 OK):

{
  "date": "Mon Jan 28 2026",
  "doses": [
    {
      "id": 1,
      "label": "Toma 1",
      "taken": false,
      "time": null,
      "alarmTime": "14:00",
      "alarmEnabled": true
    }
  ],
  "lastUpdated": "2026-01-28T10:00:00.000Z"
}


2. Guardar/Actualizar Datos

Sobrescribe el estado completo del usuario.

Método: POST

URL: /api/pastillas?id={user_id}

Body (JSON): Estructura idéntica a la respuesta del GET.

Diagrama de Secuencia (Sincronización):
``` mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Frontend (React)
    participant API as Python Server
    participant FS as File System

    U->>FE: Marca toma como "Realizada"
    FE->>FE: Actualiza State Local
    FE->>API: POST /api/pastillas?id=xyz (JSON Payload)
    activate API
    API->>API: Valida JSON
    API->>FS: open("data/user_xyz.json", "w")
    FS-->>API: Confirmación escritura
    API->>API: Log Action (INFO)
    API-->>FE: 200 OK {status: "success"}
    deactivate API
    FE->>FE: Limpia flag de sincronización

``` 

5. Modelo de Datos (Persistencia)

Dado que utilizamos archivos JSON, no existe un esquema relacional estricto (ERD), pero sí una estructura de documento definida. A continuación se muestra el esquema del objeto almacenado.

Esquema del Archivo JSON (Document Structure)
``` mermaid
classDiagram
    class UserFile {
        +String date "Key de fecha (TodayString)"
        +String lastUpdated "ISO Timestamp"
        +List~Dose~ doses
    }

    class Dose {
        +Integer id
        +String label "Nombre de la toma"
        +Boolean taken
        +String time "Hora de la toma (null si no tomada)"
        +String alarmTime "HH:MM"
        +Boolean alarmEnabled
    }

    UserFile *-- Dose : contiene 1..N
``` 

6. Diagrama de Flujo del Programa (Frontend Logic)

Este diagrama detalla el ciclo de vida de la aplicación React, desde la carga inicial hasta el bucle de comprobación de alarmas.
``` mermaid
flowchart LR
    A[Inicio App] --> B{"¿Existe UserID?"}
    B -- No --> C[Generar ID aleatorio
     y 
     guardar en LocalStorage]
    B -- Si --> D[Leer ID de LocalStorage]
    
    C --> E
    D --> E["Fetch GET /api/pastillas"]
    
    E --> F{"¿Respuesta OK?"}
    F -- No/Error --> G[Cargar backup
     de LocalStorage]
    F -- Si --> H{"¿Misma Fecha que Hoy?"}
    
    H -- No --> I["Reiniciar tomas (Reset Day)"]
    I --> J[Guardar nuevo estado
     en Servidor]
    H -- Si --> K["Setear Estado (setDoses)"]
    
    K --> L[Renderizar UI]
    
    subgraph "Bucle de Eventos
     (useEffect)"
        M[Intervalo 5 seg] --> N[Obtener Hora Actual]
        N --> O{"¿Coincide con AlarmTime?"}
        O -- Si y !Taken y !Sonado --> P["Disparar Audio (Oscilador)"]
        P --> Q[Mostrar Notificación Nativa]
        O -- No --> M
    end
    L --> M
``` 

7. Flujo de CI/CD (Propuesto)

Aunque el proyecto actual es local, para un entorno de producción se recomienda el siguiente flujo de integración y despliegue continuo.
``` 
flowchart LR
    Dev[Desarrollador] -->|Git Push| Repo[Repositorio Git]
    
    subgraph "CI Pipeline"
        Repo --> Lint["Linting (Black/ESLint)"]
        Lint --> Test["Unit Tests (PyTest)"]
    end
    
    subgraph "CD Pipeline"
        Test --> Build[Docker Build]
        Build --> Reg[Docker Registry]
        Reg --> Deploy[Deploy to Server]
    end
    Deploy -->|Restart Service| Prod[Producción]
``` 

8. Guía de Contribución

Estándares de Código

Backend (Python):

Seguir PEP 8 para estilo de código.

Manejo de errores: Usar bloques try-except en operaciones de I/O y devolver siempre JSON válido en caso de error 500.

Logging: Nunca usar print() en producción, usar self.log_action() o logging.

Frontend (React):

Single File Component: Dado que estamos en un solo archivo HTML, mantener el componente `App