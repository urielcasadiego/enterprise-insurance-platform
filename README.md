# SLU Frontend

Aplicación web de **SLU Underwriters** para la gestión del ciclo de suscripción de reaseguros: cotizaciones, facultativos, endosos, cuentas, capacidad, catálogos y usuarios.

Está construida con Vue 2 y se comunica con la API SLU mediante GraphQL.

> El código fuente de producción es privado. Este repositorio contiene únicamente documentación del portafolio, diagramas de arquitectura y ejemplos depurados.

## Tecnología

- Vue 2.7, Vue Router 3 y Vuex 3.
- Vuetify 2 para los componentes de interfaz.
- Apollo Client / Vue Apollo para GraphQL y carga de archivos.
- Socket.IO para notificaciones en tiempo real.
- Vuelidate para validaciones y ApexCharts/Chart.js para visualizaciones.
- Jest y Vue Test Utils para pruebas unitarias.

## Requisitos

- Node.js compatible con Vue CLI 4 (se recomienda Node.js 16 LTS para este proyecto).
- npm.
- Una instancia accesible del backend SLU.

## Puesta en marcha

1. Instale las dependencias:

   ```bash
   npm install
   ```

2. Cree un archivo `.env.local` a partir del siguiente ejemplo. No incluya secretos ni tokens reales en archivos versionados:

   ```dotenv
   VUE_APP_API_URL=http://localhost:5001
   ```

3. Inicie el servidor de desarrollo:

   ```bash
   npm run serve
   ```

La aplicación suele quedar disponible en la URL que informe Vue CLI (por defecto `http://localhost:8080`). La API GraphQL se configura con `VUE_APP_API_URL`; el mismo valor se usa para la conexión Socket.IO.

## Comandos

| Comando | Descripción |
| --- | --- |
| `npm run serve` | Inicia el entorno de desarrollo. |
| `npm run serve:debug` | Alias del servidor de desarrollo. |
| `npm run build` | Genera el paquete de producción en `dist/`. |
| `npm test` | Ejecuta las pruebas unitarias. |
| `npm run test:unit` | Ejecuta Jest en modo CI. |

## Estructura principal

```text
src/
├── application/       # Componentes, servicios y mixins de aplicación
├── assets/            # Estilos, fuentes e imágenes globales
├── components/        # Componentes de la interfaz histórica
├── constants/         # Constantes y catálogos estáticos
├── lib/               # Cliente Apollo y utilidades de infraestructura
├── mixins/            # Comportamientos reutilizables
├── modules/           # Módulos de la interfaz actual
├── plugins/           # Configuración de Vue/Vuetify
├── router/            # Rutas globales y guardas de navegación
├── store/             # Estado global Vuex
├── utils/             # Utilidades puras
└── views/             # Vistas de la interfaz histórica
```

La evolución funcional nueva se concentra en `src/modules/home`. Allí cada dominio dispone normalmente de sus vistas, componentes, servicios y archivo de rutas. Consulte [ARCHITECTURE.md](ARCHITECTURE.md) para el detalle de capas y flujos.

## Autenticación y permisos

El token de sesión se guarda en `localStorage` bajo la clave `sessionToken`. El enrutador protege las rutas con `meta.requiresAuth`; además, antes de acceder a una vista protegida carga y valida los permisos de la vista y la acción requerida (`read`, `create`, `update` o `delete`).

El cliente GraphQL añade automáticamente `x-view-id` y `x-view-action` según la ruta activa. Por ello, toda ruta nueva que requiera autorización debe definir sus metadatos de permiso.

## Desarrollo

- Mantenga los componentes de dominio dentro de `src/modules/home/modules/<dominio>` cuando se trate de funcionalidad nueva.
- Centralice las operaciones GraphQL reutilizables en los servicios o acciones correspondientes; evite duplicar consultas dentro de componentes.
- No codifique URLs, credenciales ni tokens en el código fuente. Use variables `VUE_APP_*` para configuración de cliente.
- Antes de entregar cambios, ejecute `npm test` y `npm run build` cuando el entorno lo permita.

## Documentación relacionada

- [Arquitectura del frontend](ARCHITECTURE.md)
- Backend: `../api_dev_slu/server/` (documentación propia en ese proyecto)


# SLU API — servidor

Servidor backend de **SLU Underwriters**. Expone una API GraphQL para el ciclo operativo de reaseguros: usuarios y permisos, suscripciones, cotizaciones, endosos, facultativos, cuentas, pagos, catálogos, documentos y notificaciones.

El servidor vive en `server/`; los comandos npm se ejecutan desde la raíz del repositorio (`api_dev_slu`).

## Tecnología

- Node.js con módulos ECMAScript.
- Express y Apollo Server 3 para GraphQL.
- Sequelize con PostgreSQL.
- Socket.IO para eventos en tiempo real.
- AWS S3 para documentos y plantillas, ExcelJS para reportes y `node-cron` para tareas programadas.
- Vitest para pruebas.

## Requisitos

- Node.js 16 LTS (la canalización de despliegue usa Node 14; para desarrollo se recomienda validar la versión acordada por el equipo).
- npm.
- Acceso a una base de datos PostgreSQL y a los servicios externos requeridos.

## Configuración

Desde la raíz del proyecto, cree un archivo `.env` local. No versiona ni comparta valores reales de credenciales.

La conexión actual de Sequelize utiliza PostgreSQL y espera SSL. Para desarrollo local sin SSL, acuerde y aplique el ajuste de configuración correspondiente antes de ejecutar la aplicación.

## Ejecución

Instale dependencias y ejecute el servidor desde la raíz del repositorio:

```bash
npm install
npm run develop
```

El endpoint GraphQL está publicado en la raíz del servidor:

```text
http://localhost:5001/
```

Otros comandos disponibles:

| Comando | Descripción |
| --- | --- |
| `npm run start2` | Inicia directamente `server/server.js`. |
| `npm run develop` | Inicia el servidor con Nodemon. |
| `npm run build` | Transpila `server/` a `dist/`. |
| `npm start` | Instala, compila e inicia el artefacto `dist/`. |
| `npm test` | Inicia Vitest. |
| `npm run coverage` | Ejecuta Vitest con cobertura. |
| `npm run run:migrations` | Aplica migraciones de Sequelize. |
| `npm run down:migration` | Revierte la última migración. Úselo con precaución. |

## Estructura

```text
server/
├── application/              # Modelos, respuestas y repositorio Sequelize
├── assets/                   # Recursos de reportes y marca
├── config/                   # Carga de entorno y conexión de base de datos
├── jobs/                     # Procesos programados
├── middlewares/              # Autenticación, permisos, documentos y correo
├── modules/                  # Módulos GraphQL por dominio
├── reports/                  # Generación de reportes Excel
├── services/                 # Servicios de negocio compartidos
├── utils/                    # Utilidades puras
├── sockets.js                # Configuración Socket.IO
└── server.js                 # Arranque Express/Apollo
```

Cada módulo GraphQL suele incluir `typeDefs.js`, `queries.js`, `mutations.js` e `index.js`. Debe registrarse en `modules/index.js` para que su esquema y resolvers formen parte de la API.

## Desarrollo de módulos GraphQL

1. Cree o actualice el módulo bajo `server/modules/<dominio>/`.
2. Declare el contrato GraphQL en `typeDefs.js`.
3. Implemente resolvers de consultas y mutaciones.
4. Exporte `typeDefs` y `resolvers` desde `index.js`.
5. Registre el módulo en `server/modules/index.js`.
6. Pruebe el contrato en el endpoint GraphQL y añada pruebas cuando sea viable.

La autorización de vista se resuelve al construir el contexto GraphQL. Las solicitudes desde el frontend pueden incluir `x-view-id` y `x-view-action`; no se deben omitir las validaciones de autorización en cambios nuevos.

## Migraciones

Los modelos y migraciones se encuentran en `server/application/repository-slu/`. Antes de modificar el esquema de una base de datos compartida, coordine una copia de seguridad con el responsable técnico.

```bash
npm run generate:migration -- --name <nombre> -c "<descripcion>"
npm run run:migrations
```

Revise siempre el archivo de migración generado antes de aplicarlo. No elimine ni modifique el historial de migraciones en entornos compartidos.
