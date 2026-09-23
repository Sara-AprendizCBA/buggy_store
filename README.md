```markdown
# Explicación de los 6 Bugs del Sistema

Este documento explica los 6 errores encontrados en el código original de `TiendaOnline` y cómo fueron corregidos.

---

## Bug 1: Argumento Mutable por Defecto

**Código problemático:**
```python
def __init__(self, inventario_inicial={}):
```

**Problema:**  
En Python, los valores por defecto se evalúan una sola vez. Al usar un diccionario `{}` como valor por defecto, todas las instancias de la clase que no reciben un inventario propio terminan compartiendo el mismo diccionario en memoria.

**Consecuencia:**  
Al modificar el inventario de una tienda, también se modifica el de las demás.

**Solución:**  
Usar `None` como valor por defecto y crear un nuevo diccionario dentro del método.

```python
def __init__(self, inventario_inicial=None):
    self.inventario = inventario_inicial if inventario_inicial is not None else {}
```

---

## Bug 2: Typo en el nombre del atributo

**Código problemático:**
```python
self.ventas_totaIes += total_pedido
```

**Problema:**  
Se escribió `ventas_totaIes` (con “I” mayúscula) en lugar de `ventas_totales`. Esto crea un atributo nuevo y nunca actualiza el verdadero `ventas_totales`.

**Consecuencia:**  
El total de ventas siempre permanece en `0.0`.

**Solución:**  
Corregir el nombre del atributo:

```python
self.ventas_totales += total_pedido
```

---

## Bug 3: Lógica del descuento invertida

**Código problemático:**
```python
if cupon_descuento == "SENA2026":
    total_pedido = total_pedido * 1.20
```

**Problema:**  
El comentario indica que se debe aplicar un 20% de descuento, pero se multiplica por `1.20`, lo que **aumenta** el precio en un 20%.

**Consecuencia:**  
El cliente paga más en lugar de pagar menos.

**Solución:**  
Multiplicar por `0.80` para aplicar correctamente el 20% de descuento.

```python
if cupon_descuento == "SENA2026":
    total_pedido = total_pedido * 0.80
```

---

## Bug 4: No valida si el producto existe

**Código problemático:**
```python
producto = self.inventario[id_prod]
```

**Problema:**  
Si el `id_producto` no existe en el inventario, el programa lanza un `KeyError` y se cae.

**Consecuencia:**  
El sistema colapsa cuando se intenta comprar un producto inexistente.

**Solución:**  
Validar la existencia del producto antes de usarlo.

```python
if id_prod not in self.inventario:
    raise ValueError(f"Producto {id_prod} no existe en el inventario")
```

---

## Bug 5: No valida el stock disponible

**Código problemático:**
```python
producto['cantidad'] -= cant_comprada
```

**Problema:**  
Se resta la cantidad solicitada sin verificar si hay suficiente stock.

**Consecuencia:**  
El inventario puede quedar con valores negativos y se permiten ventas de productos que no existen.

**Solución:**  
Validar el stock antes de restar.

```python
if producto['cantidad'] < cant_comprada:
    raise ValueError(f"Stock insuficiente de {id_prod}. Disponible: {producto['cantidad']}")
```

---

## Bug 6: Modificar un diccionario mientras se itera

**Código problemático:**
```python
for id_producto in self.inventario.keys():
    if self.inventario[id_producto]['cantidad'] <= 0:
        del self.inventario[id_producto]
```

**Problema:**  
No se puede eliminar elementos de un diccionario mientras se está iterando sobre él. Esto genera un `RuntimeError`.

**Consecuencia:**  
El método `limpiar_agotados` hace que el programa se caiga.

**Solución:**  
Primero recolectar los IDs a eliminar y luego borrarlos.

```python
ids_a_eliminar = [
    id_producto for id_producto, datos in self.inventario.items()
    if datos['cantidad'] <= 0
]

for id_producto in ids_a_eliminar:
    del self.inventario[id_producto]
```
