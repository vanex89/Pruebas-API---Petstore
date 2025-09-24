 QA API Tests – Petstore

Validación de los endpoints principales de **Petstore** para el recurso **Pet**, cubriendo CRUD (Create, Read, Update, Delete) con **casos positivos y negativos**.  
Las aserciones verifican: códigos HTTP esperados, `Content-Type` JSON y tiempos de respuesta ≤ **2000 ms**, además de consistencia de datos entre lo enviado y lo recibido.


 1) Alcance

- Recurso cubierto: **`/pet`**
- Operaciones: **POST, GET, PUT, DELETE**
- Tipos de casos: **positivos** y **negativos** (entrada inválida, límites de campo, id inexistente, idempotencia, duplicidad, caracteres especiales, etc.)


 2) Requisitos

- **Postman** (para importar y ejecutar la colección).
- **Node.js** + **Newman** (opcional pero recomendado para ejecución por CLI y reporte HTML).
- Sistema operativo: Windows 10/11 u otro compatible con Node.

> Entregable esperado: repositorio público con estructura clara y README con instrucciones de ejecución.


 3) Variables (Environment de Postman)

Variables empleadas:
- `baseUrl` → URL base de la API (por defecto: `https://petstore.swagger.io/v2`)
- `timeoutMs` → umbral de latencia para checks de tiempo (por defecto: `2000`)

> Importa el **environment** incluido en `/postman` y verifica/ajusta estos valores según sea necesario.


 4) Estructura del repositorio

```
Pruebas-API---Petstore/
├─ postman/
│  ├─ Pruebas_API_Petstore.postman_collection.json
│  └─ Pruebas_API_Petstore.postman_environment.json
├─ evidence/
|  ├─ DELETE - pet{petId}
|  ├─ GET - pet{petId}
|  ├─ POST - pet
|  ├─ PUT - pet
|  └─ Pruebas API - Petstore.postman_test_run.json
├─ .gitignore
└─ README.md
```


 5) Cobertura de pruebas (resumen)

### POST `/pet`
- **POST-01**: Crear pet válido (200, JSON, t ≤ 2000 ms, campos clave, coherencia de id/name).
- **POST-02**: JSON malformado/estructura inválida (4xx, respuesta controlada, no crea recurso).
- **POST-03**: `status` fuera de catálogo (esperado 400/422; si 200, se documenta y verifica persistencia).
- **POST-04**: Duplicidad de `id` (rechazo 400/409/422; o comportamiento observado tipo upsert/no-op documentado).
- **POST-05**: Límites/extremos en `name` (vacío, muy largo, especiales): rechazo 400/422 o aceptación/sanitización documentada.

### GET `/pet/{petId}`
- **GET-01**: Consultar pet existente (con *pre-seed* vía POST) y fallback documentado si el entorno no persiste.
- **GET-02**: `petId` inexistente (404 “not found”).
- **GET-03**: `petId` no numérico/formato inválido (400/404, mensaje controlado; sin stacktrace/HTML).
- **GET-04**: `petId` vacío/nulo (400/404; mensaje controlado).
- **GET-05**: `petId` con caracteres especiales/espacios/unicode (400/404; sin stacktrace/HTML).

### PUT `/pet`
- **PUT-01**: Actualizar `name` y `status`; verificación por GET de persistencia.
- **PUT-02**: Falta `id` en body (400/422).
- **PUT-03**: `status` inválido (rechazo 400/422 o comportamiento observado documentado).
- **PUT-04**: Actualizar `id` inexistente (preferido 404; se valida no-upsert).
- **PUT-05**: Idempotencia/no-op (enviar mismos datos → 200 sin cambios reales).

### DELETE `/pet/{petId}`
- **DELETE-01**: Eliminar existente; GET posterior 404.
- **DELETE-02**: Eliminar inexistente (404 o comportamiento documentado).
- **DELETE-03**: `petId` no numérico/formato inválido (400/404; mensaje controlado).
- **DELETE-04**: Borrado doble (idempotencia): primera vez 200/204; segunda, 404/“not found”.
- **DELETE-05**: `petId` con caracteres especiales/espacios/unicode (400/404; sin stacktrace/HTML).


 6) Cómo ejecutar

### Opción A — Postman (GUI)
1. **Importar**:
   - `postman/Pruebas_API_Petstore.postman_collection.json`
   - `postman/Pruebas_API_Petstore.postman_environment.json`
2. Selecciona el **Environment** y revisa `baseUrl`/`timeoutMs`.
3. Ejecuta por carpeta o con **Collection Runner**.

### Opción B — Newman (CLI) con reporte HTML
Instala Newman y el reporter:
```bash
npm install -g newman newman-reporter-htmlextra
```

Ejecuta (genera `evidence/run-report.html`):
```bash
newman run postman/Pruebas_API_Petstore.postman_collection.json   -e postman/Pruebas_API_Petstore.postman_environment.json   -r htmlextra --reporter-htmlextra-export evidence/run-report.html
```

> Si prefieres evitar instalaciones globales, agrega `newman` y `newman-reporter-htmlextra` como `devDependencies` y define el script `npm run test:api`.

---

 7) Evidencias

Screenshot
https://github.com/vanex89/Pruebas-API---Petstore/tree/main/Evidencias/POST%20-%20pet
https://github.com/vanex89/Pruebas-API---Petstore/tree/main/Evidencias/GET%20-%20pet%7BpetId%7D
https://github.com/vanex89/Pruebas-API---Petstore/tree/main/Evidencias/PUT%20-%20pet
https://github.com/vanex89/Pruebas-API---Petstore/tree/main/Evidencias/DELETE%20-%20pet%7BpetId%7D

Video Run
https://github.com/vanex89/Pruebas-API---Petstore/blob/5f03756184e3670f940bde2736a2f0b8da629fef/Evidencias/V%C3%ADdeo%20sin%20t%C3%ADtulo%20%E2%80%90%20Hecho%20con%20Clipchamp.mp4

 8) Notas y troubleshooting

- **Entorno público Petstore**: ocasionalmente la persistencia puede variar (p. ej., un `POST` no queda disponible inmediatamente o retorna 404 en lecturas posteriores).  
  Los casos contemplan este comportamiento y **documentan** resultados alternativos sin falsear positivamente.
- **Mensajes de error**: las pruebas negativas validan respuestas **controladas** (sin *stack traces* ni HTML crudo) y términos genéricos como `invalid/bad/not found/format/number/exception`.
- **Latencia**: se verifica `responseTime ≤ timeoutMs` (por defecto 2000 ms). Ajusta si tu red es muy lenta.


 9) Scripts útiles (opcional)

**Windows – doble clic**: crea `run_api_tests.cmd` con:
```cmd
@echo off
newman run postman\Pruebas_API_Petstore.postman_collection.json ^
  -e postman\Pruebas_API_Petstore.postman_environment.json ^
  -r htmlextra --reporter-htmlextra-export evidence
un-report.html
echo Reporte generado en: evidence
un-report.html
pause
```
Autor: **Vanessa “Vane” Velásquez** (`vanex89`)

