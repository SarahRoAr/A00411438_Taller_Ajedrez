# Cuestionario de análisis conceptual y arquitectura distribuida

## 1. Conexiones bidireccionales (BiDir) vs. callbacks tradicionales

Normalmente, un cliente envía una petición al servidor. Sin embargo, el juego de ajedrez también necesita que el servidor avise cuando cambie el tablero.

JavaFX puede recibir avisos mediante callbacks tradicionales. El navegador usa una conexión bidireccional de WebSocket: una sola conexión permite enviar movimientos y recibir las actualizaciones.

Para lograrlo, el navegador crea un adaptador sin endpoints, registra su observador y conecta ese adaptador con `connection.setAdapter()`. No crea un puerto nuevo ni espera conexiones externas.

En el servidor, `observer.ice_fixed(current.con)` indica que el observador del navegador corresponde a la conexión WebSocket por la cual llegó el registro. Así, el servidor puede responder a ese cliente por la misma conexión.

## 2. Transporte TCP vs. WebSockets en ZeroC Ice

El servidor usa una sola clase, `ChessServiceImpl`, para atender a todos los clientes y manejar las salas, los turnos y los movimientos.

JavaFX se comunica por TCP, normalmente en el puerto `10000`. El navegador se comunica por WebSocket en el puerto `10002`.

Ambos clientes usan el mismo contrato Slice, aunque utilicen transportes diferentes. Por eso no hace falta duplicar la lógica del juego para cada tipo de cliente.

## 3. Empaquetado del contrato con `slice2js`

La herramienta `slice2js` convierte el contrato `ChessService.ice` en el archivo JavaScript `ChessService.js`. Este archivo define las clases, enumeraciones y conexiones que el navegador necesita.

Por ejemplo, permite crear objetos como `ChessApp.Position` y usar proxies como `ChessApp.ChessServicePrx`.

Primero se carga `Ice.min.js`, que contiene el funcionamiento básico de Ice. Después se carga `ChessService.js`, porque este usa las clases de Ice para registrar el espacio de nombres `ChessApp` en el navegador.

## 4. Validación autoritativa de movimientos

Las casillas verdes que muestra el navegador solo son una ayuda visual. No indican con certeza que un movimiento sea válido.

Antes de aceptar una jugada, el servidor verifica que sea el turno correcto, que exista una pieza propia y que el destino sea permitido. También evita movimientos que dejarían al rey propio en jaque.

El servidor es la fuente principal de la partida. Puede detectar capturas, jaque, jaque mate y tablas, además de rechazar jugadas manipuladas o enviadas por una interfaz desactualizada.
