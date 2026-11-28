1. Identificación de riesgos de calidad
Para priorizar las pruebas utilizaré un enfoque de riesgo basado en dos variables:
- Impacto de negocio: cuánto afecta el riesgo la operación del almacén, la disponibilidad de insumos o las decisiones administrativas.
- Probabilidad: qué tan probable es que ocurra el fallo durante el uso real del sistema.
La prioridad se calcula con una matriz cualitativa:
Impacto	Probabilidad	Prioridad
Alto	Alta	Crítica
Alto	Media	Alta
Medio	Alta	Alta
Medio	Media	Media
Bajo	Alta	Media

Es importante distinguir que un defecto técnicamente pequeño puede tener un impacto de negocio alto. Por ejemplo, un error de redondeo o una actualización incorrecta del stock puede terminar generando diferencias entre el inventario registrado y el físico.
2. Matriz de riesgos – Top 12
Los siguientes riesgos están ordenados de mayor a menor prioridad, considerando impacto y probabilidad.
ID	Riesgo identificado	Impacto negocio	Probabilidad	Módulo afectado	¿Cómo lo probaría?
R01	Stock incorrecto después de movimientos. Una entrada o salida no actualiza correctamente las existencias.	Alto	Alta	Movimientos / Productos	1. Registrar una entrada de 20 unidades y verificar el incremento exacto. 2. Registrar una salida de 5 y comprobar que el stock final sea correcto.
R02	Permitir salidas superiores al stock disponible. El sistema puede dejar existencias negativas o entregar productos inexistentes.	Alto	Alta	Movimientos	1. Intentar retirar más unidades que las disponibles. 2. Verificar que el sistema rechace la operación y conserve el stock original.
R03	Errores en ajustes de inventario físico. El ajuste ADJUST suma o resta incorrectamente, o se interpreta como una entrada/salida.	Alto	Alta	Movimientos	1. Ajustar un producto de 30 unidades registradas a 24 físicas. 2. Verificar que el nuevo stock sea 24, no 54 ni 6.
R04	Inconsistencia entre stock, valor de inventario y dashboard. Las métricas no reflejan los movimientos realizados.	Alto	Alta	Dashboard / Productos / Movimientos	1. Modificar existencias y comprobar el valor total del inventario. 2. Comparar las métricas del dashboard con los registros de productos y movimientos.
R05	Acceso no autorizado por roles. Un operador puede realizar acciones reservadas al administrador.	Alto	Media	Login / Roles	1. Iniciar sesión como operador e intentar acceder a operaciones administrativas. 2. Intentar acceder directamente a las funciones restringidas mediante URL o solicitudes.
R06	Pérdida o duplicación de movimientos. Una operación se registra dos veces o no queda registrada.	Alto	Media	Movimientos	1. Enviar dos veces la misma solicitud de movimiento y verificar el resultado. 2. Simular una interrupción durante el registro y comprobar que no quede una actualización parcial.
R07	Cálculo incorrecto del valor del inventario. El precio se multiplica incorrectamente por las existencias o se manejan mal los decimales.	Alto	Media	Productos / Dashboard	1. Registrar productos con precios decimales y diferentes cantidades. 2. Comparar el valor mostrado con la suma de precio × stock de todos los productos.
R08	Alertas incorrectas de stock bajo. No se alerta cuando el producto llega al mínimo o se generan alertas cuando no corresponden.	Alto	Media	Alertas / Productos	1. Probar stock por debajo, igual y por encima del mínimo. 2. Verificar que los productos agotados también se identifiquen correctamente.
R09	Datos duplicados o inconsistentes en productos. Se permiten SKU repetidos o valores inválidos.	Medio	Alta	CRUD de productos	1. Crear dos productos con el mismo SKU. 2. Intentar guardar productos con precio o stock negativo y campos obligatorios vacíos.
R10	Exportación CSV con información incorrecta o incompleta. El archivo no coincide con los datos del inventario.	Medio	Alta	Exportación CSV	1. Comparar el CSV con los productos registrados, incluyendo SKU, precio y stock. 2. Verificar caracteres especiales, decimales y valores vacíos al abrir el archivo.
R11	Errores en los filtros y clasificación de stock. Los productos agotados se clasifican como disponibles o los de stock bajo no aparecen.	Medio	Media	Dashboard / Alertas	1. Crear productos con stock 0, menor al mínimo, igual al mínimo y superior al mínimo. 2. Verificar que cada uno aparezca en su clasificación correspondiente.
R12	Mala usabilidad durante el registro de movimientos. El usuario selecciona un producto equivocado o registra una cantidad incorrecta sin advertencia.	Medio	Media	Movimientos / Productos	1. Simular el registro de una salida con productos de nombres similares. 2. Verificar que se identifique claramente el producto, SKU, stock disponible y cantidad antes de confirmar.

3. Priorización: Top 5 para el piloto real
Prioridad crítica
Estos son los cinco riesgos que probaría antes de permitir que el almacén institucional utilice StockPascual en su operación cotidiana.
1. R01 – Exactitud del stock
Crítico

El objetivo principal del sistema es reemplazar el control manual y evitar diferencias entre existencias físicas y digitales. Si el stock no es confiable, el MVP pierde su propósito principal.

2. R02 – Control de salidas
Crítico

Una salida superior a las existencias puede generar entregas imposibles o inventarios negativos. Esto afecta directamente la disponibilidad de insumos para los laboratorios.

3. R03 – Ajustes físicos
Crítico

El almacén necesita corregir diferencias entre el inventario físico y el sistema. Si los ajustes funcionan incorrectamente, se perpetúan los errores que precisamente se busca solucionar.

4. R04 – Dashboard e integridad de métricas
Crítico

Las directivas necesitan información confiable para tomar decisiones administrativas. Un dashboard que presenta cifras diferentes a las existencias reales puede provocar decisiones equivocadas.

5. R05 – Seguridad y roles
Alta

Se debe garantizar que cada usuario tenga únicamente los permisos correspondientes. Una modificación no autorizada puede alterar las existencias y comprometer la confiabilidad del inventario.

4. Estrategia de ejecución para el piloto
No probaría todos los módulos de manera aislada. Utilizaría un escenario de operación real que permita validar la integridad del inventario de principio a fin.

Escenario propuesto:

Partimos de un producto registrado en el almacén:
Operación	Cantidad	Stock resultante
Stock inicial	50	50
Entrada (IN)	+20	70
Salida (OUT)	-15	55
Ajuste (ADJUST)	48 físicos	48
Salida (OUT)	-8	40

Al finalizar, comprobaría:
- El stock registrado debe ser exactamente 40 unidades.
- El dashboard debe reflejar las existencias actualizadas.
- El valor del inventario debe corresponder al precio unitario multiplicado por las existencias.
- Las alertas deben activarse según el stock mínimo configurado.
- El CSV debe contener los datos actualizados.
- El operador no debe poder ejecutar acciones restringidas al administrador.
5. Consideraciones adicionales de Senior QA
Hay tres aspectos que no deben confundirse al evaluar los resultados:
Concepto	Ejemplo en StockPascual
Severidad técnica	El sistema permite registrar una salida de 100 unidades cuando solo hay 20.
Impacto de negocio	El almacén podría entregar insumos que realmente no tiene y perder confiabilidad en sus registros.
Prioridad de prueba	Alta, porque afecta directamente la operación institucional.

También establecería estos criterios de aceptación para autorizar el piloto:
1. No se permiten existencias negativas mediante movimientos.
2. Los movimientos deben actualizar el stock de forma consistente.
3. Los ajustes deben establecer el inventario físico como nueva existencia.
4. El dashboard y el CSV deben coincidir con los datos registrados.
5. Los permisos de administrador y operador deben respetarse.
6. Los productos con stock bajo o agotado deben identificarse correctamente.

chatgpt