# ÚLTIMO REPORTE — SalaEspejos CERRADA: observación, restauración final y salida (`Bosque_SalaEspejos`)

**Estado: la sala se jugó de punta a punta con teclado real, en una sola partida desde el spawn. Se resolvió el Puzzle 1, aparecieron las 2 frases, se hizo la restauración final, se habilitó la Salida y cruzarla cargó `Bosque_ActoFinal`. Consola: 0 errores, 0 warnings. Escena guardada.** No hay segundo puzzle ni código nuevo.

## F1. Qué se hizo
- **Ajuste:** en `Reaccion_Herida`, la intensidad final de `Luz_Herida` pasó de 0.9 a **0.6**.
- **Mensajes:** SalaEspejos no tenía Canvas de mensajes.
  - Copié `Canvas_Mensajes` (con su `Texto`: LegacyRuntime 48, blanco, centrado, con sombra) desde `Bosque_SalaRunas`. Para copiarlo abrí SalaRunas en modo aditivo y la cerré **sin guardar**; la escena no tenía cambios antes de cerrarla y su archivo sigue con fecha del 28/09.
  - Quedó bajo un nuevo `Reflexiones`, que solo tiene `MensajesReflexion` con los mismos tiempos que en Runas: fundido 0.4 s y permanencia 1.8 s.
  - No se copiaron `PausaVero` ni `SecuenciaReflexionFinal`, porque nada debe bloquear el avance.
- **Observación (`Observacion`, 2 `ZonaVero`, triggers de 1.5 × 6):**

  | Zona | Posición | Frase |
  |---|---|---|
  | `ZonaVero_1_Conexion` | x = 19.0, entre las raíces y el agua | "El agua, el suelo y la vida no existen por separado." |
  | `ZonaVero_2_Atencion` | x = 29.5, tras la vegetación que revive | "Prestar atención también es cuidar." |

  - **Las dos empiezan desactivadas** y las activa `Reaccion_Herida.alTerminar`. Así no se pueden disparar atravesando la niebla antes de resolver el puzzle.
- **Restauración final (`Zona_Herida/Reaccion_Final`, `ReaccionAmbiental2D`, 2.5 s):**
  - la Light 2D global sube de 1.0 a **1.25**;
  - aparecen 4 plantas sanas repartidas por **toda** la herida, cada una en un tramo: `Final_Mineria` (51, x 5.2), `Final_Suelo` (21, x 11.6), `Final_Raices` (54, hongos, x 18.3) y `Final_Agua` (51, x 21.2). Crecen desde escala 0, escalonadas cada 0.3 s.
  - Toda la vegetación sana queda crecida de forma permanente: `ReaccionAmbiental2D` se ejecuta una sola vez y no revierte.
- **`ZonaVero_2.alEntrar`** dispara tres cosas: `MensajesReflexion.Mostrar("Prestar atención…")`, `Reaccion_Final.Ejecutar` y `Salida.BoxCollider2D.enabled = true`.
- **Salida:**
  - la **moví al final del recorrido**, a (36.5, −0.25) sobre el tramo D. Estaba en x = 7, detrás del punto donde ahora se habilita.
  - Su collider empieza **desactivado**.
  - `TransicionEscena` no se tocó: `escenaDestino` = `Bosque_ActoFinal`, `requiereGestoCompletado` = false. ActoFinal está en Build Settings y habilitada.

## F2. Cadena completa de eventos (todo por Inspector)
```
ControladorLuz.alIluminarSensor → NotificadorProgreso.RegistrarGesto
                                → Reaccion_Herida.Ejecutar
Reaccion_Herida.alTerminar      → ZonaVero_1.SetActive(true), ZonaVero_2.SetActive(true)
ZonaVero_1.alEntrar             → MensajesReflexion.Mostrar(frase 1)
ZonaVero_2.alEntrar             → MensajesReflexion.Mostrar(frase 2)
                                → Reaccion_Final.Ejecutar
                                → Salida BoxCollider2D.enabled = true
Salida (TransicionEscena)       → SceneManager.LoadScene("Bosque_ActoFinal")
```

## F3. Prueba final con teclado real (una sola partida, registro cada 0.1 s)
| t (s) | Evento |
|---|---|
| 11.0 | Spawn en x = −7.94. Zonas inactivas, Salida desactivada, niebla opaca. |
| 12.9 | Espejo_1 + E → 45° |
| 15.0 | Espejo_2 + E → 45°. **El rayo llega al Sensor** y empieza la revelación. |
| 20.2 | Termina `Reaccion_Herida`: niebla en 0, `Luz_Herida` en 0.6. **Se activan las 2 ZonaVero.** |
| 25.8 | x = 18.7 → aparece la frase 1 (alfa 1, texto correcto) y se desvanece sola. |
| 63.1 | x = 28.0 → aparece la frase 2 con tildes correctas (comprobado en captura de pantalla). **Se habilita la Salida.** La luz global sube a 1.25 y crecen las plantas finales. |
| 82.5 | x ≈ 34.4 → Vero toca la Salida → **se carga `Bosque_ActoFinal`**. |

Vero no se detuvo en ningún momento: las frases no piden input. Consola: **0 errores y 0 warnings**.

## F4. Notas
- **Recuadro azul al borde de la imagen:** en algunas capturas aparece un recuadro azul en el borde, más allá de la pared derecha. Es porque el Game view del Editor estaba muy panorámico (aspecto 2.99). Los límites de la cámara y el fondo están calculados para 16:9, y a 16:9 el fondo cubre toda la cámara.
- **Al llegar a ActoFinal**, Vero cae al vacío en esa escena. Es un problema de ActoFinal, que no se tocó.
- **Archivos modificados en total por el trabajo de SalaEspejos:** `Bosque_SalaEspejos.unity` y `ControladorLuz.cs` (1 línea, B2). No se tocó ninguna otra escena, ni el prefab de Vero, ni `VeroController` ni `TransicionEscena`.

---

# Reporte anterior — SalaEspejos: Zona A + Puzzle 1 con arte real y "la herida" revelada por la luz (`Bosque_SalaEspejos`)

**Estado: el Puzzle 1 se resolvió con teclado real desde el spawn. La niebla se disipa, aparece la herida, la luz de la zona sube y la vegetación revive. Consola: 0 errores, 0 warnings. Escena guardada.** No se escribió código nuevo. Las fases 5–6 (observación y segundo puzzle) no se tocaron.

## E0. Bloques anteriores en esta escena (resumen)
- **B2:** arreglo de 1 línea en `ControladorLuz.cs`. Tras rebotar, el rayo sale desde `hit.point + direccion * 0.01f`. Antes volvía a chocar con el mismo espejo a distancia 0.
- **B3:** `Vero_Greybox` en la capa `Ignore Raycast`, solo como override de esta escena. `capasDetectables` del ControladorLuz = todo excepto `Ignore Raycast`. Vero ya no corta el rayo.
- **Disposición del puzzle:** Emisor (−4.5, 3.6) a 270°, Espejo_1 (−5.0, −1.2), Espejo_2 (1.0, −1.2), Sensor (0.5, 3.6). Se resuelve girando cada espejo una vez (45°).
- **Suelo base (capa Suelo):**

  | Tramo | x | Tope y |
  |---|---|---|
  | A | −12 → 6 | −2.5 |
  | B | 6 → 14 | −1.5 |
  | C | 14 → 24 | −2.5 |
  | D | 24 → 38 | −1.25 |

  Hay paredes en los extremos y un `RetornoCaida_Fondo` que devuelve a Vero a (−8, −1).
- **Overrides de Vero en esta escena** (iguales a SalaRunas): `capaSuelo` = Suelo, hijo `PuntoSuelo` (0, −0.72), Animator con `Vero_Greybox.controller` en `Visual`, `fuerzaSalto` 11.5. No se tocó el script ni el prefab.

## E1. Qué se hizo en este bloque
- **Cámara:** `CamaraSeguimiento` en Main Camera, con objetivo = Vero y suavizado 5.
  - Los límites se calcularon con los bordes de la sala (x −13 a 39), tamaño ortográfico 5 y pantalla 16:9: `limiteMin` (−4.11, 0.5), `limiteMax` (30.11, 2.0).
  - Con y mínima 0.5 nunca se ve por debajo de la base del suelo (−4.5).
- **Salida:** su `BoxCollider2D` está **desactivado**. El objeto sigue en (7, −1.69) con `TransicionEscena` → Bosque_ActoFinal, a la espera del bloque final. Antes, cruzar x ≈ 7 cargaba ActoFinal a mitad del recorrido.
- **Emisor:**
  - Su cuadrado se ocultó (SpriteRenderer desactivado). El transform se mantiene, porque es el origen y la dirección del rayo.
  - El arte es un objeto aparte, `Santuario_Emisor` (`Runas_Asset_40`, escala 1.4, sin rotar), con la espiral sobre el origen del rayo.
  - Lleva un hijo `Luz_Santuario` (Light2D, copia de la luz de Vero, radio 3) con `PulsoAmbiental`: base 0.8, amplitud 0.3.
- **Espejos y Sensor:** los cuadrados se ocultaron y cada uno lleva un hijo visual: `Visual_Espejo` (`Prop_Espejo`, slice `04_Espejo_0`) y `Visual_Sensor` (`Prop_Sensor`). El visual del espejo gira con el espejo al pulsar E. **No se tocaron colliders, posiciones ni rotaciones.**
- **Haz:** el ancho del LineRenderer del ControladorLuz pasó de 1 a **0.25**. Con 1 u tapaba el santuario. Es solo ancho; se puede revertir.
- **Fondo:** `Fondo_Lejano`, el mismo sprite y el mismo parallax 0.3 que SalaRunas. Está en la sorting layer **Default** con orden −50, para no depender de añadir una sorting layer a la Global Light.
- **Zona A (entrada):** solo 4 elementos sanos: `A_Pilar` (38), `A_Helecho` (51), `A_Rocas` (23) y `A_Plantas` (49).
- **Vero:** el orden de dibujo de `Visual` pasó a 10 (el mismo override que en SalaRunas), para quedar por encima de la niebla.

## E2. "La herida" (`Zona_Herida`), en orden de lectura de izquierda a derecha
| Eslabón | Objetos (`Runas_Asset`) | x aprox. | Después de resolver |
|---|---|---|---|
| Minería | Bomba 01, Motor 06, Cilindro 10, Máquina 07 | 3–12 | **se queda** (huella) |
| Suelo alterado | Fosa 02, Tierra removida 11 | 10–16.5 | se queda |
| Raíces afectadas | Raíces arrancadas 18, Tocón 16, Árbol caído 14, Árbol seco 13 | 14.6–20 | se queda |
| Agua afectada | Charcas 31, 29, 33 | 19.3–24.2 | se queda |
| Vegetación dañada | Secas 17, 19, 20 (`VegetacionSeca`) | 24–28 | **se desvanecen** (alfa → 0) |
| Vegetación que revive | `VegetacionViva`: 21 y 51 sobre las secas, 51 junto a las raíces, 21 junto al agua, 49 al final (x 27.6–32.4, la zona "sin vida") | — | **crecen** desde escala 0 |

- **Niebla (`Niebla_Herida`):** cubre x ≈ 2.3–33.8 en dos capas.
  - `Velo_1..8`: `SoftSquare.png` de 2DGamekit, color (0.30, 0.26, 0.36), alfa 0.97, orden 4.
  - `Niebla_1..8`: `Mist.png` de 2DGamekit, alfa 0.93, orden 6.
  - Todo usa **Sprite-Lit-Default**, no el material original.
  - El velo fue necesario porque `Mist.png` es casi transparente (alfa máximo 106/255, promedio unos 14). Solo, apenas ocultaba la herida. Con el velo debajo se ven siluetas pero no se distingue qué son.
  - Queda por encima del ambiente y por debajo de Vero.
- **Luz de la zona:** `Luz_Herida`, un Light2D puntual en (17, 2.5), radio 14, verde cálido. Su intensidad sube de 0 a 0.9.
- **Reacción:** `Reaccion_Herida` (`ReaccionAmbiental2D`), con duración 2 s y 25 sprites:
  - velos y volutas: alfa a 0, escalonado de izquierda a derecha cada 0.15 s;
  - plantas secas: alfa a 0, empezando a 1.6 s;
  - plantas vivas: escala de 0 a 1, empezando entre 1.8 y 3.1 s;
  - `Luz_Herida`: empieza a 0.3 s.
- **Cadena de eventos (por Inspector):** `ControladorLuz.alIluminarSensor` → `NotificadorProgreso.RegistrarGesto` (ya existía) **y** `Reaccion_Herida.Ejecutar` (nuevo).

## E3. Prueba con teclado real (desde el spawn, una sola partida)
1. Vero aparece en x = −7.94. Rayo con 0°/0°: rebota en Espejo_1 hacia arriba y no completa. La niebla está opaca.
2. Con D llega a Espejo_1 (x = −5.02) y pulsa E: Espejo_1 = 45°. El rayo cruza hasta Espejo_2 y regresa.
3. Con D llega a Espejo_2 (x = 0.94) y pulsa E: Espejo_2 = 45°. **El rayo llega al Sensor.**
4. Línea de tiempo de la reacción (registro cada 0.1 s; t = 0 es el impacto en el Sensor):

   | t | Velo_1 | Velo_8 | Mist | Planta seca | Planta viva | Luz |
   |---|---|---|---|---|---|---|
   | +0.5 s | 0.79 | — | — | — | — | 0.04 |
   | +1.0 s | 0.43 | 0.75 | — | — | — | 0.30 |
   | +1.5 s | 0.10 | 0.38 | — | — | — | 0.64 |
   | +2.1 s | 0 | 0.08 | — | 0.84 | — | 0.88 |
   | +3.1 s | — | 0 | — | 0.13 | 0.29 | — |
   | +3.6 s | — | — | — | 0 | 0.88 | — |
   | +4.6 s | — | — | — | — | completa (1.6) | 0.90 |

   Las máquinas siguieron con alfa 1 todo el tiempo.
5. Siguió con D hasta x = 30.6, saltando los desniveles de x 6 y x 24. Pasó por x = 7 sin cambiar de escena. La cámara siguió a Vero hasta su límite (30.1).
6. Consola: **0 errores y 0 warnings** en toda la prueba.

## E4. Pendiente o a revisar (para Carlos)
- **Fuerza de `Luz_Herida`:** con 0.9 aclara bastante el fondo y da un tono verdoso lechoso. Si se ve demasiado, bajar `intensidadFinal` a ~0.6 en `Reaccion_Herida`. *(Aplicado en el bloque final: 0.6.)*
- **El velo empieza en x ≈ 2.3:** junto a Espejo_2 se nota un borde vertical suave.
- **Agua:** las charcas no cambian, porque así lo pediste. Quedan como huella hasta la Fase 6.
- **Colliders de los espejos:** siguen siendo los cuadrados de 2.5 × 2.6. El rayo gira en la esquina del collider, un poco por encima del óvalo visible.
- **Plantas vivas:** crecen desde su centro (pivote central del sprite). No cambié el pivote porque se comparte con SalaRunas.
- **El suelo son bloques greybox**, no tiles.
- **Sin probar:** SalaEspejos viniendo desde SalaRaices_v2 → SalaRunas. Probándola sola, la luz global queda violeta porque `GestorProgreso` no existe.

---

# Reporte anterior — Vida "La memoria del bosque" implementada; SalaRunas probada de punta a punta (`Bosque_SalaRunas`)

**Estado: las tres runas funcionan y se probaron seguidas con teclado real, en una sola partida desde el spawn. Escena guardada.** El Final (Salida) no se tocó.

## V0. Secuencia final de reflexión (última pieza de Runas)
- **Antes no existía:** al completar Vida solo aparecía "El bosque recuerda…" con `MensajesReflexion`.
- **`PanelDialogo.cs` sin modificar:** copié su Canvas desde `Bosque_SalaRaices_v2` → `Canvas_PanelDialogo` en SalaRunas, sin el contador de vidas de Raíces. `SalaRaices_v2` se abrió en modo aditivo, se copió y se cerró **sin guardar**; el archivo no cambió (última modificación: 26/09). En la copia, `duracionAutoOcultar` pasó de 5 a 8 s para que la línea de Tambopata no se oculte antes de tiempo.
- **Script nuevo `SecuenciaReflexionFinal.cs`:** llama a `PanelDialogo.MostrarTexto` / `Ocultar` con las 4 líneas, en orden, **una sola vez**, y congela a Vero con `PausaVero` durante toda la secuencia. Hace falta porque `PanelDialogo` solo muestra un texto; en Raíces el orden lo pone `EspirituBosque`, que avanza con E.
- **Disparo:** `Vida_ReaccionSantuario.alTerminar` → `SecuenciaReflexionFinal.Iniciar()`. Quité de ese evento la llamada suelta a `MensajesReflexion.Mostrar("El bosque recuerda…")`, porque la secuencia empieza con esa misma frase y se habría visto dos veces.
- **Prueba real** (partida completa con teclado desde el spawn), registro del panel:
  - 515.5 s: "El bosque recuerda lo que nosotros olvidamos." (3.5 s) → pausa 0.8 s
  - 519.8 s: "Cuando una parte del bosque desaparece, la vida que depende de ella también cambia." (4.5 s) → pausa
  - 525.2 s: "Entre 2025 e inicios de 2026, la minería ilegal deforestó 500 hectáreas dentro de la Reserva Nacional Tambopata." (6 s) → pausa
  - 532.0 s: "ODS 15 — Vida de ecosistemas terrestres." (3.5 s) → se oculta
  - Vero quieta durante toda la secuencia (con D pulsada no se movió); control recuperado a los 536 s. Se mostró una sola vez.
- **Salida:** sigue **siempre activa** (`requiereGestoCompletado = false`), como estaba. No la condicioné a la secuencia porque eso es del Final.

## V1. Qué se hizo (todo bajo `Zona_Vida`)
- **Cornisas one-way** (`Runas_Asset_37`, `PlatformEffector2D`, capa Suelo): `Vida_Cornisa_1` (x 70.3–71.3, tope 1.24), `_2` (71.3–72.3, 2.24) y `_3` (72.3–73.3, 3.24). No se tocaron tiles, las columnas del santuario, las plantas ni el Ascensor_05.
- **`RetornoCaida`:**
  - `Vida_FondoFoso` (x 63.8–70.8, y −2.5 a −0.7): lleva al borde de P1 (61.2, 0.5). Ya no se puede quedar debajo del Ascensor_05.
  - `Vida_FondoBorde` (x 79.9–86.1, y −13 a 3): lleva a (78.0, 5.3). Antes se podía caer fuera del mundo.
- **Glifos** (`Vida_Glifos`, **desactivado hasta que termina Tierra**). Cada uno tiene base `Runas_Asset_55`, icono, `Brillo` y `Runa`:

| Punto | Posición | Glifo | Icono |
|---|---|---|---|
| P1 | (60.9) borde del foso | Hoja | `Art/Glifos/Glifo_Hoja.png` (a partir de `leaf.png`) |
| P2 | (71.8) cornisa 2 | Animal | `Art/Glifos/Glifo_Animal.png` (**silueta nueva**: ave genérica, sin especie concreta) |
| P3 | (76.0) santuario | Agua | `Art/Glifos/Glifo_Agua.png` (copia de `WaterDrop.png`) |
| P4 | (79.1) santuario | Raíz | `LowerPlants4_1` |

- **`Vida_SecuenciaGlifos`:** una segunda `SecuenciaRunas`, independiente de `GestorRunas`. Orden: Agua (P3) → Raíz (P4) → Hoja (P1) → Animal (P2).
- **Demostración:** 4 símbolos sobre la puerta (`Vida_SimbolosPuerta`), apagados. Cada vez que Vero entra en la zona de la puerta (x 76.9–77.7), se encienden uno por uno en orden y se apagan.
- **Script nuevo `MemoriaSantuario.cs`:** hace la demostración y controla cuándo se puede pulsar E en cada glifo. Lo bloquea durante la demostración, si el glifo ya está acertado y al completar la secuencia. Para ello activa o desactiva el componente `Runa`; `Runa` y `SecuenciaRunas` no se modificaron.
- **Runa_C = la puerta en espiral:** en (77.3, 6.6), con el sprite oculto, el collider desactivado y un `Brillo` cálido de radio 3.5.
- **Al completar los 4 glifos:**
  - `MemoriaSantuario.Completar()` y `Vida_ReaccionSantuario`: la puerta, las columnas, las raíces y las plantas pasan de apagadas a su color, los 4 símbolos quedan encendidos y se enciende la luz del santuario.
  - Luego `GestorRunas.IntentarActivar(Runa_C)`, los brillos de Runa_C (1.6), Runa_A (2.0) y Runa_B (2.0), `PausaVero.Pausar(2.6)` y el mensaje *"El bosque recuerda lo que nosotros olvidamos."*
- **Tierra:** solo se añadió **un listener** a `Tierra_ReaccionRuna.alTerminar` → `Vida_Glifos.SetActive(true)`.
- **Gesto global:** `GestorRunas.alCompletarse` sigue teniendo un solo `RegistrarGesto`. Se dispara una vez, al completar Agua → Tierra → Vida.

## V2. Prueba de punta a punta con teclado real (una sola partida desde el spawn)
| Paso | Resultado |
|---|---|
| **Agua:** orilla → Ascensor_01 → losa → Ascensor_02 → bomba (E) → dique (E) | ✅ Runa_A (índice 1), pausa y mensaje. Se desbloquea Tierra |
| **Tierra:** corte 2 → corte 1 → Ascensor_04 → cima → corte 3 → regreso | ✅ Runa_B (índice 2), pausa y mensaje. Se desbloquea Vida |
| Túnel → Ascensor_05 → cornisas → santuario | ✅ |
| Demostración al pasar por la puerta | ✅ glifos bloqueados durante la demostración y reactivados al terminar |
| Error a propósito (E en Raíz primero) | ✅ la secuencia no avanza, sin castigo |
| Agua (P3) → E repetido en Agua | ✅ avanza a 1; el E repetido no hace nada (glifo bloqueado) |
| Raíz (P4) → Hoja (P1, bajando por el foso) → Animal (P2, cornisa 2) | ✅ 2 → 3 → 4 |
| Runa_C | ✅ secuencia principal en 3/3 (completada) |
| Pausa y mensaje | ✅ con D pulsada 0.9 s, Vero no se movió; texto visible |
| Control recuperado | ✅ |

**Incidencias durante la partida** (todas de enganches o de mi script de prueba; ninguna rompió la progresión):
- **Enganche contra paredes al saltar con D mantenida** en la escalera de la meseta (escalones 6→7 y 7→8) y en la entrada del túnel (x 48.76). Se sale soltando D y saltando de nuevo.
- Con el Ascensor_04 abajo, su costado bloquea el paso hacia la derecha hasta que sube.
- La Salida (Final) no se probó.

---

# Reporte anterior — Tierra "La red rota" implementada y probada (`Bosque_SalaRunas`)

**Estado: implementada, probada con teclado real de punta a punta y guardada.** Vida y Final no se tocaron.

## T1. Qué se hizo
Todo va bajo el padre **`Zona_Tierra`**.
- **Lectura visual:** mangueras mineras (`Runas_Asset_03`, no se tocan) → cortes → raíces reales → red restaurada. Las raíces de los cortes son **raíces de árbol reales** (`LowerPlants4_x`, del 2DGamekit), no mangueras.

| Corte | Raíz | Trigger (x) | Qué pasa al repararlo (E) |
|---|---|---|---|
| 1. Maquinaria | `Tierra_Raiz_1`, arco `LowerPlants4_2` sobre la máquina `Runas_Asset_06` (38.3, 1.35) | 37.5–39.5 | La raíz recupera color y tamaño y la máquina se apaga (se oscurece) |
| 2. Borde del foso | `Tierra_Raiz_2`, `LowerPlants4_3` colgando sobre el foso (33.2, −0.9) | 33.8–35.2 | La raíz recupera color y tamaño |
| 3. Cima de la meseta | `Tierra_Raiz_3`, `LowerPlants4_1` (54.3, 10.9) | 53.3–55.9 | La raíz recupera color y tamaño |

- **Antes de repararlas,** las raíces se ven secas: marrón grisáceo, alfa 0.9, escala 0.85. Cada corte tiene un `Brillo` ámbar que pulsa como pista.
- **`Tierra_Cortes`** (los 3 triggers con `MecanismoRaiz`) **empieza desactivado.** Las raíces se ven siempre, pero no se pueden reparar hasta completar Agua.
- **`Tierra_Contador`** (`ContadorRestauracion`, total 3) cuenta los cortes en cualquier orden. Al completarse:
  - `Tierra_ReaccionRed`: una ola recorre las raíces reales existentes de x 57 a x 42 (`LowerPlants4_7`, `4_3 (1)`, `4_4 (2)`, `4_1`, `4_4 (1)`, `4_1 (1)`, `4_4 (3)`, el tocón `Runas_Asset_16 (4)` y el tronco `Runas_Asset_18`), que pasan de secas a su color natural, y se enciende una luz cálida.
  - Se activa `Tierra_ZonaRuna`.
- **`Tierra_ZonaRuna`** (`ZonaVero`, x 39–41): cuando Vero **vuelve** frente a la runa, se ejecuta `Tierra_ReaccionRuna`: se apaga la máquina `Runas_Asset_07`, crecen 3 brotes y se enciende la luz de la runa. Al terminar:
  - `SecuenciaRunas.IntentarActivar(Runa_B)`
  - `Brillo` de Runa_B a 1.4
  - `PausaVero.Pausar(2.6)`
  - `MensajesReflexion.Mostrar("Se rompe la red que sostiene la vida.")`
- **Runa_B:** movida de (−1.28, −17.93) a **(40.0, 0.9)**. Tiene la piedra con espiral, el collider desactivado (no reacciona a E) y brillo y color activo en ámbar.
- **`Tierra_FondoFoso`** con `RetornoCaida` en el foso x 26.8–33.8. Devuelve a Vero a (35.2, 0.9).
- **Scripts nuevos:** `ContadorRestauracion.cs` y `ZonaVero.cs`. No se modificó ningún script existente.
- **Agua:** solo se añadió **un listener** a `Agua_ReaccionRestauracion.alTerminar` → `Tierra_Cortes.SetActive(true)`. Su lógica no cambió.

## T2. Cambio en Ascensor_04 (probado antes de tocarlo)
Con su `PuntoA` en y 2.12, **subir al ascensor desde abajo era imposible en la práctica:**
- **Desde la izquierda,** su collider (y 1.42–2.25) sobresale por encima de los escalones (y 1 y 2) y Vero choca con la cabeza.
- **Desde la derecha,** el ascensor no se detiene abajo y empieza a subir mientras Vero salta, así que choca contra su costado. Fallaron todos los intentos con teclado real.

**Cambio:** solo el `PuntoA` del Ascensor_04, de y 2.12 a **1.25**. Ahora su parte superior queda a 1.38, alcanzable con un salto desde el suelo. En la prueba subió al primer intento. El `PuntoB`, la velocidad y el script no se tocaron.

**Acceso al corte 3:** desde el ascensor arriba se sube por los escalones de la meseta (5 → 6 → 7 → 8). **El último escalón (8 → 10) mide 2 u, justo el límite del salto:** si se mantiene D pegado a la pared, Vero se queda enganchada. Hay que saltar y pulsar D en el aire, y así se llega a la cima. El corte 3 quedó en la cima, como estaba previsto.

## T3. Prueba con teclado real (de punta a punta)
| Paso | Resultado |
|---|---|
| Agua completa (spawn → bomba → dique → Runa_A) | ✅ índice 1, pausa y mensaje. `Tierra_Cortes` se activa al terminar (C0 → C1) |
| Corte 2 (borde del foso), E | ✅ solo ese corte (r = 1) |
| Corte 1 (máquina), E | ✅ r = 2 |
| Ascensor_04 → escalones → cima → corte 3, E | ✅ r = 3: se activan la ola y `Tierra_ZonaRuna` |
| Regreso caminando hacia Runa_B (sin teletransporte) | ✅ al entrar en la zona despierta Runa_B: índice 2 y color ámbar |
| Pausa | ✅ con A pulsada 0.9 s, Vero no se movió (x 40.21 → 40.21) y el mensaje estaba visible (alfa 1) |
| Control recuperado | ✅ con D pulsada 0.6 s, x 40.21 → 42.07 |

**Fallo encontrado y corregido durante la prueba:** los triggers de los cortes 1 y 2 estaban demasiado juntos, y con Vero entre ambos un solo E reparaba los dos. Los separé (hueco de 2.3 u, más que el ancho de Vero, 1.42 u) y repetí la prueba entera.

## T3b. Ajuste final: escalón de la cima (Tierra cerrada)
- **Tile:** se quitó **un solo tile** de `Grid/Suelo`, la **celda (62, 12)** (RuleTile `TilesetRockRules`; mundo x 53.76–54.76, y 9–10). El salto de 2 u (8 → 10) queda en dos escalones de 1 u (8 → 9 → 10). La RuleTile redibujó el borde con césped. Ningún otro tile cambió.
- **Objetos movidos por el cambio:** `Tierra_Raiz_3`, `Tierra_Corte_3` y su `Brillo` se movieron **+1 u en x**, porque la raíz estaba sobre esa columna. La planta `LowerPlants3_15 (4)` se movió **+0.6 u en x** para que no quede flotando sobre el nuevo borde.
- **Prueba con teclado real:** desde el suelo junto al Ascensor_04 (Vero colocada ahí solo para la prueba), subida al ascensor y luego escalones hasta la cima, **solo con D mantenida y saltos normales.** Después, 3 de 3 subidas desde el escalón 8 hasta la cima, en 1.3–1.8 s cada una, sin engancharse.
- **Nota:** en la primera subida Vero se enganchó una vez en el escalón **6 → 7** (1 u, sin cambios). Es el mismo enganche de siempre: la cápsula de Vero se pega a cualquier pared si salta con D mantenida y queda contra la cara del escalón. Al soltar y volver a pulsar D, sigue. La solución general sería un material de física sin fricción (en Vero o en `Suelo`). No lo hice porque no lo pediste.

## T4. Pendiente o a revisar
- ~~Salto de 2 u a la cima~~: resuelto en T3b.
- **Enganche general contra paredes** al saltar con D mantenida (ver la nota de T3b). Afecta a cualquier escalón.
- **El `RetornoCaida` del foso devuelve a Vero al lado de Tierra (35.2, 0.9).** Si cae desde el lado del Agua, aparece más adelante. No rompe nada, porque los cortes siguen bloqueados hasta completar Agua, pero es un atajo.
- **Aspecto visual:** las raíces dañadas se leen, pero el arco sobre la máquina es fino y tapa parte de ella, y la ola bajo la meseta no se ve desde la cima (se ve al regresar).
- **Hallazgo de la inspección:** las piezas "raíz" de x 33–45 son mangueras mineras (`Runas_Asset_03`). Ahora se usan así en la narrativa y no se tocaron.

---

# Reporte anterior — Zona del Agua "Seguir el veneno" implementada y probada (`Bosque_SalaRunas`)

**Estado: implementada, probada con teclado real y guardada.** Solo la zona del Agua. Tierra, Vida y el momento final no se tocaron.

## 0. Última pasada: pausa, mensaje y tubería pintada
- **Scripts nuevos:**
  - `PausaVero.cs`, método `Pausar(float)`. Usa `VeroController.puedeMoverse`, sin modificar `VeroController`.
  - `MensajesReflexion.cs`, método `Mostrar(string)`. Fundido de 0.4 s, 1.8 s visible y fundido de 0.4 s.
- **Objeto nuevo `Reflexiones`:** lleva los dos componentes y un `Canvas_Mensajes` en overlay. El texto usa la fuente legacy, cursiva, sombra, abajo al centro. En el proyecto no hay fuentes TMP importadas.
- **`Agua_ReaccionRestauracion.alTerminar` ahora también llama a:**
  - `PausaVero.Pausar(2.6)`, que es lo que dura el mensaje;
  - `MensajesReflexion.Mostrar("El río no solo lleva agua. Lleva vida.")`.
- **Tubería pintada:** dos capas azules (`Agua_FondoChorroLimpio` y `Agua_FondoCharcoLimpio`) son **hijas de `Fondo_Medio_02`**, así se mueven con su parallax, y van en la capa `FondoMedio`, orden 1. Tapan el chorro y el charco verdes pintados. Aparecen con la restauración (alfa 0 → 0.8 y 0 → 0.5), así que después la tubería pinta agua limpia. El fondo no se modificó.

**Prueba con teclado real:** spawn → orilla → Ascensor_01 → losa → Ascensor_02 → bomba (E) → dique (E) → restauración → Runa_A activa (índice 1) → `puedeMoverse = False` y mensaje visible (alfa 1). Con D pulsada 0.9 s durante la pausa, Vero **no se movió** (x 19.86 → 19.86). El control volvió a los ~2.6 s y, con D pulsada 0.6 s después, Vero **sí se movió** (x 19.86 → 21.73). ✅

Nota: en el primer intento Vero cayó al agua al subir al Ascensor_01. Fue un fallo del script de prueba (un umbral de espera demasiado estricto hizo saltar a Vero con el ascensor lejos), no del juego. El `RetornoCaida` la devolvió a la orilla y el segundo intento completó todo.

## 1. Qué se hizo
Todo lo nuevo está bajo el padre **`Zona_Agua`**. Solo se usaron sprites del set de SalaRunas; no hay arte nuevo.

| GameObject | Qué es | Posición |
|---|---|---|
| `Agua_Bomba` | Sprite `Runas_Asset_01_R1C5` + `MecanismoRaiz` + trigger + `Brillo` rojo | Sobre la losa superior, (27, 7.26). Se llega con Ascensor_02 |
| `Agua_Tuberia_Boca`, `_V0`, `_H1`, `_H2`, `_V1`, `_V2`, `_H3` | Codo `Runas_Asset_04_R1C2` + manguera `Runas_Asset_03_R1C1` | Desde la bomba bajan por la pared (x 24.3), cruzan por encima de la losa de la runa (y 3.0) y terminan en la boca (16.85, 1.38) |
| `Agua_ChorroVerde` | `Square` verde translúcido | Cae de la boca (x 16.2) al agua |
| `Agua_Superficie` | `Square` translúcido: verde → azul | Foso x 4.8–23.8, y −5 a −2.9 |
| `Agua_Fondo` + `Agua_PuntoRetorno` | Trigger + `RetornoCaida` (el existente) | Mismo rectángulo que el agua. Devuelve a la orilla izquierda (4.2, 0.9) |
| `Agua_Dique` | Tronco `Runas_Asset_18_R2C1` + `MecanismoRaiz` (**empieza desactivado**) + `Brillo` azul (empieza apagado) | Losa, (20.3, 1.57) |
| `Agua_CanalLimpio`, `Agua_ChorroLimpio` | `Square` azul, alfa 0 al inicio | Del dique al borde de la losa y la caída al agua |
| `Agua_Brote_1..3` | Plantas `Runas_Asset_51` y `_21`, escala 0 al inicio | Orilla izquierda (3.4 y 5.2) y suelo derecho (24.9) |
| `Agua_LuzRestaurada` | Light2D (copia del `Brillo`), intensidad 0 → 0.7 | Sobre el estanque (12, −1.5) |
| `Agua_ReaccionBomba`, `Agua_ReaccionRestauracion` | `ReaccionAmbiental2D` | — |

**Runa_A (existente):** movida a la losa (18.8, 1.78) y con el sprite de la piedra con espiral `Runas_Asset_48_R4C7`. **Su `CircleCollider2D` está desactivado**, así que no reacciona a E. Colores: inactivo gris azulado, activa cian. `Brillo` en cian, radio 3, pulso tenue (0.15 ± 0.08).

**Script nuevo, el único:** `Assets/Scripts/Mechanisms/ReaccionAmbiental2D.cs`. Interpola color/alfa y escala de sprites y la intensidad de luces 2D, cada cambio con su propio retraso. Aplica el estado "dañado" en `Awake` y dispara `alTerminar` al acabar.

**No se modificó:** `VeroController`, los ascensores, SalaRaices_v2, ningún script existente ni ningún objeto del mapa, aparte de Runa_A.

## 2. Cadena de eventos (todo por Inspector)
1. **Bomba**, E con Vero en rango (`MecanismoRaiz.alOfrecerse`):
   - `Agua_ReaccionBomba.Ejecutar()`: el chorro verde desaparece y la bomba se oscurece (1.2 s).
   - Activa el `MecanismoRaiz` del dique.
   - Enciende el `Brillo` del dique, que indica el siguiente paso sin texto.
2. **Dique**, E (`alOfrecerse`) → `Agua_ReaccionRestauracion.Ejecutar()`, unos 4.6 s en total:
   - El dique se disuelve.
   - Aparecen el canal y el chorro limpio.
   - El agua pasa de verde a azul.
   - El charco tóxico `Runas_Asset_31` pierde el tinte verde.
   - Las plantas `HangingPlants2_2 (6)`, `Runas_Asset_51_R5C2_0 (1)` y `LowerPlants4_6` pasan de secas a su color natural.
   - Crecen los 3 brotes.
   - Se enciende la luz del estanque.
3. **Al terminar la reacción** (`alTerminar`):
   - `GestorRunas.SecuenciaRunas.IntentarActivar(Runa_A)`: la runa se muestra activa y la secuencia pasa a índice 1.
   - `Brillo` de Runa_A → `SetIntensidadBase(1.4)`.

`GestorRunas.alCompletarse` sigue igual. Solo se dispara con las 3 runas, así que el gesto global todavía no se registra.

## 3. Prueba con teclado real (de principio a fin)
Hecha con `keybd_event` sobre la ventana de RaizYRuina y clic real en el Game view. Registro cada 0.1 s: posición de Vero, ascensores, estado de los mecanismos, índice de la secuencia y color de la runa.

| Paso | Resultado |
|---|---|
| Spawn → orilla izquierda (D + saltos) | ✅ x 4.66, en el suelo |
| Subir al Ascensor_01 en A y cruzar el estanque | ✅ Vero es llevada por el ascensor (x 7.2 → 14.8) sin caer |
| Saltar a la losa de la runa | ✅ x 18.23, y 1.51 |
| Ascensor_02 arriba → caminar a la bomba → **E** | ✅ `bomba.yaActivado = True`, dique habilitado |
| Volver a bajar en Ascensor_02 → dique → **E** | ✅ `dique.yaActivado = True` |
| Esperar la reacción (5 s) | ✅ `indiceActual = 1`, Runa_A con color activo (G = 1.00), agua azul, chorro verde con alfa 0, brotes a escala natural, luz en 0.7 |
| Caerse al agua desde la losa | ✅ `RetornoCaida` devuelve a Vero a (4.20, 0.50) |

## 4. Pendiente o a revisar
- ~~Pausa y mensaje~~ y ~~tubería pintada~~: resueltos en la sección 0.
- **La segunda tubería pintada (x≈66, zona de Vida) sigue vertiendo verde.** Es la misma imagen repetida en `Fondo_Medio_03`. Se deja para Vida o el final.
- **Las capas azules sobre la tubería pintada** son rectángulos: tapan bien el verde, pero se nota el borde recto.
- **Aspecto greybox:** los chorros y el canal son rectángulos `Square` con color plano. El azul limpio se ve muy saturado. La manguera está hecha con segmentos ondulados que no encajan del todo. Conviene sustituirlos por arte cuando puedas.
- **`Agua_Brote_3` (x 24.9) no lo vi en la captura final**, aunque su escala llegó a 1. Puede estar tapado por el helecho seco y el tocón que hay en ese punto. Revísalo o muévelo.
- `Runa_B`, `Runa_C` y `Compuerta_Greybox` siguen en y ≈ −18 y `GestorRunas.alCompletarse` sigue llamando a `Compuerta.AbrirCompuerta`. Se cambia en las fases de Tierra, Vida y final.

---

# Reporte anterior (ruta alta hasta la Salida — pendiente, sin cambios)

> Se conserva tal cual porque contiene una decisión pendiente (3 tiles). Las posiciones de runas y compuerta que propone quedaron **sustituidas** por el diseño "Las 3 Memorias del Bosque".

**Estado: NO se colocó nada.** La escena está igual que en el último guardado y sin cambios pendientes. Hay un bloqueo en la entrada de la ruta que solo se arregla quitando 3 tiles, y esa decisión es tuya.

## 1. Correcciones a lo que dije antes
- **`Runas_Asset_41 (1)` no es la variante ancha.** Su collider mide 2.48 × 1.33 u; el de `39 (1)` mide 2.94 × 1.08. Aun así cumple el mínimo de 1.5 u, así que la ruta usa `41 (1)` en todas las plataformas nuevas, como pediste.
- **S1–S4 no se habían validado con el modelo de física.** El modelo de entonces fallaba en escaleras de 1 u y la propuesta se calculó a mano, con perfiles medidos. Al validarla ahora con el modelo corregido aparecieron 3 errores:
  1. **S2 no funcionaba.** A la izquierda de x 44.76 el suelo está en y −0.25, no en 0.75, y la franja a 0.75 antes de la pirámide es demasiado corta para tomar impulso.
  2. **Bloqueo en la entrada (S1).** Ver el punto 2.
  3. **S4 dejaba una zona de pie de 0.6 u,** como ya había avisado.

## 2. BLOQUEO: techo sobre la entrada de la ruta
En x 17.76–20.76 hay un bloque de tiles de 1 fila (y 5.99–6.99, collider hasta 6.75), justo encima del suelo de y 0.75. Vero, de pie en S1 bajo ese bloque, solo puede elevarse **0.33 u** antes de darse con la cabeza. Para saltar sin golpearse necesita estar en x ≥ 21.79, pero ahí el montículo (desde x 22.76, y 3.0) no deja sitio. Recolocando plataformas no hay forma de subir: lo probé con el modelo en todas las combinaciones razonables.

**Solución mínima (verificada):** quitar **3 tiles de `Grid/Suelo`: celdas (26,9), (27,9) y (28,9)**, que en el mundo ocupan x 17.76–20.76, y 5.99–6.99. Es el peldaño inferior de la escalera de tiles que sube hacia la izquierda, hacia la cornisa de y 10.75. Esa escalera hoy es inalcanzable, y no forma parte de la ruta.
- Lo verifiqué **en Play**, donde los cambios de tiles se deshacen al salir: con esas 3 celdas vacías, el modelo encuentra la ruta completa hasta la Salida.
- **Dejé las 3 celdas seleccionadas con GridSelection** para que las borres tú. Si prefieres que lo haga yo, dímelo.

## 3. Ruta validada (con las 3 celdas vacías)
El modelo usa la física real de Vero (salto 10.5, gravedad ×2.5, velocidad 4, cápsula 2.15 × 2.92), comprueba colisiones en toda la trayectoria y **exige al menos 1.5 u de zona de pie en cada aterrizaje**.

Todas las plataformas nuevas son copias de `Runas_Asset_41_R4C2_0 (1)`: escala (1.68, 1.23), capa Suelo, collider 2.48 × 1.33. La posición es la del objeto; el tope del collider queda 0.80 u por encima de ella.

| Plataforma | Posición del objeto (x, y) | Collider x | Tope y | Zona de pie (modelo) |
|---|---|---|---|---|
| S1 | (20.67, 1.95) | 19.43–21.91 | 2.75 | 1.63 |
| SM (entrada al montículo) | (23.34, 3.95) | 22.10–24.58 | 4.75 | 1.63 |
| P1 | (39.54, 0.95) | 38.30–40.78 | 1.75 | 2.00 |
| P2 | (42.54, 2.95) | 41.30–43.78 | 3.75 | 2.00 |
| P3 | (45.54, 4.95) | 44.30–46.78 | 5.75 | 2.00 |
| P4 | (48.54, 6.95) | 47.30–49.78 | 7.75 | 2.00 |
| P5 | (51.54, 8.45) | 50.30–52.78 | 9.25 | 2.38 |
| S3a | (58.54, 10.30) | 57.30–59.78 | 11.10 | 2.00 |
| S3b | (61.54, 11.95) | 60.30–62.78 | 12.75 (se une a los tiles de 12.75) | 4.00 |
| S4 | (66.52, 13.30) | 65.28–67.76 | 14.10 | 2.50 |
| S5 | (70.54, 14.65) | 69.30–71.78 | 15.45 | 2.00 |

**Secuencia de saltos** según el modelo (subida → ancho de la zona de pie):
1. Suelo y 0.75 (**1.63**) → S1 (+2.00 → 1.63) → SM (+2.00 → 1.63) → cima del montículo, y 5.75 (+0.99 → 5.88).
2. Bajada al suelo de y −0.25 en x 33.9 (−6.0 → 3.50). Así se evitan el foso trampa y el túnel.
3. P1 (+2.00), P2 (+2.00), P3 (+2.00), P4 (+2.00): 2.00 cada una. P5 (+1.50 → 2.38).
4. Cima de la pirámide de tiles, y 9.75 (+0.49 → 2.50). **Los escalones de 1 u de la pirámide ya no se usan** y no se tocan.
5. S3a (+1.36 → 2.00) → S3b / tiles de 12.75 (+1.65 → 4.00) → S4 (+1.36 → 2.50) → S5 (+1.35 → 2.00).
6. `Runas_Asset_45`, y 16.80 (+1.35 → 6.63). Caminando a la derecha, Vero toca la **Salida** (77.11–79.11 × 15.65–17.65).

- **Ancho mínimo en la ruta alta: 1.63 u. Subida máxima: 2.00 u.**
- **¿Por qué son 11 plataformas y no "S1–S4 + 2–3"?** Desde el suelo de y −0.25 hasta la cima de la pirámide (y 9.75) hay 10 u. Con saltos de 2 u como máximo, hacen falta 5 escalones (P1–P5). S3 y S4 se dividieron en S3a/S3b y S4/S5 para que ninguna zona de pie quede bajo otra plataforma. Con eso desaparece la franja de 0.6 u.

## 4. Runas, compuerta y Salida (propuesta, sin colocar — SUSTITUIDA)
- `SecuenciaRunas` exige el orden **A → B → C**. Al completarse llama a `Compuerta.AbrirCompuerta` y a `NotificadorProgreso.RegistrarGesto`.
- La compuerta es una pared sólida de 2.64 × 4.88 u. Por eso va **justo antes de la Salida**, y las runas antes de ella, en el mismo orden en que se recorren:

| Objeto | Posición propuesta | Sobre |
|---|---|---|
| Runa_A | (42.30, 5.05) | P2 (zona de pie x 41.3–43.3, superficie 3.75) |
| Runa_B | (55.00, 11.05) | cima de la pirámide (x 53.8–56.3, superficie 9.75) |
| Runa_C | (62.30, 14.05) | S3b / tiles de 12.75 (x 60.3–64.3) |
| Compuerta_Greybox | (73.65, 19.24) | extremo izquierdo de `Runas_Asset_45` (x 72.33–74.97, y 16.80–21.68) |
| Salida | sin cambios: (78.11, 16.65) | `Runas_Asset_45` |

- Con la compuerta cerrada, desde S5 no se puede pasar: el salto llega a y 17.7 y la compuerta sube hasta 21.68. Abierta, se llega a la Salida.
- El centro de cada runa queda 1.3 u sobre la superficie, así su zona de activación (radio 0.96) coincide con el cuerpo de Vero para pulsar E.

## 5. Tramo bajo, desde el spawn (ya existía, no lo cambié)
- **La escalera del spawn (x −8 a 2) tiene peldaños de 1 u,** con zonas de pie de menos de 1.5 u. Vero la sube en el juego real, pero **no cumple la regla de 1.5 u**. Si quieres que se cumpla en toda la ruta, habría que ensancharla igual que la pirámide. No lo incluí porque no lo pediste.
- **Foso de x≈5.8–9.8:** el salto necesita 3.38 u en horizontal y el máximo de Vero es unos 3.42 u, así que es posible con 0.04 u de margen. Es muy exigente.
- Los demás aterrizajes del tramo bajo miden entre 1.63 y 5.75 u.

## Siguiente paso (del reporte anterior)
Cuando las celdas (26,9), (27,9) y (28,9) estén vacías, las borres tú o me autorices a hacerlo:
1. Coloco las 11 plataformas, las 3 runas y la compuerta con estas coordenadas.
2. Repito el modelo sobre la escena real.
3. Hago la prueba con teclado real de punta a punta, activando las runas con E en orden.
4. Guardo la escena y actualizo este reporte con el resultado.
