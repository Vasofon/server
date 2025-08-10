# **Vasofón Server: Backend de Intercomunicador Familiar**

Este repositorio contiene el código fuente del backend para la aplicación de intercomunicación familiar **Vasofón**. El servidor es el corazón del proyecto, responsable de gestionar los grupos familiares, la lógica de registro de dispositivos, la comunicación en tiempo real y la funcionalidad del **Bot Familiar**.

El servidor está construido con un enfoque moderno y asincrónico para garantizar un alto rendimiento y una respuesta rápida.

### **Servicios (API Endpoints)**

El servidor proporciona una API REST y endpoints de WebSocket para gestionar toda la lógica de la aplicación. Todos los endpoints que gestionan recursos protegidos requieren un token de autenticación válido en la cabecera Authorization: Bearer \<auth\_token\>.

#### **Endpoints de Autenticación y Grupos**

* POST /groups: Crea un nuevo grupo familiar con un nombre elegido por el usuario, registra el primer dispositivo y usuario, y devuelve un token de autenticación.  
  * **Payload:** { "user\_name": "string", "group\_name": "string", "device\_id": "string" }  
  * **Returns:** { "group\_id": "string", "auth\_token": "string" }  
  * **Excepción:** Si el group\_name ya existe, la petición fallará.  
* POST /groups/{group\_id}/members: Inicia el proceso de unirse a un grupo existente. Envía una solicitud de confirmación a los miembros actuales del grupo a través del Bot Familiar.  
  * **Payload:** { "device\_id": "string", "device\_name": "string" }  
* GET /auth/token/{device\_id}: El dispositivo solicitante obtiene su token de autenticación una vez que ha sido confirmado por otro miembro del grupo. El token se genera solo si el dispositivo ha sido habilitado.  
  * **Returns:** { "auth\_token": "string" }  
* GET /groups/{group\_id}: Permite a un miembro del grupo ver el estado actual del grupo y sus propias credenciales.  
* GET /groups/{group\_id}/members: Lista todos los miembros del grupo. Este endpoint solo es accesible para miembros validados del grupo.

#### **Endpoints de Mensajería (Requieren Token de Autenticación)**

Para todos los endpoints de mensajería, el cliente debe incluir su token en la cabecera Authorization: Bearer \<auth\_token\>.

* POST /groups/{group\_id}/members/{recipient\_id}/messages: Envía un mensaje de texto a un miembro específico o inicia un broadcast. El tipo de mensaje (texto o acción especial) se define en el payload.  
  * **Payload:** { "content": "string", "action": "string" }  
  * El campo action es opcional. Si se envía con el valor "chancla", el servidor ejecutará la acción especial en lugar de un mensaje de texto normal.  
* GET /groups/{group\_id}/members/{user\_id}/messages: Recibe el historial de mensajes para el usuario en el grupo especificado.

#### **Endpoints de WebSocket (Requieren Token de Autenticación)**

* websocket /ws/{group\_id}/{auth\_token}: Establece una conexión en tiempo real con un dispositivo usando el token para la autenticación. Utilizado para el envío instantáneo de mensajes de broadcast y directos.

### **Tecnologías Utilizadas**

* **Lenguaje:** Python 3.9+  
* **Framework:** **FastAPI**  
* **Servidor Web:** **Uvicorn**  
* **Base de Datos:** **PostgreSQL**  
* **ORM:** SQLAlchemy  
* **Despliegue:** **GCP App Engine**  
* **Notificaciones Push:** **Firebase Cloud Messaging (FCM)**  
* **Comunicación en Tiempo Real:** **WebSockets** (gestionado por FastAPI)

### **Dependencias**

Para ejecutar el servidor localmente, necesitas instalar las siguientes dependencias de Python.

fastapi  
uvicorn\[standard\]  
psycopg2-binary \# Adaptar si se usa otro cliente para PostgreSQL  
sqlalchemy  
python-socketio  
firebase-admin

### **Cómo Ejecutar (Desarrollo Local)**

1. Clona el repositorio:  
   git clone https://github.com/Vasofon/vasofon-server.git  
   cd vasofon-server  
2. Crea un entorno virtual e instala las dependencias:  
   python \-m venv venv  
   source venv/bin/activate \# En Windows usa venv\\Scripts\\activate  
   pip install \-r requirements.txt  
3. **Configura las variables de entorno:** Necesitarás una conexión a la base de datos, las credenciales de Firebase y una clave secreta para firmar los tokens.  
4. Ejecuta el servidor:  
   uvicorn main:app \--reload

El servidor se iniciará en http://127.0.0.1:8000.

### **Licencia**

Este proyecto está bajo la licencia **GNU Affero General Public License (AGPL)**.
