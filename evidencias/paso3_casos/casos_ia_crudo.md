Suite de casos de prueba – StockPascual
Paso 3 | QA Analyst – Diseño de pruebas

Vamos a construir la suite de pruebas tomando como base el archivo 02_casos_de_prueba.md, los criterios de aceptación del negocio y los riesgos que identificamos anteriormente.

El objetivo es validar que StockPascual sea confiable para operar en un almacén institucional, especialmente en el control de existencias, movimientos, alertas y reportes.
1. Preparación del ambiente de pruebas
Usuarios definidos en el ejercicio:
Usuario	Contraseña	Rol
admin	Admin123!	Administrador
operador	Oper123!	Operador

Credenciales exclusivamente para ambiente de pruebas, no para producción.

Datos iniciales de prueba:
SKU	Producto	Categoría	Precio unitario	Stock	Stock mínimo	Ubicación
LAB-001	Guantes de nitrilo	Seguridad	$25.000	50	10	Estante A1
LAB-002	Alcohol industrial	Químicos	$18.500	8	10	Estante B2
LAB-003	Papel filtro	Laboratorio	$12.000	0	5	Estante C1
LAB-004	Pipeta graduada	Instrumentos	$35.000	20	5	Estante A2
LAB-005	Probeta de vidrio	Instrumentos	$42.500	15	5	Estante A3

Estos datos permiten probar productos disponibles, con stock bajo y agotados, además de cálculos monetarios.
2. Suite de casos de prueba
La suite contempla los criterios del documento: SKU único, validaciones, ajustes físicos, búsqueda sin distinción entre mayúsculas y minúsculas, exportación CSV, dashboard, integridad de movimientos, roles y prevención de XSS. 02_casos_de_prueba.md


Convenciones de prioridad:
- P0
   Bloqueante para el piloto.
- P1
   Importante, debe corregirse antes de una liberación normal.
- P2
   Mejora o escenario de menor impacto.
A. Pruebas funcionales
ID	Módulo	Título	Precondiciones	Pasos	Datos de prueba	Resultado esperado	Prioridad	Automatizable
CP-001	Login	Inicio de sesión administrador	Usuario admin creado	1. Abrir login. 2. Ingresar credenciales. 3. Iniciar sesión.	admin / Admin123!	Acceso exitoso con permisos de administrador.	P0	Sí
CP-002	Login	Inicio de sesión operador	Usuario operador creado	1. Abrir login. 2. Ingresar credenciales. 3. Iniciar sesión.	operador / Oper123!	Acceso exitoso con permisos de operador.	P0	Sí
CP-003	Productos	Crear producto correctamente	Sesión como admin	1. Entrar a productos. 2. Crear producto. 3. Guardar.	LAB-006, Microscopio básico, Instrumentos, $850.000, stock 10, mínimo 2, Estante D1.	Producto creado y visible en el listado con todos sus datos.	P1	Sí
CP-004	Productos	Consultar producto registrado	Producto LAB-001 existente	1. Buscar LAB-001. 2. Abrir sus datos.	LAB-001	Se muestran correctamente SKU, nombre, categoría, precio, stock, mínimo y ubicación.	P1	Sí
CP-005	Productos	Editar información del producto	Sesión admin y LAB-001 existente	1. Abrir edición. 2. Cambiar nombre y precio. 3. Guardar.	Precio anterior $25.000; nuevo $27.000.	Los cambios quedan guardados y visibles al consultar nuevamente.	P1	Sí
CP-006	Productos	Eliminar producto sin inconsistencias	Sesión admin y producto existente con movimientos registrados	1. Intentar eliminar el producto. 2. Consultar movimientos e informes relacionados.	LAB-005	La eliminación no deja movimientos huérfanos ni altera incorrectamente los reportes históricos.	P0	Sí
CP-007	Movimientos	Registrar entrada de inventario (IN)	LAB-001 tiene stock 50	1. Seleccionar producto. 2. Registrar entrada. 3. Consultar stock.	IN: 20 unidades	Stock actualizado de 50 a 70 unidades.	P0	Sí
CP-008	Movimientos	Registrar salida de inventario (OUT)	LAB-001 tiene stock 50	1. Seleccionar producto. 2. Registrar salida. 3. Consultar stock.	OUT: 15 unidades	Stock actualizado de 50 a 35 unidades.	P0	Sí
CP-009	Movimientos	Ajustar inventario físico (ADJUST)	LAB-001 tiene stock 50	1. Seleccionar producto. 2. Registrar ajuste. 3. Consultar stock.	ADJUST: 42 unidades	Stock final exactamente 42, sin sumar ni restar automáticamente las 50 unidades anteriores.	P0	Sí
CP-010	Productos	Buscar producto por nombre sin distinguir mayúsculas	LAB-001 registrado	1. Buscar el nombre en minúsculas. 2. Repetir en mayúsculas.	guantes de nitrilo / GUANTES DE NITRILO	Se encuentra el mismo producto en ambas búsquedas.	P1	Sí
CP-011	Productos	Buscar producto por SKU sin distinguir mayúsculas	LAB-001 registrado	1. Buscar SKU en minúsculas. 2. Repetir en mayúsculas.	lab-001 / LAB-001	Ambas búsquedas devuelven LAB-001.	P1	Sí
CP-012	Alertas	Identificar producto con stock bajo	LAB-002 tiene stock 8 y mínimo 10	1. Abrir alertas. 2. Buscar LAB-002.	Stock 8 / mínimo 10	LAB-002 aparece identificado como producto con stock bajo.	P0	Sí
CP-013	Alertas	Identificar producto agotado	LAB-003 tiene stock 0	1. Abrir dashboard. 2. Consultar agotados y alertas.	LAB-003, stock 0	El producto se identifica como agotado y se refleja en las métricas correspondientes.	P0	Sí
CP-014	Dashboard	Consultar total de productos	Cinco productos registrados	1. Abrir dashboard. 2. Consultar total de productos.	LAB-001 a LAB-005	El total corresponde a los cinco productos registrados, según la definición de conteo del sistema.	P1	Sí
CP-015	Dashboard	Calcular valor total del inventario	Productos iniciales registrados	1. Abrir dashboard. 2. Consultar valor del inventario. 3. Comparar con cálculo manual.	Datos iniciales de LAB-001 a LAB-005	Valor esperado: $2.015.000 COP.	P0	Sí
CP-016	Dashboard	Consultar cantidad de movimientos	Movimientos de prueba registrados	1. Registrar dos entradas y una salida. 2. Abrir dashboard.	3 movimientos nuevos	La cantidad de movimientos refleja los registros efectuados, de acuerdo con el alcance del indicador.	P1	Sí
CP-017	Exportación CSV	Exportar todos los productos	Existen cinco productos registrados	1. Abrir exportación. 2. Descargar CSV. 3. Abrir archivo. 4. Comparar registros.	LAB-001 a LAB-005	El CSV contiene todos los productos contemplados por el alcance, con información consistente.	P1	Sí

B. Pruebas negativas y de validación
ID	Módulo	Título	Precondiciones	Pasos	Datos de prueba	Resultado esperado	Prioridad	Automatizable
CP-018	Productos	Impedir SKU duplicado	LAB-001 ya existe	1. Intentar crear otro producto. 2. Usar el mismo SKU.	SKU LAB-001	El sistema rechaza el registro e informa que el SKU ya existe.	P0	Sí
CP-019	Productos	Impedir precio negativo	Sesión admin	1. Crear producto. 2. Ingresar precio negativo. 3. Guardar.	Precio -$15.000	Se rechaza el registro y no se almacena el precio negativo.	P0	Sí
CP-020	Productos	Impedir stock inicial negativo	Sesión admin	1. Crear producto. 2. Ingresar stock negativo. 3. Guardar.	Stock -10	El sistema impide guardar el producto con existencias negativas.	P0	Sí
CP-021	Movimientos	Impedir salida superior al stock disponible	LAB-001 tiene 50 unidades	1. Registrar salida. 2. Ingresar una cantidad superior al stock. 3. Confirmar.	OUT: 60 unidades	Se rechaza la operación y el stock permanece en 50.	P0	Sí
CP-022	Login	Rechazar credenciales incorrectas	Usuario admin existente	1. Ingresar usuario válido. 2. Ingresar contraseña incorrecta. 3. Intentar acceder.	admin / ClaveIncorrecta123	Se deniega el acceso y no se crea una sesión autenticada.	P1	Sí
CP-023	Movimientos	Validar cantidad cero en movimiento	Producto con stock 50	1. Intentar registrar una entrada o salida. 2. Ingresar cantidad 0.	IN: 0 / OUT: 0	No se registra un movimiento sin cantidad válida.	P1	Sí
CP-024	Productos	Impedir campos obligatorios vacíos	Sesión admin	1. Crear producto. 2. Dejar vacío el SKU o nombre. 3. Guardar.	SKU vacío / nombre vacío	El sistema identifica los campos obligatorios y no crea el producto.	P1	Sí

Nota de validación: el documento no especifica expresamente las reglas para cantidades decimales en movimientos ni el comportamiento de movimientos con cantidad cero. CP-023 propone validar esta regla con el responsable del negocio antes de cerrar el criterio de aceptación.
C. Pruebas de seguridad y roles
ID	Módulo	Título	Precondiciones	Pasos	Datos de prueba	Resultado esperado	Prioridad	Automatizable
CP-025	Roles / Productos	Operador no puede crear productos	Sesión como operador	1. Ingresar al módulo productos. 2. Intentar crear un producto.	LAB-007	La operación no está disponible o es rechazada por el servidor. No se crea el producto.	P0	Sí
CP-026	Roles / Productos	Operador no puede editar productos	Sesión como operador y LAB-001 existente	1. Intentar editar LAB-001. 2. Cambiar precio. 3. Guardar.	Precio $30.000	El sistema rechaza la modificación y mantiene el precio original.	P0	Sí
CP-027	Roles / Productos	Operador no puede eliminar productos	Sesión como operador y LAB-005 existente	1. Intentar eliminar LAB-005. 2. Confirmar si aparece la opción.	LAB-005	El sistema impide eliminar el producto y conserva su información.	P0	Sí
CP-028	Seguridad	Prevenir XSS almacenado en campos visibles	Sesión admin	1. Ingresar código JavaScript en nombre, categoría o ubicación. 2. Guardar. 3. Consultar el producto y visualizarlo en dashboard o listado.	<script>alert('XSS')</script>	El contenido se trata como texto o se rechaza; nunca se ejecuta JavaScript en el navegador.	P0	Sí

D. Pruebas de integridad de datos
ID	Módulo	Título	Precondiciones	Pasos	Datos de prueba	Resultado esperado	Prioridad	Automatizable
CP-029	Movimientos / Dashboard	Consistencia entre movimientos, stock y valor del inventario	LAB-001 tiene stock 50 y precio $25.000	1. Registrar IN de 20. 2. Registrar OUT de 15. 3. Consultar stock. 4. Consultar dashboard.	IN +20 / OUT -15	Stock final 55. Valor de LAB-001: $1.375.000 COP. El dashboard debe reflejar el nuevo valor.	P0	Sí
CP-030	Movimientos / Productos	Consistencia después de ADJUST	LAB-001 tiene stock 50 y precio $25.000	1. Registrar ADJUST a 42. 2. Consultar producto. 3. Consultar dashboard.	Stock físico contado: 42	Stock final 42 y valor del producto $1.050.000 COP. No se duplica el inventario.	P0	Sí

E. Prueba End-to-End (E2E) de negocio
ID	Módulo	Título	Precondiciones	Pasos	Datos de prueba	Resultado esperado	Prioridad	Automatizable
CP-031	E2E / Inventario	Flujo completo: ingreso → salida → alerta → exportación	Admin autenticado. Producto LAB-008 creado con stock 5 y mínimo 10.	1. Registrar IN de 10 unidades. 2. Verificar stock 15. 3. Registrar OUT de 8 unidades. 4. Verificar stock 7. 5. Consultar alertas. 6. Exportar CSV. 7. Buscar LAB-008 en el archivo.	SKU LAB-008, Guantes quirúrgicos, precio $30.000, stock inicial 5, mínimo 10, IN +10, OUT -8.	Stock final 7. Se activa la alerta de stock bajo. El CSV contiene LAB-008 con stock actualizado de 7 unidades.	P0	Sí

3. Resumen de cobertura
31
Casos de prueba totales

17
Casos funcionales

7
Negativos y validaciones

4
Seguridad y roles

2
Integridad de datos

1
Escenario E2E



Casos P0
Alta prioridad

18
Casos que se consideran bloqueantes para el piloto si presentan resultados fallidos.

4. Observaciones que deben validarse con el negocio
Como Senior QA, no daría por cerrados algunos criterios sin confirmar estas reglas:
1. Stock mínimo: ¿la alerta se activa cuando el stock es menor o igual al mínimo? La suite utiliza como supuesto que se activa cuando es menor o igual.
2. Eliminación de productos: ¿se permite eliminar productos con movimientos históricos o se debe bloquear su eliminación? En ambos casos se debe preservar la integridad histórica.
3. Cantidades: ¿se permiten cantidades decimales o solamente unidades enteras?
4. CSV: ¿debe exportar productos activos o también productos eliminados?
5. Dashboard: ¿el total de productos incluye los agotados? La suite asume que sí.
Corrección de cálculo para CP-015: con los cinco productos iniciales, el valor correcto del inventario es $2.735.500 COP, no $2.015.000 COP.
5. Recomendación de automatización
Automatizaría inicialmente estos ocho casos con Playwright:
Caso	Justificación
CP-001	Validación recurrente de autenticación.
CP-003	Verificación del registro de productos.
CP-018	Evitar duplicidad de SKU.
CP-007	Validar entradas de inventario.
CP-008	Validar salidas de inventario.
CP-009	Garantizar el comportamiento correcto de ADJUST.
CP-021	Evitar stock negativo.
CP-031	Validar el flujo completo del negocio.

La automatización de estos casos permitiría detectar rápidamente regresiones que comprometan el inventario.

Mi recomendación como QA: no pasaría directamente a automatizar los 31 casos. Primero ejecutaría la suite manualmente, registraría evidencias, documentaría defectos y validaría las reglas pendientes con el responsable del almacén. Después automatizaría los ocho casos prioritarios.

El siguiente paso de tu guía es el refinamiento de la suite: eliminar redundancias, agregar cinco casos de frontera y seleccionar los ocho escenarios de automatización con Playwright.