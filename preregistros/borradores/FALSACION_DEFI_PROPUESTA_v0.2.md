# Condición de falsación de la apuesta del proyecto en DeFi — PROPUESTA v0.2 (sin sellar)

Fecha: 2026-10-23 · Sustituye a la v0.1 (sha256 `746dfbf2a77b34e8c6b4369b5455a6b76ea591f65b1be33b20a0a0ab3ea42cb0`).
Estado: **para el visto bueno final de Rod. NO sellado.** Rod aprobó el esquema y los umbrales de la v0.1 con dos enmiendas: (1) la dirección del conservadurismo y (2) el momento de la evaluación. Esta versión las incorpora y trae una decisión derivada de la enmienda 1 (§3.3).

## 0. Por qué hace falta

- **La apuesta del proyecto:** «los agentes autónomos se encontrarán con desconocidos y necesitarán decidir si cooperan, cada vez más, en terrenos donde hay dinero».
- **Hoy no tiene condición de falsación.** Este documento fija, **antes de ver los datos que la decidirán**, qué resultado diría que la apuesta **no se sostiene en DeFi**, **cuándo** se mira y **una sola vez**.

## 1. La métrica es el CENSO DEL TERRENO, no el recuento de pares

- **Por qué no los pares:** 7 de los 8 pares del ensayo están muertos porque nuestra medición no pudo medirlos: R6 excluye en cascada las 67 retiradas de Yearn, hay constantes solo de hermanos y ventanas poco cubiertas. Contar pares mediría nuestras reglas de gas, no el mercado.
- **Qué se mide:** mercado a mercado, la fracción de la actividad que mueve capital que **decide y firma su titular (autodirigida)** frente a la que hace **automatización de terceros** (un contrato actúa sobre capital ajeno), con `n` absolutos.
- **Independiente de nuestras reglas de gas:** usa todas las transacciones que producen el evento (R1), sin R6, sin hermanos, sin medianas. Solo las clasifica con la lectura R1-B': ¿la posición acaba en quien firma, en la misma transacción?

## 2. Cómo se mide exactamente

**Unidad:** el mercado (contrato).

**Acciones que cuentan:** las que asignan o retiran capital:
- `yearn_deposit/withdraw`, `erc4626_deposit/redeem` (y peticiones R2), `beefy_deposit/withdraw`;
- `pendle_buy_pt/sell_pt`;
- `curve_add_liq/remove_one`, `curve_gauge_deposit/withdraw`, `convex_deposit/withdraw`;
- `aero_add_liq/remove_liq`, `aero_gauge_deposit/withdraw`.

**Fuera:** `approve`, el `swap` del pool genérico del router y los cobros de recompensas. **Ver §3.3: esta exclusión favorece la hipótesis, y se declara y se mide aparte.**

**Fuentes:** solo on-chain, ya permitidas: `eth_getLogs` y recibos de nodos públicos, en los almacenes `gas_v2` y `gas_censo`, con la clasificación R1-B' guardada en cada fila.

**Universo:**
- los mercados del universo del D0 (los dos lados de cada par seleccionado) y los del censo, con buscadores activos durante todo el mes;
- la lista se congela en la salida, con su sha256;
- solo transacciones propias de cada mercado, nunca de hermanos.

**Clasificación de cada transacción:**

| Clase | Qué significa | Cómo cuenta |
|---|---|---|
| autodirigida | La posición acaba en quien firma: directa, vía router o reenviada por un zap (R1-B') | Decisión del titular |
| de terceros | Un contrato se queda la posición: estrategia, keeper, vault que asigna a otro vault | Intermediación |
| sin lectura | No se pudo leer el beneficiario | Fuera de la fracción; puede invalidar el mercado |

**Fracción de intermediación:** `Î = terceros / (autodirigidas + terceros)`, la **estimación directa**. Al lado se publica el intervalo de Wilson al 95 %.

**Mercado medible** si cumple todo:
- (a) `n = autodirigidas + terceros ≥ 50` en la ventana;
- (b) `sin lectura ≤ 20 %` del total observado;
- (c) su cadena tiene datos que cubren la ventana entera.

## 3. Umbrales y veredicto (aprobados por Rod, con la enmienda 1)

### 3.1 Veredicto por mercado (medible): ESTIMACIÓN DIRECTA

Enmienda 1 de Rod, que **acepto sin reservas**:
- usar la cota inferior de Wilson hacía más difícil declarar intermediación, que es justo lo que falsa la apuesta;
- era prudencia estadística **a favor de la propia hipótesis**;
- quien se pone a prueba debe ponerse fácil refutarse.

| Veredicto | Condición |
|---|---|
| **INTERMEDIADO** | `Î ≥ 0,80` |
| **AUTODIRIGIDO** | `Î < 0,80`, salvo la regla de actor único |
| **NO CONCLUYENTE** | Un AUTODIRIGIDO en el que **una sola dirección** firma más del 50 % de sus transacciones autodirigidas (la apuesta habla de desconocidos, y un solo actor no lo es) |

- El intervalo de Wilson se publica junto a cada `Î` para que el lector vea la incertidumbre, pero **no decide**.
- La igualdad (`Î = 0,80`) cuenta como intermediado: también aquí el empate se resuelve hacia la refutación.

### 3.2 Resultado global (una de tres salidas)

- **FALSADA EN DEFI:** `M ≥ 6` mercados medibles en ≥ 2 cadenas, e `INTERMEDIADOS ≥ ⌈2M/3⌉` (o la mayoría que Rod fije en §3.3).
- **NO FALSADA:** `M ≥ 6` en ≥ 2 cadenas, e `INTERMEDIADOS + NO CONCLUYENTES < ⌈2M/3⌉`.
- **INDETERMINADA:** cualquier otro caso (menos de 6 mercados medibles, una sola cadena, o NO CONCLUYENTES que podrían decidir en un sentido u otro). Se publica como indeterminada, con los números, sin forzar conclusión.

### 3.3 Dirección de TODAS las decisiones (lo que pide la enmienda 1, aplicado a todo el diseño)

Al aplicar la enmienda revisé **cada** elección del diseño, no solo Wilson, preguntando hacia qué lado se inclina.

| Decisión | ¿Hacia dónde se inclina? | Qué hago |
|---|---|---|
| Wilson inferior como criterio | **A favor de la hipótesis** | **Cambiada** a estimación directa (enmienda 1) |
| Empate `Î = 0,80` | Neutro → | Resuelto **hacia la refutación** |
| Actor único rebaja solo los AUTODIRIGIDOS | **Hacia la refutación** | Se mantiene |
| n ≥ 50, sin lectura ≤ 20 %, M ≥ 6 en 2 cadenas | Neutras: hacen más difícil cualquier conclusión, en los dos sentidos | Se mantienen |
| **Mayoría de 2/3 para FALSAR** | **A favor de la hipótesis**: exige más para refutar que la mayoría simple | **Decisión de Rod (D-23a)** |
| **Excluir los cobros de recompensas** | **A favor de la hipótesis**: los cobros los hacen sobre todo keepers, e incluirlos subiría la intermediación | **Se mantiene la exclusión** por una razón de medición (un cobro no es la decisión de cooperar; es mantenimiento), **declarada**, y el script publica **además** el resultado incluyéndolos |
| Zaps que reenvían la posición al firmante cuentan como autodirigidos | **A favor de la hipótesis** (menos intermediación) | **Se mantiene** por definición (el titular firma y recibe; usar una herramienta no es delegar), **declarada**, y el script publica el resultado contándolos como de terceros |

**D-23a — la mayoría para falsar.**
- La mayoría cualificada de 2/3 protege la hipótesis por el mismo mecanismo que Wilson: exige más evidencia para refutar que para no refutar.
- **Mi recomendación, coherente con la enmienda 1:** bajarla a **mayoría simple**, `INTERMEDIADOS > M/2`. NO FALSADA pasaría a ser `INTERMEDIADOS + NO CONCLUYENTES ≤ M/2`.
- Si Rod prefiere mantener 2/3, el documento sellado dirá textualmente que esa elección favorece la hipótesis del autor.
- **¿Mayoría simple, sí/no?**

**Lo que el documento sellado dirá en todo caso:** las decisiones que se mantienen a favor de la hipótesis (la exclusión de cobros, los zaps como autodirigidos y, si Rod lo decide, los 2/3) se listan como tales, con sus resultados alternativos publicados al lado, para que el lector los juzgue.

**Sensibilidad (informativa, no decide):** el resultado con T = 0,70 y T = 0,90; con los cobros incluidos; con los zaps como de terceros; y con la cota inferior de Wilson. Así se ve si la conclusión depende de alguna de estas elecciones.

**Contaminación previa, declarada.** El 10-oct se vieron fracciones del terreno del universo del ensayo con pocas horas de datos: Beefy ~89 % y sUSDai ~81 % autodirigidas; depósitos de Morpho 4–10 % y gauge de Curve 2/60. Quedan a los dos lados de 0,80. **Para fijar estos números no se ha calculado el censo actual.**

## 4. CUÁNDO se evalúa (enmienda 2): UNA sola vez, en un momento pre-declarado

- **Momento:** el **cierre del mes de observación**, sobre el censo de esa fecha.
  - Con el D0 previsto para el **2026-11-10**, el día 1 es el 2026-11-11 (primer día UTC completo tras el D0, PLAN_CONGELADO; la posición de 30 días se abre el día 1 y se cierra el día 30, metodología §9) y el mes cubre los días 1 a 30.
  - **Corte de la evaluación: 2026-12-11 a las 00:00:00 UTC.**
  - **Ventana evaluada:** las transacciones con marca de tiempo en **[2026-11-11 00:00, 2026-12-11 00:00) UTC**, es decir, el propio mes de observación.
- **Si el D0 no ocurre el 2026-11-10:** el corte es **D0 + 31 días a las 00:00 UTC** y la ventana son los 30 días anteriores al corte.
  - El D0 que cuenta es el de la entrada `methodology_freeze` del paper_log, que es pública, anclada e inmutable. La fecha se deriva mecánicamente de ese registro y nadie la elige después.
  - Si el D0 no llega a ocurrir, la condición no se evalúa y se declara así.
- **Ejecución:**
  - una sola vez, entre el corte y el corte + 72 h;
  - el script se niega a correr antes del corte, si ya existe la marca `FALSACION_DEFI_APLICADA.json`, o si su sha no es el sellado.
  - **Nadie mira el resultado antes:** el script no tiene modo de vista previa sobre datos reales; solo una autoprueba con datos sintéticos.
- **Si ese día el censo no alcanza el mínimo** (menos de 6 mercados medibles, o menos de 2 cadenas):
  - el resultado es **INDETERMINADA**, y **es definitivo** para este mes de observación;
  - **no hay prórroga ni segunda evaluación** sobre los mismos datos ampliados;
  - cualquier evaluación posterior sería un pre-registro nuevo, con fecha nueva, declarado como tal, y no sustituye ni reescribe este resultado.
- **Requisito operativo:** la acumulación de gas del ensayo, en modo vivo durante el mes, y el bucle del censo siguen corriendo hasta el corte. Si se interrumpen, las cadenas afectadas pierden la cobertura (2.c) y eso se refleja en M, sin rellenar a posteriori.

## 5. ALCANCE (condición de Rod): falsa LA APUESTA EN DEFI, no la apuesta entera

- **Si la condición se cumple, queda falsado:** que en DeFi (en los protocolos, cadenas y mercados observados) los agentes autónomos estén decidiendo, cada vez más, cooperar con desconocidos.
- **No queda falsado:** lo que ocurra **cuando los agentes tengan su propia infraestructura económica**: sus propios mercados y contrapartes, protocolos de agente a agente. Que DeFi esté intermediado dice cómo se mueve hoy el capital en DeFi, no nada sobre un terreno que aún no existe.
- **Límite adicional:** el universo son los mercados que **nuestra selección** observa (3 cadenas, 6 protocolos, estables con TVL ≥ 250 k y la regla de medibilidad del D0). **No es «todo DeFi».**
- **Frase obligatoria en cualquier publicación del resultado:** «Esta condición falsa la apuesta en DeFi, en los mercados observados. No dice nada sobre los terrenos en que los agentes tengan su propia infraestructura económica.»

## 6. Qué NO cae aunque la condición se cumpla

1. **La confianza vive en el protocolo, no en la inteligencia.** Se demostró puertas adentro (identidad y atestación FLP; la puerta de bias_audit) y no depende de una contraparte externa.
2. **La auto-auditoría detecta cuándo una medición se inclina.** Se demostró contra el propio aparato: rachas del libro de cambios, TH-SIM, sellos que no eran sellos, la fuga del número de ATENCIÓN, la puerta que falló por ergonomía, la falsa alarma de Convex y **la prudencia de este mismo documento, que apuntaba a favor de la hipótesis (§3.3)**.

## 7. Aviso metodológico: la Fase 1 y la Fase 2 NO son réplicas independientes

- **Preguntan cosas distintas:** la Fase 1 (Ambassador en Moltbook) preguntaba si había demanda de interacción bilateral entre agentes en un terreno social; la Fase 2 pregunta quién decide los movimientos de capital en mercados DeFi.
- **Lo que comparten es algo más débil:** en los dos terrenos, la decisión bilateral autónoma que esperábamos es rara o no aparece.
- **No son dos confirmaciones del mismo hallazgo:** un mismo sesgo nuestro puede estar detrás de las dos, y la Fase 2 depende mucho de nuestro aparato de medición. Se escribe así en cualquier síntesis.

## 8. Sellado y script (tras el visto bueno final de Rod)

- **`aplicar_falsacion_defi.py`:** solo lectura de los almacenes; una ejecución en la ventana del §4; marca anti-repetición; comprobación de su propio sha.
- **Salida:**
  - la lista congelada de mercados con su sha;
  - por mercado: `n`, autodirigidas, de terceros, sin lectura, `Î`, el intervalo de Wilson, la concentración del primer firmante y el veredicto;
  - el resultado global, **escrito antes** que la sensibilidad;
  - la tabla de decisiones a favor de la hipótesis, con sus resultados alternativos;
  - la frase de alcance.
- **Autoprueba solo con datos sintéticos.**
- **Sellado del criterio y del script:** `date -u` del VPS, 444 + `chattr +i`, entrada en `preregistros.jsonl` y sha anotado aparte en `/root`.

## 9. Para el visto bueno final de Rod

1. **D-23a:** mayoría simple (`> M/2`) en vez de 2/3, por coherencia con la enmienda 1. **Recomendación: sí.**
2. Visto bueno a esta v0.2: estimación directa, la tabla de dirección (§3.3), el corte del **2026-12-11 a las 00:00 UTC** (o D0 + 31 días) e INDETERMINADA definitiva si no se alcanza el mínimo.

Con las dos respuestas, escribo el script, lo pruebo solo con datos sintéticos y sello el criterio y el script antes del 10-nov. **¿Sí/no?**
