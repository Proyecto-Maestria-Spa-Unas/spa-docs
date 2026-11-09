# Manual de usuario de NailsSpa

Sistema de gestión y control interno del inventario de un spa de uñas. Este manual explica, paso a paso, cómo usar cada pantalla según el rol de la persona que ingresa.

> **Pendientes** El texto está completo para las pantallas que ya funcionan. Los recuadros «Captura pendiente» se reemplazan por imágenes de la versión de staging cuando estén desplegados el tablero de decisiones (F15) y las correcciones de accesibilidad (F16). Las pantallas de **Historial** y **Proveedores** aún no están implementadas y se describen como «próximamente».

## Contenido

1. [Qué puede hacer cada rol](#1-qué-puede-hacer-cada-rol)
2. [Ingresar, salir y recuperar la contraseña](#2-ingresar-salir-y-recuperar-la-contraseña)
3. [Manual para el Administrador](#3-manual-para-el-administrador)
4. [Manual para el Gerente](#4-manual-para-el-gerente)
5. [Manual para la Manicurista](#5-manual-para-la-manicurista)
6. [Mensajes frecuentes y qué hacer](#6-mensajes-frecuentes-y-qué-hacer)
7. [Consejos de accesibilidad](#7-consejos-de-accesibilidad)
8. [Lista de capturas por tomar](#8-lista-de-capturas-por-tomar)

## 1. Qué puede hacer cada rol

NailsSpa muestra solo las opciones que corresponden a tu rol. Si una opción no aparece en el menú, tu rol no tiene permiso para usarla.

| Opción del menú | Administrador | Gerente | Manicurista |
|---|:-:|:-:|:-:|
| Inventario (consultar) | Sí | Sí | Sí |
| Inventario (crear, editar, descontinuar) | Sí | No | No |
| Detalle del producto con kardex valorizado | Sí | Sí | No |
| Registrar movimiento: salida | Sí | No | Sí |
| Registrar movimiento: entrada | Sí | No | No |
| Historial de movimientos | Todos | Todos | Solo los propios |
| Alertas y tablero: lista de alertas | Sí | Sí | Sí |
| Alertas y tablero: gráficos, exportar datos y mapa | Sí | Sí | No |
| Proveedores | Sí | Sí | Sí |
| Usuarios | Sí | Sí | No |

> **Nota.** El Gerente consulta y analiza, pero no registra movimientos ni modifica el catálogo. La Manicurista solo registra salidas de insumos.

## 2. Ingresar, salir y recuperar la contraseña

### 2.1 Ingresar

1. Abre la dirección de NailsSpa en el navegador. Aparece la pantalla **Ingresar**.
2. Escribe tu **Correo** y tu **Contraseña**.
3. Pulsa **Ingresar**. Si todo está bien, entras a la pantalla de **Inventario**.

> **Captura pendiente (staging):** pantalla de ingreso con los campos Correo y Contraseña.

Si ves el mensaje **«Correo o contraseña incorrectos.»**, revisa que el correo esté bien escrito y vuelve a intentarlo. Si tu usuario fue desactivado, pide al Administrador que lo active.

### 2.2 Cerrar sesión

Pulsa **Salir** en la parte superior derecha del menú. La sesión también se cierra sola a los **30 minutos**; si pasa, vuelve a ingresar.

### 2.3 Recuperar la contraseña

Si olvidaste tu contraseña:

1. En la pantalla de ingreso, pulsa **¿Olvidaste tu contraseña?**.
2. Escribe tu **Correo** y pulsa **Enviar enlace**.
3. La pantalla muestra siempre el mismo mensaje, exista o no la cuenta. Eso protege la privacidad de los usuarios.
4. Abre el correo que recibes y pulsa el enlace. Te lleva a una pantalla para escribir la **nueva contraseña**.
5. Escribe la contraseña dos veces. Debe tener **entre 8 y 128 caracteres** y las dos deben coincidir.
6. Pulsa el botón para guardarla. Aparece **«Contraseña actualizada»**; vuelve a la pantalla de ingreso y entra con la nueva clave.

> **Captura pendiente (staging):** pantalla «Recuperar contraseña» y pantalla «Contraseña actualizada».

Si el enlace del correo venció o ya se usó, la pantalla te avisa y te deja pedir otro desde el paso 1. Si no te llega el correo, revisa la carpeta de spam y espera unos minutos antes de pedirlo de nuevo.

## 3. Manual para el Administrador

El Administrador tiene acceso a todas las opciones del menú: **Inventario**, **Registrar movimiento**, **Historial**, **Alertas y tablero**, **Proveedores** y **Usuarios**.

### 3.1 Consultar el inventario

1. Pulsa **Inventario** en el menú.
2. La tabla muestra **Código**, **Producto**, **Categoría**, **Stock actual**, **Stock mínimo** y **Estado**.
3. Para encontrar un producto:
   - Escribe en **Buscar** (nombre o código).
   - Elige una **Categoría** o **Todas las categorías**.
   - Elige un **Estado** o **Todos los estados**.
   - Marca **Incluir productos descontinuados** si quieres verlos.
4. Usa **Anterior** y **Siguiente** para cambiar de página.

El **Estado** se muestra con ícono y texto, no solo con color:

| Estado | Qué significa |
|---|---|
| ● Disponible | Hay existencias por encima del stock mínimo. |
| ▲ Bajo stock | El stock actual está en el mínimo o por debajo. Conviene reponer. |
| ■ Agotado | No quedan existencias. |
| — Descontinuado | El producto ya no se maneja; se conserva su historial. |

> **Captura pendiente (staging):** tabla de inventario con filtros y los cuatro estados.

### 3.2 Ver el detalle de un producto y su kardex

1. En la tabla, pulsa el **nombre del producto**.
2. Se abre el detalle con el **kardex valorizado**: cada entrada y salida con su cantidad, costo y saldo.
3. Para acotar el periodo, completa **Desde** y **Hasta** y pulsa el botón de filtrar.

El costo del inventario se calcula por **costo promedio ponderado**: cada vez que entra mercancía con costo, el promedio se recalcula. Las salidas se valoran al promedio vigente.

> **Captura pendiente (staging):** detalle de un producto con el kardex.

### 3.3 Crear un producto

1. En **Inventario**, pulsa **+ Crear producto**.
2. Completa el formulario. Los campos con asterisco (\*) son obligatorios:
   - **Código** (por ejemplo, ESM-001). No puede repetirse.
   - **Nombre** (por ejemplo, Esmalte rojo).
   - **Descripción** (opcional).
   - **Categoría** y **Unidad de medida**.
   - **Cantidad inicial**, si el producto ya tiene existencias.
   - **Stock mínimo** (opcional). Si lo dejas vacío, el producto no genera alertas.
3. Pulsa **Guardar producto**. Aparece un aviso de éxito y el producto queda en la tabla.

Si el código ya existe, la pantalla lo indica y no guarda el producto. Si un campo está mal, el mensaje aparece debajo de ese campo.

> **Captura pendiente (staging):** formulario «Crear producto».

### 3.4 Editar, descontinuar y reactivar un producto

- **Editar:** pulsa **Editar** en la fila del producto, cambia los datos y pulsa **Guardar producto**. Si otra persona modificó el mismo producto mientras lo editabas, la pantalla te pide recargar y repetir el cambio.
- **Descontinuar:** pulsa **Descontinuar** cuando el producto ya no se vaya a usar. No se borra: queda marcado y se conserva su historial.
- **Reactivar:** con **Incluir productos descontinuados** marcado, pulsa **Reactivar** para volver a usarlo.

### 3.5 Registrar un movimiento (entrada o salida)

1. Pulsa **Registrar movimiento** en el menú.
2. Elige **Registrar entrada** (llegó mercancía) o **Registrar salida** (se consumió o se dio de baja).
3. Completa el formulario:
   - **Producto**.
   - **Motivo del movimiento** (por ejemplo, compra o consumo de servicio).
   - **Cantidad**.
   - En las entradas por compra, el **Costo unitario** y el **Proveedor**. El costo unitario debe ser el costo neto de compra (precio menos descuentos, más transporte y otros costos directos).
   - **Observación** (opcional).
4. Envía el formulario con el botón del final. Aparece **«¡Movimiento registrado correctamente!»**.

Si después del movimiento el producto queda con stock bajo o agotado, el aviso te lo dice. Si intentas sacar más cantidad de la que hay, la pantalla muestra que el stock es insuficiente y no registra nada. Pulsar el botón dos veces no duplica el movimiento.

> **Captura pendiente (staging):** formulario de entrada y formulario de salida.

### 3.6 Historial y anulación de movimientos *(próximamente)*

La pantalla **Historial** permitirá consultar todos los movimientos, filtrarlos por producto, tipo y fechas, y **anular** un movimiento registrado por error. Los movimientos no se borran: la anulación deja constancia y exige una confirmación explícita.

### 3.7 Alertas y tablero

Pulsa **Alertas y tablero**. La pantalla tiene cuatro partes.

**Alertas de reposición.** Lista los productos que necesitan reponerse, con **Producto**, **Estado**, **Stock actual**, **Stock mínimo** y **Faltante**. Puedes filtrar por **Estado** (bajo stock o agotado). Solo aparecen productos que tienen stock mínimo definido.

**Tablero de decisiones** *(disponible con F15)*. Muestra tres cifras y cuatro gráficos de los últimos 30 días, incluido hoy:

| Indicador | Para qué sirve |
|---|---|
| Valor del inventario | Cuánto vale el stock actual (existencias por costo promedio). Si no hay costos registrados, dice «Sin costos registrados». |
| Consumo de 30 días | Cuántas unidades salieron en el periodo. |
| Productos por estado | Cuántos productos hay disponibles, con bajo stock, agotados y descontinuados. |
| Top de rotación | Los productos que más salen. |
| Consumo por categoría o por producto | Qué se consume más. Cambia la agrupación con la lista **Agrupar el consumo por**. |
| Cobertura estimada (días) | Para cuántos días alcanza el stock al ritmo de consumo actual. Muestra los 10 productos con menos cobertura. |

Cada gráfico tiene un botón **Ver como tabla** con los mismos datos, y **Ver gráfico** para volver.

**Exportar datos.** Descarga un archivo con el inventario o los movimientos:

1. En **Qué descargar**, elige **Inventario** o **Movimientos**.
2. En **Formato**, elige **CSV (se abre en Excel)** o **Parquet (análisis de datos)**.
3. Pulsa **Descargar**. Cuando el archivo está listo, aparece **«Descarga lista»** con su nombre, por ejemplo `inventario-20261108.csv`.

**Proveedores cercanos.** Un mapa con los proveedores ubicados cerca de la sede.

> **Captura pendiente (staging):** lista de alertas; tablero de decisiones con gráficos y con tablas; tarjeta «Exportar datos»; mapa de proveedores cercanos.

### 3.8 Proveedores *(pendiente)*

La pantalla **Proveedores** permitirá consultar, crear y editar proveedores, con su ubicación en el mapa. El NIT no puede repetirse.

### 3.9 Gestionar usuarios

1. Pulsa **Usuarios**. La tabla muestra **Nombre**, **Usuario** (correo), **Rol**, **Estado** y **Acciones**. Puedes ver 10, 25 o 50 usuarios por página.
2. **Crear un usuario:** pulsa **Nuevo usuario**, completa **nombre completo**, **correo**, **Rol** y **Contraseña temporal** (de 8 a 128 caracteres) y pulsa **Guardar**. Entrega la contraseña temporal a la persona por un medio seguro; luego puede cambiarla con **¿Olvidaste tu contraseña?**.
3. **Editar:** en **Acciones**, cambia el nombre o el rol. El correo no se puede modificar.
4. **Activar o desactivar:** usa el interruptor de la fila. El cambio es inmediato. Los usuarios **no se borran**: al desactivarlos, ya no pueden ingresar, pero se conserva su historial.

Los avisos de éxito se cierran solos a los pocos segundos; los de error se quedan hasta que los cierres.

> **Captura pendiente (staging):** tabla de usuarios y formulario «Nuevo usuario».

## 4. Manual para el Gerente

El Gerente consulta y analiza la información. Su menú muestra **Inventario**, **Historial**, **Alertas y tablero**, **Proveedores** y **Usuarios**. No ve **Registrar movimiento**.

| Tarea | Dónde está |
|---|---|
| Consultar el inventario y filtrarlo | [3.1](#31-consultar-el-inventario) |
| Ver el detalle y el kardex valorizado de un producto | [3.2](#32-ver-el-detalle-de-un-producto-y-su-kardex) |
| Revisar las alertas de reposición | [3.7](#37-alertas-y-tablero) |
| Analizar el tablero de decisiones (gráficos y tablas) | [3.7](#37-alertas-y-tablero) |
| Exportar el inventario o los movimientos a CSV o Parquet | [3.7](#37-alertas-y-tablero) |
| Ver los proveedores cercanos en el mapa | [3.7](#37-alertas-y-tablero) |
| Consultar el historial de todos los movimientos *(próximamente)* | [3.6](#36-historial-y-anulación-de-movimientos-próximamente) |
| Gestionar usuarios | [3.9](#39-gestionar-usuarios) |

Lo que el Gerente **no** puede hacer: crear, editar o descontinuar productos, registrar entradas o salidas y anular movimientos. En la tabla de inventario no verá los botones **+ Crear producto**, **Editar** ni **Descontinuar**.

## 5. Manual para la Manicurista

La Manicurista registra el consumo de insumos y revisa qué hay disponible. Su menú muestra **Inventario**, **Registrar movimiento**, **Historial**, **Alertas y tablero** y **Proveedores**. No ve **Usuarios**.

### 5.1 Consultar el inventario

Funciona igual que en [3.1](#31-consultar-el-inventario), solo para consulta: no aparecen los botones para crear, editar ni descontinuar, y no se abre el detalle con el kardex valorizado.

### 5.2 Registrar una salida

Cuando uses un insumo, regístralo para que el inventario siga al día.

1. Pulsa **Registrar movimiento**. Solo verás **Registrar salida**.
2. Elige el **Producto** y el **Motivo del movimiento**.
3. Escribe la **Cantidad** que usaste.
4. Agrega una **Observación** si hace falta y envía el formulario.
5. Verás **«¡Movimiento registrado correctamente!»**. Si el producto queda con stock bajo o agotado, el aviso te lo indica; avísale al Administrador para que lo reponga.

Si escribes más cantidad de la que hay, la pantalla muestra que el stock es insuficiente y no registra nada.

> **Captura pendiente (staging):** formulario «Registrar salida».

### 5.3 Ver mis movimientos *(pendiente)*



### 5.4 Revisar las alertas

En **Alertas y tablero** ves la lista de productos por reponer, con su estado, stock actual, stock mínimo y faltante. No ves los gráficos, la exportación ni el mapa, que son para el Administrador y el Gerente.

## 6. Mensajes frecuentes y qué hacer

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| Correo o contraseña incorrectos. | Los datos no coinciden. | Revisa el correo y la contraseña, o usa **¿Olvidaste tu contraseña?**. |
| Tu sesión venció. Vuelve a ingresar. | Pasaron 30 minutos de sesión. | Ingresa de nuevo. |
| No hay conexión con el servidor. | No se pudo comunicar con el sistema. | Revisa tu conexión a internet y reintenta con el botón **Reintentar**. |
| No tienes permiso para… | Tu rol no puede hacer esa acción. | Pídele al Administrador que la haga o que revise tu rol. |
| El código ya existe. | Hay otro producto con el mismo código. | Usa un código distinto. |
| Stock insuficiente. | Intentas sacar más cantidad de la que hay. | Corrige la cantidad. |
| Otra persona modificó este producto. | Dos personas editaron el mismo producto. | Recarga la página y repite el cambio. |
| El enlace venció o no es válido. | El enlace de recuperación ya no sirve. | Pide uno nuevo desde **Recuperar contraseña**. |

## 7. Consejos de accesibilidad

- Todo el sistema se puede usar con el teclado: **Tab** avanza, **Shift+Tab** retrocede, **Enter** o **Espacio** activan botones y **Esc** cierra las ventanas emergentes.
- Los estados del inventario siempre llevan ícono y texto, no solo color.
- Cada gráfico del tablero tiene su **tabla** con los mismos datos (botón **Ver como tabla**).
- Con zoom del navegador al 200 % el contenido sigue siendo legible; para ampliar, usa **Ctrl** y **+**.

