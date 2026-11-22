# Casos de prueba — Carrito de compras

Diseñados antes de escribir código. Cada caso se convierte en una prueba de Vitest en el ciclo indicado.

## 1. Interfaz prevista

Para escribir pruebas antes que el código hay que decidir primero cómo se usará el carrito:

| Operación | Uso | Requisito |
|---|---|---|
| Agregar | `carrito.agregarProducto({ id, nombre, precio, cantidad })` | RF-01 |
| Listar | `carrito.obtenerProductos()` | RF-01 |
| Subtotal | `carrito.subtotal()` | RF-02 |
| Descuento | `carrito.aplicarDescuento(porcentaje)` | RF-03 |
| Total | `carrito.total()` | RF-02, RF-03 |
| Actualizar | `carrito.actualizarCantidad(id, cantidad)` | RF-04 |
| Eliminar | `carrito.eliminarProducto(id)` | RF-04 |

## 2. Reglas aclaradas al diseñar los casos

Los requisitos no responden estas preguntas. Se fijaron aquí para poder escribir un resultado esperado concreto.

| ID | Pregunta | Decisión |
|---|---|---|
| D-01 | ¿Qué pasa si se agrega un ID que ya está en el carrito? | Se suma la cantidad; no se crea una segunda línea. |
| D-02 | ¿La cantidad puede ser decimal? | No: debe ser un entero mayor que 0. |
| D-03 | ¿El precio puede ser 0? | Sí: RF-05 solo rechaza precios negativos. |
| D-04 | ¿Actualizar la cantidad a 0 elimina el producto? | No: se rechaza con error; para quitar un producto existe `eliminarProducto`. |
| D-05 | ¿Con cuántos decimales se devuelven los importes? | Dos decimales. |
| D-06 | ¿Qué tipo de error se lanza? | `RangeError` para valores fuera de rango, `TypeError` para tipos inválidos y `Error` para productos inexistentes, siempre con un mensaje que indica el campo. |

## 3. Casos

Tipo: **N** = normal · **B** = borde · **I** = inválido.

| ID | Req. | Tipo | Entrada / acción | Resultado esperado | Ciclo |
|---|---|---|---|---|---|
| CP-01 | RF-01 | N | Agregar `{ id: "P1", nombre: "Mouse", precio: 15.5, cantidad: 2 }` | El carrito contiene 1 producto con esos datos | 1 |
| CP-02 | RF-02 | B | Carrito vacío, pedir subtotal | `0` | 1 |
| CP-03 | RF-02 | N | Agregar P1 (15,50 × 2) y P2 (4,25 × 3); pedir subtotal | `43.75` | 1 |
| CP-04 | RF-01 | N | Agregar P1 con cantidad 2 y otra vez P1 con cantidad 3 (D-01) | Una sola línea de P1 con cantidad 5 | 1 |
| CP-05 | RF-02 | B | Agregar `precio: 0.1, cantidad: 3`; pedir subtotal (D-05) | `0.3` | 1 |
| CP-06 | RF-03 | N | Subtotal 100, descuento 10 %; pedir total | `90` | 2 |
| CP-07 | RF-02 | N | Subtotal 43,75, sin descuento; pedir total | `43.75` | 2 |
| CP-08 | RF-03 | B | Subtotal 100, descuento 0 % | Total `100` | 2 |
| CP-09 | RF-03 | B | Subtotal 100, descuento 100 % | Total `0`, nunca negativo | 2 |
| CP-10 | RF-03, RF-05 | I | Descuento −1 | `RangeError` con mensaje sobre el descuento | 2 |
| CP-11 | RF-03, RF-05 | I | Descuento 100,01 | `RangeError` con mensaje sobre el descuento | 2 |
| CP-12 | RF-04 | N | P1 con cantidad 2; actualizar a 5 | P1 queda con cantidad 5 y el subtotal se recalcula | 3 |
| CP-13 | RF-04 | N | Carrito con P1 y P2; eliminar P1 | Solo queda P2 | 3 |
| CP-14 | RF-04 | B | Carrito con solo P1; eliminar P1 | Carrito vacío, subtotal `0` | 3 |
| CP-15 | RF-04, RF-05 | I | Actualizar P1 a cantidad 0 (D-04) | `RangeError` con mensaje sobre la cantidad | 3 |
| CP-16 | RF-04 | I | Actualizar o eliminar un ID que no existe | `Error` indicando que el producto no existe | 3 |
| CP-17 | RF-05 | I | Agregar con `precio: -0.01` | `RangeError` con mensaje sobre el precio | Fase C |
| CP-18 | RF-05 | B | Agregar con `precio: 0` (D-03) | Se acepta; subtotal `0` | Fase C |
| CP-19 | RF-05 | I | Agregar con `cantidad: 0` | `RangeError` con mensaje sobre la cantidad | Fase C |
| CP-20 | RF-05 | I | Agregar con `cantidad: -1` | `RangeError` con mensaje sobre la cantidad | Fase C |
| CP-21 | RF-05 | I | Agregar con `cantidad: 1.5` (D-02) | `RangeError` con mensaje sobre la cantidad | Fase C |
| CP-22 | RF-05 | I | Agregar con `precio: "15"` o `precio: NaN` | `TypeError` con mensaje sobre el precio | Fase C |
| CP-23 | RF-05 | I | Agregar con `nombre: ""` o sin `id` | `TypeError` con mensaje sobre el campo | Fase C |
| CP-24 | RF-05 | I | Descuento `"10"` | `TypeError` con mensaje sobre el descuento | Fase C |

## 4. Cobertura por requisito

| Requisito | Normal | Borde | Inválido |
|---|---|---|---|
| RF-01 | CP-01, CP-04 | — | — (sus validaciones están en RF-05) |
| RF-02 | CP-03, CP-07 | CP-02, CP-05 | — |
| RF-03 | CP-06 | CP-08, CP-09 | CP-10, CP-11 |
| RF-04 | CP-12, CP-13 | CP-14 | CP-15, CP-16 |
| RF-05 | — | CP-18 | CP-17, CP-19 a CP-24 |

Los ciclos 1, 2 y 3 de la Fase B construyen la funcionalidad principal. Los casos marcados «Fase C» amplían los casos negativos, como pide esa fase.
