# Portal Académico | San Agustín Campus

Copia de portafolio de [Dayron Brus Rojas](https://github.com/DayronBrusRojas) del [proyecto original del equipo](https://github.com/Giron-Rosa/portal-academico). Se conservan el historial y los créditos de sus colaboradores.

Plataforma académica desarrollada en equipo para centralizar la gestión escolar y la comunicación entre alumnos, padres, docentes y administración. Integra seguimiento de cursos, tareas, notas y asistencia con mensajería, reportes, asistencia mediante IA y automatizaciones con n8n.

[Backend](backend) · [Frontend](frontend) · [Base de datos](database) · [Flujos n8n](n8n/workflows)

## Funcionalidades

| Perfil | Funciones principales |
| --- | --- |
| Alumno | Consultar cursos, materiales, actividades, asistencia y tareas; enviar y consultar entregas. |
| Padre o apoderado | Seguir el progreso académico y comunicarse con docentes; acceder a herramientas de acompañamiento. |
| Docente | Gestionar contenidos, tareas, evaluaciones, notas, asistencia y comunicados. |
| Administración | Gestionar estudiantes, docentes, padres, indicadores institucionales y módulos de caja y finanzas. |

### Comunicación, IA y automatización

- Mensajería y notificaciones en tiempo real con WebSockets/STOMP y SockJS, incluido intercambio de audio.
- Servicios de IA para acompañamiento académico, planes de apoyo y respuestas estructuradas que se presentan en la interfaz.
- Integración con una API compatible con el formato de chat de OpenAI; la configuración actual utiliza Gemini.
- Alertas por correo relacionadas con notas y asistencia mediante webhooks de n8n.
- Exportación de reportes y envío manual o automatizado de boletines Excel desde el portal administrativo.
- Componentes de gamificación y seguimiento de planes de apoyo.

## Tecnologías

| Capa | Tecnologías declaradas en el repositorio |
| --- | --- |
| Backend | Java 21, Spring Boot 3.3.5, Spring Data JPA, Maven |
| Seguridad | Spring Security y JWT |
| Frontend | Angular 21.2, TypeScript 5.9, RxJS |
| Base de datos | PostgreSQL |
| Comunicación | WebSockets, STOMP y SockJS |
| Reportes y documentos | Apache POI, OpenPDF y PDFBox |
| Automatización | n8n y correo SMTP |
| Entorno de ejecución | Docker y Docker Compose; también ejecución directa con Java y Node.js |

## Estructura

```text
backend/
  src/main/java/pe/sanagustin/portal/
    controller/    # API por perfil y dominio
    service/       # lógica académica, IA, reportes y automatización
    repository/    # acceso a datos
    entity/        # modelos persistentes
    security/      # autenticación JWT
    config/        # seguridad, inicialización y WebSockets
frontend/          # aplicación Angular
database/          # esquema, migraciones y datos de demostración
n8n/workflows/     # flujos exportados en JSON
docker-compose.yml
```

## Preparar el entorno

```bash
git clone https://github.com/DayronBrusRojas/portal-academico.git
cd portal-academico
```

Utiliza una base PostgreSQL propia de desarrollo. [application.yml](backend/src/main/resources/application.yml) y [docker-compose.yml](docker-compose.yml) contienen configuración de servicios remotos del equipo; ajústala antes del arranque. El backend incluye inicializadores de tablas y datos, por lo que las pruebas deben utilizar una base aislada.

| Configuración | Uso |
| --- | --- |
| `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD` | Conexión a PostgreSQL propia. |
| `JWT_SECRET` | Sobrescribe `jwt.secret` con una clave propia en Base64 de al menos 32 bytes al decodificarla. |
| `GEMINI_API_KEY` | Clave del proveedor configurado actualmente para IA. |
| `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD` | Cuenta y servidor SMTP propios. |
| `N8N_ALERT_WEBHOOK_URL` | URL del webhook de alertas, accesible desde el backend. |
| `TEST_EMAIL_RECIPIENT` | Destinatario de pruebas que controles. |
| `ENTORNO` | Modo utilizado por n8n; revisa el servicio y el flujo antes de enviar correos. |

Las variables se configuran en la terminal para la ejecución directa o en la configuración de los servicios para Docker Compose. Mantén las credenciales propias fuera del control de versiones.

### Base de datos y usuarios de demostración

[database](database) incluye `01_init.sql`, migraciones numeradas y scripts de datos de demostración. Revisa y prepara el esquema en una base vacía antes de iniciar el backend. Los scripts `50_*` en adelante contienen datos de prueba y ajustes; `consolidated_neon.sql` es otro recurso del proyecto que debe revisarse antes de utilizarlo.

Las cuentas de demostración se definen en los scripts SQL y en los [inicializadores del backend](backend/src/main/java/pe/sanagustin/portal/config). Comprueba qué cuentas se crearon en tu entorno local.

## Ejecutar con Docker Compose

Abre Docker Desktop manualmente. Antes del primer arranque, revisa `environment` en `docker-compose.yml`: algunos valores de conexión están escritos directamente y no se sustituyen solo por crear un `.env`. El Compose actual define frontend, backend y n8n; PostgreSQL debe estar disponible por separado.

Después de configurar tus servicios:

```bash
docker compose up -d --build
```

| Servicio | Dirección local |
| --- | --- |
| Frontend | http://localhost:4200 |
| Backend | http://localhost:8080 |
| n8n | http://localhost:5678 |

PostgreSQL utiliza la dirección de tu instancia; el Compose actual no publica un servicio PostgreSQL local.

## Ejecutar sin Docker

Requiere JDK 21, Maven, Node.js y npm compatibles con Angular 21.2, además de PostgreSQL configurado. Define primero las variables del backend; Maven no carga automáticamente un `.env`.

```bash
# Primera terminal, desde la raíz del proyecto
cd backend
mvn spring-boot:run
```

```bash
# Otra terminal, desde la raíz del proyecto
cd frontend
npm install
npm start
```

El frontend utiliza varias rutas hacia `http://localhost:8080`. Si cambias el host o puerto del backend, revisa los servicios Angular y la conexión WebSocket. Si n8n se ejecuta fuera de Compose, configura un `N8N_ALERT_WEBHOOK_URL` accesible desde Java.

## Automatizaciones

Los flujos exportados están en [n8n/workflows](n8n/workflows):

- [Alertas a padres](n8n/workflows/alerta_padres.json).
- [Boletines de notas en Excel](n8n/workflows/boletin_notas_excel.json).

Impórtalos en tu instancia n8n, asigna tus credenciales y revisa conexiones de datos, destinatarios y URLs de webhook antes de activarlos. Para la integración del backend utiliza la URL de producción del webhook correspondiente al flujo publicado.

## Verificación

| Ubicación | Comando | Requisitos adicionales |
| --- | --- | --- |
| `backend` | `mvn test` | Base de pruebas y configuración propia. |
| `frontend` | `npm run build` | Dependencias instaladas. |
| `frontend` | `npm test` | Entorno compatible con el ejecutor de pruebas. |

## Autoría y aportes

Proyecto colaborativo mantenido en el repositorio de [Rosa](https://github.com/Giron-Rosa), con contribuciones de **Rosa**, **Sergio Mateo** y **Dayron Brus Rojas** registradas en Git.

Entre los aportes de [Dayron Brus Rojas](https://github.com/DayronBrusRojas) figuran integraciones de IA, mensajería y automatización. Ejemplos verificables:

- [Integración de n8n y alertas por notas y asistencia](https://github.com/Giron-Rosa/portal-academico/commit/40dd52e1fe195db6f102e1ba5f3f5184d11e70b5).
- [Envío manual y automatizado de reportes Excel](https://github.com/Giron-Rosa/portal-academico/commit/2575f12446076277f00d2abac968b678fa66ab22).

Estos aportes forman parte del trabajo conjunto. Consulta el [historial de contribuciones](https://github.com/Giron-Rosa/portal-academico/graphs/contributors) para revisar la autoría completa.
