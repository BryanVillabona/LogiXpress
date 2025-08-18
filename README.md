# 📦 LogiXpress - Sistema de Gestión Logística

**LogiXpress** es una empresa encargada de la distribución y logística en Colombia.  
Este proyecto simula su sistema de información mediante una base de datos en **MongoDB**, organizada en colecciones que representan los procesos reales de la compañía: clientes, envíos, facturación, soporte, rutas, oficinas, vehículos, incidencias y repartidores.  

El objetivo es modelar un sistema de logística realista que pueda gestionar clientes, controlar envíos, generar facturas, registrar incidencias y dar soporte a usuarios de manera eficiente.

---

## 📑 Estructura del proyecto

El proyecto está compuesto por 9 colecciones JSON que representan las áreas claves del negocio:

1. **Oficinas** → Puntos base de la empresa en distintas ciudades de Colombia.  
2. **Clientes** → Personas y empresas que envían o reciben mercancía.  
3. **Repartidores** → Personal encargado de realizar entregas, vinculados a oficinas y vehículos.  
4. **Vehículos** → Flota de transporte asignada a repartidores y oficinas.  
5. **Rutas** → Planificación de recorridos logísticos para los repartidores.  
6. **Envíos** → Registros de cada entrega realizada, vinculados a clientes, rutas y oficinas.  
7. **Facturas** → Documentos generados a partir de los envíos para los clientes.  
8. **Incidencias** → Problemas o irregularidades ocurridas durante los envíos.  
9. **Soporte** → Casos de asistencia al cliente relacionados con incidencias y servicios.

---

## ⚙️ Requisitos previos

- Tener instalado **MongoDB** (versión >= 6.0).
- Tener instalado **MongoDB Compass** o acceso a la terminal `mongosh`.
- Archivos `.json` exportados de este repositorio.

---

## 🗄️ Creación de la base de datos

Para crear la base de datos en MongoDB, ejecuta en la terminal:

```bash
use logixpress
```

Esto creará (o seleccionará) la base de datos `logixpress`.

---

## 📥 Importación de los datos

Cada colección tiene un archivo `.json` correspondiente.  
Los archivos deben importarse de la siguiente forma:

```bash
mongoimport --db logixpress --collection oficinas --file oficinas.json --jsonArray
mongoimport --db logixpress --collection clientes --file clientes.json --jsonArray
mongoimport --db logixpress --collection repartidores --file repartidores.json --jsonArray
mongoimport --db logixpress --collection vehiculos --file vehiculos.json --jsonArray
mongoimport --db logixpress --collection rutas --file rutas.json --jsonArray
mongoimport --db logixpress --collection envios --file envios.json --jsonArray
mongoimport --db logixpress --collection facturas --file facturas.json --jsonArray
mongoimport --db logixpress --collection incidencias --file incidencias.json --jsonArray
mongoimport --db logixpress --collection soporte --file soporte.json --jsonArray
```

⚠️ Nota: asegúrate de estar en la carpeta donde se encuentran los archivos `.json`.

---

## 🗂️ Archivos de datos incluidos

- `oficinas.json`
- `clientes.json`
- `repartidores.json`
- `vehiculos.json`
- `rutas.json`
- `envios.json`
- `facturas.json`
- `incidencias.json`
- `soporte.json`

---

## 🎯 Casos de uso del sistema

Algunos ejemplos de lo que este sistema permite gestionar:

- Registrar oficinas y sus direcciones.  
- Mantener información actualizada de clientes y empresas.  
- Controlar qué repartidores y vehículos están asignados a cada oficina.  
- Planificar rutas de distribución según la ciudad.  
- Gestionar envíos con trazabilidad (cliente → ruta → oficina).  
- Generar facturas automáticas por cada envío.  
- Detectar y registrar incidencias (paquetes dañados, retrasos, pérdida).  
- Atender solicitudes de soporte relacionadas con incidencias.  

---

## 🔍 Consultas con expresiones regulares

A continuación, se evidencia una serie de consultas para el sistema de logística.  
Cada consulta está agrupada por colección, combina **expresiones regulares** con **operadores lógicos y de arrays**, y tiene un propósito de negocio claro.

---

### 1. Clientes cuyos nombres empiezan por "J" o "M", seguidos de una vocal y no terminan en número  
```js
db.clientes.find({ 
  "nombre": { "$regex": "^[JjMm][aeiou].*[^0-9]$" } 
})
```
Útil para autocompletar nombres de clientes personas y evitar falsos registros con números al final.

---

### 2. Clientes con correos Gmail o Hotmail, pero no corporativos (`.com.co`)  
```js
db.clientes.find({ 
  "contacto.email": { "$regex": "^(.*@gmail.com|.*@hotmail.com)$", "$not": { "$regex": ".com.co$" } } 
})
```
Permite segmentar usuarios con correos personales, ideal para campañas de fidelización.

---

### 3. Clientes de ciudades que empiezan por "B" pero que no sean "Bogotá"  
```js
db.clientes.find({ 
  "direccion.ciudad": { "$regex": "^B", "$not": { "$regex": "^Bogotá$" } } 
})
```
Útil para segmentar en Barranquilla y Bucaramanga, excluyendo Bogotá.

---

### 4. Envíos de electrodomésticos que no sean “microondas” ni “licuadora”  
```js
db.envios.find({ 
  "detalle_paquete.descripcion": { "$regex": "Electrodoméstico", "$not": { "$regex": "(microondas|licuadora)" } } 
})
```
Filtra electrodomésticos de mayor tamaño o relevancia para logística pesada.

---

### 5. Envíos con descripciones que contengan palabras separadas por espacios (ej. “silla ergonómica”)  
```js
db.envios.find({ 
  "detalle_paquete.descripcion": { "$regex": "\w+\s\w+" } 
})
```
Ayuda a identificar productos compuestos que suelen necesitar embalaje especial.

---

### 6. Facturas con métodos de pago que sean tarjeta débito o tarjeta crédito, pero no efectivo  
```js
db.facturas.find({ 
  "metodo_pago": { "$regex": "tarjeta (crédito|débito)$" } 
})
```
Útil para generar reportes de pagos electrónicos.

---

### 7. Facturas con clientes cuyos códigos estén entre `CLI001` y `CLI005`, y terminen en número impar  
```js
db.facturas.find({ 
  "cliente_id": { "$regex": "^CLI00[1-5][13579]$" } 
})
```
Permite filtrar facturas iniciales de clientes específicos para auditorías.

---

### 8. Incidencias con tipos que empiecen con “Paquete” o “Retraso”  
```js
db.incidencias.find({ 
  "tipo": { "$regex": "^(Paquete|Retraso)" } 
})
```
Detecta las incidencias más comunes que afectan calidad del servicio.

---

### 9. Incidencias cuya descripción mencione “paquete” seguido de cualquier palabra  
```js
db.incidencias.find({ 
  "descripcion": { "$regex": "paquete\s\w+" } 
})
```
Permite identificar casos donde el paquete sufrió un evento particular (dañado, extraviado, abierto).

---

### 10. Tickets de soporte con asuntos que comiencen en “Paquete” y terminen en “do” (ej. Perdido, Abierto)  
```js
db.soporte.find({ 
  "asunto": { "$regex": "^Paquete.*do$" } 
})
```
Agrupa quejas relacionadas con pérdida o apertura de paquetes.

---

### 11. Tickets de soporte con descripciones que contengan al menos un número (ej. tiempo de espera, horas)  
```js
db.soporte.find({ 
  "descripcion": { "$regex": "\d+" } 
})
```
Detecta reclamos donde los clientes especifican tiempos o cantidades.

---

### 12. Oficinas cuyos correos inicien con la ciudad y terminen en `logixpress.com`  
```js
db.oficinas.find({ 
  "email": { "$regex": "^[a-z]+@logixpress.com$" } 
})
```
Valida la correcta nomenclatura de correos institucionales.

---

### 13. Oficinas cuya dirección contenga un número de calle con 2 o más dígitos  
```js
db.oficinas.find({ 
  "direccion": { "$regex": "\d{2,}" } 
})
```
Verifica consistencia en direcciones registradas.

---

### 14. Repartidores cuyos nombres tengan dos palabras (ej. “Carlos Ramírez”)  
```js
db.repartidores.find({ 
  "nombre": { "$regex": "^[A-Z][a-z]+\s[A-Z][a-z]+$" } 
})
```
Identifica empleados con nombre y apellido registrados correctamente.

---

### 15. Repartidores con cédulas que empiecen en `10` y terminen en dígito impar  
```js
db.repartidores.find({ 
  "cedula": { "$regex": "^10.*[13579]$" } 
})
```
Segmenta empleados de ciertas generaciones de documento.

---

### 16. Vehículos con placas que tengan exactamente 3 letras y 3 números  
```js
db.vehiculos.find({ 
  "placa": { "$regex": "^[A-Z]{3}\d{3}$" } 
})
```
Valida placas con formato correcto colombiano.

---

### 17. Vehículos en mantenimiento cuya placa termine en número par  
```js
db.vehiculos.find({ 
  "placa": { "$regex": "\d*[02468]$" }, 
  "estado": "mantenimiento" 
})
```
Identifica vehículos en mantenimiento filtrados por criterio adicional.

---

### 18. Vehículos cuyo modelo sea de 2018 a 2020  
```js
db.vehiculos.find({ 
  "modelo": { "$regex": "20(18|19|20)" } 
})
```
Permite ubicar modelos con antigüedad específica para renovación.

---

### 19. Rutas con paradas que tengan al menos dos palabras (ej. “Santa Marta”)  
```js
db.rutas.find({ 
  "paradas": { "$elemMatch": { "$regex": "^[A-Z][a-z]+\s[A-Z][a-z]+$" } } 
})
```
Ayuda a planificar rutas con ciudades compuestas.

---

### 20. Rutas cuyo destino empiece con consonante y termine con vocal  
```js
db.rutas.find({ 
  "destino": { "$regex": "^[^aeiouAEIOU].*[aeiouAEIOU]$" } 
})
```
Clasifica destinos con reglas fonéticas para segmentación.

---

### 21. Rutas cuyo estado sea “planificada” o “completada”  
```js
db.rutas.find({ 
  "estado": { "$regex": "^(planificada|completada)$" } 
})
```
Ayuda a revisar rutas finalizadas o listas para ejecución.

---

### 22. Clientes con identificación que tenga guion y termine en un dígito  
```js
db.clientes.find({ 
  "identificacion": { "$regex": "-\d$" } 
})
```
Detecta empresas registradas con NIT válido.

---

### 23. Clientes con nombres que contengan al menos un espacio (nombre compuesto o empresa S.A.)  
```js
db.clientes.find({ 
  "nombre": { "$regex": "\s" } 
})
```
Permite diferenciar clientes individuales de empresas.

---

### 24. Clientes cuyo teléfono empiece con `+57 31` y termine en número impar  
```js
db.clientes.find({ 
  "contacto.telefono": { "$regex": "^\+57 31.*[13579]$" } 
})
```
Segmenta clientes que usan líneas móviles específicas.

---

### 25. Tickets de soporte con respuestas donde el agente tenga nombre compuesto  
```js
db.soporte.find({ 
  "respuestas": { 
    "$elemMatch": { "agente": { "$regex": "^[A-Z][a-z]+\s[A-Z][a-z]+$" } } 
  } 
})
```
Verifica consistencia en nombres de agentes de soporte.

---

## 👤 Autor

| Nombre del Integrante | GitHub |
|---|---|
| Sergio Lievano | [@sergiosteven66](https://github.com/sergiosteven66)|
| Bryan Villabona | [@BryanVillabona](https://github.com/BryanVillabona)|
