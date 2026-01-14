# Documentación Técnica: Asistencia Técnica Hub

## 1. Resumen del Sistema
**Asistencia Técnica Hub** es una solución de gestión de colas de soporte diseñada para integrarse nativamente en **Microsoft Teams**. Permite a los usuarios internos solicitar asistencia técnica y a los agentes de IT gestionar las solicitudes mediante un panel centralizado con triaje asistido por Inteligencia Artificial.

---

## 2. Stack Tecnológico

### Frontend (Cliente)
- **Framework:** React 19 (ES6+).
- **Estilizado:** Tailwind CSS (Arquitectura Utility-first).
- **Iconografía:** Lucide React.
- **Bundler:** Vite.
- **Integración:** Microsoft Teams JavaScript SDK (v2.11.0).
- **Estado Global:** React Hooks (useState, useEffect, useCallback, useMemo).

### Backend (Servidor / API)
- **Entorno de Ejecución:** Node.js 20.x.
- **Arquitectura:** Azure Functions V4 (Serverless).
- **Lenguaje:** TypeScript.
- **IA:** Google Gemini API (`@google/genai`).
  - **Modelo:** `gemini-3-flash-preview`.
  - **Función:** Clasificación automática de prioridad, resumen de incidentes y categorización.

### Persistencia (Base de Datos)
- **Motor:** Azure SQL Database (MSSQL).
- **Driver de Conexión:** `mssql` (Node.js).

---

## 3. Infraestructura y Despliegue

### Hosting
- **Azure Static Web Apps:** 
  - Aloja el contenido estático del frontend.
  - Gestiona el escalado automático de las Azure Functions (API).
  - Integración continua (CI/CD) mediante **GitHub Actions**.

### Seguridad y Variables de Entorno
El sistema depende de las siguientes variables configuradas en el Portal de Azure (Application Settings):
- `SqlConnectionString`: Cadena de conexión cifrada hacia Azure SQL.
- `API_KEY`: Clave de acceso para Google Gemini API.

---

## 4. Arquitectura de Datos (Esquema SQL)

La tabla principal `requests` contiene los siguientes campos críticos:
| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `id` | VARCHAR | Identificador único del ticket (ej: T-1234). |
| `userId` | VARCHAR | UPN/Email del usuario en Teams. |
| `userName` | VARCHAR | Nombre para mostrar del usuario. |
| `subject` | VARCHAR | Título breve del problema. |
| `description` | TEXT | Detalles extendidos del incidente. |
| `status` | VARCHAR | Estados: `waiting`, `in-progress`, `completed`, `cancelled`. |
| `createdAt` | BIGINT | Timestamp de creación. |
| `startedAt` | BIGINT | Timestamp de cuando un agente toma el caso. |
| `completedAt` | BIGINT | Timestamp de cierre del ticket. |
| `priority` | VARCHAR | Nivel asignado por IA: `low`, `medium`, `high`. |
| `aiSummary` | TEXT | Resumen ejecutivo generado por Gemini. |
| `category` | VARCHAR | Categoría detectada (Hardware, Software, Redes, etc). |

---

## 5. Flujos de Trabajo Clave

### A. Triaje Automático
Cuando un usuario envía un ticket, el backend intercepta la solicitud y la envía a **Gemini 3 Flash**. La IA devuelve un objeto JSON que define la prioridad y la categoría sin intervención humana, optimizando el tiempo de respuesta del equipo de IT.

### B. Ciclo de Vida del Ticket
1. **Waiting:** El ticket entra en la cola general.
2. **In-Progress:** Un agente "Toma el caso". Se limpia el estado de espera y comienza el cronómetro de atención.
3. **Re-encolado:** Si el agente no puede resolverlo, puede devolverlo a la cola, reseteando los tiempos de atención.
4. **Finalización:** El ticket se mueve al historial como Resuelto o No Solucionado.

### C. Integración con Microsoft Teams
- **Contexto:** La app detecta automáticamente quién es el usuario logueado.
- **Deep Linking:** El botón "Contactar" utiliza el protocolo `msteams://` para abrir chats directos con mensajes pre-cargados, eliminando la necesidad de buscar manualmente al usuario.

---

## 6. Mantenimiento y Logs
- **Logs de Aplicación:** Disponibles a través de Azure Portal -> Monitor -> Log Stream.
- **Actualizaciones:** Cualquier cambio en la rama `main` de GitHub dispara automáticamente un despliegue hacia Azure.

---
*Documento actualizado el: ${new Date().toLocaleDateString()}*
