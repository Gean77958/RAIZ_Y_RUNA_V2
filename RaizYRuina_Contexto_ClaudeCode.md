# RAÍZ Y RUINA — Contexto técnico completo (para Claude Code)

> Preparado a partir de la conversación de diseño y desarrollo con Carlos (Gean Carlos Mandujano Coronel). No contiene propuestas nuevas ni cambios sugeridos — es documentación del estado real. Todo lo marcado NO CONFIRMADO es incertidumbre real, no una suposición disfrazada.

---

## 1. VISIÓN DE RAÍZ Y RUINA

Juego de aventura y puzzle 2D donde Vero, un pequeño espíritu guardián del bosque en formación, reaprende tres gestos de cuidado ambiental (Ofrenda, Memoria, Atención) para revertir la corrupción de su ecosistema. Es una alegoría directa — no literal — de la crisis real de minería ilegal en la Reserva Nacional de Tambopata, Madre de Dios, Perú. El núcleo filosófico: la corrupción del bosque no es un mal externo arbitrario, es la consecuencia de una reciprocidad de cuidado que se rompió. El antagonista (El Guardián) no es "un villano", es la forma que tomó el abandono.

## 2. OBJETIVO DEL PROYECTO Y PRESENTACIÓN

- Proyecto capstone **individual** de Carlos, curso de Desarrollo de Videojuegos, Universidad Continental.
- Debe presentarse **funcionando y jugable** en el parcial de la Semana 8.
- Al momento de esta conversación quedaban aproximadamente 3.5 semanas.
- Vinculado explícitamente al ODS 15 (Vida de Ecosistemas Terrestres), con investigación real de respaldo (ver sección 3).
- GDD elaborado y **ya presentado en clase** en una plantilla de Figma ("Template GDD for Universidad Continental students").

## 3. GDD Y GAMEPLAY

El GDD en Figma (ya presentado) describe una versión **más ambiciosa** que el alcance MVP acordado para el parcial. Esto es importante: **el documento y el build actual no coinciden todavía**, y esa reconciliación queda pendiente (ver sección 15).

**Contenido del GDD tal como está presentado:**
- Género: "Puzzle-adventure 2D/3D"
- Estructura declarada: 1 bosque 2D (3 salas de puzzle) + 1 zona 3D (templo interior) — **la parte 3D fue recortada del MVP** (ver sección 17), pero el documento de Figma aún la menciona.
- Core Loop declarado: "Explorar bosque 2D → activar mecanismo ambiental → resolver puzzle → avanzar y revelar más corrupción → llegar a la grieta → transición a templo 3D → confrontar al Guardián → restaurar el equilibrio." (incluye la transición 3D recortada)
- Sistema de recursos declarado: 4 recursos (Luz Vital, Semilla, Agua, Esencia natural) — **recortado a solo Luz Vital** para el MVP.
- Estilo visual declarado en el documento: "ilustración atmosférica 2D (**pixel art** + escenas pintadas)" — **esto es una inconsistencia real**: en esta conversación se decidió explícitamente abandonar el pixel art por completo a favor de ilustración pintada estilizada. El documento de Figma no se actualizó para reflejar ese cambio. **Bandera para no resolver unilateralmente — confirmar con Carlos cuál versión es la vigente antes de asumir.**
- Aspectos técnicos declarados incluyen "escenas 3D con NavMesh" y "cámara top-down 2D / tercera persona en templo 3D" — mismo caso: recortado del MVP, documento no actualizado.
- Tabla de features del propio GDD ya marca "Transición 2D→3D" como prioridad **Baja** — coherente con la decisión de recortarla.

**Investigación real que respalda el ODS 15 (ya confirmada con fuentes):**
- Reserva Nacional Tambopata, Madre de Dios, Perú — minería ilegal de oro.
- ~500 hectáreas deforestadas entre 2do semestre 2025 e inicios de 2026 (fuente: MAAP #241 / Conservación Amazónica-ACCA).
- ~1,000 personas vinculadas a la actividad ilícita dentro de la reserva; 183 infraestructuras mineras; 67 campamentos detectados.
- Contraste temporal citado como argumento de urgencia: en 2019 (año de la Operación Mercurio) la deforestación anual bajó a menos de 10 hectáreas; hoy son 500.
- 370 campamentos destruidos y +75 operativos estatales en 2026 (la crisis sigue activa, no es histórica).
- Juego de referencia estudiado (benchmark de diseño, NO código reutilizado): "Defensora Shihua" (estudiantes de Toulouse Lautrec + ONG Arbio Perú) — mecánica de metros cuadrados jugados = metros cuadrados reales protegidos en la cuenca del río Las Piedras, Madre de Dios.

## 4. PERSONAJE Y CONTROL

**Vero — protagonista.** Espíritu guardián del bosque en formación. Silueta redondeada, colgante que emite luz. Verbos narrativos: explorar, ofrendar, recordar, atender, sanar.

- Control: top-down 2D, 4 direcciones, `Rigidbody2D`.
- Script: `VeroController.cs` — **IMPLEMENTADO**.
  - `gravityScale = 0`, `freezeRotation = true`, `CollisionDetectionMode2D.Continuous`, `RigidbodyInterpolation2D.Interpolate` — todos seteados en `Awake()`.
  - Input: `Input.GetAxisRaw` (Input Manager legacy) — confirmado funcionando pese a que el proyecto tiene el asset `InputSystem_Actions` presente; no se ha migrado al nuevo Input System.
  - Movimiento suavizado con `Vector2.SmoothDamp` en `FixedUpdate()`.
  - Campos: `velocidad = 4f`, `suavizado = 0.08f`.
- Collider: `Capsule Collider 2D`, `Is Trigger` **desmarcado** (Vero es sólida; los triggers los llevan los objetos interactuables, no ella).
- Tag: `Vero` (creado manualmente, usado por todos los mecanismos para detectarla).
- Arte: sprite pintado real (`Vero_Frente.png`), extraído de una hoja de referencia generada y con fondo removido (herramienta `rembg`, modelo `u2netp`). Reemplazó la cápsula gris placeholder.
- Point Light 2D como hijo de Vero, posicionado sobre el colgante: color `#FCDE5A`, `Volumetric Intensity ≈0.3`, `Outer Radius ≈0.5`.
- **Prefab**: Vero fue convertida en Prefab (`Assets/Prefabs/Vero_Greybox`) y se reutiliza arrastrándola a cada escena nueva. **No usa `DontDestroyOnLoad`** — cada escena tiene su propia instancia local; al cargar una escena nueva, la instancia de la escena anterior se destruye junto con toda la escena (comportamiento por defecto de `SceneManager.LoadScene` en modo Single).

**El Guardián — antagonista/jefe final.** Antiguo protector corrompido por el abandono. Silueta angular, grietas violetas emisivas que transicionan a ámbar al resolverse. Verbos: patrullar, advertir, confrontar, ceder. **Estado: DISEÑADO (arte conceptual + paleta definidos), NO IMPLEMENTADO en ninguna escena todavía.** No existe una escena de Acto 3.

**La Voz de la Raíz Vieja** — presencia narrativa sin modelo ni animación; pensada como fragmentos de texto en un códice/diario. **PENDIENTE**, no implementada.

## 5. ESCENAS

Tres escenas existen actualmente, cada una en su propio archivo de escena:

| Escena | Gesto | Estado mecánico |
|---|---|---|
| `Bosque_SalaRaices` | Ofrenda | IMPLEMENTADO, probado funcionando de punta a punta |
| `Bosque_SalaRunas` | Memoria | IMPLEMENTADO, probado funcionando de punta a punta |
| `Bosque_SalaEspejos` | Atención | IMPLEMENTADO, haz de luz confirmado visible y funcionando |

No existe todavía una escena de Acto 3 (confrontación con El Guardián) — **PENDIENTE**, ni un menú principal — **PENDIENTE**.

Registro en Build Profiles (Unity 6 renombró "Build Settings" a "Build Profiles"): las 3 escenas fueron agregadas a la Scene List. Hubo un momento en que `Bosque_SalaRunas` faltaba en la lista; se dieron instrucciones para agregarla y ordenar la lista como Raíces(0)→Runas(1)→Espejos(2). El usuario confirmó "asunto solucionado" pero **no se re-confirmó visualmente el estado final** — tratar como PARCIAL/NO CONFIRMADO al 100%.

## 6. SISTEMAS YA IMPLEMENTADOS

- Movimiento de Vero — IMPLEMENTADO
- Mecanismo de activación simple (Ofrenda / Raíz+Puente) — IMPLEMENTADO
- Mecanismo de secuencia ordenada (Runas) — IMPLEMENTADO
- Mecanismo de raycasting de luz con rebotes (Espejos) — IMPLEMENTADO
- Patrón Observer vía `UnityEvent` en los tres mecanismos — IMPLEMENTADO
- Iluminación 2D básica (Light2D global + point light en Vero) — IMPLEMENTADO (mood/tono definitivo pendiente de ajuste fino)
- Sistema de progreso global (`GestorProgreso`, singleton) — código **escrito**, integración en escenas **NO CONFIRMADA** (ver sección 7)
- Sistema de restauración ambiental por luz (`RestauradorAmbiente`) — código **escrito**, instalación en las 3 escenas **NO CONFIRMADA**
- Transición entre escenas (`TransicionEscena`) — código **escrito**; objetos `Salida` en las escenas **PENDIENTES** de creación (se dieron las instrucciones, no hubo confirmación de que se hayan creado)

## 7. SCRIPTS C# Y RESPONSABILIDADES

Todos ubicados conceptualmente bajo `Assets/Scripts/` (Player, Mechanisms, Core según carpetas creadas).

| Script | Responsabilidad | Estado |
|---|---|---|
| `VeroController.cs` | Movimiento top-down de Vero | IMPLEMENTADO |
| `MecanismoRaiz.cs` | Detecta a Vero, activa con tecla E, invoca `alOfrecerse` (UnityEvent) | IMPLEMENTADO |
| `PuenteRaiz.cs` | Escucha `alOfrecerse`, habilita su collider (empieza intransitable) | IMPLEMENTADO |
| `Runa.cs` | Runa individual; reporta activación a `SecuenciaRunas`, no sabe si es su turno | IMPLEMENTADO |
| `SecuenciaRunas.cs` | Coordinador de orden correcto; invoca `alCompletarse` al terminar la secuencia | IMPLEMENTADO |
| `Compuerta.cs` | Empieza BLOQUEANDO (polaridad opuesta a `PuenteRaiz`); `AbrirCompuerta()` la abre | IMPLEMENTADO (en Sala de runas); uso en Sala de espejos NO CONFIRMADO |
| `Espejo.cs` | Espejo rotable por el jugador (tecla E, +45° por pulsación) | IMPLEMENTADO |
| `ControladorLuz.cs` | Raycasting del haz de luz con rebotes; invoca `alIluminarSensor` una sola vez | IMPLEMENTADO, confirmado funcionando |
| `GestorProgreso.cs` | Singleton `DontDestroyOnLoad`; cuenta gestos completados y hectáreas restauradas | ESCRITO — integración/wiring en escena NO CONFIRMADA |
| `RestauradorAmbiente.cs` | Interpola color/intensidad del Light2D de cada escena según progreso global | ESCRITO — instalación en escenas NO CONFIRMADA |
| `TransicionEscena.cs` | Carga la siguiente escena al cruzar un trigger | ESCRITO — objetos `Salida` NO CONFIRMADOS como creados |

**Detalle de arquitectura importante para `GestorProgreso` + `RestauradorAmbiente`:** `RestauradorAmbiente` se suscribe al evento `alCambiarProgreso` de `GestorProgreso` **en código, dentro de `Start()`**, no por Inspector — esto es deliberado, porque `GestorProgreso` sobrevive entre escenas (`DontDestroyOnLoad`) pero un listener de Inspector apuntando a un objeto de una escena que se descarga rompería la referencia. Se desuscribe en `OnDestroy()` para evitar excepciones de referencia nula al cambiar de escena. **No revertir este patrón a wiring por Inspector.**

**Ajuste pendiente de una línea, dado pero no confirmado como aplicado:** agregar a `GestorProgreso.cs` la propiedad `PorcentajeRestaurado => gestosTotales > 0 ? (float)gestosCompletados / gestosTotales : 0f`. **NO CONFIRMADO si ya está en el archivo real del proyecto.**

## 8. PREFABS / ASSETS / RECURSOS

- `Assets/Prefabs/Vero_Greybox` — Prefab de Vero, con `VeroController`, `Rigidbody2D`, `Capsule Collider 2D`, `Sprite Renderer` (sprite pintado), Point Light 2D hijo, tag `Vero`.
- `Vero_Frente.png` — sprite pintado de Vero, fondo removido, generado a partir de una hoja de referencia (Gemini) recortada y procesada.
- Hojas de referencia conceptual (turnaround + poses) generadas para Vero y El Guardián, en estilo pintado, paleta definida por hex codes (ver sección 17). Estas hojas son **referencia de diseño**, no assets listos para animar por huesos (para eso se necesitarían partes del cuerpo separadas, no poses completas).
- Estructura de carpetas del proyecto: `Assets/Scenes`, `Assets/Scripts/{Player,Mechanisms,Core}`, `Assets/Art/{Vero,Guardian}`, `Assets/Prefabs`, `Assets/Tilemaps`.
- No hay assets de audio todavía. No hay tilemaps de fondo reales (las salas usan geometría placeholder / cuadrados de color).

## 9. INTERACCIÓN

Patrón consistente en los tres mecanismos: un `Collider2D` marcado `Is Trigger` detecta cuándo Vero (por tag) entra/sale de un radio de interacción; mientras está dentro, `Input.GetKeyDown(KeyCode.E)` dispara la acción. Cada mecanismo **reporta** su activación mediante un `UnityEvent` propio (`alOfrecerse`, `alCompletarse` vía `IntentarActivar`, `alIluminarSensor`), sin conocer directamente a quien escucha — así se conectan recompensas (puentes, compuertas) sin acoplar el código.

## 10. PUZZLES

1. **Ofrenda (Sala de raíces):** activación simple. Vero se acerca a la raíz, presiona E, la raíz se activa y el puente se forma. IMPLEMENTADO.
2. **Memoria (Sala de runas):** secuencia ordenada de 3 runas (`Runa_A`, `Runa_B`, `Runa_C`), coordinadas por `GestorRunas` (objeto con `SecuenciaRunas`). El orden correcto se define arrastrando las runas en el orden deseado en el Inspector — no depende de su posición visual. Activar fuera de orden reinicia la secuencia tras 0.6s (feedback rojo). IMPLEMENTADO.
3. **Atención (Sala de espejos):** un `Emisor` fijo dispara un rayo (`Physics2D.Raycast`) que rebota en `Espejo_1` y `Espejo_2` (rotables por el jugador con tecla E, +45° por pulsación) hasta llegar a un `Sensor`. El camino se recalcula y dibuja cada frame con un `LineRenderer`. IMPLEMENTADO y confirmado visualmente funcionando (haz amarillo visible).

## 11. RESTAURACIÓN DEL ECOSISTEMA

Sistema de dos partes:
- `GestorProgreso` (singleton persistente) cuenta cuántos de los 3 gestos se completaron y calcula hectáreas restauradas proporcionalmente.
- `RestauradorAmbiente` (uno por escena, en el Light2D global de cada sala) interpola color e intensidad de luz desde un tono corrupto (violeta apagado) hacia uno sano (verde cálido) según el progreso global.

Este sistema reemplaza conceptualmente el "0%/50%/100%" del GDD de Figma, pero implementado como **interpolación continua 0–1**, no como 3 estados discretos con arte distinto. Es una decisión de diseño tomada explícitamente para caber en el tiempo disponible sin requerir arte de fondo adicional por estado. **No revertir a un sistema de sprites de fondo intercambiables sin discutirlo — fue una decisión deliberada de alcance.**

## 12. ÍNDICE DE RESTAURACIÓN

Anclado a datos reales: `hectareasTotales = 500f` (las hectáreas deforestadas reales en Tambopata, fuente MAAP #241), `gestosTotales = 3`. Cada gesto completado restaura simbólicamente 500/3 ≈ 166.67 hectáreas. `HectareasRestauradas` se expone como propiedad pública de `GestorProgreso` para que la UI (pendiente) la muestre en pantalla.

## 13. UI / HUD

**Estado: PENDIENTE. No existe ningún Canvas ni elemento de UI implementado todavía.** El GDD de Figma menciona una "UI de progreso" (prioridad Media) — no construida. Es el paso 3 del plan MVP acordado (barra de progreso + texto de hectáreas, alimentado por `GestorProgreso.HectareasRestauradas` y `PorcentajeRestaurado`).

## 14. PROGRESIÓN

- Progresión entre salas: mediante `TransicionEscena.cs` en objetos `Salida` — **código listo, objetos NO CONFIRMADOS como creados** en ninguna escena todavía.
- Progresión narrativa/de restauración: mediante `GestorProgreso` — código listo, wiring de los 3 mecanismos hacia `RegistrarGestoCompletado()` **NO CONFIRMADO**.
- No hay sistema de guardado (decisión explícita del GDD: "Persistencia: sin guardado, fuera de alcance").
- No hay niveles de experiencia ni desbloqueos adicionales más allá de avanzar de sala en sala.

## 15. SISTEMAS PENDIENTES

- Objetos `Salida` (transición entre escenas) en las 3 salas
- Confirmar/completar wiring de `GestorProgreso.RegistrarGestoCompletado()` en los 3 mecanismos
- Confirmar/instalar `RestauradorAmbiente` en el Light2D de las 3 escenas
- Agregar la propiedad `PorcentajeRestaurado` a `GestorProgreso.cs` (si no está ya)
- UI de progreso (barra + hectáreas)
- Menú principal y menú de pausa
- Escena de Acto 3 (confrontación con El Guardián) — **en 2D**, no en 3D
- Arte/implementación de El Guardián en escena (hoy solo existe como concepto)
- Códice/diario narrativo con los datos reales de Tambopata
- Audio ambiental por sala
- Reconciliar el documento GDD de Figma con el alcance MVP real (pixel art vs. pintado, recorte de 3D, recorte de recursos)
- Build final compilado y probado en otra máquina

## 16. ERRORES O PROBLEMAS CONOCIDOS

Historial de errores ya encontrados y resueltos (útil para no repetir el diagnóstico si reaparecen patrones similares):
- Vero sin ningún Collider2D al inicio → los triggers nunca se disparaban, sin ningún error en consola (causa: sin gravedad, Vero nunca "chocaba" con nada, así que la ausencia pasó desapercibida). Resuelto agregando Capsule Collider 2D.
- Confusión recurrente de arrastrar el objeto incorrecto a un campo del Inspector (ej. el campo `Sprite` de la raíz apuntando a Vero por error; el campo `Emisor` de `ControladorLuz` apuntando al propio `ControladorLuz` en vez de al triángulo `Emisor`).
- `Puente_Greybox` tenía originalmente el script `MecanismoRaiz` en vez de `PuenteRaiz`, y su collider marcado `Is Trigger` cuando debía estar sólido.
- Line Renderer de `ControladorLuz` con `Materials → Element 0` en `None` → haz invisible sin error de consola. Resuelto asignando el material `Default-Line`.
- Un `UnassignedReferenceException` sobre el campo `emisor` apareció en la Console incluso después de que el haz ya se veía funcionando en el Game view — se interpretó como un mensaje viejo no limpiado de la Console (Console no se limpia automáticamente entre sesiones de Play). **NO CONFIRMADO al 100% que no sea un problema real intermitente** — si Claude Code encuentra este error de nuevo, vale la pena verificarlo con la Console limpia antes de asumir que es residual.
- Unity 6 renombró "Build Settings" a "Build Profiles" — no es un bug, solo reubicación de menú, causó confusión inicial.
- Escena `Bosque_SalaRunas` faltante en la lista de Build Profiles en un momento dado — atribuido a que no estaba abierta al usar "Add Open Scenes".

## 17. DECISIONES DE DISEÑO QUE YA ESTÁN TOMADAS

- Dirección de arte: **ilustración pintada estilizada** (painterly), explícitamente NO pixel art, NO fotorrealista. Referencias: Ori and the Blind Forest, Hollow Knight, Spiritfarer, GRIS.
- Paleta Vero: cuerpo/capa `#4B7A3B`, sombra `#2F5A28`, piel `#D9A066`, colgante/ojos `#F2A623`→`#FCDE5A`, acentos de hoja `#8FBF6B`.
- Paleta El Guardián: corteza/piedra `#4A3B2E` / `#55524A`, grietas de corrupción `#8B3FA0`→`#B47FD1`, ojos `#C23B6B`, musgo enfermo `#5C6B4A`.
- El vínculo ODS 15 está anclado específicamente a la Reserva Nacional Tambopata (no es un "bosque genérico"), con datos reales citables.
- El Acto 3 (confrontación con El Guardián) se resuelve en **2D**, no en 3D — se eliminó la zona 3D explorable y la IA por NavMesh del alcance MVP, por restricción de tiempo (proyecto individual, 3.5 semanas). El Guardián no es destruido; es confrontado con los 3 gestos reaprendidos, y sus grietas pasan de violeta a ámbar.
- Sistema de recursos recortado a un único recurso: **Luz Vital**. Semilla, Agua y Esencia natural (mencionados en el GDD de Figma) NO se implementan.
- No hay fauna ambiental ni enemigos con IA de combate. No hay sistema de derrota tradicional ni combate letal.
- No se parte de ningún proyecto Unity de terceros (se evaluó y se descartó explícitamente — ver sección siguiente).
- El sistema de restauración se implementa como interpolación continua de luz (0–1), no como 3 estados discretos de arte de fondo distinto.
- Arquitectura basada en `UnityEvent` para desacoplar mecanismos de sus recompensas (patrón Observer), consistente en los 3 puzzles.
- `RestauradorAmbiente` se suscribe a `GestorProgreso` en código (no por Inspector) por la razón técnica explicada en la sección 7.

## 18. QUÉ NO DEBEMOS CAMBIAR SIN CONSULTARME

- No reintroducir la transición 2D→3D ni el templo 3D en el alcance del MVP, aunque el GDD de Figma todavía lo mencione.
- No reintroducir los 4 recursos del GDD original; el juego usa solo Luz Vital.
- No agregar combate, enemigos con IA o condición de derrota tradicional.
- No cambiar el patrón `UnityEvent`/Observer por otro paradigma (ej. interfaces, singletons acoplados) sin discutirlo — es una decisión de arquitectura ya tomada y ya evaluada en rúbrica técnica.
- No decidir unilateralmente si el estilo de arte final es "pixel art" o "pintado" — el GDD de Figma dice una cosa, la conversación de diseño decidió otra. **Esto se resuelve preguntando a Carlos, no asumiendo ninguna de las dos.**
- No forkear ni integrar código de proyectos Unity de terceros (Lost in the Woods, Forest Assassin, u otros) — evaluado y descartado explícitamente.
- No tomar código del juego "Defensora Shihua" — es propiedad de terceros (Toulouse Lautrec + Arbio Perú), sin licencia de uso; solo sirve como referencia de diseño.
- No mover el sistema de restauración de "interpolación continua" a "3 sprites de fondo discretos" sin confirmarlo — fue una decisión deliberada de alcance.

## 19. PRIORIDADES PARA EL MVP

En orden de impacto acordado:
1. Conectar las 3 escenas (objetos `Salida` + confirmar Build Profiles)
2. UI de progreso (barra + hectáreas restauradas)
3. Menú principal + menú de pausa
4. Acto 3 en 2D (El Guardián, los 3 gestos, transición final de color)
5. Contenido narrativo (códice con datos de Tambopata) + audio ambiental
6. Build final compilado y probado en otra máquina

## 20. ESTADO ACTUAL DEL PROYECTO

Estimación honesta discutida en la conversación: **entre 10% y 15% del alcance total del GDD de Figma**, pero un porcentaje mucho más alto del **riesgo técnico** ya está resuelto — las tres arquitecturas de mecánica distintas (activación simple, secuencia, raycasting) ya están construidas y probadas, que es la parte más impredecible de estimar. Lo que falta (conectar escenas, UI, menús, audio, contenido narrativo) es trabajo más predecible en tiempo. El juego, hoy, **no es jugable de principio a fin todavía** porque las escenas no están conectadas y no hay Acto 3.

---

## CONTEXTO PARA CLAUDE CODE

**Proyecto:** Raíz y Ruina — Unity 6 (6000.0.43f1 LTS), URP 2D, C#, Windows. Capstone individual, ODS 15 anclado a la crisis real de minería ilegal en la Reserva Nacional Tambopata (Madre de Dios, Perú). Entrega funcional en ~3.5 semanas desde el inicio de esta conversación.

**Arquitectura central:** patrón Observer vía `UnityEvent` en cada mecanismo de puzzle — los mecanismos no conocen a sus recompensas, solo invocan un evento (`alOfrecerse`, `alCompletarse`, `alIluminarSensor`) al que otros objetos se suscriben desde el Inspector. Excepción deliberada: `RestauradorAmbiente` se suscribe a `GestorProgreso` (singleton `DontDestroyOnLoad`) **en código** dentro de `Start()`/`OnDestroy()`, no por Inspector, porque los listeners de Inspector no sobreviven cambios de escena cuando el emisor del evento sí sobrevive.

**3 escenas existentes, cada una con su mecánica funcionando y probada:**
- `Bosque_SalaRaices` — Ofrenda (activación simple: `MecanismoRaiz` + `PuenteRaiz`)
- `Bosque_SalaRunas` — Memoria (secuencia ordenada: `Runa` + `SecuenciaRunas` + `Compuerta`)
- `Bosque_SalaEspejos` — Atención (raycasting con rebotes: `Espejo` + `ControladorLuz`)

**Vero** (protagonista, prefab en `Assets/Prefabs/Vero_Greybox`): `VeroController.cs`, Rigidbody2D top-down sin gravedad, Capsule Collider 2D sólido, tag `Vero`, sprite pintado real ya integrado (`Vero_Frente.png`), Point Light 2D hija en el colgante.

**Sistemas escritos pero con integración NO CONFIRMADA en el proyecto real:** `GestorProgreso.cs` (progreso global + hectáreas restauradas, ancladas a 500 ha reales de Tambopata / 3 gestos), `RestauradorAmbiente.cs` (interpola Light2D según progreso), `TransicionEscena.cs` (carga de escena al cruzar un trigger `Salida`). **Verificar el estado real de estos tres en el proyecto antes de asumir que están wireados.**

**Lo NO implementado:** UI/HUD completo, menú principal y de pausa, Acto 3 (El Guardián, diseñado conceptualmente pero sin escena), códice narrativo, audio.

**Decisiones firmes de alcance (no revertir sin preguntar a Carlos):** sin transición 2D→3D, sin combate/enemigos con IA, un solo recurso (Luz Vital), sin sistema de guardado, arte pintado estilizado (no pixel art — aunque el GDD de Figma presentado en clase todavía dice "pixel art", es una inconsistencia de documentación pendiente de corregir, no una instrucción vigente).

**Discrepancia activa a resolver con el usuario, no unilateralmente:** el GDD de Figma ya presentado en clase describe un alcance mayor (3D, 4 recursos, pixel art) que el MVP acordado en esta conversación. Ambas cosas coexisten hoy; Claude Code debe tratar el MVP descrito arriba como la fuente de verdad para el desarrollo, y señalar la discrepancia si el usuario pide implementar algo del documento de Figma que contradiga estas decisiones.
