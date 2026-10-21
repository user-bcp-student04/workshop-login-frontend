# PLAN

## Alcance seleccionado

Como no se indicaron IDs concretos en el prompt, este plan toma por defecto todos los items con estado Pendiente en HALLAZGOS.md: H-01, H-02, H-03, H-04, H-05, H-06 y H-07.

Se preserva el contrato vigente de docs/ENDPOINTS.md.

## Item: H-01 | SQL concatenado en autenticacion

- Item de HALLAZGOS.md: H-01.
- Cambio propuesto: reemplazar la consulta construida por concatenacion por una consulta parametrizada con PreparedStatement y constantes con nombre para evitar strings SQL magicos.
- Archivos a tocar:
  - src/main/java/com/bcp/workshop/auth/LegacyDatabase.java
  - src/test/java/com/bcp/workshop/auth/AuthResourceTest.java
- Prueba a escribir antes de tocarlo:
  - Prueba de caracterizacion de login con intento de inyeccion en username/password que hoy debe terminar en 401 con el contrato actual.
  - Prueba de control para login valido (200) y login invalido normal (401).

## Item: H-02 | Contrasenas en texto plano en persistencia

- Item de HALLAZGOS.md: H-02.
- Cambio propuesto: migrar almacenamiento/verificacion de contrasenas a hash lento con salt (BCrypt), sin exponer valores sensibles en logs.
- Archivos a tocar:
  - src/main/java/com/bcp/workshop/auth/LegacyDatabase.java
  - src/main/java/com/bcp/workshop/auth/AuthResource.java
  - src/main/resources/application.properties (si se parametriza costo de hash)
  - src/test/java/com/bcp/workshop/auth/AuthResourceTest.java
- Prueba a escribir antes de tocarlo:
  - Caracterizacion de login exitoso y fallido por password incorrecta.
  - Caracterizacion del body 200 para verificar que mantiene token, expiresIn, user y debug.

## Item: H-03 | Token reversible (Base64) con secreto embebido

- Item de HALLAZGOS.md: H-03.
- Cambio propuesto: reemplazar token reversible por token firmado con expiracion real y claims explicitos (usuario y rol), con validacion de firma en el componente de seguridad.
- Archivos a tocar:
  - src/main/java/com/bcp/workshop/auth/LegacyTokenUtil.java (o su reemplazo en capa de seguridad)
  - src/main/java/com/bcp/workshop/auth/AuthResource.java
  - src/main/resources/application.properties
  - src/test/java/com/bcp/workshop/auth/AuthResourceTest.java
- Prueba a escribir antes de tocarlo:
  - Caracterizacion de login 200 validando contrato externo (token no nulo, expiresIn, user).
  - Prueba que preserve la estructura exacta de la respuesta documentada en docs/ENDPOINTS.md.

## Item: H-04 | Secreto y CORS hardcodeados en configuracion

- Item de HALLAZGOS.md: H-04.
- Cambio propuesto: externalizar secreto a configuracion (variable de entorno/propiedad) y restringir CORS a origenes configurables por ambiente.
- Archivos a tocar:
  - src/main/resources/application.properties
  - src/main/java/com/bcp/workshop/auth/LegacyTokenUtil.java (o componente de seguridad equivalente)
  - README.md (solo para documentar variables de entorno de laboratorio)
  - src/test/java/com/bcp/workshop/auth/AuthResourceTest.java
- Prueba a escribir antes de tocarlo:
  - Caracterizacion de login 200 para verificar que no cambia el contrato.
  - Prueba de configuracion que confirme lectura del secreto desde entorno de prueba.

## Item: H-05 | Manejo de errores expone detalles internos

- Item de HALLAZGOS.md: H-05.
- Cambio propuesto: devolver mensajes genericos al cliente y mover el detalle tecnico al logging del servidor con nivel adecuado.
- Archivos a tocar:
  - src/main/java/com/bcp/workshop/auth/AuthResource.java
  - src/test/java/com/bcp/workshop/auth/AuthResourceTest.java
- Prueba a escribir antes de tocarlo:
  - Caracterizacion de error interno para congelar comportamiento actual y luego validar la respuesta generica tras la correccion.

## Item: H-06 | Dependencias con version reportada en reto (SCA)

- Item de HALLAZGOS.md: H-06.
- Cambio propuesto: revisar advisories/CVEs aplicables y actualizar/remover dependencias vulnerables con el menor impacto funcional posible.
- Archivos a tocar:
  - pom.xml
  - src/test/java/com/bcp/workshop/auth/AuthResourceTest.java (si hay impactos por upgrades)
- Prueba a escribir antes de tocarlo:
  - Baseline de pruebas actuales del endpoint de login (200 y 401) antes de actualizar dependencias.
  - Re-ejecutar la misma bateria despues de actualizar para validar no regresion de contrato.

## Item: H-07 | Resource concentra validacion, negocio, persistencia y respuesta

- Item de HALLAZGOS.md: H-07.
- Cambio propuesto: separar responsabilidades en capas Resource -> Service -> Repository -> Security, aplicar guard clauses para reducir anidamiento, y reemplazar numeros magicos por constantes con nombre.
- Archivos a tocar:
  - src/main/java/com/bcp/workshop/auth/AuthResource.java
  - src/main/java/com/bcp/workshop/auth/LegacyDatabase.java (o repositorio nuevo)
  - src/main/java/com/bcp/workshop/auth/LegacyTokenUtil.java (o servicio de token nuevo)
  - src/main/java/com/bcp/workshop/auth/LoginRequest.java
  - nuevos archivos de servicio/DTO en src/main/java/com/bcp/workshop/auth/
  - src/test/java/com/bcp/workshop/auth/AuthResourceTest.java
- Prueba a escribir antes de tocarlo:
  - Caracterizacion completa del contrato HTTP actual: 400 (request nulo/username vacio/password vacia), 401 (credenciales invalidas), 200 (credenciales validas).
  - Validacion de estructura JSON de respuesta alineada con docs/ENDPOINTS.md.
