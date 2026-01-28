# **Control de Pastillas (Pill Tracker)**
Para facilitar la vida a los enfermos en el control de tomas de los medicamentos

![captura de pantalla de la aplicación](./docs/captura-de-pantalla.png)  

## **Descripción General**

**Control de Pastillas** es una aplicación web full-stack ligera diseñada para la gestión y seguimiento de la toma de medicamentos diaria.  
Este proyecto destaca por su arquitectura minimalista y **"Zero-Dependency"** en el lado del servidor. El backend está construido íntegramente con la librería estándar de Python (http.server), eliminando la necesidad de frameworks pesados o gestores de paquetes. El frontend es una SPA (Single Page Application) reactiva que utiliza React y Tailwind CSS, servida directamente sin necesidad de pasos de compilación previos.  
El sistema permite la sincronización en tiempo real entre dispositivos dentro de la misma red, persistencia de datos basada en archivos JSON y un sistema de alertas visuales y auditivas.

## **Características Principales**

* **Arquitectura Ligera:** No requiere npm, node\_modules, ni librerías externas de Python (pip). Funciona "out-of-the-box".  
* **Sincronización Multi-dispositivo:** Los datos se sincronizan automáticamente entre dispositivos (móvil, tablet, PC) utilizando un ID de usuario único.  
* **Persistencia Híbrida:**  
  * **Remota:** Almacenamiento en servidor mediante archivos JSON planos.  
  * **Local:** Respaldo en localStorage para resiliencia ante fallos de red.  
* **Gestión de Tomas:**  
  * Control visual de estado (tomado/pendiente).  
  * Registro automático de la hora de la toma.  
  * Reinicio diario automático o manual.  
* **Sistema de Notificaciones:** Alarmas auditivas y notificaciones nativas del navegador configurables por toma.  
* **Interfaz Adaptativa:** Diseño *Mobile-First* responsivo construido con Tailwind CSS.  
* **Logging:** Sistema de registro de eventos en servidor (server.log) para auditoría y depuración.

## **Instalación y Despliegue**

### **Requisitos Previos**

* Python 3.6 o superior.  
* Navegador web moderno con soporte para ES6.

### **Pasos para ejecutar**

1. **Clonar el repositorio:**  
   git clone \[https://github.com/jsbsan/Control-de-Pastillas.git\](https://github.com/jsbsan/Control-de-Pastillas.git)  
   cd control-pastillas

2. **Iniciar el servidor:**  
   No es necesario instalar dependencias. Simplemente ejecuta:  
   python server.py

3. **Acceder a la aplicación:**  
   Abre tu navegador y visita:  
   * **Local:** http://localhost:8000  
   * **Red Local (Móvil):** http://\<TU\_IP\_LOCAL\>:8000 (Ej: 192.168.1.35:8000)

## **Credenciales y Autenticación**

Este sistema utiliza un modelo de autenticación implícito basado en **Tokens de Sincronización (User ID)**, diseñado para la simplicidad en entornos familiares o de red local.

* **Usuario por defecto:** Al acceder por primera vez, el sistema genera automáticamente un ID único (ej: user\_k92lx8m).  
* **Compartir sesión:** Para ver los mismos datos en otro dispositivo (ej: monitorizar desde el PC lo que se marca en el móvil):  
  1. Ve a la pestaña **Info** en el dispositivo origen.  
  2. Copia el "Código de Sincronización".  
  3. En el dispositivo nuevo, ve a **Info**, pulsa el icono de la **llave** e introduce el código.

**Nota:** No existen contraseñas. La seguridad radica en el conocimiento del ID de usuario (URL param ?id=).

## **Estructura del Proyecto**

El proyecto mantiene una estructura plana para facilitar el mantenimiento:  
/  
├── data/                  \# Directorio generado automáticamente para los JSON de usuarios  
│   └── user\_xxxx.json     \# Base de datos plana por usuario  
├── index.html             \# Frontend: Lógica React, Estilos y Markup (Todo en uno)  
├── server.py              \# Backend: API REST y Servidor de Archivos estáticos  
├── server.log             \# Log de actividad del servidor (generado en ejecución)  
└── readme.md              \# Documentación técnica

## **Tecnologías Utilizadas**

### **Backend**

* **Python 3:**  
  * http.server & socketserver: Para servir la aplicación y manejar peticiones HTTP.  
  * json: Serialización y almacenamiento de datos.  
  * logging: Registro de actividad.

### **Frontend**

* **React 18:** Renderizado de la interfaz de usuario.  
* **Babel Standalone:** Transpilación de JSX en tiempo de ejecución (in-browser).  
* **Tailwind CSS:** Framework de estilos utilitarios (vía CDN).   
* **API Fetch:** Comunicación asíncrona con el backend Python.  

## **Licencia**  

Este proyecto se distribuye bajo la licencia **GNU General Public License v3.0 (GPLv3)**.  
Eres libre de:  

* Usar el software para cualquier propósito.  
* Cambiar el software para adaptarlo a tus necesidades.    
* Compartir el software con tus amigos y vecinos.  

Manteniendo siempre la autoría original y liberando las modificaciones bajo la misma licencia.   
**Autor:** Julio Sánchez Berro  