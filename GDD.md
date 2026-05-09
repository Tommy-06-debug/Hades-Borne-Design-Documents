# GDD: PROYECTO "GATO MAFIAS" (Nombre Provisional)

> [!summary]- 1. Visión General
> 
> - **Género:** Action-RPG ligero (inspiración Soulslike).
>     
> - **Cámara:** Cenital (Top-Down) o isometrico.
>     
> - **Estética:** Estilizada y "Cozy" (vibras Cult of the Lamb / Tunic). Fuerte contraste entre lo tierno del mundo (gatos, patitos) y la brutalidad mecánica del combate.
>     
> - **Filosofía de Diseño:** Profundidad mecánica sobre cantidad de contenido. Diseño ortogonal, combate de empuje (Push-forward combat) y sinergia estricta entre estados de personaje.
>     

## ⚔️ 2. Pilares de Jugabilidad y Bucle Central

> [!quote]- El Bucle Sinergético (La Regla de Oro del Combate)
> 
> - **Fase de Riesgo (Cuerpo a Cuerpo):** El jugador usa ataques melee para dañar la postura (equilibrio) del enemigo. Cada golpe exitoso recarga munición de las armas de fuego.
>     
> - **Fase de Ejecución (Distancia):** Al agotar la postura enemiga, este entra en estado Stunned. El jugador usa un dash para ganar distancia segura y descarga su munición (obtenida en la fase 1) para infligir daño masivo a la salud real del enemigo. Las balas son ineficaces contra enemigos con la postura intacta.
>     

- **Sistema de Postura (Estilo Sekiro adaptado):** Los enemigos tienen una barra invisible de equilibrio. Se desgasta con ataques melee y parries perfectos.
    
- **Interfaz Dielgética:** El estado Stunned (postura rota) se representa visualmente con patitos amarillos girando sobre la cabeza del enemigo. No hay barras de estado flotantes que ensucien la pantalla.
  El enemigo marcado tiene un circulo en los pies que lo marca.
    
- **Salud Segmentada:** 9 Vidas (estilo Hollow Knight). Cada golpe recibido resta una unidad entera, sin matemáticas ocultas de mitigación de daño.
    
- **Letalidad:** Sin barras de vida para enemigos comunes; el jugador debe aprender el ritmo y la cantidad de balas/golpes necesarios para matar. Las barras de vida aplican solo para Jefes.
    

> [!info]- Sistema de Letalidad Progresiva
> 
> - **Daño Base (Ranged):** Las armas a distancia infligen un daño mínimo (aprox. 20-30%) a enemigos con la barra de postura activa. Útil para control de masas débiles.
>     
> - **Bonificador por Ejecución (Stun Multiplier):** Al impactar a un enemigo en estado Stunned, el daño aumenta un +150%. Esto incentiva el uso de armas de fuego como herramientas de finalización.
>     

## 🎮 3. Esquema de Controles

Todas las acciones de combate y evasión están mapeadas en los gatillos/bumpers. Los pulgares nunca deben abandonar los sticks analógicos.

| Input                 | Acción Principal                | Función Técnica                                                                                                                                           |
| --------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Stick Izquierdo**   | Movimiento                      | Desplazamiento del personaje.                                                                                                                             |
| **Stick Derecho**     | Fijación de enemigos (softlock) | Fija un enemigo o cambia el enemigo fijado, de tal forma que el player no se gira hacia el enemigo pero en el caso de atacar el ataquese dirige hacia el. |
| **R2 (Gatillo Der.)** | Disparo a distancia             | Gasto de recurso / Daño a salud.                                                                                                                          |
| **R1 (Bumper Der.)**  | Ataque Melee                    | Generación de recurso / Daño a postura.                                                                                                                   |
| **L2 (Gatillo Izq.)** | Dash / Esquiva                  | Un solo botón dedicado de pulsación única para evitar Input Lag. Otorga frames de invulnerabilidad y reposicionamiento.                                   |
| **L1 (Bumper Izq.)**  | Parry                           | Acción estática de alto riesgo/alta recompensa para quebrar drásticamente la postura enemiga si se ejecuta con el timing perfecto.                        |

## ⚙️ 4. Sistemas de Personaje (FSM)

El jugador se rige por transiciones lógicas estrictas para asegurar que los controles respondan fluidamente:

> [!note]- Estados de Movimiento Base
> 
> - **Idle:** Estado de reposo.
>     
> - **Move:** Desplazamiento dictado por el Stick Izquierdo.
>     

> [!note]- Estado: Dash
> 
> - **Condición de entrada:** Pulsación única del Bumper L2 (Trigger instantáneo).
>     
> - **Lógica:** Otorga inmediatamente un flag booleano de invulnerabilidad (is_invincible = true). Desplaza al personaje a alta velocidad en la dirección actual del Stick Izquierdo (o hacia donde mira si el stick está neutro) por una cantidad fija de frames o distancia. Ignora colisiones de hitbox de daño enemigas.
>     
> - **Salidas:** Transición forzada a Idle al finalizar la animación. No puede cancelarse a sí mismo para evitar spam infinito (requiere un cooldown interno mínimo).
>     

> [!note]- Estado: Melee Attack
> 
> - **Condición de entrada:** Pulsación del Bumper R1.
>     
> - **Lógica:** Fija la rotación del personaje hacia el vector del Stick Derecho. Activa una hitbox de colisión en arco frente al personaje. Si la hitbox detecta una entidad enemiga, ejecuta evento: Damage_Posture() y evento: Add_Ammo().
>     
> - **Salidas:** Termina la animación y vuelve a Idle. Puede ser interrumpido por Dash (Cancelación por evasión) o Hit Reaction.
>     

> [!note]- Estado: Ranged Attack (Ofensiva Táctica)
> 
> - **Condición de entrada:** Pulsación única del Gatillo R2 (Semi-automático, no mantener) Y munición actual > 0.
>     
> - **Lógica:** El movimiento de las piernas sigue activo (Move en paralelo), pero la velocidad general se reduce un 10% durante la animación de disparo para dar peso mecánico. El torso se orienta hacia el enemigo fijado e instancia un único proyectil rápido. Ejecuta evento: Consume_Ammo() y aplica un recoil visual y físico. Si el proyectil impacta, ejecuta evento: Damage_Health() (escala masivamente si el objetivo tiene la postura rota).
>     
> - **Salidas:** Requiere que finalice la recuperación del arma antes de transicionar a Idle/Move. Puede ser cancelado de emergencia por un Dash.
>     

> [!note]- Estado: Parry (Active)
> 
> - **Condición de entrada:** Pulsación del Bumper L1.
>     
> - **Lógica:** Reduce la velocidad de movimiento a 0 y activa una ventana estricta de frames activos (ej. 10 frames). Si una hitbox de daño enemiga colisiona durante esta ventana, se anula el daño, se ejecuta un gran impacto al evento Damage_Posture() del atacante y se reproduce un efecto visual/sonoro. Si falla, el jugador queda expuesto al recovery de la animación.
>     
> - **Salidas:** Finaliza la animación y vuelve a Idle. Interrumpido por Hit Reaction si el ataque impacta fuera de la ventana activa.
>     

> [!note]- Estado: Hit Reaction (Jugador)
> 
> - **Condición de entrada:** Colisión con hitbox enemiga sin flags de invulnerabilidad o Parry activo.
>     
> - **Lógica:** No anula los inputs del jugador. Resta -1 a la variable de Vidas. Desencadena un Hitstop global, aplica fuerza de Knockback y activa invulnerabilidad por 1.5 segundos con parpadeo.
>     
> - **Salidas:** Se resuelve casi instantáneamente volviendo al estado de control presionado.
>     

## 🔫 6. Arsenal (Matriz Ortogonal)

El juego cuenta con un total de 6 armas base:

### 🗡️ Armas Cuerpo a Cuerpo

> [!quote]- 1. Navaja Mariposa / Puño Americano (Arma Base)
> 
> - **Función:** Combate sostenido y seguridad táctica.
>     
> - **Input:** Pulsaciones consecutivas de R1 (Combo de hasta 3 golpes).
>     
> - **Ejecución:** El jugador conserva el 60% de velocidad. Hitbox en cono corto y estrecho. Daño de postura moderado por golpe.
>     
> - **Economía:** Solo el tercer golpe recarga munición, obligando a mantener la presión.
>     
> - **Riesgo:** Muy bajo. Frames de recuperación mínimos, cancelable con Dash casi siempre.
>     

> [!quote]- 2. Katana Yakuza (Embestida Ofensiva)
> 
> - **Función:** Reposicionamiento rápido y cazar retaguardia.
>     
> - **Input:** Pulsación única de R1 con cooldown moderado.
>     
> - **Ejecución:** Sobrescribe temporalmente el estado Move. El jugador sale disparado a gran velocidad. El jugador _es_ la hitbox y atraviesa enemigos. I-frames exclusivos durante el desplazamiento. Daño medio-alto a todo lo atravesado.
>     
> - **Economía:** Genera 1 bala por cada enemigo atravesado, excelente retorno si se alinean objetivos.
>     
> - **Riesgo:** Altísimo. No puede cancelarse con Dash durante el trayecto ni en el envainado.
>     

> [!quote]- 3. Bate de Béisbol (Control de Masas)
> 
> - **Función:** Botón de pánico táctico para despejar zonas.
>     
> - **Input:** Pulsación pesada de R1.
>     
> - **Ejecución:** Velocidad reducida al 10%. Arco masivo de 180 grados. Aplica un Knockback brutal a los impactados (daño extra contra muros).
>     
> - **Economía:** Munición proporcional a la cantidad de enemigos impactados.
>     
> - **Riesgo:** Medio-alto. Animación de preparación lenta. Cancelable en primeros frames, in-cancelable una vez inicia el golpe. El Knockback compensa la lentitud.
>     

### 🎯 Armas a Distancia

> [!quote]- 1. Revólver (Arma Base)
> 
> - **Capacidad:** 6 cargas.
>     
> - **Input:** Pulsación única (Semi-automático).
>     
> - **Ejecución:** Conserva el 90% de velocidad. Instancia 1 proyectil Hitscan o Rigidbody rápido. Retroceso visual moderado en cámara.
>     
> - **Cooldown/Riesgo:** Recuperación corta (ej. 0.3 seg). Cancelable por Dash en cualquier momento del cooldown.
>     

> [!quote]- 2. Escopeta (Arma de Riesgo)
> 
> - **Capacidad:** 2 a 3 cargas.
>     
> - **Input:** Pulsación única (Semi-automático pesado).
>     
> - **Ejecución:** Conserva 100% de velocidad, pero recibe Knockback físico hacia atrás. Múltiples proyectiles en cono amplio (rango corto). Daño individual bajo, requiere quemarropa para aplicar el Stun Multiplier. Shake violento en cámara.
>     
> - **Cooldown/Riesgo:** Animación de bombeo obligatoria (ej. 0.8 seg) donde la velocidad baja al 50%. Si se cancela con Dash, el arma no estará lista para disparar.
>     

> [!quote]- 3. Fusil de Precisión (Rango Largo)
> 
> - **Capacidad:** 1 o 2 cargas máximas.
>     
> - **Input:** Mantener pulsado para apuntar, soltar para disparar.
>     
> - **Ejecución:** Velocidad de movimiento a 0 al apuntar. El personaje se ancla. Proyectil Hitscan con Pierce_Count infinito. Daño 100% al primero, 20% a los subsiguientes.
>     
> - **Cooldown/Riesgo:** Animación larga y pesada (ej. 1.2 seg). SÍ puede cancelarse con Dash para sobrevivir, pero pierde la bala cargada.
>