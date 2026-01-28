El sistema de guardado que he implementado utiliza una arquitectura **Cliente-Servidor** muy sencilla pero efectiva. Aquí te explico paso a paso cómo viaja la información desde tu dedo hasta el disco duro:

### 1. La "Llave": El Código de Usuario

Todo gira en torno al **Código de Sincronización** (ej. `user_x9s2`) que ves en la pestaña **Info**.

- Este código se genera aleatoriamente la primera vez que entras.
- Actúa como el nombre del archivo donde se guardarán tus datos.

### 2. El Proceso de Guardado (Paso a Paso)

1. **Acción en el Navegador (Frontend):** Cuando tocas una pastilla para marcarla como "Tomada", la aplicación web (`index.html`) empaqueta toda la información (qué pastillas tomaste, la fecha de hoy, las alarmas) en un paquete de texto formato **JSON**.
    
2. **Envío al Servidor:** La aplicación usa una función llamada `fetch` para enviar este paquete al servidor Python a través de la dirección `/api/pastillas?id=user_x9s2`. Es como enviar una carta por correo.
    
3. **Recepción en Python (Backend):** El script `server.py` recibe esta "carta".
    
    - Mira el remitente (`id=user_x9s2`).
    - Busca en su carpeta `data` si existe un archivo llamado `user_user_x9s2.json`.
    - Si existe, lo sobrescribe con la nueva información. Si no, lo crea.
4. **Almacenamiento Físico:** La información se queda guardada en tu disco duro (o en el del móvil) dentro de esa carpeta `data` como un archivo de texto plano. No es una base de datos compleja, es simplemente un archivo de texto que puedes abrir con el Bloc de Notas.
    

### 3. El Proceso de Carga (Lectura)

Cuando abres la aplicación de nuevo:

1. El navegador le pregunta al servidor: _"¿Tienes datos para el usuario `user_x9s2`?"_.
2. Python busca el archivo `data/user_user_x9s2.json`.
3. Python lee el texto del archivo y se lo envía de vuelta al navegador.
4. La aplicación web "pinta" las pastillas verdes o rojas según lo que diga ese archivo.

### Resumen Técnico

- **Protocolo:** HTTP (POST para guardar, GET para leer).
- **Formato de datos:** JSON.
- **Base de Datos:** Sistema de archivos local (archivos `.json` en la carpeta `data`).

Es un sistema muy robusto porque, al usar archivos simples, es muy fácil hacer copias de seguridad: solo tienes que copiar la carpeta `data` y tendrás todos los historiales de todos los usuarios a salvo.
