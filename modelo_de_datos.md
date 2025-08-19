# Modelo de datos — LogiXpress

> Documento técnico del modelo de información utilizado en el taller de **MongoDB** para la empresa de logística **LogiXpress**.

> **Importante:** En este proyecto **no** se referencian documentos usando `_id`/`ObjectId`. Todas las relaciones entre colecciones se realizan mediante **claves naturales legibles** (códigos como `OFI001`, `CLI001`, `ENV001`, etc.).


---

## 1) Principios de diseño adoptados

- **Claves naturales y no `_id`**: Las relaciones se hacen con campos de negocio (`cliente_id`, `codigo_envio`, `oficina_asignada`, etc.). Facilita importaciones con `mongoimport` y lectura por parte de negocio.

- **Embebidos donde aporta valor**: Se embeben datos **intrínsecamente ligados** al documento (p. ej., `detalle` en facturas, `detalle_paquete` e `historial_estados` en envíos, `acciones_tomadas` en incidencias, `respuestas` en soporte) para lectura directa.

- **Consistencia de códigos**: Todos los códigos siguen prefijos y numeración de 3 dígitos (ej.: `CLI001`, `ENV010`).

- **Fechas como string ISO corto `YYYY-MM-DD`**: simplifica importación y permite filtros por prefijo (p. ej., `^2025-08`).


---

## 2) Convenciones de claves (claves naturales)


| Entidad | Campo clave | Ejemplo | Patrón (regex) |
|---|---|---|---|
| Oficina | `codigo` | `OFI001` | `^OFI\d{3}$` |
| Cliente | `codigo` | `CLI001` | `^CLI\d{3}$` |
| Repartidor | `codigo` | `REP001` | `^REP\d{3}$` |
| Ruta | `codigo` | `RUT001` | `^RUT\d{3}$` |
| Envío | `codigo_envio` | `ENV001` | `^ENV\d{3}$` |
| Factura | `codigo_factura` | `FAC001` | `^FAC\d{3}$` |
| Incidencia | `codigo_incidencia` | `INC001` | `^INC\d{3}$` |
| Ticket soporte | `ticket_id` | `SUP001` | `^SUP\d{3}$` |
| Vehículo | `codigo` | `VEH001` | `^VEH\d{3}$` |
| Vehículo (placa) | `placa` | `ABC123` | `^[A-Z]{3}\d{3}$` |


> **Nota placas:** los vehículos además se identifican por `placa` con patrón colombiano `AAA999`.


---

## 3) Colecciones y campos principales


### oficinas

**Documento de ejemplo:**

```json

{
  "codigo": "OFI001",
  "ciudad": "Bogotá",
  "direccion": "Cra. 68 #24-39, Fontibón",
  "telefono": "+57 1 3901234",
  "email": "bogota@logixpress.com",
  "horario": {
    "apertura": "08:00",
    "cierre": "18:00"
  },
  "servicios": [
    "recepción",
    "almacenamiento",
    "distribución"
  ]
}

```

**Campos / Tipos (nivel superior):**

- `codigo`: string

- `ciudad`: string

- `direccion`: string

- `telefono`: string

- `email`: string

- `horario`: object

- `servicios`: array<string>


**Estructuras embebidas destacadas:**

- `horario` *(object)*:
  - `apertura`: string
  - `cierre`: string


**Relaciones y referencias (por código natural):**

- Referenciada por `clientes.oficina_registro`, `envios.origen_oficina`, `envios.destino_oficina`, `repartidores.oficina_asignada`, `vehiculos.oficina_asignada`, `rutas.origen` (cuando usa código).


### clientes

**Documento de ejemplo:**

```json

{
  "codigo": "CLI001",
  "nombre": "Juan Pérez",
  "tipo": "persona",
  "identificacion": "1023456789",
  "contacto": {
    "telefono": "+57 3104567890",
    "email": "juan.perez@gmail.com"
  },
  "direccion": {
    "calle": "Cra. 45 #23-12",
    "ciudad": "Bogotá",
    "departamento": "Cundinamarca"
  },
  "oficina_registro": "OFI001",
  "envios_previos": [
    "ENV001",
    "ENV003"
  ]
}

```

**Campos / Tipos (nivel superior):**

- `codigo`: string

- `nombre`: string

- `tipo`: string

- `identificacion`: string

- `contacto`: object

- `direccion`: object

- `oficina_registro`: string

- `envios_previos`: array<string>


**Estructuras embebidas destacadas:**

- `contacto` *(object)*:
  - `telefono`: string
  - `email`: string
- `direccion` *(object)*:
  - `calle`: string
  - `ciudad`: string
  - `departamento`: string


**Relaciones y referencias (por código natural):**

- Referenciada por `envios.cliente_id`, `facturas.cliente_id`, `soporte.cliente_id`.

- Mantiene histórico liviano con `envios_previos` (códigos `ENV###`).


### repartidores

**Documento de ejemplo:**

```json

{
  "codigo": "REP001",
  "nombre": "Carlos Ramírez",
  "cedula": "1012456789",
  "telefono": "+57 3104567890",
  "email": "carlos.ramirez@logixpress.com",
  "oficina_asignada": "OFI001",
  "vehiculo_asignado": "VEH001",
  "rutas_asignadas": [
    "RUT001",
    "RUT002"
  ]
}

```

**Campos / Tipos (nivel superior):**

- `codigo`: string

- `nombre`: string

- `cedula`: string

- `telefono`: string

- `email`: string

- `oficina_asignada`: string

- `vehiculo_asignado`: string

- `rutas_asignadas`: array<string>


**Relaciones y referencias (por código natural):**

- Referenciado por `rutas.repartidor_asignado`, `vehiculos.repartidor_asignado`, `incidencias.repartidor_id`.

- Puede tener múltiples rutas (`rutas_asignadas`).


### vehiculos

**Documento de ejemplo:**

```json

{
  "codigo": "VEH001",
  "tipo": "Camión",
  "placa": "ABC123",
  "capacidad_kg": 5000,
  "modelo": "Chevrolet NPR 2020",
  "estado": "activo",
  "oficina_asignada": "OFI001",
  "repartidor_asignado": "REP001"
}

```

**Campos / Tipos (nivel superior):**

- `codigo`: string

- `tipo`: string

- `placa`: string

- `capacidad_kg`: number

- `modelo`: string

- `estado`: string

- `oficina_asignada`: string

- `repartidor_asignado`: string


**Relaciones y referencias (por código natural):**

- Referenciado por `rutas.vehiculo_asignado` y `repartidores.vehiculo_asignado`.

- Alineado a una oficina (`oficina_asignada`).


### rutas

**Documento de ejemplo:**

```json

{
  "codigo": "RUT001",
  "origen": "OFI001",
  "destino": "Medellín",
  "kilometros": 415,
  "vehiculo_asignado": "VEH001",
  "repartidor_asignado": "REP001",
  "paradas": [
    "La Pintada",
    "Santa Bárbara",
    "Envigado"
  ],
  "estado": "en curso"
}

```

**Campos / Tipos (nivel superior):**

- `codigo`: string

- `origen`: string

- `destino`: string

- `kilometros`: number

- `vehiculo_asignado`: string

- `repartidor_asignado`: string

- `paradas`: array<string>

- `estado`: string


**Relaciones y referencias (por código natural):**

- Referenciada por `envios.ruta_asignada` y `repartidores.rutas_asignadas`.

- Conecta `origen` (oficina o ciudad) con `destino`.


### envios

**Documento de ejemplo:**

```json

{
  "codigo_envio": "ENV001",
  "cliente_id": "CLI001",
  "origen_oficina": "OFI001",
  "destino_oficina": "OFI002",
  "ruta_asignada": "RUT001",
  "detalle_paquete": {
    "peso_kg": 2.5,
    "dimensiones_cm": {
      "largo": 30,
      "ancho": 20,
      "alto": 10
    },
    "descripcion": "Electrodoméstico pequeño - licuadora",
    "valor_declarado": 350000
  },
  "estado": "en tránsito",
  "fecha_envio": "2025-08-01",
  "fecha_entrega_estimada": "2025-08-03",
  "historial_estados": [
    {
      "estado": "pendiente",
      "fecha": "2025-08-01"
    },
    {
      "estado": "en tránsito",
      "fecha": "2025-08-02"
    }
  ]
}

```

**Campos / Tipos (nivel superior):**

- `codigo_envio`: string

- `cliente_id`: string

- `origen_oficina`: string

- `destino_oficina`: string

- `ruta_asignada`: string

- `detalle_paquete`: object

- `estado`: string

- `fecha_envio`: string

- `fecha_entrega_estimada`: string

- `historial_estados`: array<object>


**Estructuras embebidas destacadas:**

- `detalle_paquete` *(object)*:
  - `peso_kg`: number
  - `dimensiones_cm`: object
  - `descripcion`: string
  - `valor_declarado`: number
- `historial_estados` *(array<object>)*: ejemplo de ítem:
  - `estado`: string
  - `fecha`: string


**Relaciones y referencias (por código natural):**

- Entidad transaccional central. Referencia a `clientes` (`cliente_id`), `oficinas` (`origen_oficina`, `destino_oficina`) y `rutas` (`ruta_asignada`).

- Relacionada con `facturas.envio_id` e `incidencias.envio_id`.


### facturas

**Documento de ejemplo:**

```json

{
  "codigo_factura": "FAC001",
  "envio_id": "ENV001",
  "cliente_id": "CLI001",
  "fecha_emision": "2025-08-01",
  "monto_total": 380000,
  "detalle": {
    "costo_envio": 30000,
    "seguro": 15000,
    "valor_paquete": 335000
  },
  "estado": "pagada",
  "metodo_pago": "tarjeta crédito"
}

```

**Campos / Tipos (nivel superior):**

- `codigo_factura`: string

- `envio_id`: string

- `cliente_id`: string

- `fecha_emision`: string

- `monto_total`: number

- `detalle`: object

- `estado`: string

- `metodo_pago`: string


**Estructuras embebidas destacadas:**

- `detalle` *(object)*:
  - `costo_envio`: number
  - `seguro`: number
  - `valor_paquete`: number


**Relaciones y referencias (por código natural):**

- Referencia a `envios.envio_id` y `clientes.cliente_id`.

- Embebe `detalle` para snapshot financiero.


### incidencias

**Documento de ejemplo:**

```json

{
  "codigo_incidencia": "INC001",
  "envio_id": "ENV002",
  "repartidor_id": "REP002",
  "tipo": "Retraso en entrega",
  "descripcion": "El vehículo se averió en carretera y la entrega se retrasó 1 día.",
  "fecha_reporte": "2025-08-03",
  "estado": "resuelta",
  "acciones_tomadas": [
    "Cliente notificado",
    "Entrega reprogramada"
  ]
}

```

**Campos / Tipos (nivel superior):**

- `codigo_incidencia`: string

- `envio_id`: string

- `repartidor_id`: string

- `tipo`: string

- `descripcion`: string

- `fecha_reporte`: string

- `estado`: string

- `acciones_tomadas`: array<string>


**Relaciones y referencias (por código natural):**

- Referencia a `envios.envio_id` y `repartidores.repartidor_id`.

- Embebe `acciones_tomadas` para trazabilidad.


### soporte

**Documento de ejemplo:**

```json

{
  "ticket_id": "SUP001",
  "cliente_id": "CLI001",
  "incidencia_id": "INC003",
  "asunto": "Paquete dañado",
  "descripcion": "Mi paquete llegó con la caja rota y con contenido en mal estado.",
  "fecha_creacion": "2025-08-01",
  "estado": "abierto",
  "respuestas": [
    {
      "fecha": "2025-08-01",
      "agente": "Mariana López",
      "mensaje": "Hemos recibido su caso, lo estamos revisando."
    }
  ]
}

```

**Campos / Tipos (nivel superior):**

- `ticket_id`: string

- `cliente_id`: string

- `incidencia_id`: string

- `asunto`: string

- `descripcion`: string

- `fecha_creacion`: string

- `estado`: string

- `respuestas`: array<object>


**Estructuras embebidas destacadas:**

- `respuestas` *(array<object>)*: ejemplo de ítem:
  - `fecha`: string
  - `agente`: string
  - `mensaje`: string


**Relaciones y referencias (por código natural):**

- Referencia a `clientes.cliente_id` e `incidencias.codigo_incidencia`.

- Embebe `respuestas` (historial conversacional).


---

## 4) Reglas de consistencia y validación

- Mantener los patrones de códigos (`CLI\d{3}`, `ENV\d{3}`, etc.) durante la carga de datos.

- Validar que **todas** las referencias por clave natural **existan** (ej.: no registrar `facturas.envio_id = ENV999` si no existe ese envío).

- Evitar modificar retroactivamente códigos ya referenciados por otras colecciones.

- Usar `regex` sobre fechas (`^2025-08`) y códigos (`^ENV\d{3}$`) para búsquedas eficientes y autocompletado.


---

## 5) Justificación por colección

- **oficinas:** ancla geográfica/operativa; provee `horario` y `servicios` para segmentar flota, rutas y atención. Es referenciada desde múltiples entidades (clientes, envíos, repartidores, vehículos).

- **clientes:** entidad comercial; snapshot mínimo de contacto y dirección (objeto) para evitar duplicación. `envios_previos` mantiene un historial liviano por código.

- **repartidores:** vincula oficina, vehículo y rutas por **claves naturales**; `rutas_asignadas` permite planear cobertura; útil para reporte de cargas y desempeño.

- **vehiculos:** identificados por `placa` y código; asignación a oficina y repartidor para control operativo; `capacidad_kg` habilita validaciones previas del paquete.

- **rutas:** conectan origen/destino y consolidan asignación (`vehiculo_asignado`, `repartidor_asignado`) y puntos de paso (`paradas`); clave para análisis de tiempos y SLA.

- **envios:** **transacción central**; embebe `detalle_paquete` e `historial_estados` para trazabilidad sin joins; relaciona cliente, oficinas y ruta por códigos.

- **facturas:** documento legal con **snapshot** financiero (`detalle` embebido) y trazabilidad al envío y cliente; soporta conciliación y cobranza.

- **incidencias:** registro operativo con `acciones_tomadas` embebidas; enlaza al envío y al repartidor para accountability; insumo de mejora continua.

- **soporte:** ticket de atención enlazado al cliente e incidencia; `respuestas` embebidas preservan el hilo conversacional para auditoría.
