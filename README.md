# UDEE

API Java con Spring Boot para trabajar con clientes, domicilios, medidores,
marcas, modelos, tarifas, mediciones y facturas. El código se organiza en
controladores REST, servicios, repositorios JPA y entidades.

Este repositorio es el fork [tunonalves/UDEE](https://github.com/tunonalves/UDEE)
de [NicoRoldan28/UDEE](https://github.com/NicoRoldan28/UDEE). La rama de referencia
de este fork es `develop`; las contribuciones destinadas a este repositorio deben
abrir su PR aquí, no en el proyecto de origen por defecto.

## Tecnologías y requisitos

El [pom.xml](pom.xml) declara:

- Java 11 y Spring Boot 2.4.5.
- Spring Web, Data JPA, Validation y Security.
- MySQL Connector/J 8.0.21, Lombok y ModelMapper.
- JJWT, Actuator y Resilience4j, entre otras dependencias.
- Spring Boot Starter Test para las pruebas.

Se incluye Maven Wrapper para Linux/macOS (`mvnw`) y Windows (`mvnw.cmd`). Hace
falta un JDK 11 y acceso a los repositorios Maven al resolver dependencias.
La ejecución requiere preparar una base MySQL local compatible con las entidades;
la versión del conector no fija por sí sola la versión del servidor.

## Preparación local segura

```bash
git clone --branch develop git@github.com:tunonalves/UDEE.git
cd UDEE
sh mvnw --version
```

Antes de iniciar la aplicación o sus pruebas:

1. Preparar una base de datos de desarrollo aislada y un usuario con permisos
   limitados a ella. No reutilizar conexiones ni credenciales de producción.
2. Crear fuera del repositorio un archivo de configuración local de Spring Boot.
   Definir allí `spring.datasource.url`, `spring.datasource.username` y
   `spring.datasource.password` para esa base; revisar también el esquema y la
   política `spring.jpa.hibernate.ddl-auto` antes de permitir cambios de tablas.
3. Revisar los perfiles necesarios y la configuración de seguridad con el
   mantenedor. La presencia de Spring Security/JWT no certifica que la API sea
   apta para exponerse públicamente.
4. Revisar manualmente [bdd.sql](bdd.sql) antes de considerar su uso en una base
   desechable. No se propone importarlo automáticamente ni se garantiza que
   coincida con las entidades actuales. También hay modelos `modelo.mwb` y
   `schema.mwb` en la raíz.

Ejemplo de arranque **solo después de preparar la configuración local** (reemplazar
la ruta de ejemplo por el archivo propio existente):

```bash
sh mvnw spring-boot:run -Dspring-boot.run.arguments="--spring.config.location=file:/ruta/absoluta/udee-local.properties --server.address=127.0.0.1 --server.port=8080"
```

`spring.config.location` permite seleccionar explícitamente la configuración
externa en lugar de depender de los archivos incluidos. El ejemplo limita la
escucha a la máquina local. No publicar el archivo local ni secretos en Git.
No se reprodujeron valores de los archivos de configuración del repositorio.

## Rutas declaradas

Las siguientes rutas se obtienen de las anotaciones de
[src/main/java/com/api/UDEE/controller](src/main/java/com/api/UDEE/controller).
Son un inventario estático, no una comprobación de disponibilidad o autorización.

| Recurso | Alta (`POST`) | Listado (`GET`) | Consulta (`GET`) |
| --- | --- | --- | --- |
| Domicilios | `/api/address` | `/api/address` | `/api/address/{id}` |
| Facturas | `/api/bills` | `/api/bills` | `/api/bills/{id}` |
| Marcas | `/api/brands` | `/api/brands/brands` | `/api/brands/{id}` |
| Clientes | `/api/clients` | `/api/clients/clients` | `/api/clients/{id}` |
| Mediciones | `/api/measurements` | `/api/measurements/measurements` | `/api/measurements/{id}` |
| Medidores | `/api/meters` | `/api/meters/meters` | `/api/meters/{id}` |
| Modelos | `/api/models` | `/api/models/models` | `/api/models/{id}` |
| Tarifas | `/api/rates` | `/api/rates/rates` | `/api/rates/{id}` |
| Usuarios | `/users2` | `/users2` | `/users2/{id}` |

También se declara `POST /auth/login2`. Los segmentos repetidos de algunos
listados son intencionales en esta tabla: reflejan la combinación de las
anotaciones de clase y método, no una corrección de las rutas del código.
No se presuponen operaciones `PUT` o `DELETE` ni un CRUD completo.

Por ejemplo, los listados de facturas y mediciones reciben `Pageable`, devuelven
`X-Total-Count` y `X-Total-Pages`, y usan estado 204 si no hay contenido. Consultar
las entidades y cada controlador antes de construir un cuerpo JSON; no se
incluyen cuentas, contraseñas ni tokens de ejemplo.

## Organización

| Ruta | Contenido |
| --- | --- |
| [controller](src/main/java/com/api/UDEE/controller) | Rutas y respuestas HTTP. |
| [service](src/main/java/com/api/UDEE/service) | Lógica y acceso a repositorios. |
| [repository](src/main/java/com/api/UDEE/repository) | Persistencia JPA. |
| [domain](src/main/java/com/api/UDEE/domain) | Entidades, asociaciones y respuestas. |
| [dto](src/main/java/com/api/UDEE/dto) | Objetos de intercambio, incluidos los de login. |
| [config](src/main/java/com/api/UDEE/config) y [filter](src/main/java/com/api/UDEE/filter) | Configuración y filtro de autorización. |
| [src/test/java/com/api/UDEE](src/test/java/com/api/UDEE) | Pruebas de controladores, servicios y carga de contexto. |

## Compilación, pruebas y límites

Una vez revisado el entorno local, los objetivos Maven habituales son
`sh mvnw test` y `sh mvnw package`. Las pruebas incluyen `@SpringBootTest`, por lo
que pueden cargar configuración y necesitar servicios: seleccionar también una
configuración y base de pruebas aisladas antes de ejecutarlas. `package` ejecuta
las pruebas por defecto; `sh mvnw -DskipTests package` las omite explícitamente y
no debe presentarse como validación de su resultado. En Windows usar `mvnw.cmd`.

Esta revisión solo contrastó documentación con el POM, rutas y fuentes; no leyó
credenciales, no ejecutó Maven, no importó SQL, no consultó APIs ni conectó a bases
de datos. El arranque, las pruebas y la compatibilidad del esquema no están
verificados. El POM contiene declaraciones duplicadas de ModelMapper y
`javax.el`; no se modificaron dependencias en esta revisión de documentación.
