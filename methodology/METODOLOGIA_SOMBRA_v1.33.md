---
documento: Metodología de la cartera sombra (c_solo) y calibración del cost model
fase: RALL Fase 2 — observación de yield farming
version: 1.33-draft
fecha_borrador: 2026-10-08
sustituye_a: 1.32-draft (sha256 0195e38c…)
estado: BORRADOR (no congelado; no vale para puntuar nada hasta que Rod lo congele)
---

# Metodología de la cartera sombra — v1.33-draft

> **Esto es un MODELO, no una medición.** Toda cifra derivada de este documento se
> etiqueta `modelo:sombra-v1.26` en datos, reportes y paper (de la v1.27 a la v1.31 no cambia ninguna regla aplicada que
> altere una predicción: la v1.31 valora las recompensas extra de Convex solo si pagan —hoy 0— y registra el hallazgo
> de medición «hermanos más bajos»; verificado sobre datos reales: predicciones IDÉNTICAS). La cartera sombra no
> tiene capital, wallet ni transacciones: simula qué habría costado hacer lo mismo
> "a mano", con precios y gas reales de cada día.

## Pendiente antes del congelado

1. F1–F6 se aplicaron en la v1.12, G1–G2 en la v1.13, la medición on-chain del gas en la v1.14 y, en la v1.15, I1–I3, R1, R2, la ventana del APY por ciclo de cosecha y el contraste corregido (Rod). Pendientes:
   - **CONDICIÓN DE SALIDA DEL ENSAYO (Rod, 2026-10-10).** El ensayo **todavía no ha producido ninguna predicción completa**: todas han salido anuladas, por `sin_dato_gas` (R3) o por `sin_dato_apy` (la ventana del APY no se acorta, §7.1). P-D no tendrá APY hasta el 2026-10-14, cuando haya 7 días de fotos propias en Arbitrum (sin nodo de archivo público). **El congelado no se hace hasta ver predicciones completas saliendo limpias varios días seguidos.** La fecha se mueve lo que haga falta; **no se recorta ninguna regla para llegar a tiempo.**
   - **PROBLEMA ABIERTO (Rod, 2026-10-16): el valor de una constante de gas que se congele el D0 DEPENDERÍA DEL DÍA QUE TOQUE.** **Resuelto el 2026-10-31 por el criterio pre-registrado: V3** (ver «Agregación del gas», más abajo, y el §5). El problema no es que la constante varíe; es que, con la regla actual (mediana de las 50 transacciones válidas más recientes, C8), **el número congelado cambiaría hasta un 40 % —y en algún caso más— según el día exacto del D0**. Eso contradice el propósito del congelado: que el resultado no dependa de elecciones arbitrarias, y el día del D0 lo es respecto de estas cifras. Medido sobre 4 instantáneas consecutivas (12–15 oct, mismas reglas), máximo cambio de un día al siguiente con la regla vigente: Curve `add_liq` (hermanos) **74 %**; ERC-4626 `f42f` en Base **107 %**; Convex `deposit` (hermanos) **46 %**; Curve `remove_one` (hermanos) **41 %**; swap en Base **33 %**; Merkl en Ethereum **28 %**; swap en Ethereum **20 %**. Causas observadas: muestras que se renuevan enteras cada día; poblaciones heterogéneas (llamadas directas frente a routers, bots y zaps) y bimodales (Convex: ~660.000 directo frente a 2,7–4,1 millones por zap); y un solo actor que domina la muestra (36 de 50 en Curve `remove_one` el 14-oct). **Medición de agregaciones alternativas en `medicion_agregacion_2026-10-16.json` y en `HECHOS_v1.28_ensayo.md`. No se ha cambiado ninguna regla.** Cualquier cambio se decidiría DESPUÉS de ver esos números y se declarará como post-hoc. Queda planteado así aunque al final se decida no tocar nada.
   - **⚠️ CON/SIN DE BEEFY-ESTR: VIENE DE UNA COSECHA DE DATOS DISTINTA AL RESTO (2026-10-10 07:25 UTC, cálculo G8; `HECHOS_v1.22_ensayo.md` §1).** Sin la regla (533 hermanos por código del vault): **3 de 8 pares vivos**; con la regla (misma estrategia): **1 de 8**. Se acepta como su con/sin (Rod, 2026-10-16) porque regenerarlo con los datos actuales exigiría repetir el descubrimiento de 535 candidatos y acumular el grupo sin filtrar —horas de nodo público para un dato que ya existe—; la diferencia de cosecha sí es información y por eso se declara aquí, en primer plano. **Se vuelve a medir UNA vez en el D0**, que es donde importa.
   - **M (medibilidad): su con/sin es el censo del D0.** En el ensayo queda declarado como pendiente del D0 y no bloquea; **en el D0, la puerta del documento final EXIGE el censo publicado y bloquea si falta** (Rod, 2026-10-16).
   - **⚠️ HALLAZGO DE MEDICIÓN DE PRIMER ORDEN (2026-10-24, a raíz de una observación de Rod): las constantes de gas que salen de contratos HERMANOS son sistemáticamente MÁS BAJAS que las propias, y en el lado delegado eso EMPUJA HACIA DELEGAR.** Detalle en `ANALISIS_FUENTE_SIGNO_2026-10-24.md` y `BANDA_COMPROBACION7_PROPUESTA.md`.
     - **El hallazgo:**
       - **En las 5 comparaciones posibles** (4 unidades independientes), los hermanos salen por debajo de la propia, con un cociente de **0,58 a 0,90**.
       - Solo 1 comparación es buena (n propias ≥ 10); las otras 3 unidades son aproximadas (pocas filas propias, sin R6) y probablemente exageran el sesgo. **La dirección es firme; la magnitud, incierta.**
       - **Mecanismo plausible:** composición distinta. La muestra propia incluye las transacciones reenviadas por zap, más caras (R7: 2,21 M frente a 1,04 M).
     - **Efecto en las clases:**
       - **P-B (Beefy frente a LP+gauge): nulo.** DELEGAR 8/8 con las constantes de hermanos ×0,5, ×2 y con las medianas propias.
       - **P-C (Convex): marginal en la mayoría, real en los márgenes.** Con el lado delegado ×2, 2 de 12 puntos pasan a DIRECTO.
       - **Ocurrió en vivo el 24-oct:** `convex_deposit` (de hermanos) saltó de 1,22 M a 2,27 M y P-C pasó a tener 2 DIRECTO.
     - **La correlación «hermanos → DELEGAR» de los tres pares completos está confundida con el tipo de comparación:** los Beefy frente a un vault simple, con la retirada también de hermanos, dan DIRECTO en sus referencias.
     - **Hueco de la capa que destapa:** la comprobación 6 marca que una constante se apoya en hermanos, pero ninguna comprobación medía cuánto depende el resultado de esa fuente.
     - **Comprobación 7, «sensibilidad a la fuente»** (aprobada por Rod el 2026-10-26, **declarada post-hoc**): la banda se **deriva** de la discrepancia medida, no se elige. El criterio se selló el 2026-10-26, se integró en el diagnóstico completo el 2026-10-31 (G21) y su primera ejecución oficial fue el diagnóstico del 2026-11-03.
     - **Nadie debe leer un DELEGAR apoyado en constantes de hermanos sin mirar su comprobación 7.**
   - **PARES QUE LLEGAN AL D0 (Rod, 2026-10-26).**
     - La selección del D0 es **nueva** (semilla BTC del día del congelado, regla M); los 8 pares del ensayo salieron de la semilla del 7-oct y no tienen por qué repetirse.
     - El documento del D0 lista **cada mercado elegible y por qué entra o se cae**, separando:
       - **no medible**: límite de NUESTRA medición (sin dato de gas tras C8, regla M);
       - **mercado encogido o desaparecido**: TVL por debajo del mínimo de elegibilidad, contrato muerto o vencimiento (F2), que es una propiedad del mercado.
     - **Ejemplo del ensayo:** el TVL del lado más chico de P-B base 8bd6 bajó a ~198,5 k el 25-oct (el mínimo de elegibilidad es 250 k). Si sigue así, ese par se caería por **mercado encogido**, no por medición. Es legítimo, y se dirá así.
   - **AGREGACIÓN DEL GAS — CRITERIO PRE-REGISTRADO (Rod, 2026-10-16), se aplica UNA vez sobre las instantáneas del 16 al 30-oct; el resultado va a Rod antes del D0.** Archivo sellado en el VPS: `auditoria/CRITERIO_AGREGACION_GAS.json`, **registrado 2026-10-16T05:34:54Z** (marca escrita por el propio servidor al crear el archivo; coincide con su mtime), sha256 `e286adbc0253e4493adad6e0a694b6ff0ea5db8d651a4b467b719564f5cda618`, modo 444. Script que lo aplica, sellado con él: `aplicar_criterio_agregacion.py`, sha256 `d4e12a13f2810c514bd6e5862f18d90814d83e7dee77bfbc382de751ee4ed583`. Candidatas V0 (vigente), V1, V3 y V5 (tope por emisor, declarada hoy y sin medir). Ganadora: menor mediana, sobre todas las constantes, del máximo cambio entre instantáneas consecutivas; descartes: deja alguna constante sin dato, o desplaza más de un 15 % el nivel de una constante estable (≤ 10 % de cambio con V0); **si ninguna mejora estrictamente a la regla actual, se queda la actual**. Las 4 instantáneas ya vistas (12–15 oct) quedan fuera. **APLICADO una sola vez el 2026-10-31 (01:05–01:07 UTC), con el script sellado y las 15 instantáneas del 16 al 30-oct: GANA V3, la mediana de las medianas diarias.** Métrica (mediana, sobre las 43 constantes, del máximo cambio entre instantáneas consecutivas): V0 0,0911; **V3 0,0048**; V1 0,0109, descartada porque desplaza un 18,5 % una constante estable; V5 0,0268, descartada porque deja una constante sin dato. 24 constantes estables. Salida: `auditoria/AGREGACION_RESULTADO_2026-10-31.json`, sha256 `1cd80778b812cf713c4e4626b957901904420c353b7623c33feeb7121c3a45a3`. **Por el propio criterio, V3 sustituye a la regla de las 50 más recientes (§5).**
   - **PENDIENTE (Rod, 2026-10-16): ¿existe de verdad la recompensa del 5 % que DefiLlama empezó a sumar al APY de Aave en Base el 2026-10-15, y podría cobrarla un depositante directo?** Si la respuesta es sí, afectaría al **lado directo de otros mercados** (no solo al control): sería una fuente de rendimiento del lado directo que hoy no se cuenta (G1 solo cuenta Merkl con identificador exacto). Se averigua antes del congelado; no se cambia nada hasta tener la respuesta.
   - **CALENDARIO (Rod, 2026-10-11): se ESPERA al censo completo; D0 hacia el 2026-11-10.** Aplicar el criterio de medibilidad (§2.3) con uno o dos días de datos lo convertiría en **decorativo**, así que se espera a los 30 días de acumulación de todos los elegibles: Base desde el 2026-10-08, Ethereum desde el 2026-10-07 y los 30 pools de P-C del censo desde el 2026-10-10 por la noche, porque el nodo sirve unas 31 h hacia atrás. **El mes de observación iría de ~10-nov a mediados de diciembre.** La condición de salida del ensayo sigue en pie.
   - ~~**H1 — DECIDIDO por Rod (2026-10-07), PENDIENTE DE APLICAR antes del congelado.**~~ **APLICADO en la v1.25 (2026-10-11):** cierre a las **18:50 UTC**; predicción a las 18:55, siempre antes de las 00:00. Texto original: El reintento de la foto de las 18:00 (18:30–18:50) cae después del cierre del día (18:20) y nunca cuenta. Arreglo decidido: **mover el cierre del día a después del último reintento (18:50 UTC)**, no adelantar el reintento (dejaría de ser útil como reintento); la predicción se sigue registrando antes de las 00:00 UTC. Se aplica tras medir en el ensayo cuántas veces hace falta (día 1: una vez). Desde la v1.15 la implementación ya respeta el cierre en todos los recuentos del día (§9), así que al aplicar H1 basta con cambiar la hora.
   - **M — CRITERIO DE MEDIBILIDAD EN LA SELECCIÓN: APROBADO por Rod (2026-10-11), se aplica en la selección del D0** (§2.1, §2.3). Antes del congelado faltan:
     - implementarlo en la herramienta del D0, con el censo completo publicado;
     - tener 30 días de acumulación de gas de **todos** los elegibles. La de los 31 de P-C que no están en el ensayo empezó el 2026-10-11 (§2.3, requisito operativo).
   - ~~**R7 — Zap de Beefy y la R1 (EN ESPERA a propósito; se mide en `HECHOS_v1.15_ensayo.md`).**~~ **Resuelto en la v1.22 (R7).** Con la R1, el depósito en Beefy ya incluye las transacciones que pasan por su zap; la regla del §5.2 punto 3 (zap = depósito + añadir liquidez + swap) lo contaría dos veces, en contra de delegar. Se declara y no se toca hasta que Rod decida.
2. **Rellenar con datos de la Fase 0** (mediciones, no decisiones):
   - **el censo de incidentes de la clase A completa** (A.2.8, punto 7), en el paso D0·2;
   - el `TVL_A` on-chain de Vyper 2023 y de yUSDT 2023 (A.2.8, punto 2);
   - la selección de mercados con la regla del §2.1 (semilla del día del congelado);
   - las fechas de creación on-chain de cada mercado, o de su factory en los mercados que vencen (A.3.4);
   - las auditorías que cubren cada implementación;
   - las unidades de gas (§5, con respaldo) y la calldata representativa de cada acción (§5, I3: de las propias transacciones medidas, on-chain);
   - el ciclo de cosecha de cada mercado por precio por share y su ventana del APY (§7.1, v1.15);
   - si rsETH era colateral en Aave v3 Base el 2026-04-17;
   - el censo del A.2 y la clase del A.2.7 recalculados ese día.
3. **Infraestructura del congelado:**
   - Protección de `main` **verificada** el 2026-09-24: force push rechazado (GH013) con la clave del VPS y push normal aceptado. El borrado de `main` se rechaza por ser la rama por defecto.
   - **La regla `deletion` del ruleset queda declarada como NO PROBADA por separado** (decisión de Rod). Probarla exigiría crear una rama de sacrificio, que después no se podría borrar porque el ruleset cubre todas las ramas. Eso dejaría basura permanente en un repo público solo para demostrar una regla que no se va a usar. `main` ya está protegida contra el borrado en la práctica.
   - Se instala el buzón `methodology_freeze` (plan §5).

## Historial de cambios

### v1.32-draft → v1.33-draft (2026-11-05): dos decisiones de Rod sobre el texto entregado

**No cambia ninguna regla de medición.** Escribe en el texto dos decisiones de Rod del 2026-11-05:
- **I3 con V3** (§5.2 punto 7): la calldata representativa se elige entre las 50 transacciones más recientes, no entre todas. Se declara que ahí V3 no se aplica del todo, y por qué.
- **Elegibles tardíos** (§2.3): los elegibles del D0 que no lo eran el 7-oct salen como no medibles, declarándolo, con su número, la causa (cobertura del censo) y la dirección del sesgo (desconocida).

### v1.31-draft → v1.32-draft (2026-11-05): texto final que se entrega a Rod para el congelado

**Cambia una regla aplicada: la agregación de las constantes de gas pasa de «las 50 últimas» a V3**, por el criterio pre-registrado el 2026-10-16 y aplicado una sola vez el 2026-10-31 (§5 y Pendientes). Lo demás:
- **Anclaje de la selección del D0** (orden de Rod, 2026-10-30): §2.3.
- **Cuatro líneas neutralizadas** por exposición operativa (pausas del ciclo diario, arquitectura de acceso a una clave e historial de un hallazgo de seguridad, permisos de una clave). No cambia ninguna regla.
- **Estado de la comprobación 7** puesto al día (sellada, integrada, primera ejecución oficial).
- **Ninguna versión anterior de este documento se aprobó como texto completo por su sha.** Rod aprobó los cambios uno a uno; este texto final es la primera línea base completa (declaración D0-6).

### v1.30-draft → v1.31-draft (aprobado por Rod, 2026-10-21 a 26)

**No cambia ninguna predicción** (verificado sobre datos reales el 2026-10-23 y el 2026-10-24: 96/96 idénticas).

- **D-21d — recompensas extra de Convex:**
  - solo cuentan, y solo dan nota, si **pagan de verdad** (`rewardRate > 0` y `periodFinish` futuro). Si pagan, se valoran en el lado delegado;
  - si el gauge de Curve paga recompensas extra que no se valoran, se avisa del lado DIRECTO, por simetría.
  - **Corrección para el rastro:** la advertencia anterior («rewards extra que no se valoran, lado delegado infravalorado»), presente desde la v1.12 y tratada como contaminación real, era una **FALSA ALARMA**. La ranura de P-C nunca se financió (`periodFinish = 0`, medido el 2026-10-21) y **la infravaloración fue CERO**.
- **D-21e — presentación:** la síntesis muestra base y recompensas por separado. El «0,98 % / 0,98 %» de la síntesis del 20-oct era solo la base.
- **Hallazgo «hermanos más bajos» y comprobación 7:** ver Pendientes.
- **Condición de falsación de la apuesta en DeFi: SELLADA** el 2026-10-23T07:01:04Z (`auditoria/FALSACION_DEFI.json`; texto `fde04d14…`; script `c3391dc3…`). Censo del terreno, estimación directa, mayoría simple, evaluación única el 2026-12-11 a las 00:00 UTC (o D0 + 31 días) y **tabla visible de las decisiones que favorecen la hipótesis del autor**.

### v1.29-draft → v1.30-draft (aprobado por Rod, 2026-10-16)

**No cambia ninguna regla aplicada.** Fija cómo se publica el con/sin de BEEFY-ESTR (tabla histórica, de otra cosecha, declarada) y de M (censo del D0, que bloquea el documento final si falta), y añade una regla operativa.

**Regla operativa OP-1 (lección de BEEFY-ESTR, reutilizable para el laboratorio clonable): el "con y sin" de una regla SOLO se puede calcular si se conservaron los datos previos a aplicarla. Antes de aplicar un filtro, guarda el conjunto sin filtrar.** Aquí se perdió la lista de los 535 hermanos sin el filtro de estrategia y por eso hubo que tirar de un cálculo histórico. Desde la G15, el descubrimiento de hermanos guarda también la lista sin filtrar, y la capa de auditoría (`bias_audit`, comprobación 4) marca cualquier regla de tipo "filtro" que no conserve el conjunto previo.

### v1.28-draft → v1.29-draft (aprobado por Rod, 2026-10-16)

**No cambia ninguna regla aplicada.** Pre-registra **cómo** se elegirá la agregación de las constantes de gas (Pendientes).

**Razonamiento (para el rastro, reutilizable).** La elección de la agregación iba a ser **post-hoc**: se habían visto cuatro instantáneas y sus números. Escribir y sellar HOY el criterio de elección —candidatas, métrica, descartes y la regla de "si nada mejora, se queda lo actual"— **antes** de que existan los datos sobre los que se aplica, convierte esa decisión post-hoc en una **regla pre-registrada**. Es el principio del proyecto —**congela la regla, no el número**— aplicado a la pregunta de **cómo congelar un número**. El valor entero del criterio está en su marca de tiempo: por eso se sella en el servidor, con la hora real de creación (la lección del archivo de umbrales de la capa, cuya primera marca fue estimada y posterior a su uso), junto con el sha256 del script que lo aplicará, y se aplica una sola vez sobre instantáneas creadas después.

**Declarado:** el criterio lo propuso el asistente después de ver las 4 instantáneas (12–15 oct) y la medición de variantes; lo que no se ha visto son los datos sobre los que se aplicará. La variante V5 se declara sin medirla.

### v1.27-draft → v1.28-draft (2026-10-16)

**No cambia ninguna regla aplicada.** Deja planteado en Pendientes el problema de fondo (la constante congelada dependería del día del D0, con sus números) y registra dos mediciones, sin decidir nada:
- agregaciones alternativas de las constantes de gas (`HECHOS_v1.28_ensayo.md` §1);
- la recompensa del 5 % de Aave Base: **no aparece ni en el controlador de incentivos on-chain (su única recompensa registrada terminó el 2024-09-03) ni como oportunidad LIVE en Merkl**; solo la reporta DefiLlama (`apyReward` = 5, token USDC, desde el 2026-10-14). Por esas dos vías no la cobraría un depositante directo; su origen sigue sin verificar (`HECHOS_v1.28_ensayo.md` §2). No se cuenta, igual que manda G1.

### v1.26-draft → v1.27-draft (aprobado por Rod, 2026-10-16)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| CONTR-AAVE | **(Implementación, no regla.) El contraste del APY de Aave compara nuestra lectura on-chain con el `apyBase` de DefiLlama**, no con su `apy` | Lo mismo con lo mismo, igual que con Convex en la v1.15 (CONTR). Nuestra lectura on-chain de Aave es el tipo **base** de suministro. Desde el 2026-10-15, DefiLlama suma a su `apy` un `apyReward` de exactamente 5 %, y su `apyBase` (3,77–4,28 %) coincide con nuestra medición (3,83–4,16 %). La discrepancia (4,04 % frente a 8,95 % el 15-oct; 3,83 % frente a 8,76 % el 16-oct) venía de comparar cosas distintas, no de nuestra lectura | §7.1 |

**Declarado:** se hizo **después** de ver la discrepancia (2 días seguidos), al revisar la lectura como manda el §7.1. No cambia ninguna predicción: Aave es el mercado de control y no está en ningún par. Queda **pendiente** averiguar si la recompensa del 5 % existe y si la podría cobrar un depositante directo (ver Pendientes).

### v1.25-draft → v1.26-draft (aprobado por Rod, 2026-10-15)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| TH-SIM | **Tiempo humano simétrico.** En la forma "con tiempo humano", la **entrada y la salida se cobran a LOS DOS LADOS**; el **compounding y el rebalanceo, solo a quien los hace** (si el lado delegado tiene acciones manuales propias, como reclamar recompensas, también se le cobran) | Comprar y vender, o entrar y salir de un vault, son acciones manuales tanto si delegas como si no. **Lo único que ahorra delegar es el compounding y el rebalanceo.** Cobrar la entrada y la salida a un solo lado era un **error de simetría que empujaba hacia DELEGAR**. En P-D era exactamente lo que convertía un DIRECTO en EMPATE: la forma "con" sumaba 2 acciones × (US$2,08 o US$25) solo al lado directo | §1, §13 |

**Declarado sin maquillar:** se encontró **DESPUÉS de ver el primer resultado completo** (P-D, 2026-10-14). En la forma "con", la ventaja del lado delegado subía exactamente 2 × el tiempo humano por acción (de −2,97 a +1,19 y +47,03 en 1.000 $ a 7 días), y eso destapó que el lado delegado no pagaba su entrada ni su salida. **El argumento se sostiene en los dos sentidos:** entrar y salir requiere una acción manual en los dos lados, sea cual sea el efecto en el resultado. **Se publica el resultado con y sin el cambio en TODOS los pares** (`comparacion_humano_v126_2026-10-14.json`; resumen en `HECHOS_v1.26_ensayo.md`). La implementación lleva un interruptor (`predictor.HUMANO_SIMETRICO`) para poder calcular siempre las dos.

**Caso de prueba para la capa de auditoría (bias_audit):** esta asimetría es la clase de error que la comprobación 3 (aplicación uniforme, con simetrías declaradas) debería cazar sola. Cuando existan el registro de reglas y la comprobación 3, se verificará si la habría cazado **con las simetrías que estaban declaradas antes de este hallazgo**, y el resultado se dejará escrito, sea cual sea.

### v1.24-draft → v1.25-draft (aprobado por Rod, 2026-10-11)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| H1 | **APLICADO: el cierre del día pasa de las 18:20 a las 18:50 UTC**, después de la ventana del reintento de las 18:00 (`[18:30, 18:50)`), para que pueda contar. La predicción se registra después del cierre (18:55 en el ensayo) y siempre antes de las 00:00 UTC. Un reintento que termine después de las 18:50 sigue sin contar, y el `ensayo_status` lo marca "POSTERIOR AL CIERRE" | Decidido el 2026-10-07 y pendiente de aplicar desde entonces. En el ensayo hizo falta una vez (día 1) | §9 |
| CAL | **Calendario: se espera al censo completo; D0 hacia el 2026-11-10** | Con uno o dos días de datos, el criterio M sería decorativo. El mes de observación iría de ~10-nov a mediados de diciembre | Pendientes |
| D7 | **Síntesis semanal construida** (`fase2/sintesis.py`): criterio de **madurez** (ejecución verificable, desempeño ajustado a riesgo, alineación tesorería-usuario), tablas deterministas, sin LLM. La envía el **Orquestador una vez por semana** con un disparo propio. **El ciclo diario sigue pausado.** En el ensayo, cada salida va marcada ENSAYO y no cuenta | Estaba aplazada "para durante el ensayo" (plan §2.5, D7; PLAN_CONGELADO A2/A3) | Plan |

### v1.23-draft → v1.24-draft (aprobado por Rod, 2026-10-11)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| M | **Se adopta el criterio de medibilidad con `M` = 10** y la salvaguarda (a) + (b) tal como estaba escrita | `M` = 10 es **el mínimo que ya fijaba el C8 (R5)**: no introduce ninguna constante nueva. Adelanta a la selección lo que R3 haría igualmente durante el mes. Queda escrito el orden de aplicación en el D0: **elegibles → censo de medibilidad SIN usar la semilla → la semilla ordena solo a los medibles → los N primeros de cada grupo.** Si un grupo se queda sin candidatos medibles, **se queda sin par y NO se baja `M`** | §2.1, §2.3 |
| BEEFY-ESTR | **Se mantiene**, aunque cueste los dos pares `ce3996` y `ae021e` (`HECHOS_v1.22_ensayo.md` §1) | Revisarla ahora por su efecto sería **elegir después de ver los datos**, que es exactamente lo que el método prohíbe. **El coste se reporta, no se evita** | §5 |
| CENSO | **La acumulación de gas se amplía a todos los elegibles** (39 en la selección del ensayo; los 31 de P-C que faltaban), en un almacén aparte (`gas_censo`), con sus ventanas fuera de las fotos y de las pasadas del ensayo | Es medición, no cambia ninguna regla. El censo necesita 30 días de datos y en Base y Ethereum solo existen si se acumulan: cada día sin medir sería un hueco el día del congelado | §2.3 |

### v1.22-draft → v1.23-draft (2026-10-11)

**No cambia ninguna regla aplicada.** Recoge dos decisiones de Rod y añade una propuesta sin aplicar.

| # | Contenido | Estado |
|---|---|---|
| D1 | **El approve del activo de P-C se queda como está** (sin C8 ii para `approve`). **Rastro:** el asistente había dejado escrita, antes de ver los resultados de G8, la recomendación de admitir el C8(ii) para `approve` con las monedas de los pools de la misma familia de Curve. Al leer los datos comprobó que **no rescataba P-C**: también le faltan el depósito y la retirada del gauge y el CRV mint, que la R1-B' deja sin dato porque casi todo es de terceros. **Retiró su propia recomendación**, porque solo añadía complejidad sin mover el recuento, y Rod aprobó no tocar nada | Aprobado por Rod, 2026-10-11 |
| D2 | **El ensayo sigue tal cual, sin relajar ninguna regla.** Se revisa el 2026-10-14 si P-D tiene ya APY directo. Con las reglas vigentes, solo P-D puede dar predicciones completas | Aprobado por Rod, 2026-10-11 |
| M | **Criterio de medibilidad en la selección de mercados, con salvaguarda de sesgo** (§2.3) | **PROPUESTA, NO APLICADA** |

**Corrección del rastro (coste de BEEFY-ESTR).** En el informe de estado del 2026-10-11 se dijo que BEEFY-ESTR no quitaba ningún par. **Era falso:** `ce3996` y `ae021e` están fuera **solo** por Beefy, porque su Morpho ya tiene dato. Con el grupo anterior de 533 hermanos habría 3 pares vivos de 8; con la regla, 1 de 8 (`HECHOS_v1.22_ensayo.md` §1). La regla no se revisa por su efecto.

### v1.21-draft → v1.22-draft (aprobado por Rod, 2026-10-10)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| R7 | **No se compone el zap de Beefy.** Su coste es el depósito (y la retirada) de Beefy **medido tal cual**; se retira la regla "depósito + añadir liquidez + swap" | Con la R1-B' la constante ya es la transacción completa del zap (mediana 2,21 M de gas en 73 depósitos reenviados por el zap, frente a 1,04 M en 3 directos); componerla lo contaba dos veces e inflaba el coste de delegar. Es medir en lugar de sumar supuestos, el mismo criterio que se aplicó al APY y al gas | §5.2 |
| BEEFY-ESTR | **Hermanos de Beefy (C8 ii): además del código del vault, la misma implementación de estrategia** (`strategy()` del vault y `eth_getCode` de esa estrategia, mismo hash), verificable on-chain | Los vaults de Beefy son clones idénticos, pero mezclan estrategias de LP (que necesitan zap) con estrategias de un solo activo (que no). Eso encarecía artificialmente a los Beefy de un solo activo (`ce3996`, `ae021e`) y sesgaba en contra de delegar | §5 |

**Declarado:** las dos se propusieron **después** de ver los números; el argumento se sostiene en ambos sentidos (contar dos veces una misma transacción o comparar con vaults de otro tipo es un error sea cual sea el signo de su efecto). **Coste de BEEFY-ESTR:** el grupo de hermanos se reduce. {coste_b}

**Corrección de la lectura del terreno (§5.6).** En la v1.20 se escribió una lectura fuerte —"el camino directo está desierto; cabe que el mercado ya haya respondido delegando"— **con más fuerza de la que los datos aguantaban**: era en buena parte un artefacto de la R1-B medida sobre el destinatario inmediato, que contaba como "de terceros" a los zaps y routers que reenvían la posición al usuario. Con la R1-B' el cuadro correcto es que **depende del protocolo**:
- **mayoritariamente autodirigido**, aunque pase por zaps o routers que reenvían la posición: Beefy (65 de 73 depósitos en hermanos; 50 de 58 retiradas), sUSDai (83 de 103 depósitos), approve en Arbitrum (57 de 66);
- **realmente desierto**: depósitos en vaults de Morpho (4–10 % propios; 5 de 100 en 2.289 hermanos), depósito en el gauge de Curve (2 de 60), CRV mint (0 de 92); y bajo en los swaps y cobros de Merkl en Base (4–11 %).
Salvedades que siguen: un solo salto (cota inferior), *smart wallets* (cota inferior), posible sobreconteo de pagos a keepers en retiradas y cobros.

### v1.20-draft → v1.21-draft (aprobado por Rod, 2026-10-09)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| R1-B' | **La posición se sigue hasta su destino final DENTRO DE LA MISMA TRANSACCIÓN.** Si el beneficiario inmediato del evento es un contrato que en esa misma transacción transfiere la posición (en los depósitos, el token de la propia posición; en retiradas, peticiones, swaps y cobros, lo obtenido) a quien firma, la transacción cuenta como autodirigida. Las estrategias, keepers y zaps que **se quedan** la posición siguen fuera | **No es una regla nueva: es la R1-B implementada como pretendía** —la misma relación que la R1-B tenía con la R1—. La implementación anterior miraba al destinatario inmediato y por eso contaba como "de terceros" justo los casos que la regla quería incluir: un gestor que llega al contrato a través de un router | §5.2 |

**Declarado sin maquillar:** la R1-B' se propuso **después** de ver los números (con la R1-B, 0 de 8 pares; el depósito en sUSDai, 0 de 103 autodirigidas). **El argumento se sostiene en ambos sentidos:** que un router actúa para quien firma es cierto sea cual sea el efecto en el gas o en el recuento de pares; si al seguir la posición hubieran salido menos pares, la corrección sería la misma.

**Trazabilidad de un error (para el paper).** El 2026-10-08 se afirmó, antes de medirlo, que "la mayoría de las transacciones del zap de Beefy tienen al usuario como beneficiario". **Era incorrecto.** Se detectó al aplicar la R1-B y medir el terreno (2026-10-09 ≈ 03:33 UTC): solo 4 de 73 depósitos en hermanos de Beefy tenían al firmante como beneficiario inmediato. La causa: el zap (`0x6f19…`) acuña las participaciones a su nombre y, en la misma transacción, se las transfiere al usuario. Ese hallazgo mostró que la R1-B medía el destinatario inmediato y no el final, y motivó la R1-B'.

**El "0 de 8 pares" obtenido con la R1-B no es un resultado:** es probablemente un artefacto de la implementación. No se reporta como resultado; el recuento válido es el de la R1-B' (`HECHOS_v1.21_ensayo.md`).

**Límites que siguen (y su sentido):**
- se sigue **un salto** (beneficiario inmediato → firmante); una cadena router → router → firmante, o una entrega en otra transacción posterior, no cuenta → la fracción autodirigida sigue siendo una **cota inferior por este motivo**;
- una cuenta externa que firma para su propia *smart wallet* (posición en la wallet) no cuenta → también cota inferior;
- en retiradas, peticiones, swaps y cobros vale cualquier token entregado al firmante; un pago a un keeper (*bounty*) en una transacción de mantenimiento podría contarse como autodirigido → posible **sobreconteo**, acotado a esas acciones. En los depósitos se exige el token de la posición, que excluye ese caso.

### v1.19-draft → v1.20-draft (aprobado por Rod, 2026-10-09)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| TERRENO | **La proporción de actividad autodirigida frente a la que pasa por automatización de terceros se reporta como hallazgo**, mercado a mercado y acción por acción, con `n` absolutos (§5.6). Deja de ser solo un filtro interno del cálculo del gas | Al excluir de la medición del gas la automatización que actúa sobre posiciones ajenas (R1-B), aparece una pregunta propia: cuánta de la actividad real en estos mercados es autodirigida. Es una observación del terreno, de primer orden para el paper, y se reporta por separado de la puntuación del experimento | §5.6 |

**Aclaración (Rod, 2026-10-09):** la R1-B no excluye la automatización; excluye **actuar sobre posiciones ajenas**. Un agente que gestiona su propio capital (beneficiario = firmante) cuenta como cualquier otro actor; los agentes de la Fase 3 entrarían como actores válidos.

### v1.18-draft → v1.19-draft (aprobado por Rod, 2026-10-09)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| R1-B | **Una transacción cuenta solo si el BENEFICIARIO del evento es quien la firma**: el dueño de la posición o destinatario del evento (`owner`/`receiver`/`provider`/`to`, según la acción) = el `from` de la transacción. En los pools de Curve y en la liquidez de Aerodrome, cuyo evento no trae beneficiario, el destino/origen del `Transfer` del LP acuñado/quemado. Se aplica igual a transacciones propias y de hermanos, en todos los mercados | **No es una regla nueva: es la R1 escrita correctamente.** La justificación de la R1 era "un gestor directo real llega al contrato a través de un router"; "beneficiario = firmante" es la traducción medible de esa intención. Lo que entraba sin ella eran estrategias de otros protocolos, keepers y zaps —es decir, la **automatización del lado delegado**—, y eso inflaba el coste de gestionar directo y **sesgaba a favor de delegar**: un sesgo apuntado a la pregunta del experimento | §5.2 |

**Declarado sin maquillar:** la R1-B se propuso **después** de ver los números (constantes por hermanos 2–8 veces las directas; 0 directas de 52 en el depósito del gauge de Curve; 58 de 62 depósitos en Beefy vía otro contrato). **Contrapeso:** el argumento no depende de esos números. Que la automatización del lado delegado no es lo que paga un gestor directo es cierto sea cual sea su gas; la regla se sostiene igual si el efecto hubiera sido el contrario (si esas transacciones hubieran sido más baratas que las de un gestor, también habría que excluirlas). Por eso se publica la comparación R1 frente a R1-B **en todos los mercados**, incluidos aquellos donde la R1-B empeora el resultado o deja sin dato (`HECHOS_v1.19_ensayo.md`).

**Límite declarado:** una transacción firmada por una cuenta externa en nombre de su propia *smart wallet* (beneficiario = la wallet, firmante = la cuenta) no cuenta. Es una exclusión conservadora y visible; no se intenta resolver la propiedad de la wallet.

### v1.17-draft → v1.18-draft (aprobado por Rod, 2026-10-08)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| MUESTREO | **Muestreo pre-registrado de hermanos (C8 ii) para grupos grandes**, con disparo, orden y tamaño fijados de antemano (§5.5) | Es el mismo mecanismo de orden por semilla pública que ya usa la selección de mercados (§2.1). Lo decisivo: **se pre-registra ahora, sin saber si se disparará ni qué números dará**; un muestreo pre-registrado no se puede acomodar al resultado. El tamaño de la muestra es un **número** (transacciones), no "hasta que vaya rápido": si el criterio de parada fuera el reloj, el reloj decidiría el resultado | §5.5 |
| R2-HERM | (Implementación de la R2, no regla.) No se buscan hermanos de las acciones de **petición** (R2) en contratos que no emiten ninguna petición propia | La petición solo entra en el par si el contrato propio la emite; sus hermanos no se usarían nunca | §5.2 |

### v1.16-draft → v1.17-draft (aprobado por Rod, 2026-10-08)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| R6 | **Filtro de contaminación de la R1.** (i) **approve:** cuentan solo las transacciones cuyo **único** evento es el `Approval`. (ii) **Resto de acciones:** fuera las transacciones que producen además el evento característico (par emisor:evento) de **otra entrada de la tabla**; se **permite** el `Approval` del activo de entrada, porque es parte del mismo depósito | La constante del approve medía **592.437** unidades cuando un approve real cuesta **55.437**, porque el **88 %** de esas transacciones no eran approves (eran `transferFrom` u otras operaciones que también emiten `Approval`); y en Arbitrum la mediana saltó de **173.926 a 51.105** entre dos mediciones consecutivas según qué mezcla cayera en las últimas 50. **Una constante que cambia así no es una medición.** El filtro se apoya en pares emisor:evento ya registrados en cada transacción, así que es objetivo y se aplica igual a todos los mercados | §5.2 |
| I2-ARB | **Fecha de creación en Arbitrum (y contraste en Ethereum) con la PRO API de Blockscout, sin que el agente tenga la clave:** la consulta la hace un proceso aparte, que devuelve **solo** el bloque y la fecha de creación de una dirección. La clave no se imprime ni se registra | El agente no necesita la clave para obtener la fecha de creación | A.3.4 |
| DEDUP | (Implementación, no regla.) Los buscadores que comparten el mismo conjunto de hermanos hacen **una sola** búsqueda | Coste de las búsquedas en nodos públicos con límite de peticiones | §5.2 |

### v1.15-draft → v1.16-draft (aprobado por Rod, 2026-10-08)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| C8ii-F | **De dónde salen los candidatos a hermanos (C8 ii):** (a) la factory por `eth_call`, donde se pueda enumerar; (b) para el resto, listas públicas del propio protocolo en hosts ya permitidos (ydaemon para Yearn, la API de Beefy; DefiLlama antes que una API nueva). **La lista solo propone; la cadena decide:** un candidato es hermano solo si su código (`eth_getCode`) tiene **el mismo hash** que el nuestro | Hallazgo al aplicar la v1.15: el nodo público de Base rechaza `eth_getLogs` sin dirección, así que la búsqueda por eventos no sirve allí. La verificación on-chain mantiene la regla reproducible por un tercero aunque los candidatos vengan de una API | §5 |
| C8ii-S | **Sesgo de muestreo declarado:** la lista de candidatos puede estar incompleta, y entonces la mediana del C8(ii) sale de un **subconjunto** de los hermanos | Ver §5.4 | §5.4 |
| VYPER | **Pertenencia por código excluyendo el bloque final de valores fijos** (contratos desplegados desde un *blueprint*, ERC-5202): mismo tamaño total y **código ejecutable idéntico**, con la longitud del código ejecutable fijada por una regla (§5.3) | **No relaja la condición de pertenencia: la implementa correctamente.** Dos contratos salidos de la misma implementación con parámetros distintos SON la misma implementación; exigir identidad byte a byte sería exigir "misma implementación Y mismos parámetros", que nunca fue la intención de la regla. Medido: los pools y gauges de Curve stableswap-ng hermanos difieren solo en ese bloque final | §5.3 |
| MORPHO | **Candidatos de Morpho: la API de Morpho** (`blue-api.morpho.org`, **solo GET**, añadida a la lista blanca con el mismo estándar que las demás) | La alternativa (los vaults a los que apuntan las estrategias de Yearn y Beefy) tiene un **sesgo correlacionado con la pregunta del experimento**: serían exactamente los vaults que eligieron los agregadores, y lo que se evalúa es si conviene delegar en agregadores. Un sesgo apuntado a la pregunta es peor que un host más de solo lectura (ya había cuatro APIs de terceros permitidas) | §5, §5.4 |
| LOTE | Las búsquedas de logs sobre varios hermanos van en lotes de **5 direcciones** | Medido el 2026-10-08: el nodo público de Ethereum bloquea `eth_getLogs` con 10 o más direcciones ("Request blocked") y acepta 5 | §5.2 |

### v1.14-draft → v1.15-draft (aprobado por Rod, 2026-10-08)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| R1 | **Cuentan las transacciones que PRODUCEN el evento del contrato**, no solo las dirigidas a él (routers, zaps y bundlers incluidos). Se aplica a **todos** los mercados y acciones por igual, y se publica la comparación con la regla anterior en cada uno, empeore o mejore | Un gestor directo real llega al contrato a través de un router, y el gas que paga es el de la transacción que manda de verdad. La regla anterior medía solo llamadas directas, que no es lo que paga nadie | §5.2 |
| R2 | **Acciones en dos pasos: se cuentan los dos.** Si un vault es asíncrono (ERC-7540: emite `RedeemRequest` o `DepositRequest`), salir (o entrar) cuesta la petición más la ejecución | Salir cuesta los dos pasos; medir solo el segundo subestima el coste de gestionar directo | §5.2 |
| I1 | **Ventana del gas:** los 30 días previos a la fecha de referencia (el D0), con lo que haya: el histórico que sirve el nodo (Arbitrum, ≥ 30 d) y la **acumulación propia** desde el ensayo (Base y Ethereum). La cobertura real de cada constante (desde/hasta y bloques examinados) se publica | Opción (a), recomendada y aprobada: ninguna fuente de pago y nada elegido después | §5 |
| I2 | **Fecha de creación en Base:** bloque de despliegue por búsqueda binaria de `eth_getCode` en un nodo de archivo | Medición propia y reproducible; Blockscout no sirve Base (F5) | A.3.4 |
| I3 | **Calldata representativa por acción:** de entre las transacciones que dan la constante de gas, la de tamaño de dato mediano (16 por byte no nulo + 4 por byte nulo); su coste L1 se lee en cada foto. Entra `eth_getTransactionByHash` (solo lectura) en la lista blanca | El coste L1 depende de la calldata de cada acción; un único placeholder por cadena no lo medía | §5, §5.2 |
| APY | **Ventana del APY ≥ un ciclo de cosecha completo** del mercado, con el ciclo medido sobre datos observables | Un vault que cosecha a intervalos no se puede medir en una ventana más corta que su propio ciclo. La duda la levantó Beefy en el ensayo, pero la regla se justifica sola | §7.1 |
| CONTR | **El contraste con DefiLlama compara lo mismo con lo mismo:** si nuestro APY es solo la base del pool (Curve, Convex), contra `apyBase`; y Convex contra su propio pool | Nuestra base de Convex ya coincidía con la base de DefiLlama: era el contraste el que estaba mal planteado (implementación, no regla de medición) | §7.1 |
| TS | **Cada lectura lleva la marca de tiempo de su bloque** (principal o reintento), no la del slot | Es lo medido; con una ventana de 1 día el error era ≈ 2 % relativo del APY | §9 |
| R3 | **Contrapartes parciales:** a cada contrato sin dato se le aplica el C8(ii) (mediana de hermanos de la misma implementación); si tampoco hay, **el par queda fuera** | Es la regla que ya existe para un contrato, aplicada contrato a contrato, sin inventar nada nuevo | §5 |
| R4 | **Convex y Curve en el C8(ii):** solo cuentan los `pid` cuyo pool de Curve es de la **misma familia** (misma factory stableswap-ng, mismo número de monedas, misma implementación); la misma familia da los hermanos de los pools y gauges del lado directo de Curve | El gas depende sobre todo del tipo de pool; mezclar pools de 3 o 4 monedas inflaría la cifra; y la misma regla trata igual a los dos lados del par | §5 |
| R5 | **Ámbito del C8(i) y del C8(ii): dentro de la ventana de 30 días**, con un mínimo de 10 transacciones; se corrige el Anexo C para que coincida con el §5 | Es la única lectura medible con nodos públicos y trata igual a las tres cadenas. **Consecuencia declarada: más mercados pequeños caerán al C8(ii) o a `sin_dato_gas`** | §5, Anexo C |

**Erratas de implementación corregidas en el mismo paquete (no son cambios de regla):** candado en la medición del gas (dos pasadas se pisaban el estado); la ventana de 30 días no se aplicaba al calcular; el tope de los eventos frecuentes recortaba también el histórico; la acumulación de Base dejaba ≈ 30 min sin examinar de cada 6 h (ahora hay una pasada cada 3 h); el recuento del día contaba un reintento posterior al cierre; las fotos llevaban la etiqueta de la v1.13. Una suite de tests nueva cubre la medición del gas (`fase2_gas_tests`).

**Reconocido explícitamente:** R1 y la ventana del APY se decidieron después de ver datos del ensayo: la R1, después de ver que la regla anterior dejaba sin dato el depósito de sUSDai (y fuera a P-D), **sin haber mirado el gas** de esas transacciones; la ventana, después de ver la discrepancia de Beefy. Por eso la R1 se aplica a todos los mercados y se publica su efecto en cada uno, y la ventana se define por una regla de medición que no depende del número.

### v1.13-draft → v1.14-draft (aprobado por Rod, 2026-10-08)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| GAS | **Las unidades de gas se miden ON-CHAIN**: `eth_getLogs` para localizar las transacciones de cada acción y sus recibos para leer el `gasUsed`. Blockscout pasa a ser **contraste** donde funciona (Arbitrum con la clave del F5; Ethereum sin clave), **no fuente**. No se paga ningún plan | Es medición propia, reproducible y sin depender de un tercero que puede cambiar su política o su precio. **Es el mismo criterio que ya se adoptó con el APY** (§7.1): se mide on-chain y la fuente externa queda como contraste | §5, §5.2 |
| LECTURA | **`eth_getLogs` entra en la lista blanca de lectura del guard** (lista completa en el §5.2) | Hace falta para localizar transacciones; es de solo lectura | §5.2 |

**Hecho observado al construirlo (2026-10-08), separado de la razón:** la ventana de 30 días solo es medible gratis en Arbitrum (ver I1 en Pendiente).

### v1.12-draft → v1.13-draft (aprobado por Rod, 2026-10-07)

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| G1 | **Los incentivos que reparte Merkl a quien deposita en un contrato del par se cuentan**, en los dos lados, leídos de la API de Merkl (como ya se usa DefiLlama para precios): solo oportunidades con identificador **exacto** = la dirección del contrato y estado **LIVE**; la sombra paga el gas de reclamar y el swap de la recompensa. Si la API no responde, es un **hueco declarado** (cuenta 0 y se anota), nunca una estimación | **Un gestor directo también podría cobrarlas**: no contarlas le resta rendimiento al lado directo y sesga a favor de delegar | §7, §7.2 |
| G2 | **En la forma "con tiempo humano", el coste de compoundear que compara la regla k incluye el tiempo humano de esa acción** | **Ningún gestor directo compoundearía a diario si cada compounding le cuesta tiempo**: la regla tiene que ver el mismo coste que paga | §4, §13 |

**Hecho observado al aplicar G1 (2026-10-07), separado de la razón:** de las seis contrapartes directas de hoy, solo una (Moonwell Flagship USDC, en Morpho) tiene una campaña LIVE en Merkl (WELL, APR ≈ 0,17 %). La del vault Yearn OG USDC está terminada (PAST). Así que la diferencia de rendimiento vista en la demostración (6,17 % de la estrategia frente a 4,83 % de su vault) **no** la explican incentivos vivos: lo más probable es un *harvest* que cobra de una vez recompensas acumuladas antes, y que cae dentro de la ventana de 7 días. Es ruido de medición que el ensayo tiene que cuantificar.

### v1.11-draft → v1.12-draft (aprobado por Rod, 2026-10-07)

Seis decisiones que aparecieron al construir la herramienta de selección del D0 (F1–F6 de la v1.11).

| # | Cambio | Razón | Dónde |
|---|---|---|---|
| F1 | **P-B se redefine: "vault de Beefy frente a su plataforma subyacente directa"** (antes: vault de Beefy sobre un pool de Curve frente a LP de Curve + gauge) | Es la comparación que responde a la pregunta de investigación: **delegar en un agregador frente a hacerlo directo**. La plataforma de debajo es justo lo que el agregador gestiona por nosotros. *Hecho observado, aparte de la razón: el par original quedó vacío por el mínimo de TVL* (3 vaults de Beefy sobre Curve en Base y Arbitrum, con US$60, US$120.783 y US$932; ninguno en Arbitrum) | §2, §2.2 |
| F2 | **Un mercado que vence solo es elegible si vence ≥ 31 días después del D0** | Un mercado que vence a mitad del mes no se puede observar durante el mes (la posición de 30 días tiene que existir hasta el día 30). **Consecuencia declarada:** hoy solo sobrevive en P-D el PT de sUSDai con vencimiento 25-feb-2027 (US$3,5M), así que **P-D queda con un solo mercado** | §2 |
| F3 | **Codificación exacta de la semilla y del orden** | Sin ambigüedad, cualquiera puede reproducir la selección | §2.1 |
| F4 | **La sombra replica la contraparte directa realista:** lo que un gestor directo podría hacer de verdad. Si el delegado deposita en otro vault (p. ej. una estrategia de Yearn que deposita en un vault de Morpho), la sombra **deposita en ese vault**, no reconstruye a mano la pila de debajo. Mismo criterio en cualquier otra pila, sin reglas ad hoc | Nadie reconstruye la pila entera a mano: depositaría en el vault de Morpho. Es la contraparte honesta | §2.2 |
| F5 | **Blockscout con API key gratuita de solo lectura** (fuera del repositorio y nunca impresa). **No se esquiva el bloqueo falseando el User-Agent, ni ahora ni más adelante** | El bloqueo de Cloudflare se resuelve con la vía que ofrece el propio servicio, no evadiéndolo | §5, A.3.4 |
| F6 | **"Estable" = anclado al dólar**, la unidad en la que se mide. EURC y cualquier estable no-dólar quedan fuera | Los nocionales y la comparación están en dólares: un estable en euros mete movimiento de divisa dentro de una medición de rendimiento | §2 |

### v1.10-draft → v1.11-draft (aprobado por Rod, 2026-10-07; F1–F6 de la lista de pendientes NO están aplicados)

Tres huecos de redacción que aparecieron al construir el colector (Fase A). Ninguno cambia un número ya calculado.

| # | Cambio | Dónde |
|---|---|---|
| G | **Agregación del gas dentro de una foto, por escrito:** mediana, sobre todos los bloques de la ventana, de (baseFee + priority fee del percentil p). Es lo que ya hacía el colector; faltaba que constara | §5 |
| APY | **El APY de cada foto se mide ON-CHAIN** (índice o precio por share), con una regla por tipo de mercado. DefiLlama pasa a ser **contraste, no fuente**. **Razón (Rod):** DefiLlama da un punto diario y no puede alimentar cuatro fotos; y una medición propia es más defendible que un dato de tercero. Margen de discrepancia y protocolo de actuación definidos de antemano | §7.1 |
| L1 | **Arbitrum, coste L1:** se mide con `NodeInterface.gasEstimateL1Component` sobre la calldata representativa (el análogo exacto del `getL1Fee` de Base), y se registra también el estado del *pricer* L1. **Se explica y se deja escrito que un 0 es un valor real**, no un fallo de lectura: el *pricer* de Arbitrum es adaptativo y baja el precio, hasta 0, mientras tiene superávit | §5.1 |

### v1.9-draft → v1.10-draft (aprobado por Rod, 2026-10-06)

| # | Cambio | Dónde |
|---|---|---|
| E1 | **El censo de incidentes se amplía a toda la clase A** (30 protocolos comparables) con las mismas reglas que se aplican a los seis, y con exposición = años-protocolo de la clase A. **Esta corrección SUBE la tasa** (por dólar, de ~0,39 % a ~0,92 %/año, indicativo): va en contra de delegar y no se adopta por conveniencia. El censo completo es una medición del paso D0·2 | Anexo A.2.8, punto 7; Anexo E |
| — | Tasa base central provisional: [0,16 %, 14,84 %] → **[0,50 %, 13,90 %]**. Listón central: 0,15–2,5 pp → **0,35–2,5 pp** | Anexo A.2.8, punto 7 |
| — | Corregida la numeración de la lista de pendientes de la v1.9, que tenía dos puntos "2" | Pendiente |

### v1.8-draft → v1.9-draft (aprobado por Rod, 2026-10-06, salvo lo marcado)

| # | Cambio | Dónde |
|---|---|---|
| D.5a | **La tasa de incidentes pasa a estar en la unidad correcta: por dólar depositado.** Cada incidente se pondera por la fracción `f` del TVL del protocolo que alcanzó. Si no consta, `f` ∈ [pérdida / TVL, 1]. La incertidumbre se propaga a la banda. `λ_inc`: **19,71 %/año por protocolo → 0,39 %/año por dólar** (IC [0,16 %, 12,13 %]) | Anexo A.2.8 |
| D.5b | **Muerte ≠ pérdida de principal.** La pérdida depende de si los depositantes **pudieron salir**, medido en la curva de salida del TVL: las graduales no cuentan en `L`, y las bruscas, las *rugged* y las indeterminadas sí. Clase A: 6 de 6 graduales. Clase B: 76 de 258 sin salida | Anexo A.2.7, punto 11 |
| — | **Antes y después** de la tasa base (central [9,54 %, 50,20 %] → **[0,16 %, 14,84 %]**), de los `T_ij` y del **listón en APY** (central 6–12 pp → **0,15–2,5 pp**) | Anexo A.2.8, A.3.7 |
| — | **Pasada simétrica (Anexo E):** 7 hallazgos sobre posibles sesgos *a favor* de delegar abiertos por la v1.9. **E1 queda como DECISIÓN PENDIENTE** | Anexo E |

### v1.7-draft → v1.8-draft (aprobado por Rod, 2026-10-06)

| # | Cambio | Dónde |
|---|---|---|
| 9b | **La regla de muerte relativa se aplica también a la clase A**, con los mismos parámetros que la B (≤ 2,03 % del pico, ≥ 365 d), sin tocarlos. "A · relativa" pasa a ser la variante **central**; la "A · amplia absoluta" sale de la banda. Tasa base central: **21,67 % [8,33 %, 46,34 %] → 24,11 % [9,54 %, 50,20 %]** | Anexo A.2.7 (puntos 7, 8 y 9b) |
| — | Antes y después del riesgo a 30 días de los seis mercados candidatos (`T` no cambia; cambia la tasa base) | Anexo A.3.6 |
| — | **Anexo D (nuevo): prueba de la objeción contraria.** Solo análisis, **ningún cambio** | Anexo D |

### v1.6-draft → v1.7-draft (aprobado por Rod, 2026-09-24, salvo lo marcado)

| # | Cambio | Dónde |
|---|---|---|
| 1 | **Clase B: se mantiene en la banda** con una **muerte relativa**: TVL ≤ 2,03 % del pico propio, sin recuperación, durante ≥ 365 días. Sustituye al umbral absoluto de US$250k. Los parámetros salen de una medición (un orden de magnitud por debajo de la peor caída del TVL total de DeFi) y de constantes ya existentes, y se declaran. **Resultado: de 26,28 % a 16,05 %/año**, que se mantiene | Anexo A.2.7 (puntos 2, 7 y 9) |
| 2 | La regla `deletion` del ruleset queda **declarada como no probada**, con su motivo | Pendiente, punto 3 |
| 3 | **Hallazgo:** la regla absoluta infracuenta las muertes grandes en la clase A (3 frente a 6). Se propone aplicar también ahí la regla relativa. **DECISIÓN PENDIENTE DE ROD** | Anexo A.2.7, punto 9b |
| — | La rejilla de la banda (ii) pasa a 11.664 combinaciones (23.328 con tiempo humano) | §11 |

### v1.5-draft → v1.6-draft (aprobado por Rod, 2026-09-24, salvo lo marcado)

| # | Cambio | Dónde |
|---|---|---|
| C2 | **Eliminado el caso (c).** Los incidentes de mercados que no comparten mecanismo con el nuestro ya no cuentan, igual que ya no contaban sus meses limpios. Desaparece la dilución por el TVL del protocolo, que era la cuarta capa de sesgo pro-tamaño | Anexo A.3.1 |
| C3 | **Corregido el sesgo de supervivencia:** la tasa base suma una **tasa de muerte** calculada sobre una clase de protocolos comparables que incluye a los muertos y abandonados. El criterio está escrito con el mismo rigor que el censo. ARMA (Giza) y Eliza/ai16z quedan **fuera, con su motivo**. El sesgo residual queda declarado | Anexo A.2.7 |
| C5 | **Mercados que vencen por diseño:** cuentan la antigüedad de su **familia de implementación** (misma factory o versión, en esa cadena) | Anexo A.3.4 |
| C6 | **Selección de mercados sin elección humana:** todos los elegibles hasta N = 3 por protocolo y cadena, ordenados con una semilla pública | §2.1 |
| C8 | **Regla de respaldo** para las unidades de gas en contratos con poca actividad | §5 |
| — | **Antes y después** de los seis `T_ij` y de la tasa base | Anexo A.3.5, A.2.7 |
| — | **La clase B (US$250k) queda fuera de la banda**, como **propuesta pendiente de Rod**, porque se decidió después de ver el resultado. *(Sustituido en v1.7: la B se mantiene con muerte relativa.)* | Anexo A.2.7 |

### v1.4-draft → v1.5-draft (2026-09-24)

| # | Cambio | Estado | Dónde |
|---|---|---|---|
| 1 | **Opción (b) del A.3.3:** la evidencia positiva (meses limpios) se cuenta sobre **el mismo mercado** donde se mide la negativa, no sobre el protocolo entero | **Aprobado por Rod** | Anexo A.3, A.3.4 |
| 1b | Por coherencia con (b), las **auditorías** solo cuentan si cubren la implementación del mercado de la posición (antes: todas las del protocolo) | Aplicado como consecuencia directa de (b); **Rod puede revertirlo** | Anexo A.3, C1 |
| 2 | **Antes y después de los seis `T_ij`** con mercados candidatos elegidos por regla. Yearn **sigue** igual que Curve, pero ahora por una razón explícita: los dos mercados tienen la misma edad | Resultado | Anexo A.3.4 |
| 3 | **Barrido deliberado de sesgos a favor del incumbente** en toda la metodología: 10 hallazgos (1 aplicado, 4 declarados, 5 pendientes de decisión) | Para revisión de Rod | **Anexo C (nuevo)** |

### v1.3-draft → v1.4-draft (aprobado por Rod, 2026-09-23)

| # | Cambio | Dónde |
|---|---|---|
| 1 | **Se ACTIVA el reset proporcional** de la evidencia positiva: sustituye a la resta en la evidencia negativa de `T_ij`. Queda escrita la razón de Rod | Anexo A.3, A.3.2 |
| 2 | **Antes y después de los seis protocolos** sobre el censo actual, no solo el caso hipotético | Anexo A.3.2 |
| 3 | **Hallazgo al activarlo:** en el censo actual, Yearn sigue igual que Curve por dos causas que no son la escala (el alcance asimétrico de la evidencia y la media vida negativa). **Para revisión de Rod**; no bloquea el congelado | Anexo A.3.3 |

### v1.2-draft → v1.3-draft (aprobado por Rod, 2026-09-23, salvo lo marcado)

| # | Cambio | Dónde |
|---|---|---|
| 1 | **Peso negativo de `T_ij`:** el denominador pasa del TVL del protocolo al **TVL del mercado o pool de nuestra posición**. Se define el mercado en cada uno de los seis protocolos, cómo se tratan los incidentes que tocan varios mercados, el mecanismo compartido y los incidentes a nivel de protocolo. Se muestra el antes y el después | Anexo A.3.1 |
| 1b | **Hallazgo:** el arreglo es correcto pero numéricamente insuficiente, porque la evidencia positiva domina. Se propone un *reset proporcional* sin constantes nuevas. *(Adoptado en v1.4.)* | Anexo A.3.2 |
| 2 | La saturación por `TRUST_SCALE` = 5 queda **declarada intencional** | Anexo A.3 |
| 3 | **Aprobados:** ventanas `[hh:00, hh:20)`, reintento a +30 min, día válido con ≥ 3 de 4 fotos, cierre a las 18:20 UTC (**18:50 desde la v1.25, H1**) con la predicción registrada antes de las 00:00, y los criterios C3 y C4 | §9, §10, §14 |
| 5 | **Mecánica concreta del congelado:** buzón en el daemon del Ambassador (único escritor del `paper_log`), anclaje diario de la cabeza de la cadena en el repo público de hashes y OpenTimestamps | §0.2 |

### v1.1-draft → v1.2-draft (aprobado por Rod, 2026-09-22)

| # | Cambio | Dónde |
|---|---|---|
| 1 | **La banda de indiferencia incluye TODAS las sensibilidades pre-registradas:** también el percentil de gas, las variantes de la regla de riesgo, la media vida positiva y los extremos del tiempo humano. Motivo: si se pre-registra una incertidumbre y se deja fuera de la banda, se reconoce pero se factura como si no existiera. Habrá más empates, y son empates reales | §11, §12 |
| 2 | **Media vida positiva de 730 días: APROBADA.** Se **elimina el tope de 36 meses** de D6: la exponencial ya satura sola | Anexo A.3 |
| 3 | **El umbral central de materialidad es US$0.** US$1M y US$10M pasan a sensibilidades. Motivo en el A.2. El rango se recalcula (entra la vault legacy de Yearn del 16-dic-2025) | Anexo A.2 y Anexo B |
| 4 | **Tiempo humano:** ya no se propone un valor. Se pre-registran sus extremos {US$2,08; US$25} por acción y los dos entran en la banda de la forma "con". Se mantiene reportar con y sin | §13 |
| 5 | **TVL mínimo:** baja de US$1M a **US$250k**. El tope del 5 % del TVL ya protege de la iliquidez | §2 |
| 6 | **Cuatro fotos al día** (00:00, 06:00, 12:00 y 18:00 UTC) y se usa la **mediana** | §5, §9 y §10 |
| 7 | Nocionales de 1k / 10k / 100k con tope del 5 %: **sin cambios** | §3 |
| 8 | **Horizontes de 7 y 30 días:** los dos pre-registrados y reportados; ninguno se elige como principal | §3 y §11 |
| 9 | **D1:** se arranca **sin agentes**. Criterio de admisión escrito aquí y congelado antes del día 1. Mamo es el primer candidato, no un compromiso. Si no sobrevive ninguno, es un resultado publicable | §14 |

### v1.0-draft → v1.1-draft (aprobado por Rod, 2026-09-22)

| # rev. | Cambio | Dónde |
|---|---|---|
| — | Principio transversal de pre-registro | §0.5 |
| 19 | `risk_min` / `risk_max` salen de una regla de derivación sobre el historial público de incidentes | Anexo A.2 y Anexo B |
| 20 | Dos medias vidas en `T_ij`: la negativa en 90 días | Anexo A.3 |
| 13 | Tiempo humano reportado con y sin | §13 |
| 18 | LGD pre-registrado {0,3 · 0,6 · 1,0} | Anexo A.1 |
| 11 | MEV pre-registrado: L2 {0 · 10 · 30} pb; mainnet {10 · 30 · 60} pb | §6 |
| 15 | Banda de indiferencia derivada de las sensibilidades | §11 |
| 9 | Cota superior del gestor directo con la `k` óptima a posteriori | §4 y §11 |
| — | Conversión anual → horizonte en forma de Poisson; precisión del "TVL en ese momento" | Anexo A |

## 0. Regla de congelado y principio de pre-registro

1. Este documento pasa a **v1.2** (o a la versión vigente) el día que Rod lo apruebe, **antes del día 1** de la observación.
2. Al congelarlo se registra su sha256 en tres sitios:
   - (a) el catálogo privado, en `rall/fase2/`;
   - (b) la primera entrada de Fase 2 del `paper_log` (`methodology_freeze`, firmada y encadenada). La escribe el **propio daemon del Ambassador**, a través de un buzón (`freeze_request.json`), porque es el único escritor de la cadena. Si el sha256 del archivo no coincide con el aprobado, el daemon **rechaza** el congelado y deja constancia;
   - (c) el anclaje externo aprobado en D5. Cada día, después del cierre, un proceso separado y sin acceso a la identidad del Ambassador publica la **cabeza de la cadena** del `paper_log` (el sha256 de la última línea) en el repo público de hashes y la sella con **OpenTimestamps**. El primer anclaje después del congelado lleva además el sha256 de la metodología.
3. Durante el mes **no se cambia ningún parámetro.** Solo se admiten erratas por bugs de implementación. Una errata se publica como vX.Y.z con su diff, se registra en el `paper_log` y los resultados se reportan con las dos versiones.
4. Las sensibilidades del §12 están **pre-registradas** y **todas** entran en la banda del §11 (salvo la cota `k*`, que es una cota y no una incertidumbre). No se añaden otras después de ver los datos.
5. **Principio transversal (Rod, 2026-09-22).** Cuando un supuesto manda mucho en el resultado y no se puede medir bien, **no se elige un valor**. Se hace una de dos cosas:
   - (a) se congela la **regla de derivación**, y el número es consecuencia de ella;
   - (b) se **pre-registran varios valores**, se publica la sensibilidad y todos entran en la banda.

   **Nunca se elige un valor después de ver los datos.** Cuando un valor central podría estar condicionado por datos ya vistos, el central es el que **no requiere ninguna decisión humana** (ver A.2).

## 1. Qué estima

`c_solo(par, nocional, horizonte)` es el coste, en USD, de obtener **por cuenta propia** la misma exposición que da el servicio delegado:
- gas de entrada, de salida y de cada compounding o rebalanceo;
- slippage y MEV de los swaps, sin la protección que dan los agregadores;
- el arrastre de rendimiento por no compoundear continuamente;
- en la forma "con tiempo humano" (§13), además el coste de cada acción manual. **Desde la v1.26 (TH-SIM), la entrada y la salida se cobran también al lado delegado**: la diferencia entre los dos lados es solo el compounding y el rebalanceo que hace cada uno.

El riesgo de la capa delegada (`L`, `T_ij`) **no** forma parte de la sombra: pertenece al cost model (anexo A).

## 2. Universo de pares (se fija con direcciones exactas al congelar)

**Regla de elegibilidad**, evaluada el día del congelado. Un par entra si cumple todo lo siguiente:
- el vault delegado y el subyacente existen en **Base o Arbitrum**, salvo P-C;
- el TVL de cada lado es **≥ US$250k** según DefiLlama. El tope del 5 % del §3 ya protege de la iliquidez; el suelo solo descarta pools prácticamente vacíos;
- **los activos están anclados al dólar estadounidense**, que es la unidad en la que se mide todo (F6, v1.12). Un estable anclado a otra divisa (EURC, por ejemplo) **no** es elegible aunque DefiLlama lo marque como estable: metería movimiento de divisa dentro de una medición de rendimiento. Operativamente: marcado como estable en DefiLlama **y** con símbolo de un estable en dólares (lista cerrada que se publica con la selección);
- **un mercado con vencimiento** solo es elegible si vence **≥ 31 días después del D0** (F2, v1.12), porque un mercado que vence a mitad del mes no se puede observar durante el mes;
- todos los datos necesarios se obtienen de fuentes públicas gratuitas (plan §4).

| Id | Delegado | Directo (sombra) | Chain | Nota |
|---|---|---|---|---|
| P-A | Vault Yearn v3 (stable) | Réplica de su asignación en la **contraparte directa realista** de cada estrategia (§2.2) | Base | Fees del vault leídas de su *accountant*, por vault; en una estrategia tokenizada, su `performanceFee()` |
| P-B | Vault Beefy (cualquier plataforma) | **La plataforma subyacente directa** del vault, con el criterio del §2.2 (p. ej. el vault de Morpho; el pool de Aerodrome + gauge, con claim y compounding manual) | Base o Arbitrum | **Redefinido en la v1.12 (F1).** El par original (Beefy sobre Curve) quedó vacío por el mínimo de TVL |
| P-C | Convex | LP de Curve + gauge directo | **Ethereum mainnet** | **D2: un único par, tratado y reportado APARTE; nunca se promedia con L2** |
| P-D | PT de Pendle (tipo fijo) | Mantener el subyacente a tipo variable | Arbitrum | Pregunta distinta: se puntúa aparte. Con la regla de vencimiento (F2), hoy **un solo mercado** |
| Control | Aave v3 supply | — | Base y Arbitrum | Referencia sin capa delegada y **control cruzado entre cadenas (D3)** |

**Cadenas (D3):** Base es la principal y Arbitrum la secundaria. Optimism queda fuera.

**Replicación de la asignación (P-A).** La sombra copia la asignación del delegado observada on-chain **con un día de retraso**. Rebalancea solo si algún peso objetivo cambió más de **10 pp**, y como mucho una vez al día.

### 2.1 Selección de mercados (C6, v1.6): nada elegido a mano

1. **Elegibles:** todos los mercados que cumplen la regla del §2 el día del congelado, agrupados por protocolo y cadena. La lista completa se publica.
   - **Censo de medibilidad (§2.3, v1.24):** se evalúa sobre **todos** los elegibles, **sin usar la semilla**, y se publica completo.
2. **Semilla pública:** el hash del **primer bloque de Bitcoin minado después de las 00:00 UTC del día del congelado**. Nadie lo conoce de antemano, cualquiera puede verificarlo y es la misma cadena que ancla los sellos de OpenTimestamps.
   - **Definición exacta (F3, v1.12):** el bloque de **menor altura** cuyo campo `timestamp` de la cabecera es **≥ 00:00:00 UTC** del día del congelado. La semilla son los **32 bytes** de su hash, en el orden en que se muestra habitualmente (el de los exploradores, big-endian; p. ej. `0000…ee02c`).
3. **Orden:** dentro de cada grupo, los mercados **medibles (§2.3)** se ordenan por `sha256(semilla ‖ dirección_del_mercado)` de forma ascendente. Un grupo sin ningún medible se queda sin par.
   - **Codificación exacta (F3, v1.12):** la entrada del sha256 son **52 bytes**: los 32 bytes de la semilla seguidos de los **20 bytes** de la dirección del mercado. Ni texto, ni `0x`, ni mayúsculas o minúsculas: bytes. El resultado se compara como número de 256 bits (o, lo que es igual, como cadena hexadecimal en minúsculas). Para P-C (un único par, D2) se toma el primero.
4. **Máximo:** los **3 primeros** de cada grupo, o todos si hay menos de 3.
   - N = 3 es un **límite operativo**: cada mercado añade lecturas en cada una de las cuatro fotos diarias y filas en cada reporte.
   - No favorece a nadie, porque el orden es aleatorio y verificable.
   - Queda declarado como la única constante que se introduce aquí.
5. **Consecuencia declarada:** los mercados seleccionados pueden ser grandes o pequeños. Los nocionales que choquen con el tope del 5 % (§3) se excluyen caso por caso, como ya estaba establecido.

### 2.2 Contraparte directa: qué replica la sombra (F4, v1.12, Rod)

**Criterio general.** La sombra replica **lo que un gestor directo podría hacer de verdad**. Si el delegado deposita en otro contrato que un usuario puede usar por sí mismo, la sombra **deposita en ese contrato**. No reconstruye a mano la pila de debajo: nadie lo haría, y compararse con algo que nadie hace no responde la pregunta.

**Cómo se aplica, sin reglas ad hoc:**
1. Se sigue la cadena de custodia del delegado (vault → estrategia → …) **hasta el primer contrato en el que un usuario cualquiera puede depositar directamente** el mismo activo. Esa es la contraparte directa.
2. Ejemplos de hoy:
   - Yearn v3 con estrategia "Morpho … Compounder" → **el vault de Morpho** donde deposita la estrategia;
   - Beefy sobre Morpho → **el vault de Morpho**;
   - Beefy sobre Aerodrome → **el pool de Aerodrome + su gauge** (con claim y compounding manual);
   - Convex → **el pool de Curve + su gauge** (como ya estaba).
3. **Si el delegado reparte entre varias estrategias** (vault asignador de Yearn), la sombra replica el reparto entre las contrapartes de cada estrategia, con la regla de rebalanceo del §2 (10 pp, 1 día de retraso).
4. **El riesgo de la contraparte** (por ejemplo, el del vault de Morpho) lo comparten los dos lados, así que no entra en la comparación. El término de riesgo del Anexo A es solo el de **la capa delegada**.
5. **Consecuencia que queda a la vista:** el delegado y la sombra comparten el rendimiento de base, y lo que se mide es lo que el agregador añade (compounding, rebalanceo, gas ahorrado) frente a lo que quita (comisiones, riesgo propio).

### 2.3 Criterio de medibilidad en la selección (M) — **APROBADO por Rod (2026-10-11), v1.24**; se aplica en la selección del D0 (propuesto en la v1.23)

**Problema.** La regla del §2 comprueba el TVL y la elegibilidad, pero **nunca exigió que el mercado fuera medible**. Por eso salen pares que nacen muertos: en el ensayo, 7 de 8 caen por `sin_dato_gas` (R3) y la semilla gasta sus plazas en ellos.

**Regla.**
1. **Qué se exige.** Un par candidato es **medible** si **cada acción del par, de los dos lados** (delegado y directo), tiene en la ventana de 30 días previa al D0 **al menos `M` transacciones válidas**:
   - válidas con las reglas vigentes (R1-B' + R6 + R2);
   - propias o de hermanos verificados (C8 i / ii), contrato a contrato como en R3;
   - y la misma regla para las acciones compuestas.
2. **Valor de `M`: 10 (aprobado), el mismo mínimo que ya fija el C8 (R5).** No introduce ninguna constante nueva: adelanta a la selección lo que R3 haría igualmente durante el mes, de modo que **no excluye ningún par que habría producido predicciones**. Solo evita que la semilla elija un par muerto en lugar de uno vivo del mismo grupo.
   - Un `M` mayor daría medianas algo más estables, a cambio de una constante nueva y de **más** sesgo de selección (abajo). No se recomienda.
   - **Declarado:** esta recomendación se hace habiendo visto que con `M` = 10 o 20 sobrevive P-D y con `M` = 30 no. El argumento (no añadir ninguna constante) no depende de eso.
3. **Se fija antes del D0 y se aplica igual a todos los grupos.** Es un criterio de **viabilidad de la medición**: no mira ni el APY, ni el resultado, ni qué lado gana.

**Encaje con la selección por semilla (§2.1).**
- **Orden de los pasos el día del congelado:**
  1. elegibles del §2, con la lista completa;
  2. **censo de medibilidad de TODOS los elegibles**, no solo de los que saldrían elegidos;
  3. orden por `sha256(semilla ‖ dirección)` **solo entre los medibles**;
  4. los N primeros de cada grupo (N = 3; en P-C, 1).
- Es lo mismo que recorrer el orden completo de la semilla saltándose los no medibles. Se publican las dos listas con sus claves de orden, para que cualquiera compruebe que el salto fue mecánico.
- **Anclaje de la selección (orden de Rod, 2026-10-30).** El mismo día del congelado, antes de que empiece la medición, se anclan con OpenTimestamps las dos listas con sus claves de orden, la semilla y los datos de entrada con los que se calcularon (las respuestas de DefiLlama y los recuentos de actividad de los 30 días). Los datos de un día no se pueden reconstruir después; sin ese anclaje, «cualquiera puede reproducir la selección» no sería cierto.
- **El censo no usa la semilla**, así que la medibilidad de cada candidato queda decidida sin saber cuál saldría elegido.
  - Excepción declarada: en los grupos de hermanos grandes, el muestreo del §5.5 recorre el grupo en el orden de la misma semilla, con el tope de recibos por grupo de la implementación.
  - Sin ese tope, el veredicto "≥ 10" no depende del orden, porque el muestreo solo para antes de agotar el grupo cuando ya tiene 50.
  - Con el tope, en un grupo enorme y casi inactivo el veredicto **puede** depender de la semilla. Se declara. La semilla es pública y está fijada de antemano, así que no admite elección.
- **Grupo sin ningún candidato medible:** el grupo se queda sin par, se dice así y se publica su censo. No se rellena con un no medible ni se baja `M`.
- **Requisito operativo:** en Base y Ethereum el gas solo existe si el colector lo ha acumulado (los nodos públicos no sirven historial; Base exige dirección en `eth_getLogs`). Para que el censo del D0 tenga 30 días, **la acumulación tiene que cubrir desde 30 días antes a TODOS los elegibles**, y no solo a los 8 pares del ensayo. Hoy cubre solo a los 8.
  - Un mercado que pase a ser elegible más tarde tendrá menos ventana, y se declara con su cobertura real.
  - **Elegibles el D0 que no lo eran el 2026-10-07 (Rod, 2026-11-05):** salen como **no medibles**, declarándolo. El censo acumuló datos solo de los elegibles del 7-oct, y en Base los nodos públicos no sirven historial, así que esos mercados no tienen datos o tienen menos ventana. **La causa es una limitación de cobertura de nuestro censo, no una propiedad del terreno.** El documento del D0 dice cuántos son y cuáles, con su TVL. **Dirección del sesgo: desconocida a priori.** Son mercados que cruzaron el umbral de elegibilidad hace menos de un mes (nuevos o que crecieron), y no hay dato que diga si en ellos domina delegar o gestionar directo. Por eso se publica qué fracción del universo elegible queda fuera por esta causa, en número y en TVL.

**SALVAGUARDA (Rod) — la parte importante.**
- **El criterio introduce un sesgo correlacionado con la pregunta.** Si solo entran mercados donde la gente gestiona directo, se están eligiendo mercados donde delegar domina menos, y eso **empuja el resultado hacia DIRECTO**. Es el mismo tipo de sesgo que Rod rechazó con Morpho.
- **Dos matices de medición:**
  - El sesgo **ya existe hoy implícitamente**: R3 deja fuera durante el mes los pares no medibles, así que cualquier resultado sale ya de un subconjunto medible. El criterio no lo crea: lo hace **explícito, pre-registrado y publicado**.
  - El signo no es seguro a priori. En el ensayo, 5 de los 7 pares muertos caen por acciones del lado **delegado** (Yearn y Beefy) y 2 por el lado **directo** (Aerodrome y el gauge de Curve). Se mantiene declarada la dirección hacia DIRECTO como la presunción prudente, y **la salvaguarda no depende del signo**.
- **Por eso no se sustituye una cosa por otra: se conservan las dos.**
  - **(a) Se publica el CENSO completo:** cada mercado elegible, si es medible o no, y con qué números. Por acción y por lado: transacciones válidas, autodirigidas y de terceros, la cobertura real de la ventana y la causa de cada "no medible" (ventana corta, R6, hermanos inexistentes o inactivos). **Ese censo es en sí un hallazgo del terreno (§5.6)** y se reporta aunque el cost model no se corra en esos mercados.
  - **(b) El cost model se corre solo donde hay datos, DECLARANDO EXPLÍCITAMENTE** en cada resultado, tabla y conclusión del paper que ese subconjunto **no es representativo del mercado**. Se dice en qué dirección se presume que sesga (hacia DIRECTO) y qué fracción del universo elegible representa, en número de mercados y en TVL.
- **Qué NO se puede concluir del subconjunto medible:**
  1. Nada sobre "el mercado" ni sobre el universo elegible: ni qué fracción de mercados compensa delegar, ni un resultado medio extrapolable.
  2. Que el camino directo sea viable en general: en el subconjunto lo es **por construcción**.
  3. Nada sobre los mercados no medibles. Que un grupo (p. ej. P-C) se quede sin pares **no** es evidencia ni a favor ni en contra de delegar en él.
  4. Comparaciones entre grupos cuando uno de ellos no tiene mercados medibles, ni promedios que mezclen grupos con distinta cobertura.
  5. Que la proporción de mercados medibles mida cuánta gente gestiona directo: cuenta transacciones, no personas ni capital, y es cota inferior (un solo salto, *smart wallets*).
  6. Cualquier frase incondicional. Lo único válido es la forma "en los mercados con actividad autodirigida suficiente para medir el lado directo, …".

**Números del ensayo** (`HECHOS_v1.22_ensayo.md`; Base ≈ 1,7 días y Ethereum ≈ 2,7 días de datos, así que son **una cota baja, no el censo del D0**):
- con `M` = 10 o 20 sobrevive **1 de los 8** pares seleccionados (P-D);
- con `M` = 30 o 50, **0 de 8**.

En P-A, P-B y P-D los seleccionados son **todos** los elegibles; en P-C hay 32 elegibles y solo se ha medido 1. El censo real se hace el D0.

## 3. Nocionales y horizontes

- **Nocionales:** US$1,000 / US$10,000 / US$100,000 por par. Un nocional que supere el **5 % del TVL** del pool más chico del par se excluye (supuesto de no dilución).
- **Horizontes, los dos pre-registrados y los dos reportados; ninguno es "el principal":**
  - **7 días rodantes:** posiciones abiertas cada día d = 1…23 y cerradas el día d+7 (23 posiciones por par y nocional);
  - **30 días:** posición abierta el día 1 y cerrada el día 30. **Limitación declarada:** es n = 1 por par y nocional en el mes, así que se reporta como caso, no como estadística.

## 4. Acciones de la sombra y compounding

| Acción | Contenido |
|---|---|
| Entrada | Swap al activo, si hace falta; depósito o `add_liquidity`; stake en el gauge, si aplica |
| Compounding | Claim de rewards; swap al subyacente; nuevo depósito |
| Rebalanceo | Retirar de un mercado y depositar en otro (solo P-A) |
| Salida | Unstake; retirada; swap de vuelta al activo de partida |

**Regla de compounding:** se evalúa una vez al día, al cierre del día de observación (§9). Se compoundea si `rewards_acumulados_usd ≥ k × coste_estimado_compounding_usd`. Al salir siempre se liquida todo.
- **G2 (v1.13, Rod):** en la forma "con tiempo humano" (§13), `coste_estimado_compounding_usd` **incluye el tiempo humano de esa acción**. Ningún gestor directo compoundearía a diario si cada vez le cuesta tiempo: la regla ve el mismo coste que paga. En la forma "sin", el coste es solo gas, swaps y MEV.

- `k` ∈ {2, 5, 10}. **Los tres entran en la banda.** El 5 es solo el valor central que se muestra en las tablas.
- **Cota superior del gestor directo (punto 9):**
  - `k*` = argmin del `c_solo` realizado sobre la rejilla K* = {1, 1,5, 2, 3, 5, 7, 10, 15, 20, 30, 50, ∞}, donde ∞ significa liquidar solo al salir. Se calcula para cada par, nocional, horizonte y forma (con o sin tiempo humano), con los valores centrales de las demás sensibilidades.
  - **Usa información futura: es una cota, no una predicción.** Nunca entra en la predicción diaria ni en la banda.

## 5. Gas (precios reales de ese día)

- **Unidades de gas por acción (v1.14, Rod): medidas ON-CHAIN.** Se congelan como **constante por acción** y se publican en la versión congelada. Detalle en el §5.2.
  - **Razón:** medición propia, reproducible y sin depender de un tercero que puede cambiar su política o su precio; el mismo criterio que el APY (§7.1). Blockscout queda como **contraste** donde funciona, no como fuente.
  - **Ventana (I1, v1.15) y agregación (V3, 2026-10-31):** **todas** las transacciones válidas de los **30 días previos al congelado**, agregadas con **V3: la mediana de las medianas diarias** (criterio pre-registrado, aplicado el 2026-10-31; sustituye a «las 50 últimas»), con lo disponible: el histórico que sirve el nodo (Arbitrum) y la acumulación propia desde el ensayo (Base y Ethereum: desde lo que servía el nodo en la primera pasada con la regla v1.15, ≈ 5,3 h y ≈ 33 h antes; esa fecha se publica). La fecha de la transacción más antigua y la más reciente de cada constante se publican junto a ella. La acumulación corre en cada foto, en cada reintento y en una pasada suelta cada 3 h (el nodo de Base sirve ≈ 5,3 h de logs).
  - **Respaldo (C8, v1.6; ámbito y alcance fijados en la v1.15: R3, R4, R5)**, para contratos con poca actividad. **Todo dentro de la ventana de 30 días** (R5):
    - (i) con al menos 10 válidas propias en la ventana, la constante es su agregado V3 (el corte de 50 desaparece con V3);
    - (ii) si aun así hay menos de 10, se usa el agregado V3 de la **misma acción** en los contratos **hermanos** de la **misma implementación** en esa cadena, también dentro de la ventana. Los hermanos se encuentran on-chain: por **hash de bytecode** (`eth_getCode`) entre los contratos que emiten el mismo evento en el tramo reciente (`eth_getLogs` sin filtrar por dirección); en **Curve y Convex (R4)**, por **familia**: los pools de la misma factory stableswap-ng con el mismo número de monedas y la misma implementación, sus gauges, y los `pid` del Booster de Convex cuyo pool es de esa familia (con sus contratos de recompensas). La misma familia sirve a los dos lados del par;
    - (iii) si tampoco hay 10, la acción queda `sin_dato_gas`.
  - **Candidatos y pertenencia (v1.16):** los candidatos a hermanos salen (a) de la factory por `eth_call` donde se pueda enumerar o (b) de las listas públicas del protocolo en hosts ya permitidos; **la lista solo propone y la cadena decide**: un candidato cuenta como hermano solo si su `eth_getCode` tiene el mismo hash que el nuestro. Las transacciones de un candidato no verificado no entran nunca en la mediana. Si una cadena permite buscar eventos sin dirección (no es el caso de Base), los emisores encontrados son candidatos y pasan la misma verificación. Ver el sesgo de muestreo en el §5.4.
  - **Varios contratos en una acción del par (R3):** el C8 se aplica **contrato a contrato**; si un contrato sigue sin dato, la acción queda sin dato y **el par queda fuera**. Con todos medidos, la constante del par es la media de las medianas.
  - **Consecuencia declarada (R5):** al no salir de la ventana, más mercados pequeños caen al C8(ii) o a `sin_dato_gas` (y su par, fuera). Es el precio de una regla medible con nodos públicos e igual en las tres cadenas.
  - El criterio aplicado (propias / C8 ii / iii, siempre con V3) y el número de transacciones quedan anotados junto a cada constante.
  - **Declarado (lectura de quien redacta; la decide Rod):** V3 se calcula sobre **todas** las transacciones válidas almacenadas de la ventana, que es como lo midió el criterio (el mismo motor sin el corte de 50 ni el tope de muestra). El muestreo de hermanos del §5.5 no cambia: decide qué se recoge, no cómo se agrega.
- **Precio L2 (ejecución) de cada foto:** `eth_feeHistory` sobre **todos** los bloques de su ventana (§9). Para cada bloque se calcula `baseFee + priorityFee_p`, con la priority fee del **percentil p** de ese bloque y p ∈ {25, 50, 75}; el precio de la foto es la **mediana sobre los bloques de la ventana** de ese valor (v1.11). Los tres percentiles entran en la banda.
- **Coste L1 de datos de cada foto:**
  - Base (OP-stack): `GasPriceOracle.getL1Fee(calldata_representativa)` en el primer bloque de la ventana, todas las acciones en la misma llamada (Multicall3). La calldata representativa de cada acción se congela (I3, §5.2 punto 7).
  - Arbitrum: `NodeInterface.gasEstimateL1Component(destino, false, calldata_representativa)` en el primer bloque de la ventana, que devuelve las **unidades de gas L2 que se cobran por el dato L1**; se multiplican por el precio L2 de la foto. Se registra además el estado del *pricer* (`ArbGasInfo`: precio por unidad, superávit, fondos debidos). Ver §5.1 (v1.11).
- **Mainnet (solo P-C):** el mismo método de percentiles.
- **Coste del día:** la **mediana de las fotos válidas del día** (§9 y §10).
- **ETH/USD:** precio de DefiLlama en cada foto; mediana del día.

### 5.1 Arbitrum: el coste del dato L1 puede ser 0, y es un valor real (v1.11)

**Hallazgo (2026-10-07, al construir el colector).** En la primera foto real, Arbitrum reportó **0** como precio del dato L1 (`ArbGasInfo.getPricesInWei`, índice 1). Como Pendle (P-D) vive en Arbitrum, si ese 0 fuera un error la sombra **subestimaría** el coste de gestionar directo. Se investigó antes del ensayo.

**Cómo factura Arbitrum el dato L1.** Arbitrum no cobra el coste de publicar en L1 a precio de mercado transacción a transacción. Cobra **unidades de gas L2 adicionales** (`gasUsedForL1`), con un precio por unidad que fija un ***pricer* adaptativo** (ArbOS). El *pricer* compara lo recaudado con lo que se debe al *batch poster*:
- con **déficit**, el precio por unidad sube;
- con **superávit**, baja, y **puede llegar a 0** y quedarse ahí mientras dure el superávit.

**Medición (2026-10-07, bloque ≈ 512.447.700):**

| Lectura | Valor |
|---|---|
| `ArbGasInfo.getL1BaseFeeEstimate()` (precio por unidad) | 0 |
| `ArbGasInfo.getL1PricingSurplus()` | **+0,0062 ETH** (superávit) |
| `NodeInterface.gasEstimateL1Component` (supply de Aave, 164 bytes) | **0 unidades de gas L1** |
| Precio por unidad hace ≈ 7 días | 37.518.013 wei (superávit +0,0060 ETH) |
| Precio por unidad hace ≈ 90 días | 8.190.204 wei (déficit ≈ −0,00015 ETH) |

**Comprobación con transacciones reales (2026-10-07).** Recibos de 38 transacciones de usuario en 10 bloques recientes de Arbitrum, con calldata de 68 a 7.364 bytes: las **38 tienen `gasUsedForL1 = 0`**. Coincide con el `NodeInterface` y con el *pricer*. Una foto del colector instalado, unos 27 minutos después del diagnóstico, volvió a dar 0 con un superávit de +0,0059 ETH.

**Conclusión.** El 0 es **el precio que se cobra de verdad** en ese momento: el estimador del propio nodo (el que usan las wallets) dice que una transacción enviada entonces paga 0 por el dato L1. **No es un fallo de lectura ni el oráculo equivocado.** Pero **no es una constante**: hace una semana no era 0. Por eso:
- el coste L1 de Arbitrum se **mide en cada foto** con el `NodeInterface` (§5) y nunca se supone;
- el colector guarda también el estado del *pricer*, para poder auditar cada 0;
- si durante el mes el precio vuelve a subir, la sombra lo recoge sin cambiar ninguna regla.

**Límite declarado.** La lectura es el coste que pagaría una transacción **en el primer bloque de la ventana**. Si el *pricer* cambia dentro de la ventana, la foto no lo ve; la mediana de cuatro fotos al día lo suaviza.

### 5.2 Cómo se miden las unidades de gas on-chain (v1.14; R1, R2 e I3 en la v1.15)

**1. Qué transacción representa una acción.** Cada acción de la sombra o del delegado (§4) tiene un **contrato** y un **evento característico** que ese contrato emite cuando se ejecuta. Una transacción cuenta si:
- **produce** ese evento en ese contrato (se localiza con `eth_getLogs`), **vaya directa o a través de un router, *zap* o *bundler*** (R1, v1.15);
- su recibo es de éxito (`status = 1`).

**Destino final de la posición (R1-B', v1.21, Rod).** Si el beneficiario inmediato no es quien firma, se mira el resto de la misma transacción: si ese beneficiario transfiere al firmante el token de la posición (depósitos) o lo obtenido (retiradas, peticiones, swaps, cobros), la transacción cuenta. Es la R1-B implementada como pretendía (ver el historial). La columna "R1-B" (destinatario inmediato) se sigue publicando para comparar.

**Beneficiario = firmante (R1-B, v1.19, Rod).** De las transacciones que producen el evento, cuentan solo aquellas en las que el beneficiario del evento es quien firma la transacción. Por acción: depósitos en vaults (ERC-4626, Yearn) → `owner` del `Deposit`; retiradas → `owner` del `Withdraw`; peticiones ERC-7540 → `owner`; Beefy → destino del mint / origen del burn de shares; Pendle → `receiver` del `Swap`; gauges de Curve → `provider`; CRV → `recipient` del `Minted`; Convex → `user`; Aerodrome → `to` del `Burn` y del `Deposit` del gauge, `from` del `Withdraw`/`ClaimRewards`; approve → `owner`; swap → `recipient`/`to`; Merkl → `user`; liquidez de Curve y de Aerodrome → destino/origen del `Transfer` del LP acuñado/quemado por el pool en la misma transacción. Si el beneficiario no se puede leer, la transacción **no cuenta** (no se supone). **No es una regla nueva: es la R1 escrita correctamente** (ver el historial).

**Filtro de contaminación (R6, v1.17, Rod).** De esas transacciones se excluyen:
- en la acción **approve**, todas las que emiten algún evento además del `Approval` (solo cuenta un approve de verdad);
- en las demás acciones, las que producen además el evento característico (par emisor:evento) de **otra entrada de la tabla** (otro contrato, u otra acción del mismo contrato). Se permite el `Approval` del activo de entrada, que es parte del mismo depósito.
Razón: sin el filtro, la constante del approve en Base medía 592.437 frente a los 55.437 de un approve real (el 88 % no eran approves), y la de Arbitrum saltó de 173.926 a 51.105 entre dos mediciones consecutivas. Una constante así no es una medición. El filtro usa solo los pares emisor:evento registrados en cada recibo: es objetivo e igual para todos los mercados. Se publican también las constantes sin el filtro (R1 sola) y con la regla anterior, para comparar.

**Razón (Rod, v1.15):** un gestor directo real llega al contrato a través de un router, y el gas que paga es el de la transacción que manda de verdad. Hasta la v1.14 solo contaban las llamadas directas (`recibo.to` = el contrato), que no es lo que paga nadie. La regla se aplica **a todos los mercados y acciones por igual**. Cada transacción guarda si era directa, su `to`, su `from` y los eventos que emitió, para publicar la comparación con la regla anterior y medir la contaminación (R6).

| Acción | Contrato | Evento |
|---|---|---|
| Depósito / retirada en un vault ERC-4626 (Morpho, Euler, Fluid, Yearn v3, sUSDai) | el vault | `Deposit(address,address,uint256,uint256)` / `Withdraw(address,address,address,uint256,uint256)` |
| Depósito / retirada en un vault de Beefy | el vault | `Transfer` desde / hacia la dirección cero (mint / burn) |
| Añadir / quitar liquidez en Curve (stableswap-ng) | el pool | `AddLiquidity(address,uint256[],uint256[],uint256,uint256)` / `RemoveLiquidityOne(address,int128,uint256,uint256,uint256)` |
| Stake / unstake en el gauge de Curve; reclamar CRV | el gauge; el Minter | `Deposit(address,uint256)` / `Withdraw(address,uint256)`; `Minted(address,address,uint256)` |
| Depósito / retirada / reclamar en Convex | el Booster (filtrado por `pid`); el reward pool | `Deposited(address,uint256,uint256)`; `Withdrawn(address,uint256)` / `RewardPaid(address,uint256)` |
| Liquidez y gauge de Aerodrome | el pool (vía Router); el gauge | `Mint` / `Burn`; `Deposit` / `Withdraw` / `ClaimRewards` |
| Comprar / vender PT de Pendle | el market (vía Router de Pendle) | `Swap(address,address,int256,int256,uint256,uint256)`, con el signo de `netPtOut` |
| `approve` | el token | `Approval(address,address,uint256)` |
| Swap genérico de una recompensa | router de referencia por cadena, sobre un pool de referencia USDC/WETH | `Swap` del pool, con el router como `sender` |
| Reclamar incentivos de Merkl | el distribuidor de Merkl | `Claimed(address,address,uint256)` |
| Petición de salida / de entrada en un vault asíncrono (ERC-7540) (R2, v1.15) | el vault | `RedeemRequest(address,address,uint256,address,uint256)` / `DepositRequest(address,address,uint256,address,uint256)` |

Si un evento no aparece en ninguna transacción de la ventana, la acción queda en el respaldo C8 y se revisa a mano si la firma del evento es la correcta antes de congelar.

**2. Qué se mide.** `unidades = gasUsed` del recibo. En **Arbitrum** se resta `gasUsedForL1` (el dato L1 se cobra aparte, §5.1); en **Base** el `gasUsed` ya es solo la ejecución L2 (el coste L1 va aparte, §5).

**3. Cómo se agregan varias transacciones en una unidad.** **V3 (desde el 2026-10-31): la mediana de las medianas diarias** de todas las válidas de la ventana (solo transacciones con fecha dentro de la ventana). Sustituye a «la mediana de las 50 últimas».

**Acciones en dos pasos (R2, v1.15):** si en la ventana el contrato emite al menos una petición ERC-7540 (`RedeemRequest` o `DepositRequest`), la salida (o la entrada) es de dos pasos y su coste es **la suma** de la mediana de la petición y la de la ejecución (`Withdraw` o `Deposit`). Si la petición tiene menos de 10 transacciones, se le aplica el C8 como a cualquier acción. Razón (Rod): salir cuesta los dos pasos; medir solo el segundo subestima el coste de gestionar directo. Revisados los demás mercados del universo: Yearn v3, Beefy, Morpho, Pendle y Convex salen en una sola transacción; las salidas de Curve y Aerodrome ya se modelaban como varias acciones (retirar del gauge + quitar liquidez). Si una acción del par se hace en **varios contratos** (P-A con varias contrapartes), se toma la media de las medianas de cada contrato. ~~Las acciones que no se pueden medir aparte (el *zap* de Beefy sobre Aerodrome) son la suma de sus componentes medidos.~~ **Retirado en la v1.22 (R7):** el coste del *zap* de Beefy es el depósito/retirada de Beefy medido tal cual, que con la R1-B' ya es la transacción completa del *zap*.

**4. Contraste.** Donde Blockscout funciona (Arbitrum con la clave del F5, Ethereum sin clave), se compara la mediana propia con la de Blockscout para el mismo contrato y la misma función. Una discrepancia se anota; **nunca sustituye** al valor on-chain.

**5. Lista blanca de lectura del guard, completa, después de este cambio.** En los nodos RPC solo se admiten estos métodos, todos de **solo lectura**:
`eth_call`, `eth_chainId`, `eth_blockNumber`, `eth_getBlockByNumber`, `eth_feeHistory`, `eth_getCode`, `eth_getTransactionReceipt`, `eth_getLogs` (v1.14) y **`eth_getTransactionByHash`** (nuevo en la v1.15, I3: leer la calldata de las transacciones medidas).
Cualquier otro método se bloquea. Los de firma o envío (`eth_sendTransaction`, `eth_sendRawTransaction`, `eth_sendUserOperation`, `eth_sign*`, `personal_*`, `wallet_*`) se bloquean **antes** de mirar la lista, aunque el host estuviera permitido.

**6. Valores provisionales.** Durante el ensayo se miden valores provisionales en las tres cadenas (y se acumulan en cada foto, en cada reintento y cada 3 h). **Los definitivos son los del día del congelado**, con la ventana de 30 días previos (§5).

**7. Calldata representativa de cada acción (I3, v1.15).** De entre **las 50 más recientes** de las transacciones que dan la constante de gas de una acción en un contrato, se lee su calldata (`eth_getTransactionByHash`) y se elige la de **tamaño de dato mediano**, medido como 16 por byte no nulo + 4 por byte nulo (con un número par, la menor de las dos centrales; empate, por hash). Esa calldata, con su destino real (el router si la transacción iba por un router), es la que se pasa en cada foto a `getL1Fee` (Base) o a `gasEstimateL1Component` (Arbitrum). Se congela el D0. Si una acción del par usa varios contratos, su coste L1 es la media de los de cada contrato, igual que las unidades.
- **Declarado (Rod, 2026-11-05): aquí V3 no se aplica del todo.** Desde V3 las unidades salen de TODAS las válidas de la ventana, pero la calldata representativa se elige entre las 50 más recientes. Motivo: leer la calldata de todas exigiría miles de llamadas `eth_getTransactionByHash` a nodos públicos. El efecto es que el coste L1 de cada acción representa a las transacciones recientes, no a toda la ventana.

**Búsquedas sobre varios hermanos (v1.16):** en lotes de 5 direcciones por llamada a `eth_getLogs` (límite medido del nodo público de Ethereum); el resultado se junta en orden de bloque.

**8. Candado y cobertura.** Las pasadas de medición no pueden solaparse (candado de archivo). Cada pasada anota el tramo de bloques que examinó de verdad; la cobertura se publica.

### 5.3 Pertenencia de contratos con valores fijos en el código (v1.16, Rod)

**Por qué.** Muchos contratos guardan valores de la instancia (tokens, pool, `pid`, dueños…) dentro de su propio código desplegado. Dos hermanos de la misma implementación tienen entonces códigos distintos aunque ejecuten exactamente lo mismo. La regla de pertenencia del C8(ii) pide **la misma implementación**, no los mismos parámetros: comparar el código excluyendo esos valores **implementa** la regla, no la relaja.

**Regla fija y reproducible (contratos desplegados desde un *blueprint*, ERC-5202, como los pools y gauges de Curve stableswap-ng).** En estos contratos el código desplegado es el código ejecutable seguido de un bloque final con los valores fijos.
1. El *blueprint* es el que la propia factory declara: `get_implementation_address(pool)` para los pools y `gauge_implementation()` para los gauges (lecturas `eth_call`). Su código (`eth_getCode`) empieza por la cabecera `0xFE7100` del ERC-5202.
2. **Longitud del código ejecutable `L`** = el prefijo más largo del código desplegado del contrato de nuestro par que aparece tal cual dentro del código del *blueprint*. No depende de los hermanos ni de "lo que difiera".
3. Un candidato es hermano si su código desplegado tiene **el mismo tamaño total** que el nuestro y **sus primeros `L` bytes son idénticos** a los nuestros. El bloque excluido son los bytes finales a partir de `L`.

Un tercero lo reproduce sin criterio propio con dos `eth_call` y dos `eth_getCode`. Medido el 2026-10-08: pools `L` = 21.491 de 23.635 bytes (bloque final de 2.144); gauges `L` = 10.597 de 10.725 (bloque final de 128).

**Cuándo NO se aplica.** Si los valores fijos no están en un bloque final sino **dispersos** dentro del código (típico de Solidity: los `immutable` se incrustan en cada punto de uso), no hay un recorte limpio y **no se fuerza**: los hermanos no se pueden verificar y la acción queda `sin_dato_gas` si no tiene datos propios. Medido el 2026-10-08: los **gauges de Aerodrome** difieren en 10–15 tramos dispersos entre los bytes 543 y 4.516: sin recorte limpio → sin hermanos verificables.

### 5.4 Sesgo de muestreo del C8(ii): las listas de candidatos pueden estar incompletas (v1.16, Rod)

**Qué implica.** Cuando una constante de gas sale del C8(ii), su mediana se calcula sobre las transacciones de **los hermanos que conocemos**, no de todos los que existen. Si la lista de candidatos (factory o lista pública) no incluye a algunos hermanos, la mediana sale de un **subconjunto**: es un **sesgo de muestreo**. No es aleatorio: las listas públicas tienden a incluir los contratos que el propio protocolo promociona o mantiene activos, y a omitir los abandonados o los desplegados por terceros, que pueden tener un patrón de uso distinto (y por tanto un gas distinto).

**Qué se hace.**
- La pertenencia nunca depende de la lista: se verifica on-chain (mismo hash de `eth_getCode`). La lista solo puede **quitar** hermanos, nunca **añadir** uno que no lo sea.
- Para cada constante del C8(ii) se publican la fuente de candidatos, cuántos candidatos propuso, cuántos pasaron la verificación y cuántos tuvieron transacciones en la ventana.
- **Completitud medida donde se puede comparar** la lista con la enumeración de la factory (ver `HECHOS_v1.16_ensayo.md`): la API de Curve frente a la factory stableswap-ng en Ethereum, **1.069 de 1.069 (100 %)**. Donde no hay factory enumerable (Beefy, Yearn y Morpho en Base), la completitud se declara **desconocida**. La API de Morpho enumera los vaults que indexa (en Base, 513 de la v1 y 2.547 de la v2 el 2026-10-08, de ellos solo 24 y 11 "listed"); no hay forma on-chain de saber si indexa todos.

**Qué no se hace.** No se corrige la mediana por la posible incompletitud (no hay con qué estimarla sin elegir un supuesto); se declara.

### 5.5 Muestreo pre-registrado de hermanos en grupos grandes (v1.18, Rod)

**Cuándo se dispara (umbrales fijos).** Por cadena, si **una pasada completa de acumulación** en esa cadena, medida y registrada, **dura más de 20 minutos** con la implementación vigente (deduplicación de grupos y sin búsquedas de peticiones R2 innecesarias). Una vez disparado en una cadena, queda activo hasta el día del congelado, y en el D0 se registra y se congela su estado (activo o no) por cadena. Solo afecta a los **grupos de hermanos de más de 500 direcciones**; los menores se recorren enteros.

**Orden (fijo, público).** Las direcciones del grupo se recorren por orden ascendente de `sha256(semilla ‖ dirección)`, con la dirección en sus 20 bytes. La semilla es la del D0 (§2.1: el hash del primer bloque de Bitcoin tras las 00:00 UTC del día del congelado); durante el ensayo, la semilla de la selección del ensayo.

**Tamaño de la muestra (un número, no el reloj).** Se recorren las direcciones en ese orden, en lotes de 5, examinando la ventana de 30 días, **hasta reunir 50 transacciones válidas** (las que pasan R1 y R6) o agotar el grupo. Con menos de 50 al agotar el grupo se usan todas, con el mínimo de 10 del C8; con menos de 10, `sin_dato_gas`. El tiempo que tarde **no** interviene en cuándo se para.

**Razón (Rod).** Es el mismo mecanismo de orden por semilla pública que la selección de mercados (§2.1), y se pre-registra **antes** de saber si se disparará y qué números dará: un muestreo pre-registrado no se puede acomodar al resultado. El sesgo que introduce es el de un muestreo aleatorio del grupo (sin relación con la pregunta del experimento), y se suma al de la posible incompletitud de la lista de candidatos (§5.4).

**Estado del disparo (ensayo, medido con la implementación vigente G5d; duraciones registradas en `gas_v2/estado.json` → `pasadas`): **Base: DISPARADO** — la primera pasada completa tras G5d duró 20,4 min (fin 2026-10-08 23:06:58 UTC; las dos siguientes, 18,8 y 17,9 min); por la regla, una vez disparado queda activo. **Ethereum: no disparado** (pasadas de 16,0, 2,7 y 2,0 min). **Arbitrum: no disparado** (≤ 0,4 min). En Base, el muestreo afecta a los grupos de más de 500 direcciones: Beefy (534) y Morpho v2 (2.289); los de Yearn (7) se recorren enteros. En el D0 se vuelve a registrar y se congela el estado por cadena.**

### 5.6 Observación del terreno: actividad autodirigida frente a automatización de terceros (v1.20, Rod)

**Qué se reporta (datos).** Para cada mercado del universo y cada acción de la tabla del §5.2, sobre **todas** las transacciones observadas que producen el evento característico (R1, sin los filtros R6 ni R1-B), dentro de la ventana de 30 días:
- `n` total observado;
- **autodirigidas**: el beneficiario del evento es quien firma la transacción (R1-B), separadas en directas al contrato y vía router;
- **de terceros**: el beneficiario es otro (estrategias de otros protocolos, keepers, zaps, routers que actúan sobre posiciones ajenas), separadas igual;
- sin lectura del beneficiario.
Se reporta para las transacciones del propio contrato y, aparte, para las de sus hermanos verificados (C8 ii). Se lista además qué acciones dependen de hermanos porque su vía propia autodirigida no llega a 10 transacciones en la ventana. Siempre con `n` absolutos, no solo porcentajes.

**Clasificación (v1.21):** autodirigida según la R1-B' (la posición acaba en el firmante dentro de la misma transacción), desglosada en directa del firmante, vía router con el firmante como beneficiario inmediato, y reenviada al firmante por un router. Con la R1-B' la fracción deja de ser cota inferior **por el motivo del destinatario inmediato**, pero sigue siéndolo por los límites del historial (un salto, *smart wallets*), y puede sobrecontar *bounties* en retiradas y cobros.

**Limitación declarada.** Las fracciones salen de la ventana y de los nodos públicos disponibles (en Base y Ethereum, la acumulación propia desde el ensayo; en Arbitrum, 30 días), de los hermanos que proponen las listas y verifica la cadena (§5.4), y —donde se dispara— del muestreo del §5.5. Son **indicativas, no un censo**.

**Interpretación (separada de los datos; corregida en la v1.22, ver el historial).** La observación depende del protocolo: hay mercados mayoritariamente autodirigidos y mercados donde el camino directo está realmente desierto. Allí donde lo está, la comparación "delegar frente a gestionar directo" deja de ser una elección real para la mayoría de los participantes y pasa a ser una hipótesis: cabe que el mercado ya haya respondido la pregunta por su cuenta, y la haya respondido delegando. Es material de primer orden para el paper **tanto si el cost model acaba diciendo DELEGAR como si no**.

**Qué NO se puede concluir de esta observación.** (i) No es un resultado del experimento ni sustituye a la puntuación del §11: no dice si delegar compensa. (ii) Una proporción alta de terceros no prueba que delegar sea mejor: la gente puede delegar por comodidad, por inercia o por incentivos ajenos al coste. (iii) No mide personas ni capital: cuenta transacciones, y un solo keeper o una sola estrategia pueden generar muchas. (iv) No dice nada de los mercados fuera del universo ni del periodo fuera de la ventana. (v) Una transacción "autodirigida" puede ser de un agente automático que gestiona su propio capital; la categoría distingue de quién es la posición, no si hay una persona detrás.

## 6. Slippage y MEV (sin protección)

- **Slippage:** cotización on-chain en cada foto (`get_dy` del pool de Curve o el *quoter* del DEX, vía `eth_call`) comparada con el precio medio de DefiLlama. Se toma la mediana del día.
- **MEV:** recorte sobre el nocional de cada swap. Pre-registrado y dentro de la banda:
  - Base y Arbitrum: {0, 10, 30} pb;
  - mainnet (P-C): {10, 30, 60} pb.
- **Lado delegado:** el coste de los swaps del harvest ya está incluido en el precio por share. No se le suma nada.

## 7. Valoración de rewards

- **Rewards acumulados:** tasa de emisión del gauge, leída on-chain en cada foto, multiplicada por la fracción del gauge que tendría la sombra. Se valoran a la mediana diaria del precio de DefiLlama.
- **Lado delegado:** retorno realizado = cambio del precio por share (`pricePerShare`, `getPricePerFullShare` o `convertToAssets`) entre la mediana diaria del día de entrada y la del día de salida, más los fees y el gas de entrada y salida (§8). El mismo método de mediana se aplica a los dos lados.

### 7.1 APY de cada foto: medido on-chain; DefiLlama solo como contraste (v1.11, Rod)

**Regla.** El APY de cada foto se **mide on-chain** en el primer bloque de la ventana. DefiLlama **no es fuente**: es contraste.

**Razón (Rod):** DefiLlama publica **un punto diario** y no puede alimentar cuatro fotos al día. Y una medición propia, reproducible por cualquiera desde la cadena, es **más defendible** que un dato de tercero.

**Cómo se mide, por tipo de mercado** (todas son lecturas `eth_call`):

| Tipo | Lectura on-chain | APY de la foto |
|---|---|---|
| Aave v3 (control, réplica de P-A) | `Pool.getReserveData(activo).currentLiquidityRate` (APR en ray) | `(1 + APR/S)^S − 1`, con S = segundos del año (capitalización por segundo de Aave) |
| Vaults con precio por share (Yearn v3: `convertToAssets`; Beefy: `getPricePerFullShare`) | precio por share en la foto | crecimiento anualizado del precio por share entre esta foto y la **foto del mismo slot con al menos `V` días de antigüedad**, donde `V` es la ventana del mercado (v1.15, abajo): `(PPS_t / PPS_t−V)^(365/V) − 1`, con `V` medido con las marcas de tiempo de los bloques. Si no hay ninguna foto tan antigua (propia o reconstruida de un nodo de archivo), `sin_dato_apy`: **nunca se acorta la ventana** |
| Pool de Curve (P-B directo, P-C) | `get_virtual_price()` (comisiones de swap) + emisión del gauge (`inflation_rate` × peso relativo / `working_supply`, y `reward_data` de cada token extra) valorada a precio DefiLlama | base: crecimiento anualizado del `virtual_price` con la misma ventana de 7 días; rewards: APR de emisión de la foto |
| Convex (P-C delegado) | lo anterior sobre el mismo pool, menos las comisiones de Convex leídas on-chain (§8) | igual que Curve, neto de comisiones |
| PT de Pendle (P-D) | `Market.readState().lastLnImpliedRate` | `exp(lnImpliedRate) − 1` (tipo fijo implícito a vencimiento) |

Las fotos de la Fase A (ensayo) **sí pueden** servir de referencia para el arranque, porque son lecturas on-chain reproducibles; también las fotos **reconstruidas** de un nodo de archivo (hoy solo Base), marcadas como tales. Pero **nunca** cuentan como predicción.

**Ventana del APY por mercado: al menos un ciclo de cosecha completo (v1.15, Rod).**
- **Regla:** `V = min(30, max(7, ⌈ciclo / 24 h⌉))` días.
- **Razón (de medición, no depende del número):** un vault que cosecha a intervalos no se puede medir en una ventana más corta que su propio ciclo: si la ventana no contiene ninguna cosecha, el crecimiento medido es 0; si contiene una, es el de todo un ciclo comprimido en la ventana. **La duda la levantó Beefy en el ensayo (2026-10-08: precio por share plano un día entero y luego un salto), pero la regla se justifica sola** y se aplica a todos los mercados por precio por share.
- **Cómo se determina el ciclo, con datos observables:** se lee el precio por share (la misma lectura de la foto) a intervalos regulares durante los 30 días previos a la fecha de referencia. Una **cosecha** es cualquier cambio del valor bruto entre dos lecturas consecutivas (sin umbral: entre cosechas, el de un vault que cosecha a saltos no cambia nada). El **ciclo** es el mayor intervalo entre cambios consecutivos, contando los dos bordes (inicio de la serie → primer cambio; último cambio → fin). Sin ningún cambio en 30 días, `V = 30`.
- **Fuente y resolución:** Base, nodo de archivo, una lectura por hora (resolución 1 h); Arbitrum y Ethereum, sin archivo público, las fotos propias (resolución ≈ 6 h, y lo disponible si hay menos de 30 días). Un mercado que acumula de forma continua cambia en cada lectura: su ciclo es la resolución y su ventana, 7 días.
- **Se mide el D0 y se congela** con el resto de constantes; durante el mes no se recalcula.
- **Fuera:** Aerodrome (su "precio" es el del LP en USD, no un precio por share de vault; su base se toma 0, ver la predicción).

**Contraste con DefiLlama.** Cada día, para cada mercado con pool en DefiLlama, se compara la mediana diaria del APY on-chain con el punto diario de DefiLlama. **Se compara lo mismo con lo mismo (v1.15):** si nuestro APY incluye las recompensas reinvertidas (Beefy, Yearn, Morpho), contra el `apy` total; si es solo la base del pool (Curve y Convex: crecimiento del `virtual_price`), contra `apyBase`; y **Convex, contra su propio pool de DefiLlama**, no contra el de Curve. Hasta la v1.14 Convex se comparaba (base) contra el total del pool de Curve: un error del contraste, no de la lectura (la base coincidía: 0,82 % frente a 0,82 %).
- **Margen:** hay **discrepancia** si `|APY_onchain − APY_llama| > max(1,0 pp ; 25 % del APY_llama)`.
  - El 1,0 pp cubre la diferencia de ventanas: DefiLlama usa el cambio de 24 horas del precio por share, que salta con cada *harvest*, y nosotros 7 días.
  - El 25 % relativo cubre los APY altos (Pendle, rewards).
  - El margen se fija **ahora**, antes de ver ningún dato del ensayo, y **solo dispara una alerta: no cambia ningún resultado**.
- **Qué se hace con una discrepancia:**
  1. el valor on-chain **sigue siendo el que cuenta**: nunca se sustituye por el de DefiLlama, ni ese día ni después;
  2. se registra `apy_discrepancia` con los dos valores en el log del día;
  3. si en un mismo mercado se repite **3 días seguidos**, se revisa la lectura. Si es un **bug de implementación**, se corrige como errata (§0.3: se publica, se registra y se reporta con las dos versiones). Si la lectura es correcta, se deja constancia y sigue igual;
  4. la síntesis semanal enseña, por mercado, el número de días con discrepancia.

### 7.2 Incentivos de Merkl (G1, v1.13, Rod)

**Regla.** Además de las emisiones de gauge (§7), cuentan los incentivos que reparte **Merkl** a quien deposita en un contrato del par, **en los dos lados** (delegado y sombra), con la misma regla.
- **Fuente:** la API de Merkl (`/v4/opportunities`), leída en cada foto, igual que los precios de DefiLlama. Es un dato de tercero: la distribución se calcula fuera de la cadena y solo la raíz Merkle es on-chain.
- **Qué cuenta:** solo las oportunidades cuyo identificador es **exactamente** la dirección del contrato del par y cuyo estado es **LIVE**. Las terminadas (PAST) no cuentan: un gestor que entra hoy no las cobra.
- **Coste:** la sombra (y el delegado, si los recibe) paga el gas de reclamar (`claim` del distribuidor de Merkl), el swap de la recompensa al activo de partida y el nuevo depósito, con la regla k del §4.
- **Hueco declarado:** si la API no responde o no se puede leer de forma fiable, ese incentivo **cuenta 0 y se anota** en la predicción. No se estima.
- **Limitación declarada:** el slippage del swap de la recompensa (p. ej. WELL → USDC) no se mide; solo entra la rejilla de MEV.

**Razón (Rod):** un gestor directo también podría cobrar estos incentivos. No contarlos le resta rendimiento al lado directo y sesga a favor de delegar.

## 8. Coste del lado delegado (`c_ij`)

- **Fees de depósito y retirada:** leídos on-chain.
- **Performance y management fee:** leídos del contrato de cada vault (el *accountant* en Yearn v3, el *fee config* en Beefy, el parámetro en Convex, el fee de YT y de swap en Pendle). **No se asume ningún valor.** Cada card cita su fuente on-chain.
- **Gas de entrada y salida:** mismo método que el §5.

## 9. Fotos diarias y sin mirar hacia delante (punto 6; detalles aprobados en v1.3)

**Cuatro fotos al día.** Las horas no las elige nadie: son la **partición del día UTC en cuatro intervalos iguales anclada al comienzo del día**, es decir, **00:00, 06:00, 12:00 y 18:00 UTC**.
- Cada foto cubre la ventana **[hh:00, hh:20) UTC**. Así queda siempre dentro del mismo día UTC.
- Cada foto captura todos los datos: APY (medido on-chain, §7.1), precios, gas, coste L1, precio por share, cotizaciones y tasas de gauges.

**Valor del día:** la mediana de las fotos válidas. Con 4 fotos, la media de los dos valores centrales; con 3, el valor central.

**Cierre del día de observación: 18:50 UTC** (H1, v1.25; antes, 18:20). Queda después de la ventana del reintento de las 18:00 (`[18:30, 18:50)`), que así puede contar. Después del cierre, y antes de las 00:00 UTC del día siguiente:
- se calculan las decisiones de la sombra y la predicción;
- se registran en el `paper_log`, junto con el sha256 de los datos brutos de las fotos del día;
- se ancla el hash (D5).

**Sin mirar hacia delante:** cada decisión usa solo fotos con timestamp igual o anterior al cierre de su día. La única excepción declarada es la cota `k*` (§4). Una foto (o un reintento) terminada después del cierre queda registrada, pero **no cuenta** para el valor ni para la validez de ese día (desde la v1.15 también en los reportes del ensayo).

**Marca de tiempo de cada lectura (v1.15):** la del **bloque** del que sale (el primero de la ventana de la foto principal o, si la lectura viene del reintento, el del reintento), no la del slot. Es lo medido.

## 10. Huecos de datos (aprobado en v1.3)

- Si falla una foto, se reintenta **una vez a los +30 min**, dentro de su mismo intervalo de 6 horas.
- Un día es válido para un par si tiene **al menos 3 de las 4 fotos** válidas. Si no, el día queda como `sin_dato` para ese par:
  - no se interpola;
  - la predicción de ese par y ese día se anula (se registra como anulada, no se borra).

## 11. Resultado, banda de indiferencia y puntuación

**Rejilla de incertidumbre reconocida (G) = el producto de TODAS las sensibilidades pre-registradas del §12.**

| Puntuación | Dimensiones de G | Puntos (sin tiempo humano) | Puntos (con tiempo humano) |
|---|---|---|---|
| **(i) Solo costes** | `k` (3) × MEV (3) × percentil de gas (3) | 27 | 54 |
| **(ii) Decisión completa** | lo anterior × LGD (3) × regla del A.2 [definición (2) × umbral (3) × intervalo (2)] × H_pos (3) × clase y muerte del A.2.7 (4: A estricta, A amplia, B estricta, B relativa; v1.7) | 11.664 | 23.328 |

La columna "con tiempo humano" multiplica por los dos extremos del §13.

**Banda de indiferencia derivada (sin ningún umbral en dólares).** Para cada par, nocional, horizonte, forma (con o sin tiempo humano) y día:
- `ventaja(g) = neto_delegado(g) − neto_sombra(g)` para cada punto g de G;
- **DELEGAR** si `ventaja(g) > 0` en **todos** los puntos; **DIRECTO** si es `< 0` en todos; **EMPATE** en cualquier otro caso.

El modelo solo declara un ganador cuando sobrevive a **toda** la incertidumbre que el propio modelo reconoce. Habrá más empates, y son empates reales.

La misma banda se aplica a la **predicción** (valores ex-ante) y al **resultado** (valores realizados). La puntuación compara clases {DELEGAR, DIRECTO, EMPATE}. Se reportan:
- la matriz de confusión 3×3;
- el acierto;
- la fracción de días en EMPATE;
- el error de magnitud en los días no empatados, con el punto central de G.

**Qué significa cada puntuación:**
- **(i) Solo costes:** `c_ij` frente a `c_solo`, sin el término de riesgo.
- **(ii) Decisión completa:** incluye `risk(T_ij)·L`. Sin un incidente durante el mes, el término de riesgo nunca "se realiza". Por eso (ii) se reporta como *decisión bajo riesgo*, no como error de predicción.

**Cota del gestor directo (punto 9).** Cada resultado se contrasta también con `c_solo(k*)` y se clasifica en una de tres categorías:
- "delegar gana incluso contra el mejor compounding posible a posteriori";
- "delegar gana contra `k` fija, pero no contra `k*`" (se dice explícitamente);
- "directo gana".

**Reportes separados:**
- cada horizonte (7 y 30 días);
- cada forma (con y sin tiempo humano);
- P-C (mainnet);
- P-D (Pendle).

## 12. Sensibilidades pre-registradas (todas entran en la banda, salvo `k*`)

El valor en **negrita** es solo el **valor central de visualización** que se usa para mostrar magnitudes. **No decide nada.**

| Parámetro | Valores | Dónde entra |
|---|---|---|
| `k` de compounding | 2 · **5** · 10 | Banda (i) y (ii) |
| MEV L2 (pb) | 0 · **10** · 30 | Banda (i) y (ii) |
| MEV mainnet (pb), solo P-C | 10 · **30** · 60 | Banda (i) y (ii) |
| Percentil del precio de gas | p25 · **p50** · p75 | Banda (i) y (ii) |
| LGD | 0,3 · **0,6** · 1,0 | Banda (ii) |
| Regla A.2: definición | **amplia** · estricta | Banda (ii) |
| Regla A.2: umbral de materialidad | **US$0** · US$1M · US$10M | Banda (ii) |
| Regla A.2: intervalo | **95 %** · 80 % | Banda (ii) |
| A.2.7: clase y definición de muerte | **A · relativa** · A · estricta · B · estricta · B · relativa | Banda (ii): las 4 combinaciones (v1.8; la "A · amplia absoluta" queda sustituida por la "A · relativa"). Desde la v1.9 solo cuentan las muertes **sin salida** (A.2.7, punto 11) |
| A.2.8: fracción `f` del TVL alcanzada | medida, o [pérdida / TVL, 1] si no consta | Dentro de `λ_min`/`λ_max` de cada variante (banda ii), por la propagación del A.2.8, punto 3 |
| Media vida positiva de `T_ij` | 365 · **730** · 1460 d | Banda (ii) |
| Tiempo humano por acción (forma "con") | US$2,08 · US$25 | Banda (i) y (ii) de la forma "con"; no hay valor central y se muestran los dos extremos |
| Cota `k*` a posteriori | rejilla K* | **Fuera de la banda:** es una cota, no una incertidumbre (§4) |

## 13. Tiempo humano y lo que NO se modela

**Tiempo humano (puntos 13 y 4).** El reporte de cierre enseña **las dos formas**, ambas pre-registradas:
- **Sin tiempo humano:** `c_solo` tal como se define en §1–§8.
- **Con tiempo humano:** a `c_solo` se suma un coste por cada acción manual de la sombra (entrada, compounding, rebalanceo, salida). **Desde la v1.26 (TH-SIM)**, el lado delegado paga también **su** entrada y **su** salida (y cualquier acción manual propia, como reclamar recompensas): entrar y salir son acciones manuales en los dos lados, y lo único que ahorra delegar es el compounding y el rebalanceo. **No se elige un valor.** Se pre-registran los dos extremos del rango considerado:
  - 5 min × US$25/h ≈ **US$2,08 por acción**;
  - 15 min × US$100/h = **US$25 por acción**.

  Es un rango de 12 veces, y **los dos extremos entran en la banda** de esa forma.
- **G2 (v1.13):** en esta forma, la regla de compounding del §4 compara las rewards con el coste **incluido el tiempo humano** de la acción.

**Qué NO modela:**
- impuestos, rampas de entrada y salida y bridges;
- transacciones fallidas;
- MEV fuera de la rejilla del §6;
- el riesgo de la capa delegada (va en el anexo A);
- el efecto de la propia posición sobre el pool (acotado por la regla del 5 %).

## 14. Agentes observados: admisión (D1)

**Se arranca sin ningún agente.** Un agente entra en observación solo si cumple **todos** los criterios siguientes, verificados y documentados. Este criterio se congela con el documento. **C1–C4 aprobados por Rod (C3 y C4 en v1.3).**

| # | Criterio | Cómo se verifica |
|---|---|---|
| C1 | **Publicación oficial:** el equipo del agente publica la dirección (EOA, smart account o vault) en un canal oficial suyo: web o documentación, GitHub de la organización o cuenta oficial | Captura archivada (archive.org) y sha256 de la página en la fecha de verificación |
| C2 | **Es la cuenta operativa del agente:** la publicación la describe como la dirección con la que el agente ejecuta, o el vault que el agente gestiona. No valen el contrato del token, una tesorería no operada por el agente ni direcciones atribuidas por terceros (Arkham, Nansen, listas de holders) | Texto de la publicación citado |
| C3 | **Actividad viva:** al menos 1 transacción exitosa iniciada por esa dirección, o una UserOperation suya, en los 30 días anteriores a la verificación | Blockscout (sin clave) |
| C4 | **Actividad en el alcance:** al menos una de esas transacciones interactúa con un contrato de lending, LP o vault en Base, Arbitrum (o mainnet, solo si es de Convex o Curve) | Blockscout y ABI verificada |

**Reglas de aplicación:**
- **Mamo** es el primer candidato a verificar en la Fase 0. No es un compromiso.
- Se puede admitir a un agente también **durante** el mes, siempre con este mismo criterio. Su observación empieza el día de la admisión; no se usan datos anteriores para predecir.
- **Reverificación semanal de C3.** Si un agente deja de cumplirla, se marca `inactivo` desde esa fecha y sigue en el reporte.
- **El criterio no se relaja** para tener a alguien a quien observar.
- **Si al final del mes ningún agente sobrevive a la verificación, eso es un resultado publicable por sí mismo**, igual que lo fue el cierre de RALL.

---

## Anexo A — Calibración del cost model (se congela junto con la sombra)

```
coop_cost = c_ij + risk(T_ij) · L        delegar  ⇔  coop_cost < c_solo
risk(T)   = λ_max − T · (λ_max − λ_min)   (tasa anual)
p_h       = 1 − exp(−risk(T) · h / 365)   (probabilidad sobre el horizonte h, en días)
```

- **Unidades:** USD sobre el horizonte.
- **`c_solo` ex-ante:** estimación de la sombra con los datos de los 7 días anteriores (mismo método, sin mirar hacia delante).

### A.1 `L` y LGD

`L` = nocional × LGD, con LGD ∈ {0,3 · 0,6 · 1,0}. Los tres entran en la banda (ii). Las recuperaciones históricas (Anexo B) informan este rango, pero no eligen un valor.

### A.2 `λ_min` / `λ_max`: regla de derivación

**1. Qué cuenta como incidente con pérdida de principal.** Un evento cuenta si cumple las cuatro condiciones:
- **(a) Rol:** sufren pérdida los usuarios que tienen el **mismo rol que nuestra posición** (depositante, LP o suministrador). Las pérdidas que sufren solo los borrowers no cuentan.
- **(b) Atribución contrafactual:** el evento se atribuye al protocolo X **solo si** un usuario que tuviera el mismo subyacente *sin* X no habría sufrido la pérdida. La mala deuda que un lending protocol socializa a sus propios suministradores sí es del lending protocol, aunque la cause un colateral de terceros.
- **(c) Contratos del protocolo:** los contratos afectados están listados bajo X (o su *parent*) en la taxonomía de DefiLlama. Quedan fuera los proyectos distintos construidos encima, los ataques solo al frontend o al DNS y los contratos periféricos que no custodian principal de depositantes.
- **(d) Materialidad:** la pérdida de los usuarios en el rol (a), **en el momento del evento**, antes de cualquier reembolso, es > **umbral**. Los reembolsos posteriores no cambian el conteo: informan el LGD.

**Unidad de conteo:** un incidente es una misma causa raíz dentro de 7 días.

**2. Fuentes y procedimiento de censo.** El censo se hace sobre cuatro fuentes:
- el leaderboard de rekt.news;
- defillama.com/hacks (web);
- SlowMist Hacked;
- los post-mortems e informes de incidentes de cada protocolo.

Se busca por el nombre de cada protocolo y de sus productos. Un evento se incluye si lo confirma el post-mortem del protocolo o **dos** fuentes independientes. Cada decisión queda en el Anexo B con su motivo.

**3. Exposición (años-protocolo).**
- Desde la **primera fecha con TVL > 0** en la API gratuita de DefiLlama, tomada como el mínimo entre todos los slugs del *parent*.
- Hasta la **fecha del congelado**.
- Se suma sobre los seis protocolos.
- **Limitación declarada:** DefiLlama empieza a seguir algunos protocolos después de su lanzamiento real (Aave, Beefy), así que la tasa sale algo sobrestimada.

**4. Tasa y mapeo.**
- `λ̂ = N / E`.
- **`λ_min` y `λ_max` = cotas inferior y superior del intervalo de Poisson exacto (Garwood).**
- Con T = 0 se toma el extremo pesimista de lo que permiten los datos de la clase; con T → 1, el optimista.

**5. Variante central y sensibilidades.**
- **Central: definición amplia · umbral US$0 · intervalo 95 %.**
- Variantes: estricta; US$1M; US$10M; 80 %. **Todas entran en la banda (ii).**

**Por qué el umbral central es cero (Rod, 2026-09-22).** El umbral de US$1M se propuso *después* de ver el censo, y hay un evento que queda fuera solo por él: la vault legacy de Yearn del 16-dic-2025. Por eso el central tiene que ser el que **no requiere ninguna decisión humana** y, por tanto, no puede acusarse de estar ajustado a los datos. Se pierde algo de limpieza en el censo, porque entran incidentes pequeños que pueden estar peor documentados. A cambio, el número central es inatacable.

**6. Resultado provisional** (censo al 2026-09-22; se **recalcula con esta misma regla el día del congelado**, y ese número es el que se congela):
- **Central (amplia · US$0 · 95 %):** **N = 7**, **E = 35,51** años-protocolo, **λ̂ = 19,71 %/año**.
- **`λ_min` = 7,93 %/año · `λ_max` = 40,62 %/año.**
- Por horizonte:
  - 7 días: `risk_min` = 0,152 %, `risk_max` = 0,776 %;
  - 30 días: `risk_min` = 0,649 %, `risk_max` = 3,283 %.

| Variante | N | λ̂ | IC 95 % | IC 80 % |
|---|---|---|---|---|
| **amplia · US$0 (central)** | **7** | **19,71 %** | **[7,93 %, 40,62 %]** | [10,97 %, 33,15 %] |
| amplia · US$1M | 6 | 16,90 % | [6,20 %, 36,78 %] | [8,88 %, 29,66 %] |
| amplia · US$10M | 4 | 11,26 % | [3,07 %, 28,84 %] | [4,91 %, 22,51 %] |
| estricta · US$0 | 5 | 14,08 % | [4,57 %, 32,86 %] | [6,85 %, 26,12 %] |
| estricta · US$1M | 4 | 11,26 % | [3,07 %, 28,84 %] | [4,91 %, 22,51 %] |
| estricta · US$10M | 3 | 8,45 % | [1,74 %, 24,69 %] | [3,10 %, 18,81 %] |

**Advertencia:** 4 de los 7 incidentes del central son de Yearn. La tasa agregada trata los seis protocolos como una sola clase; es `T_ij` (A.3) lo que los diferencia dentro de ella.

### A.2.7 Sesgo de supervivencia: tasa de muerte de la clase comparable (C3, v1.6)

**Problema.** La tasa de incidentes del A.2 sale de seis protocolos elegidos *porque* siguen vivos y son grandes. Un protocolo que muere, se abandona o cierra también es una forma de perder la posición, y eso no se contaba en ningún sitio. Así, `λ` salía subestimado de forma sistemática y delegar salía favorecido.

**Estructura de la corrección.** La tasa base pasa a ser la **suma de dos riesgos que no se solapan**:
- **`λ_inc`:** incidentes con pérdida de principal (A.2.1–A.2.6), estimados sobre los seis. Es la tasa condicionada a que el protocolo siga vivo.
- **`λ_muerte`:** muertes, estimadas sobre una **clase comparable que incluye a los muertos**.

```
λ̂ = λ̂_inc + λ̂_muerte
λ_min = LB95(λ_inc) + LB95(λ_muerte)        λ_max = UB95(λ_inc) + UB95(λ_muerte)
```

Sumar las cotas de los dos intervalos da un intervalo **más ancho** que el exacto de la suma. Es conservador, y queda declarado.

**1. Qué es un protocolo comparable.** Se exigen las tres condiciones, todas con datos públicos de DefiLlama:
- **(a) Misma categoría funcional:** la categoría del protocolo en DefiLlama es una de las cuatro que tienen los seis (*Lending*, *Dexs*, *Yield*, *Yield Aggregator*).
- **(b) Mismo alcance:** está desplegado en al menos una de las cadenas del experimento (Ethereum, Arbitrum o Base).
- **(c) Escala comparable:** su pico de TVL en la ventana es **≥ el menor pico de los seis** (Beefy, US$1.299,8M). El umbral **no se elige**: sale de los propios seis.
- **Unidad:** el protocolo *parent* de DefiLlama, con todas sus versiones sumadas. Así, una migración de v1 a v2 no cuenta como muerte.
- **Ventana:** desde la primera fecha con TVL de los seis (2020-02-09) hasta el congelado.

**2. Qué es muerte, y qué no.**
- **Muerte estricta:** DefiLlama marca como muertas (`deadFrom`) **todas** las piezas del protocolo, o lo marca como `rugged`.
- **Muerte amplia** (incluye la estricta): además, **abandono por TVL**. El TVL del protocolo cae por debajo del **suelo de elegibilidad del §2 (US$250k)** y **no vuelve a superarlo nunca** hasta el final de la serie, durante al menos **un horizonte mensual (30 días)**.
  - Fecha de la muerte: el primer día de ese tramo final.
  - Las dos cifras son constantes que ya existían en la metodología; no se introduce ninguna nueva.
- **No es muerte:**
  - una caída de TVL que **se recupera** por encima de US$250k;
  - una caída que sigue por encima de US$250k, por grande que sea;
  - la migración a otra versión del mismo *parent*;
  - un cambio de nombre (se comprueba con `previousNames`).
- **Variante central: la amplia.** Las marcas de DefiLlama solo cubren una parte de los abandonos: en la clase A, **0** muertes estrictas frente a **3** amplias. La estricta dejaría la corrección a medias. Las dos entran en la banda (ii).
- **Muerte relativa (clase B, v1.7, decisión de Rod):** la estricta, más una **caída sostenida respecto al propio pico**. El TVL del protocolo cae a **≤ 2,03 % de su propio pico** en la ventana y **no vuelve a superar ese nivel** hasta el final de la serie durante **≥ 365 días**. Fecha de la muerte: el primer día de ese tramo final.
  - **Por qué relativa y no absoluta (Rod):** el umbral absoluto de US$250k convierte en "muerte" el ruido normal de los microprotocolos que rondan esa cifra. En cambio, una caída relativa al propio pico mide lo mismo para un protocolo grande que para uno pequeño. Así no se descarta un dato incómodo ni se acepta ruido, y la regla no depende de un juicio hecho después de ver el número.
  - **Cómo se eligió el 2,03 %, sin mirar el resultado de la clase:**
    - La muerte tiene que distinguirse de un mercado bajista.
    - **Medición de referencia:** la peor caída del **TVL total de DeFi** en la ventana (DefiLlama, `/v2/historicalChainTvl`) llevó el mercado entero al **20,3 % de su pico** (−79,7 %, del 2021-11-09 al 2023-10-13).
    - El umbral de muerte es **un orden de magnitud por debajo** de ese valle: 20,3 % / 10 = **2,03 %**. No se redondeó ni se ajustó.
    - Un protocolo que simplemente acompaña al mercado bajista **no** muere con esta regla; uno que cae diez veces más hondo que el mercado entero y no se recupera, sí.
  - **Cómo se eligieron los 365 días:**
    - Es un ciclo anual completo, más largo que las temporadas de incentivos o de puntos (normalmente trimestrales), cuyos flujos de salida no son muertes.
    - Son más de 4 medias vidas de la evidencia negativa (90 días): pasado ese tiempo, el propio modelo ya considera casi olvidado un fallo transitorio.
  - **Transparencia:** los dos parámetros se fijaron con estos criterios **antes** de calcular la clase B con la regla nueva. Sí se conocía el resultado de la regla absoluta (26,28 %), que es lo que motivó cambiar de definición. **No** se conocía el de la relativa.
  - **Censura declarada:** con 365 días de duración mínima, las muertes posteriores al 2025-09-22 todavía no son detectables. Eso hace que la tasa quede algo subestimada en el tramo final de la ventana.

**3. Fuentes:** la API gratuita de DefiLlama (`/protocols` para categorías, cadenas, marcas y *parent*; `/protocol/{slug}` para la serie de TVL). Los datos brutos se guardan con su sha256.

**4. Años-protocolo (denominador):** para cada protocolo de la clase, desde `max(primera fecha con TVL, 2020-02-09)` hasta `min(fecha de muerte, congelado)`.

**5. Numerador:** el número de muertes con fecha dentro de la ventana. Los seis están en la clase (suman exposición) y no han muerto.
- **Sin doble conteo:** los incidentes del A.2 son solo de los seis, y ninguno de los seis ha muerto.
- Una muerte provocada por un hack cuenta **una vez**, como muerte.

**6. Casos citados por Rod, evaluados con el criterio:**

| Caso | ¿Comparable? | Motivo |
|---|---|---|
| **ARMA (Giza)** | **No entra en la tasa** | (i) DefiLlama clasifica a Giza como *AI Agents*, fuera de las cuatro categorías. (ii) **Giza no murió**: lo que cerró (marzo de 2026) fue un producto, y los fondos se devolvieron a los usuarios, sin pérdida de principal. Es el cierre de un **mercado**, no la muerte de un protocolo. En el modelo de la Fase 2, un cierre así es una **salida forzada**: su coste es el gas de salida más la pérdida de rendimiento, no `L`. |
| **Eliza / ai16z** | **No entra en la tasa** | No es un protocolo de depósito: DefiLlama no lo lista como protocolo DeFi. Sus usuarios tenían **tokens**, no posiciones depositadas en un contrato. Su colapso (agosto de 2026) es riesgo de precio de un token, no la pérdida de una posición de yield. Queda fuera por categoría funcional (1a). |

**7. Resultado provisional** (al 2026-09-22; se recalcula con esta misma regla el día del congelado):

| Clase y definición | Protocolos | Muertes | Años-protocolo | `λ_muerte` | IC 95 % |
|---|---|---|---|---|---|
| **A · relativa (CENTRAL desde v1.8)** | 30 | **6** | 136,3 | **4,40 %** | [1,62 %, 9,58 %] |
| A · estricta | 30 | 0 | 154,7 | 0,00 % | [0,00 %, 2,38 %] |
| B · estricta (en la banda) | 662 | 42 | 2.232,1 | 1,88 % | [1,36 %, 2,54 %] |
| B · relativa (en la banda) | 662 | 258 | 1.607,5 | 16,05 % | [14,15 %, 18,13 %] |
| ~~A · amplia absoluta (US$250k)~~ (central en v1.6–v1.7; sustituida en v1.8) | 30 | 3 | 153,1 | 1,96 % | [0,40 %, 5,73 %] |
| ~~B · amplia absoluta (US$250k)~~ (sustituida en v1.7) | 662 | 390 | 1.484,0 | 26,28 % | [23,74 %, 29,02 %] |

**Clase B: de 26,28 % a 16,05 %.** La definición relativa filtra el ruido de los microprotocolos (390 → 258 muertes), pero **la tasa sigue siendo alta y se queda así**: esa es la gracia. Con la clase B, la tasa base total queda en **35,76 % [22,08 %, 58,75 %]**.

Muertes de la clase A con la definición **relativa** (central desde v1.8), sin ningún caso de migración ni de cambio de nombre. Las fechas son el inicio del tramo final ≤ 2,03 % del pico:

| Protocolo | Categoría | Pico de TVL | Muerte (≤ 2,03 % del pico, ≥ 365 d) |
|---|---|---|---|
| Royco (V1 + V2) | Yield | US$2.988,2M | 2025-05-09 |
| CREAM Finance | Dexs / Lending | US$2.310,8M | 2023-05-09 |
| Badger DAO | Yield Aggregator | US$2.321,4M | 2023-02-24 |
| ShibaSwap (V1 + V2) | Dexs | US$1.757,7M | 2024-03-16 |
| Iron Bank | Lending | US$1.497,9M | 2023-02-26 |
| Rari Capital | Yield Aggregator | US$1.302,9M | 2022-09-22 |

Con la regla absoluta de la v1.6–v1.7 solo se contaban CREAM, DeSyn e Iron Bank, y con fechas más tardías (cuando por fin bajaron de US$250k). DeSyn no cumple la relativa: su caída final es posterior al 2025-09-22, así que cae en la censura de 365 días.

**8. Antes y después de la tasa base** (A.2 central: amplia, US$0, IC 95 %):

| | `λ̂` | `λ_min` | `λ_max` | `risk_min` 30 d | `risk_max` 30 d |
|---|---|---|---|---|---|
| v1.5 (solo incidentes de los seis) | 19,71 % | 7,93 % | 40,62 % | 0,649 % | 3,283 % |
| v1.6–v1.7 (incidentes + muertes, clase A amplia absoluta) | 21,67 % | 8,33 % | 46,34 % | 0,682 % | 3,738 % |
| **v1.8 (incidentes + muertes, clase A relativa)** | **24,11 %** | **9,54 %** | **50,20 %** | **0,781 %** | **4,042 %** |
| Variante A estricta (en la banda) | 19,71 % | 7,93 % | 43,00 % | 0,649 % | 3,473 % |
| Variante B estricta (en la banda) | 21,59 % | 9,28 % | 43,16 % | 0,760 % | 3,485 % |
| Variante B relativa (en la banda) | 35,76 % | 22,08 % | 58,75 % | 1,798 % | 4,714 % |

**9. Clase B: RESUELTO (Rod, v1.7, opción iii).** La clase B **se mantiene en la banda (ii)** con dos definiciones de muerte: la estricta (marcas de DefiLlama) y la **relativa** del punto 2. La exclusión que se había propuesto en la v1.6 queda descartada: se había propuesto después de ver el número.

**9b. RESUELTO (Rod, v1.8): la regla relativa se aplica también a la clase A, con los mismos parámetros que la B (≤ 2,03 % del pico, ≥ 365 días), sin tocarlos.**
- **Razón (Rod):** los parámetros se fijaron para la B antes de ver ningún resultado y se reutilizan sin ajuste. Esto es coherencia, no un ajuste a posteriori. Además, el umbral absoluto de US$250k se queda corto con los protocolos grandes: es la misma asimetría pro-incumbente que ya se corrigió cuatro veces.
- La variante **central** pasa de "A · amplia absoluta" a **"A · relativa"**. La "A · amplia absoluta" sale de la banda; "A · estricta", "B · estricta" y "B · relativa" se quedan.

*Texto original del hallazgo (v1.7), conservado para la trazabilidad:*
- Con la definición relativa, la clase A pasa de **3 a 6 muertes** (1,96 % → **4,40 %**). Aparecen Royco (pico US$2.988M), Badger DAO (US$2.321M), ShibaSwap (US$1.758M) y Rari Capital (US$1.303M).
- **Causa:** el umbral absoluto de US$250k es irrelevante a escala grande. Un protocolo con un pico de US$1.300M puede quedarse en US$5M, el 0,4 % de su pico y muerto a efectos prácticos, sin bajar nunca de US$250k.
- **Es el mismo patrón pro-incumbente**, por quinta vez: una regla absoluta que no escala con el tamaño.
- El argumento de Rod para la clase B ("mide lo mismo para un grande que para un pequeño") vale igual para la A.
- **Propuesta (no aplicada):** usar la **misma definición relativa en la clase A**, en lugar de la amplia absoluta.
  - Tasa de muerte central: 4,40 % [1,62 %, 9,58 %].
  - Tasa base central: **24,11 % [9,54 %, 50,20 %]**, frente a 21,67 % [8,33 %, 46,34 %] en la v1.6.
  - La variante A estricta se queda en la banda.
- Esta propuesta **no se hizo después de elegir parámetros para la A**: son los mismos parámetros, fijados antes, para la B.

**10. Sesgo residual declarado (el residuo, no el problema entero):**
- **Tasa de incidentes de los no supervivientes:** `λ_inc` sigue saliendo solo de los seis, que están vivos. Los protocolos comparables que murieron probablemente tenían **más** incidentes antes de morir, y esos incidentes no están en `λ_inc`. Queda subestimado, aunque su muerte ya cuenta en `λ_muerte`.
- **Muertes que DefiLlama no ve:** protocolos comparables que DefiLlama nunca listó.
- **Cierres ordenados con devolución** (como ARMA): no son pérdida de principal, y su coste va a `c` y no a `L`. Si se contaran como muerte, sobrestimarían la pérdida (el LGD mínimo del modelo es 0,3).
- **Censura de la regla relativa** (v1.7): las muertes del último año de la ventana todavía no se pueden detectar.
- *(v1.7: la variante central de la A seguía con la regla absoluta. Resuelto en v1.8, punto 9b.)*
- **Censura en la clase A:** DeSyn, muerta con la regla absoluta (2025-11-28), todavía no cumple los 365 días de la relativa. Con más tiempo podría entrar.

**11. Muerte ≠ pérdida de principal: solo cuentan las muertes SIN oportunidad de salida (v1.9, Rod).**
- **Problema:** hasta la v1.8 se aplicaba un LGD de {0,3 · 0,6 · 1,0} a todas las muertes. Pero con la muerte relativa, al morir el protocolo solo retenía ≤ 2,03 % de su pico: los depositantes ya habían salido.
- **Matiz de Rod:** un rug o un cierre abrupto no da esa oportunidad y tiene que seguir costando caro. La pérdida depende de si los depositantes **pudieron salir**, y eso se **mide**, no se supone.
- **Criterio, sobre la curva observable del TVL** (serie diaria de DefiLlama), con dos umbrales que ya existían: el 20,3 % del pico (valle del mercado entero, punto 2) y el 2,03 % (umbral de muerte).
  - Se toma el **colapso final**: desde el último día con TVL ≥ 20,3 % del pico hasta el primer día del tramo ≤ 2,03 % que define la muerte.
  - **Gradual (sí hubo oportunidad de salida):** dentro de ese colapso hay **al menos una observación diaria intermedia**, con el TVL entre el 2,03 % y el 20,3 % del pico, y **ningún hueco de datos de más de un día**. Los depositantes vieron caer el TVL y tuvieron al menos un día completo para retirar. Por eso **no cuenta en `L`**. Su coste (salida forzada: gas, slippage, rendimiento perdido) es de tipo `c` y se declara no modelado más allá del gas de salida normal (Anexo E, E3).
  - **Brusca (no hubo oportunidad de salida):** el TVL pasa de ≥ 20,3 % a ≤ 2,03 % del pico **de un día al siguiente**, sin observación intermedia. **Cuenta en `L`** con el LGD de la banda {0,3 · 0,6 · 1,0}.
  - **Marcada `rugged` por DefiLlama:** cuenta como brusca, sea cual sea su curva.
  - **Indeterminada:** hay huecos de datos de más de un día en el colapso, o el protocolo nunca llegó a vaciarse antes de la fecha de muerte (por ejemplo, una muerte estricta con TVL todavía alto). **Se trata como brusca**: sin evidencia de que se pudiera salir, no hay beneficio de la duda (principio de FLP, §3.2).
- **Por qué la curva es la fuente principal y la etiqueta solo un complemento:** la curva es observable y se puede reproducir para todas las muertes. Pero con datos diarios **no distingue** un robo de una retirada masiva en un solo día; por eso las dos cosas se tratan igual (brusca). La etiqueta `rugged` añade los casos en que un tercero documentó un robo que la curva diaria podría no mostrar como un salto de un día.
- **Resultado:**

| Clase y definición | Muertes | Graduales (no cuentan en `L`) | Bruscas de 1 día | *Rugged* | Indeterminadas | **Cuentan en `L`** | Tasa v1.8 (todas) | **Tasa v1.9 (sin salida)** | IC 95 % v1.9 |
|---|---|---|---|---|---|---|---|---|---|
| **A · relativa (central)** | 6 | **6** | 0 | 0 | 0 | **0** | 4,40 % | **0,00 %** | [0,00 %, 2,71 %] |
| A · estricta | 0 | 0 | 0 | 0 | 0 | 0 | 0,00 % | 0,00 % | [0,00 %, 2,38 %] |
| B · estricta | 42 | 11 | 6 | 9 | 16 | 31 | 1,88 % | 1,39 % | [0,94 %, 1,97 %] |
| B · relativa | 258 | 182 | 32 | 10 | 34 | 76 | 16,05 % | 4,73 % | [3,72 %, 5,92 %] |

- **Las 6 muertes de la clase A** (Royco, CREAM, Badger, ShibaSwap, Iron Bank, Rari) fueron **graduales**: sus depositantes tuvieron días para salir. Eso no quiere decir que esos protocolos no hicieran perder dinero antes, por ejemplo CREAM en sus hackeos de 2021. Esas pérdidas son **incidentes**, no muertes, y el censo de incidentes solo cubre a los seis: ver el Anexo E, E1.

### A.2.8 Tasa de incidentes por dólar depositado (v1.9, Rod)

**Problema.** Hasta la v1.8, `λ_inc` era la probabilidad de que el protocolo sufriera un incidente **en cualquiera de sus mercados**, y se usaba como la probabilidad de que **nuestro** depósito perdiera principal (se multiplica por `L` = nuestro nocional × LGD).

**Razón (Rod), que lo justifica independientemente del número.** Es la **misma inconsistencia de alcance** que ya se corrigió en la v1.5 para `T_ij` (evidencia medida sobre nuestro mercado) y que se había quedado viva en la tasa base. La corrección se aplica porque **la unidad estaba mal**, no porque el resultado convenga. Se habría aplicado igual si hubiera subido el riesgo.

**1. Cómo se mide la fracción `f` del TVL alcanzada por cada incidente.**
- `f_i = max(TVL_A, P_A) / TVL_protocolo`, con `TVL_A` = el TVL de los mercados afectados (el conjunto `A` del A.3.1) y `TVL_protocolo` = el del *parent* en DefiLlama, ambos **el día anterior** al evento.
- `P_A` es la pérdida: el TVL alcanzado nunca puede ser menor que lo perdido.
- **Fuentes de `TVL_A`, por orden:**
  - (i) DefiLlama: `tvlUsd` del pool en la API de *yields* o el slug propio del producto;
  - (ii) el valor on-chain en el bloque anterior al evento (Blockscout o un nodo de archivo);
  - (iii) la cifra del post-mortem o del informe del incidente.
  - Se documenta cuál se usó.

**2. Si `TVL_A` no consta:** `f_i ∈ [P_A / TVL_protocolo, 1]`.
- La cota inferior es un hecho: al menos se alcanzó lo que se perdió.
- La cota superior asume, sin beneficio de la duda, que el incidente pudo alcanzar **todo** el protocolo.
- Las dos cotas entran en la banda.
- **Tarea de la Fase 0:** medir `TVL_A` on-chain (fuente ii) para los incidentes que hoy no lo tienen, lo que estrecharía la cota superior.

**3. Propagación de la incertidumbre:**

```
λ_inc,min = LB95(N) / E × media(f_inf)        λ_inc,max = UB95(N) / E × media(f_sup)
```

Se combinan los extremos de las dos incertidumbres: la del recuento (Poisson exacto) y la de la fracción alcanzada. Es conservador, y queda declarado. Las variantes del A.2.5 (definición, umbral, intervalo) se calculan igual, con los incidentes de cada una.

**4. Incidentes y fracción alcanzada (al 2026-09-22):**

| Incidente | Pérdida | `TVL_protocolo` (día anterior) | `TVL_A` | Fuente de `TVL_A` | `f` |
|---|---|---|---|---|---|
| yDAI v1 (Yearn, 2021-02-05) | US$11,0M | US$948,3M | US$35,0M | Post-mortem de Yearn: 35M DAI en la vault | **3,69 %** |
| Harmony (Aave, 2022-06) | US$1,2M | US$5.761,7M | ~US$3,0M | Documentado: ~3M depositados en el despliegue | **0,05 %** |
| yUSDT (Yearn, 2023-04-13) | US$11,4M | US$447,8M | **sin dato** | — | **[2,55 %, 100 %]** |
| Vyper (Curve, 2023-07-30) | US$69,3M | US$3.253,1M | **sin dato** (los pools atacados ya no están listados) | — | **[2,13 %, 100 %]** |
| yETH (Yearn, 2025-11-30) | US$9,0M | US$617,4M | US$8,82M (< pérdida → se usa la pérdida) | DefiLlama, slug `yearn-ether` | **1,46 %** |
| iearn (Yearn, 2025-12-16) | US$0,29M | US$583,0M | ~US$0,50M | rekt.news: 0,29M robados + 0,21M bloqueados | **0,09 %** |
| Kelp/rsETH (Aave, 2026-04-18) | US$196M | US$26.106,4M | US$997,8M | DefiLlama *yields*: las 3 reservas afectadas | **3,82 %** |

**5. Resultado:**
- `λ_inc` por dólar (central: amplia, US$0, IC 95 %): **0,39 %/año** con `f` en su cota inferior, e **IC [0,16 %, 12,13 %]**. Antes era **19,71 %/año por protocolo**.
- **La cota superior la dominan los dos incidentes sin `TVL_A`.** Si se midieran y resultaran cerca de su cota inferior, `λ_inc,max` bajaría a ~0,80 %.

**6. Tasa base total, antes y después** (incidentes por dólar + muertes sin salida, punto 11):

| | `λ_min` | `λ_max` | `risk_min` 30 d | `risk_max` 30 d |
|---|---|---|---|---|
| v1.8 (central: incidentes por protocolo + todas las muertes A) | 9,54 % | 50,20 % | 0,781 % | 4,042 % |
| **v1.9 (central: incidentes por dólar + muertes A sin salida)** | **0,16 %** | **14,84 %** | **0,013 %** | **1,212 %** |
| v1.9, peor variante de la banda | 3,77 % | 22,68 % | — | — |
| **v1.10 (central provisional: incidentes de la clase A por dólar + muertes A sin salida)** | **0,50 %** | **13,90 %** | **0,041 %** | **1,136 %** |

**7. Censo de incidentes de la clase A completa (E1, v1.10, Rod).**
- **Regla:** el censo de incidentes del A.2 deja de cubrir solo a los seis y cubre **todos los protocolos de la clase A** (30 comparables, A.2.7). Se usan **las mismas reglas**: rol, atribución contrafactual, contratos del protocolo, materialidad, unidad de conteo y las cuatro fuentes. La fracción alcanzada `f` se mide igual (puntos 1–3). La exposición `E` pasa a ser la de la clase A (136,3 años-protocolo, provisional).
- **Razón (Rod):** la clase de comparables tiene que ser **la misma para los dos componentes** de la tasa base. La supervivencia se corrigió en las muertes (A.2.7) y quedó sin corregir en los incidentes, y eso es incoherente.
- **Dirección declarada:** esta corrección **SUBE** la tasa (por dólar, de ~0,39 % a **~0,92 %/año**, indicativo). Va **en contra de delegar**, así que no se adopta por conveniencia.
- **Sigue siendo una cota inferior:** el censo indicativo sale solo del leaderboard de rekt.news, que solo recoge pérdidas grandes. El censo completo, con las cuatro fuentes, se hace en el **paso D0·2** del plan del congelado, y su número es el que se congela.
- **Censo indicativo** (al 2026-10-06), 7 incidentes que cuentan con las reglas del A.2:

| Incidente | Pérdida | `f_inf` (= pérdida / TVL del protocolo el día anterior) | `f_sup` |
|---|---|---|---|
| CREAM (2021-08-30) | US$18,8M | 1,13 % | 1 |
| CREAM (2021-10-27) | US$130M | 7,54 % | 1 |
| Balancer (2023-08-27) | US$2,1M | 0,31 % | 1 |
| Balancer (2025-11-03) | US$128M | 16,51 % | 1 |
| Euler (2023-03-13) | US$197M | 84,56 % | 1 |
| Venus (2022-05-13) | US$21,8M | 1,59 % | 1 |
| Venus (2026-03-15) | US$3,7M | 0,29 % | 1 |

  Excluidos con las reglas del A.2 (como ya figuraba en el Anexo E de la v1.9): Badger 2021 (frontend), Compound 2021 (error de reparto de COMP, sin pérdida de principal) y PancakeBunny (otro protocolo). Probable faltante: Rari/Fuse 2022. En el paso D0·2 se documentan, para cada incidente de la clase, la fuente, el `TVL_protocolo` y el motivo de inclusión o de exclusión.
- **Resultado provisional:** 14 incidentes en 136,3 años-protocolo. `λ_inc` por dólar: punto 0,92 %, IC **[0,50 %, 11,19 %]**. Las cotas superiores de `f` de los 7 externos son 1, porque su `TVL_A` no está medido.
- **Listón central provisional** (LGD 0,6):

| Posición | `λ(T)` v1.10 | **Listón central** | APY actual |
|---|---|---|---|
| Aave · Arbitrum USDC | 0,64 % | **0,39 pp** | 3,01 % |
| Curve · sUSDai-USDC | 1,49 % | **0,90 pp** | 4,48 % |
| Yearn · Base USDC | 1,49 % | **0,90 pp** | 4,88 % |
| Convex · DOLA-sUSDe | 0,99 % | **0,59 pp** | 6,06 % |
| Pendle · PT-USDai | 0,59 % | **0,35 pp** | 11,37 % |
| Beefy · Base USDC | 4,20 % | **2,52 pp** | 4,60 % |

### A.3 `T_ij` con dos medias vidas, sin tope

`T_ij = 1 − exp(−(pos − neg)/5)`, con `pos − neg ≤ 0 → 0`. Es la función `_evidence_to_trust` del repo de FLP, con `TRUST_SCALE` = 5.

**Evidencia:**
- **Positiva (alcance del MERCADO de la posición, v1.5; ver A.3.4):**
  - +1 por cada 30 días que **el mercado de nuestra posición** lleva operando, contados desde su creación on-chain, **sin tope** (Rod eliminó el de 36 meses);
  - +1 por cada auditoría independiente pública **que cubra la implementación de ese mercado** (misma versión de código), fechada en la fecha del informe. Ya no cuentan las auditorías de otras partes del protocolo.
- **Negativa (v1.4):** cada incidente del Anexo B atribuido al protocolo, con la variante vigente de la regla del A.2, **reduce proporcionalmente** la evidencia positiva acumulada antes de él. El factor es `(1 − min(1, s) · w_neg(edad))` (**reset proporcional**, A.3.2). `s` es la **severidad relativa a nuestra posición**, según el A.3.1. *Sustituye a la resta fija `+5 × min(1, s)` de la v1.3.*

**Decaimiento:**
- **Negativa: H_neg = 90 días** (aprobado).
- **Positiva: H_pos = 730 días** (**APROBADO**, 2026-09-22). Las variantes 365 y 1460 días entran en la banda (ii).
- **Por qué sin tope:** la exponencial satura sola, así que el tope no protegía de nada y solo impedía distinguir antigüedades.

**Efecto** (solo los meses limpios, sin auditorías ni evidencia negativa):

| Antigüedad | `T` (H_pos = 730 d, sin tope) | `T` con el tope de 36 meses (v1.1) |
|---|---|---|
| 6 meses | 0,673 | 0,673 |
| 24 meses | 0,971 | 0,971 |
| 36 meses | 0,990 | 0,990 |
| 72 meses | 0,998 | 0,990 |
| Aave (76 meses) | 0,998 | 0,990 |

**Riesgo a 30 días con el A.2 central:** Aave 0,654 % frente a un protocolo de 6 meses 1,517 %.

**Saturación por `TRUST_SCALE` = 5: es INTENCIONAL, no un descuido (Rod, 2026-09-22).** Aun sin el tope, un protocolo de 3 años y uno de 6 quedan cerca (0,990 frente a 0,998). Se deja así a propósito: llega un punto en que más años limpios no deberían mover la aguja. `TRUST_SCALE` es la constante de referencia de FLP (§5.6) y **no se toca**.

### A.3.1 Evidencia negativa: severidad relativa al mercado de nuestra posición (v1.3)

**Problema que corrige.** Hasta la v1.2, la severidad era `pérdida / TVL del protocolo entero`. Eso hacía casi inmunes a los protocolos grandes: los US$196M de Aave movían su `T_ij` solo −0,013. El modelo quedaba empujado hacia los gigantes **por construcción**, no por evidencia, y sesgaba justo la conclusión que el experimento quiere poner a prueba. El denominador pasa a ser el **mercado o pool donde estaría nuestra posición**, de forma coherente con el censo, que ya atribuye cada incidente por rol afectado.

**1. Qué es "el mercado" en cada protocolo** (la unidad donde se deposita el principal):

| Protocolo | Mercado | Identificador |
|---|---|---|
| Aave | Una **reserva** (un activo) dentro de una **instancia** del Pool en una chain. Ejemplos: Aave v3 Base · USDC; Aave v3 Ethereum Core · WETH | Dirección del Pool + dirección del activo |
| Curve | Un **pool** (contrato del pool en esa chain). En LlamaLend, el mercado de préstamo (controller + vault) | Dirección del pool o del controller |
| Convex | Un **pool del Booster** (`pid`): el reward pool que custodia el LP de un pool de Curve | Booster + `pid` |
| Yearn | Una **vault** (yToken v1/iearn, vault v2 o vault v3). Sus estrategias son internas a la vault | Dirección de la vault |
| Pendle | Un **market** (AMM de PT/SY con vencimiento) y sus PT/YT | Dirección del market |
| Beefy | Una **vault** (contrato mooToken) | Dirección de la vault |

**2. TVL de un mercado.**
- Es el `tvlUsd` de ese pool en la API gratuita de *yields* de DefiLlama (`/chart/{pool}`), tomado el **día anterior** al evento. La regla es uniforme para los seis protocolos y se puede reproducir.
- **Limitación declarada:** en los mercados de lending, el `tvlUsd` de DefiLlama es la **liquidez disponible** (suministrado − prestado), no el total suministrado. Eso aumenta `s`, es decir, es conservador.
- **Si no hay historial en DefiLlama** (incidentes anteriores a 2022, pools no listados): se usa el valor on-chain en el bloque anterior al evento o la cifra del post-mortem, y el Anexo B documenta cuál se usó.

**3. Cálculo de `s` para nuestra posición en el mercado `p`.**

Sea `A` el conjunto de mercados afectados por el incidente, con pérdida total `P_A` y TVL total `TVL_A` (sumas).

| Caso | Condición | Severidad `s` |
|---|---|---|
| **(a) Nos tocó directamente** | `p ∈ A` | `s = P_p / TVL_p` (la pérdida en nuestro mercado sobre su TVL) |
| **(b) Mecanismo compartido** | `p ∉ A`, pero el **componente que falló estaba presente en `p` el día del incidente** | `s = P_A / TVL_A` (la fracción que ese fallo le hizo perder a los mercados que alcanzó) |
| ~~(c) Otro mercado, sin mecanismo compartido~~ | `p ∉ A` y el componente no estaba en `p` | **No cuenta** (`s` = 0) desde la **v1.6 (C2)**. Antes era `P_A / TVL_protocolo`: una dilución por el tamaño del protocolo que hacía casi inmunes a los grandes (la cuarta capa de sesgo). Coherencia: los meses limpios de otros mercados tampoco cuentan (A.3.4). Se pierde a propósito la señal de "cultura del protocolo" |

**"Componente presente en `p`"** significa que se da **al menos una** de estas condiciones, evaluadas **en la fecha del incidente**:
- **(i)** el mismo contrato vulnerable: misma implementación, con igual hash de bytecode; o el mismo compilador y versión con el patrón vulnerable (p. ej. Vyper 0.2.15–0.3.0 con reentrancy en pools de ETH);
- **(ii)** el mismo contrato o adaptador de oráculo implicado;
- **(iii)** en lending, el mismo activo que falló habilitado como colateral (LTV > 0) en la instancia de `p`;
- **(iv)** la misma cuenta de gobernanza o admin cuyo abuso causó el fallo.

Se evalúa en la fecha del incidente, no hoy. La pregunta contrafactual es: *si hubiéramos estado en `p` ese día, ¿nos alcanzaba?* Que el protocolo arreglara el fallo después ya lo recoge el decaimiento de 90 días.

**4. Varios mercados afectados.** `A` es la unión de todos los mercados tocados por la misma causa raíz (la unidad de conteo del A.2). Si `p` está entre ellos, se usa su propia pérdida (caso a). Si no, pero comparte el mecanismo, se usa la severidad agregada `P_A / TVL_A` (caso b).

**5. Incidentes a nivel de protocolo.** Son los que el **componente que falla lo comparten todos los mercados**: la gobernanza, el timelock o una clave admin; un agregador de oráculo global que usan todas las instancias; o la implementación core común a todas.
- En ese caso `A` = **todos los mercados** y `TVL_A` = **TVL del protocolo**, así que `s = P_A / TVL_protocolo`, pero por el caso (b), porque el mecanismo es compartido por definición.
- **Criterio:** aquí el TVL total **sí** es el denominador correcto, porque el conjunto expuesto *es* el protocolo entero. El denominador sale del mecanismo, no del tamaño.
- Los incidentes de DNS o frontend ya quedan fuera del censo (A.2, 1c), así que no llegan aquí.
- **En el censo actual no hay ningún incidente a nivel de protocolo.**

**6. Antes y después** (posiciones candidatas; `T_ij` a 2026-09-22 con H_pos = 730 días sin tope y H_neg = 90 días; solo meses limpios, sin auditorías):

| Posición | Incidentes que la alcanzan | neg v1.2 | `T` v1.2 | neg v1.3 | `T` v1.3 |
|---|---|---|---|---|---|
| Aave v3 Arbitrum · USDC (rsETH era colateral en esa instancia) | Kelp por (b): `s` = 196 / 997,8 = 0,196 | 0,0112 | 0,9982 | 0,2931 | 0,9981 |
| Aave v3 Base · USDC (si rsETH **no** era colateral ahí; se verifica al congelar) | Kelp por (c): `s` = 196 / 26.106 | 0,0112 | 0,9982 | 0,0112 | 0,9982 |
| Vault Yearn v3 en Base | Los 4 de Yearn en código legacy, por (c) | 0,0078 | 0,9983 | 0,0078 | 0,9983 |
| Pool stable de Curve en L2 | Vyper 2023 por (c) (otro compilador y patrón) | ≈0 | 0,9983 | ≈0 | 0,9983 |
| Convex · Pendle · Beefy | Ninguno | 0 | — | 0 | — |

Datos de `TVL_A` (DefiLlama *yields*, 2026-04-17): WETH Ethereum Core US$834,5M + WETH Arbitrum US$70,2M + wstETH Arbitrum US$93,1M = **US$997,8M**. TVL de Aave ese día: **US$26.106M**.

### A.3.2 Reset proporcional de la evidencia positiva — ACTIVO (Rod, 2026-09-23)

**Razón (Rod).** Si un incidente no puede mover la aguja, el modelo premia a los protocolos grandes por antigüedad y acaba confirmando lo que ya se asumía al empezar, en vez de ponerlo a prueba. El diagnóstico que lo demostraba era que Yearn, con 4 incidentes, puntuaba igual que Curve.

**Problema que resuelve** (hallazgo de la v1.3). Con la resta fija, un protocolo maduro acumula unas 31 unidades de evidencia positiva y un incidente resta como mucho 5. Incluso una pérdida del **100 % de nuestro mercado** dejaba a Aave con `T` = 0,9951.

**Regla:**

```
pos_efectiva = Σ_meses  w_pos(edad_mes) · Π_{incidentes posteriores a ese mes} (1 − min(1, s) · w_neg(edad_incidente))
T = 1 − exp(−pos_efectiva / 5)
```

- **No introduce ninguna constante:** usa la severidad `s` del A.3.1 y las dos medias vidas aprobadas (w_pos con 730 días, w_neg con 90 días).
- **Pérdida total de nuestro mercado** (`s` = 1): la confianza acumulada antes del incidente se borra y el protocolo vuelve a empezar como uno nuevo.
- **Pérdida del 20 %:** se borra el 20 % de lo acumulado.
- **El borrado se revierte** con la media vida negativa de 90 días.
- La evidencia positiva acumulada **después** del incidente no se ve afectada.
- Las auditorías son evidencia positiva y reciben el mismo tratamiento, según su fecha.

**Efecto en el caso de referencia (Aave v3 Arbitrum · USDC):**

| Caso | `T` con resta (v1.3) | `T` con reset (v1.4) |
|---|---|---|
| Kelp, hoy (hace 157 días) | 0,9981 | 0,9976 |
| Kelp, si hubiera ocurrido hoy | 0,9978 | 0,9940 |
| Pérdida total de nuestro mercado, hoy | 0,9951 | **0,1813** |

**Antes y después de los seis protocolos sobre el censo actual.** Son las posiciones candidatas, a fecha 2026-09-22, solo con meses limpios y sin auditorías. El riesgo a 30 días se calcula con el A.2 central.

| Posición | Meses | `T` v1.2 (resta, TVL protocolo) | `T` v1.3 (resta, TVL mercado) | **`T` v1.4 (reset)** | Riesgo 30 d v1.3 | **Riesgo 30 d v1.4** |
|---|---|---|---|---|---|---|
| Aave v3 Arbitrum · USDC | 77 | 0,9982 | 0,9981 | **0,9976** | 0,655 % | **0,656 %** |
| Curve · pool stable L2 | 80 | 0,9983 | 0,9983 | **0,9983** | 0,654 % | **0,654 %** |
| Yearn · vault v3 Base | 80 | 0,9983 | 0,9983 | **0,9983** | 0,654 % | **0,654 %** |
| Convex · pid mainnet | 65 | 0,9975 | 0,9975 | **0,9975** | 0,656 % | **0,656 %** |
| Pendle · market Arbitrum | 64 | 0,9974 | 0,9974 | **0,9974** | 0,656 % | **0,656 %** |
| Beefy · vault L2 | 64 | 0,9974 | 0,9974 | **0,9974** | 0,656 % | **0,656 %** |

**Lectura honesta:**
- En el censo actual, el reset **solo mueve a Aave**, y muy poco.
- **Yearn sigue igual que Curve.**
- Los seis quedan con prácticamente el mismo riesgo, alrededor de `λ_min`.

El reset corrige lo que tenía que corregir: una pérdida grave y reciente **sí** hunde la confianza. Pero el empate Yearn–Curve tenía además otras dos causas, que se explican en el A.3.3.

### A.3.3 Hallazgo al activar el reset: por qué Yearn sigue igual que Curve *(para revisión de Rod; no bloquea el congelado)*

**Causa 1. Alcance asimétrico de la evidencia.**
- La evidencia **negativa** se mide sobre **nuestro mercado** (A.3.1). Los 4 incidentes de Yearn fueron en código antiguo (v1/iearn y el pool de yETH) que la vault v3 no comparte, así que caen en el caso (c) y se diluyen contra el TVL de todo Yearn: `s` entre 0,0005 y 0,025.
- La evidencia **positiva** se sigue midiendo sobre el **protocolo entero**: 80 meses desde el primer TVL de Yearn. Una vault v3 desplegada mucho después hereda seis años de historial limpio que no son suyos.
- **Resultado:** el protocolo le presta su antigüedad a la vault, pero no sus incidentes.

**Causa 2. Media vida negativa de 90 días.**
- Incluso una pérdida **total** se olvida en torno al 90 % en menos de un año.
- **Contraste:** si nuestra posición hubiera estado en el mismo pool de yETH, con pérdida total el 2025-11-30, hoy daría `T` = 0,9973 con el reset (0,9981 con la resta).
- Esto no es un error: es la media vida de 90 días que Rod aprobó para que "un protocolo que arregló un fallo hace un año ya no sea el mismo". Pero implica que el reset solo se nota durante unos meses después de cada incidente.

**Opciones** (ninguna está aplicada; la decisión es de Rod):
- **(a) Aceptarlo como resultado.** Para *nuestras* posiciones, los seis protocolos son hoy equivalentes en `T`, y es la evidencia la que lo dice. La comparación la decidirán `c_ij`, `c_solo` y la banda, no la confianza. Se publica como hallazgo.
- **(b) Alcance simétrico:** medir la evidencia positiva sobre el mismo alcance que la negativa, es decir, los meses limpios del **mercado** (o de la familia de código que comparte mecanismo), no los del protocolo entero. No añade constantes. Haría que una vault v3 reciente tuviera menos confianza que un pool con años de historial propio. Requiere la fecha de despliegue de cada mercado, que es un dato on-chain.
- **(c) Memoria de historial del protocolo:** tratar los incidentes del caso (c) con la media vida positiva (730 días) en vez de la negativa, como "historial del protocolo" y no como "fallo reciente". Tampoco añade constantes, pero mezcla dos conceptos, y por eso la recomiendo menos que (b).

*(Resuelto en v1.5: Rod adoptó la opción (b), ver A.3.4.)*

### A.3.4 Alcance simétrico: la evidencia positiva se cuenta sobre el mismo mercado — ACTIVO (Rod, 2026-09-24)

**Regla.**
- Los meses limpios se cuentan desde la **creación on-chain del mercado** de nuestra posición: el bloque de creación del contrato de la reserva, pool, `pid`, vault o market, según la definición del A.3.1.
- Las auditorías solo cuentan si cubren **la implementación de ese mercado**.
- La evidencia negativa sigue siendo la del A.3.1 (casos a, b y c) con el reset del A.3.2.
- **Motivo:** la evidencia positiva y la negativa se miden con el mismo criterio. El protocolo deja de prestarle su antigüedad a un mercado nuevo.
- **Fuente de la fecha:** la creación on-chain, fijada el día del congelado. **Base (I2, v1.15):** el primer bloque en el que `eth_getCode(dirección)` deja de estar vacío, por búsqueda binaria en un nodo de archivo (`mainnet.base.org` sirve estado antiguo). **Arbitrum y Ethereum:** Blockscout (PRO API con la clave de solo lectura del F5 en Arbitrum; sin clave en Ethereum). **Desde la v1.17, el agente no tiene la clave:** la consulta la hace un proceso aparte, que devuelve solo el bloque y la fecha de una dirección. Límite declarado: un contrato destruido y vuelto a desplegar en la misma dirección daría la fecha de un despliegue con código en la cabeza; se comprueba que hay código en la cabeza.

**Antes y después** (2026-09-22). Los mercados candidatos salen de una regla, no de una elección: para cada protocolo, el pool **stable** elegible de mayor TVL en su cadena según DefiLlama. Como fecha de inicio se usa, por ahora, la del primer dato en DefiLlama; en la versión congelada se usará la creación on-chain. En Base no hay ningún pool stable de Curve que pase el suelo de US$250k, así que se usa Arbitrum. Sin auditorías; riesgo con el A.2 central.

| Posición candidata | Meses del protocolo | Meses del mercado | `T` v1.4 | **`T` v1.5** | Riesgo 30 d v1.4 | **Riesgo 30 d v1.5** |
|---|---|---|---|---|---|---|
| Aave · Arbitrum USDC (desde 2023-06) | 77 | 39 | 0,9976 | **0,9896** | 0,656 % | **0,678 %** |
| Curve · Arbitrum sUSDai-USDC (desde 2025-05) | 80 | 16 | 0,9983 | **0,9262** | 0,654 % | **0,846 %** |
| Yearn · Base USDC v3 (desde 2025-05) | 80 | 16 | 0,9983 | **0,9261** | 0,654 % | **0,847 %** |
| Convex · DOLA-sUSDe (desde 2024-11) | 65 | 22 | 0,9975 | **0,9637** | 0,656 % | **0,746 %** |
| Pendle · PT-USDai en Arbitrum (desde 2026-02) | 64 | 7 | 0,9974 | **0,7240** | 0,656 % | **1,384 %** |
| Beefy · Base USDC (Morpho) (desde 2026-01) | 64 | 7 | 0,9974 | **0,7240** | 0,656 % | **1,384 %** |

**¿Deja Yearn de empatar con Curve? No, pero ahora el empate está justificado.**
- Los dos mercados candidatos tienen **la misma edad (16 meses)**, así que acumulan la misma evidencia positiva.
- Los incidentes de Yearn (código legacy, pool de yETH) **no alcanzan** a esta vault según la atribución contrafactual (caso c, severidad diluida). Si el mercado candidato de Yearn fuera otro, por ejemplo uno más antiguo, dejarían de empatar.
- Antes, `T` dependía de la antigüedad del protocolo; ahora depende de la del mercado y de los incidentes que de verdad lo alcanzan.

**Efecto nuevo que aparece:** los mercados jóvenes (Pendle y Beefy, 7 meses) pasan a tener el **doble de riesgo** que Aave. Los mercados de Pendle **vencen por diseño**, así que nunca acumulan antigüedad. Es un sesgo nuevo, pro-longevidad del mercado, y se trata en el barrido (Anexo C, C5).

**Mercados que vencen por diseño (C5, v1.6).**
- **Qué son:** los que tienen una fecha de vencimiento en su contrato (hoy, los markets de Pendle).
- **Regla:** la antigüedad se cuenta desde la creación on-chain de su **familia de implementación**, es decir, la **factory** (misma versión) que los despliega **en esa cadena**. No desde la creación del market concreto.
- **Simetría:** la evidencia negativa usa ese mismo alcance. Un incidente en cualquier market de la misma factory cuenta como mecanismo compartido (A.3.1, caso b).
- **Los mercados sin vencimiento** siguen con la regla general: una vault nueva **es** joven, no por diseño sino de verdad.
- **Ilustración:** para Pendle v2 en Arbitrum se usa marzo de 2023, cuando Pendle v2 se desplegó en Arbitrum. En la versión congelada se usará la fecha de creación on-chain de la factory de cada market seleccionado.

### A.3.5 Antes y después de la v1.6 (C2 + C3 + C5)

Las mismas posiciones candidatas del A.3.4. El riesgo a 30 días de la v1.5 usa la tasa base de la v1.5; el de la v1.6 usa la tasa base de la v1.6 (A.2.7: incidentes + muertes).

| Posición candidata | `T` v1.5 | **`T` v1.6** | Riesgo 30 d v1.5 | **Riesgo 30 d v1.6** | Qué cambia |
|---|---|---|---|---|---|
| Aave · Arbitrum USDC (39 meses) | 0,9896 | **0,9896** | 0,677 % | **0,715 %** | Solo la tasa base. Kelp sigue contando por (b) |
| Curve · Arbitrum sUSDai-USDC (16 meses) | 0,9262 | **0,9262** | 0,846 % | **0,911 %** | Solo la tasa base |
| Yearn · Base USDC v3 (16 meses) | 0,9261 | **0,9262** | 0,846 % | **0,911 %** | C2: dejan de contar los incidentes legacy (caso c). Ahora **idéntico** a Curve |
| Convex · DOLA-sUSDe (22 meses) | 0,9637 | **0,9637** | 0,746 % | **0,795 %** | Solo la tasa base |
| Pendle · PT-USDai en Arbitrum | 0,7240 (7 meses del market) | **0,9935** (familia v2 en Arbitrum, desde 2023-03) | 1,383 % | **0,703 %** | C5: deja de estar penalizado por vencer |
| Beefy · Base USDC (Morpho) (7 meses) | 0,7240 | **0,7240** | 1,383 % | **1,535 %** | La vault es de verdad joven y no vence: C5 no aplica |

**Lectura:**
- **C3 sube el riesgo de todos**, entre un 5 % y un 11 % relativo. Es la corrección del sesgo de supervivencia, que favorecía a delegar en general.
- **C2 deja a Yearn exactamente igual que Curve.** Los dos mercados tienen la misma edad y ningún incidente que los alcance. Es el resultado correcto según la atribución contrafactual; ya no hay ninguna capa que favorezca al protocolo más grande o más antiguo.
- **C5 quita a Pendle la penalización** por vencer.
- **Beefy queda como el más castigado** por una razón legítima: su vault candidata tiene 7 meses.
- Con la selección aleatoria del §2.1, los mercados finales serán otros y estos valores cambiarán; la tabla ilustra las reglas, no el resultado.

**Nota de fidelidad:** FLP §4.6 usa una sola media vida. La Fase 2 calcula los pesos con dos; es una extensión documentada aquí y **no modifica el repo del protocolo**.

### A.3.6 Antes y después de la v1.8 (9b)

La v1.8 no cambia ninguna regla de `T_ij`, así que los `T` de los seis mercados candidatos son los mismos que en la v1.6–v1.7. Cambia la tasa base (A.2.7, punto 8) y, con ella, el riesgo: `λ(T) = λ_max − T·(λ_max − λ_min)`.

| Posición candidata | `T` (sin cambios) | `λ(T)` v1.6–v1.7 | **`λ(T)` v1.8** | Riesgo 30 d v1.6–v1.7 | **Riesgo 30 d v1.8** |
|---|---|---|---|---|---|
| Aave · Arbitrum USDC | 0,9896 | 8,73 % | **9,96 %** | 0,715 % | **0,816 %** |
| Curve · Arbitrum sUSDai-USDC | 0,9262 | 11,14 % | **12,54 %** | 0,911 % | **1,025 %** |
| Yearn · Base USDC v3 | 0,9262 | 11,14 % | **12,54 %** | 0,911 % | **1,025 %** |
| Convex · DOLA-sUSDe | 0,9637 | 9,71 % | **11,02 %** | 0,795 % | **0,901 %** |
| Pendle · PT-USDai en Arbitrum | 0,9935 | 8,58 % | **9,80 %** | 0,702 % | **0,803 %** |
| Beefy · Base USDC (Morpho) | 0,7240 | 18,82 % | **20,76 %** | 1,535 % | **1,692 %** |

Todos los riesgos suben entre un 10 % y un 14 % relativo. El orden entre posiciones no cambia.

### A.3.7 Antes y después de la v1.9 (D.5a + D.5b): `T_ij`, riesgo y listón en APY

`T_ij` **no cambia**: la v1.9 no toca la evidencia. Cambia la tasa base (A.2.8), que pasa de [9,54 %, 50,20 %] a [0,16 %, 14,84 %].

| Posición candidata | `T` | `λ(T)` v1.8 | **`λ(T)` v1.9** | Riesgo 30 d v1.8 | **Riesgo 30 d v1.9** |
|---|---|---|---|---|---|
| Aave · Arbitrum USDC | 0,9896 | 9,96 % | **0,31 %** | 0,816 % | **0,025 %** |
| Curve · sUSDai-USDC | 0,9262 | 12,54 % | **1,24 %** | 1,025 % | **0,102 %** |
| Yearn · Base USDC | 0,9262 | 12,54 % | **1,24 %** | 1,025 % | **0,102 %** |
| Convex · DOLA-sUSDe | 0,9637 | 11,02 % | **0,69 %** | 0,901 % | **0,057 %** |
| Pendle · PT-USDai | 0,9935 | 9,80 % | **0,25 %** | 0,803 % | **0,021 %** |
| Beefy · Base USDC | 0,7240 | 20,76 % | **4,21 %** | 1,692 % | **0,345 %** |

**Ahora `T` sí discrimina.** Con un rango de tasa base más estrecho en términos absolutos pero más amplio en términos relativos (0,16 % → 14,84 %, un factor de 95), Beefy (vault de 7 meses) tiene 17 veces el riesgo de Pendle. En la v1.8 la diferencia era de 2 veces.

**Listón en APY: lo que tendría que rendir de más el delegado para compensar el término de riesgo.** Es `λ(T) × LGD`, y es igual para los tres nocionales (US$1k, 10k y 100k), porque `L` es proporcional al nocional. El gas fijo, que va en `c_ij` y `c_solo` y todavía no está medido, sube el listón en los nocionales pequeños.

| Posición candidata | APY actual | Listón central v1.8 | **Listón central v1.9** (LGD 0,6) | Peor de la banda v1.8 | **Peor de la banda v1.9** (LGD 1,0) | Mejor de la banda v1.9 (LGD 0,3) |
|---|---|---|---|---|---|---|
| Aave · Arbitrum USDC | 3,01 % | 5,98 pp | **0,19 pp** | 26,01 pp | **4,38 pp** | 0,07 pp |
| Curve · sUSDai-USDC | 4,48 % | 7,53 pp | **0,74 pp** | 27,58 pp | **5,18 pp** | 0,31 pp |
| Yearn · Base USDC | 4,88 % | 7,53 pp | **0,74 pp** | 27,58 pp | **5,18 pp** | 0,31 pp |
| Convex · DOLA-sUSDe | 6,06 % | 6,61 pp | **0,41 pp** | 26,65 pp | **4,67 pp** | 0,17 pp |
| Pendle · PT-USDai | 11,37 % | 5,88 pp | **0,15 pp** | 25,91 pp | **4,34 pp** | 0,05 pp |
| Beefy · Base USDC | 4,60 % | 12,46 pp | **2,53 pp** | 32,60 pp | **8,99 pp** | 0,99 pp |

Nota: los valores del "peor de la banda v1.8" de esta tabla se recalcularon con la rejilla completa de variantes del A.2.5, así que son algo mayores que los del Anexo D (22–32 pp), que usaban solo la variante B · relativa.

**Lectura: ¿sigue siendo una tautología?**
- **Ya no.** En el central, el listón (0,15–2,5 pp) queda **por debajo del APY total** de los seis mercados.
- **Pero la comparación que de verdad decide no es contra el APY total**, sino contra la **ventaja** de delegar (lo que se ahorra en gas, compounding y MEV). Para nocionales pequeños esa ventaja es del orden de décimas de punto a unos pocos puntos. Con un listón central de 0,15–0,74 pp en cinco mercados, **DELEGAR puede ganar o perder según los datos.** El resultado ya no está decidido de antemano.
- **En la banda completa sigue siendo exigente:** su peor punto (4,3–9,0 pp) solo queda por debajo del APY total en Convex y Pendle. Así que en la puntuación (ii) habrá muchos EMPATES, que son los "empates reales" que Rod aceptó en la v1.2.
- **La cota superior de la banda la dominan los dos incidentes sin `TVL_A` medido** (A.2.8, punto 5). Medirlos en la Fase 0 es lo que más puede estrecharla.

### A.4 Registro diario y cards

**Qué se registra cada día, después del cierre de las 18:50 UTC (H1, v1.25) y antes de conocer el resultado:** un evento `prediction` por par, nocional, horizonte y forma, con:
- `c_ij`, `T_ij` (por H_pos), `λ_min` y `λ_max` (por variante del A.2) y `p_h`;
- `L` para cada LGD;
- `c_solo_ex_ante` para cada punto de G;
- la clase predicha (DELEGAR / DIRECTO / EMPATE) y la ventaja en el punto central;
- el sha256 de los datos brutos de las fotos del día;
- la versión de esta metodología.

Si el tamaño lo exige, el `paper_log` lleva el hash del vector completo de G y el vector va aparte.

**Card por protocolo:** va en formato nativo FLP (`needs` / `surplus` / `vocab`). Es una **card proxy firmada por el Ambassador**, no por el protocolo. `c_ij`, `T_ij` y `L` **no se publican en la card** (FLP §5.3 / §8.3).

---

## Anexo B — Censo de incidentes (provisional, al 2026-09-22)

**Exposición** (primera fecha con TVL > 0 en DefiLlama → 2026-09-22):

| Protocolo | Primera fecha con TVL | Años-protocolo |
|---|---|---|
| Aave (parent, 7 slugs) | 2020-05-20 | 6,34 |
| Curve (parent, 4 slugs) | 2020-02-09 | 6,62 |
| Yearn (parent, 3 slugs) | 2020-02-12 | 6,61 |
| Pendle (parent, 2 slugs) | 2021-06-16 | 5,27 |
| Convex | 2021-05-17 | 5,35 |
| Beefy | 2021-05-26 | 5,33 |
| **Total** | | **35,51** |

**Incidentes que cuentan** (definición amplia; ✓ indica que entra con ese umbral):

| # | Fecha | Protocolo | Evento | Pérdida en rol (a) al evento | Causa | US$0 (central) | US$1M | US$10M | Estricta |
|---|---|---|---|---|---|---|---|---|---|
| 1 | 2021-02-05 | Yearn | Vault yDAI v1 | ~US$11,0M | Estrategia propia | ✓ | ✓ | ✓ | ✓ |
| 2 | 2022-06 | Aave | Despliegue v3 en Harmony: mala deuda a lenders de ONE/LINK tras el hack del bridge Horizon | ~US$1,2M | Bridge de terceros | ✓ | ✓ | — | — |
| 3 | 2023-04-13 | Yearn | yUSDT legacy (iearn) mal configurado | ~US$11,4M | Configuración propia | ✓ | ✓ | ✓ | ✓ |
| 4 | 2023-07-30 | Curve | Reentrancy por el compilador Vyper en pools de ETH | ~US$69,3M (bruto) | Contratos propios | ✓ | ✓ | ✓ | ✓ |
| 5 | 2025-11-30 | Yearn | Pool yETH (stableswap legacy) | ~US$9,0M | Contrato propio | ✓ | ✓ | — | ✓ |
| 6 | 2025-12-16 | Yearn | Vault v1 legacy (iearn) | ~US$0,29M (+ ~US$0,21M bloqueados) | Configuración propia | ✓ | — | — | ✓ |
| 7 | 2026-04-18 | Aave | Mala deuda en V3/V4 por rsETH sin respaldo (bridge de Kelp DAO) | ~US$196M de mala deuda propia de Aave; Umbrella absorbe la primera pérdida | Colateral o bridge de terceros | ✓ | ✓ | ✓ | — |

**Evaluados y excluidos:**

| Fecha | Protocolo | Evento | Motivo |
|---|---|---|---|
| 2022-06 | Convex | Secuestro de DNS (~US$14k) | Frontend (1c) |
| 2022-08-09 | Curve | Secuestro de DNS (~US$575k) | Frontend (1c) |
| 2024-05 | Beefy | Vaults de Sonne en Optimism y Base | Subyacente de terceros: no pasa el contrafactual (1b) |
| 2024-08 | Aave | Adapter de ParaSwap (US$56k) | Periférico, sin principal de depositantes (1c) |
| 2024-09 | Pendle | Penpie (US$27M) | Proyecto distinto construido encima (1c) |
| 2025-06-26 | Convex / Yearn | Resupply (~US$9,6M) | Proyecto distinto (sub-DAO) (1c) |
| 2025-11 | Beefy | Pausa por el hack de Balancer V2 | Subyacente de terceros (1b) |
| 2026-03-02 | Curve | LlamaLend sDOLA (~US$0,82M) | Solo borrowers; los lenders no se vieron afectados (1a) |
| 2026-03-10 | Aave | Fallo del oráculo CAPO de wstETH (~US$26–27M liquidados; 512,19 ETH reembolsados) | Solo borrowers (1a) |

**Fuentes:**
- [rekt.news leaderboard](https://rekt.news/leaderboard/)
- [Yearn Rekt IV](https://rekt.news/yearn-rekt4)
- [Check Point: yETH](https://research.checkpoint.com/2025/16-wei/)
- [Cyvers: yUSDT](https://cyvers.ai/blog/misconfigured-yudt-results-in-11-6m-exploit-for-yearn-finance-hack-analysis)
- [Nexus Mutual: Curve 30-jul-2023](https://forum.nexusmutual.io/t/curve-finance-loss-event-30-july-2023-claims-guide/1188)
- [The Defiant: Aave en Harmony](https://thedefiant.io/news/defi/aave-assets-are-still-stuck-over-a-year-after-usd100m-harmony-bridge-hack)
- [Aave: rsETH Incident Report](https://governance.aave.com/t/rseth-incident-report-april-20-2026/24580)
- [CoinDesk: Aave y Kelp](https://www.coindesk.com/tech/2026/04/20/aave-could-face-up-to-usd230-million-in-losses-after-kelp-dao-bridge-exploit-triggers-defi-chaos)
- [Aave: reembolso CAPO](https://governance.aave.com/t/direct-to-aip-wsteth-capo-oracle-incident-user-reimbursement/24275)
- [LlamaRisk: post-mortem sDOLA](https://www.llamarisk.com/research/llamalend-sdola-long2-post-mortem)
- [Convex DNS (Quadriga)](https://www.quadrigainitiative.com/hackfraudscam/convexfinancemaliciousdnshijack.php)
- [Beefy: Sonne](https://www.chaincatcher.com/en/article/2124424)
- [The Defiant: Penpie](https://thedefiant.io/news/defi/pendle-pauses-contracts-after-yield-protocol-penpie-suffers-usd27-million-exploit)
- [SolidityScan: adapter de ParaSwap](https://blog.solidityscan.com/aave-repay-adapter-hack-analysis-aafd234e15b9)
- API gratuita de DefiLlama (`api.llama.fi/protocols`, `api.llama.fi/protocol/{slug}`)

**Limitaciones del censo provisional:**
- defillama.com/hacks devolvió 403 a la consulta automática; el censo del día del congelado debe revisarlo a mano.
- Con el umbral central en US$0, el censo es más sensible a incidentes pequeños y peor documentados. Se acepta a cambio de que el central no dependa de ninguna decisión humana.
- La cifra del caso 4 es bruta, antes de recuperaciones. La del caso 7 depende de la resolución final entre Kelp, LayerZero y Aave.

---

## Anexo C — Barrido de sesgos a favor del incumbente (v1.5, 2026-09-24)

**Por qué existe.** Se pidió tras encontrar el mismo patrón tres veces seguidas: el denominador (v1.3), la escala (v1.4) y el conteo asimétrico (v1.5). Cada capa tenía un sesgo distinto a favor de los protocolos grandes y antiguos. Se revisó toda la metodología buscando cualquier punto donde el **tamaño, la antigüedad o el TVL** entren en el cálculo y favorezcan al incumbente sin que sea una decisión consciente y declarada. También se anotan los sesgos en sentido contrario.

**Estado de cada hallazgo:**
- **APLICADO:** ya está en esta versión.
- **DECLARADO:** se mantiene como decisión consciente y queda escrito.
- **DECISIÓN:** Rod tiene que elegir.

| # | Dónde | Qué favorece | Mecanismo | Estado y propuesta |
|---|---|---|---|---|
| C1 | A.3: **auditorías** | Incumbentes con más recursos | Contaban todas las auditorías del protocolo. Aave o Yearn acumulan decenas, sin relación con el mercado de nuestra posición. Además era inconsistente con (b) | **APLICADO en v1.5:** solo cuentan las auditorías que cubren la implementación del mercado. Rod puede revertirlo |
| C2 | A.3.1: **caso (c), incidentes en otros mercados** | **Protocolos grandes** | Se diluyen con `P / TVL_protocolo`: un incidente de US$10M pesa un 0,04 % en Aave (US$26.000M) y un 8 % en Beefy (US$120M). Es el mismo patrón que el denominador original. Además, tras (b), la negativa cuenta otros mercados y la positiva ya no | **APLICADO en v1.6 (Rod):** caso (c) eliminado (A.3.1). Ni la evidencia positiva ni la negativa usan mercados que no comparten mecanismo con el nuestro. Se pierde a propósito la señal de "cultura del protocolo" |
| C3 | A.2: **clase del censo (supervivencia)** | **Delegar y a los supervivientes** | La tasa base sale de seis protocolos elegidos *porque* siguen vivos y son grandes. Los protocolos equivalentes que murieron por exploits no están en la clase, así que `λ` sale subestimado. El término de riesgo pesa menos y delegar sale favorecido | **CORREGIDO en v1.6 (Rod: "no basta con declararlo"):** se suma una tasa de muerte calculada sobre una clase comparable con los muertos dentro (A.2.7). Clase A: el umbral es el menor pico de los seis, no uno elegido. La tasa base central sube de [7,93 %, 40,62 %] a **[8,33 %, 46,34 %]**. ARMA y Eliza quedan fuera, con su motivo. **Residuo declarado** en A.2.7, punto 10. **Pendiente:** la clase B (A.2.7, punto 9) |
| C4 | §3: **tope del 5 % del TVL** | Pools grandes (normalmente de incumbentes) | Efecto de selección: los nocionales grandes (US$100k) solo se evalúan en pools grandes, así que las conclusiones a ese tamaño hablan solo de ellos | **DECLARADO:** la regla protege de la dilución y se mantiene. El reporte indica, para cada nocional, qué mercados entran |
| C5 | A.3.4: **mercados que vencen por diseño** *(nuevo con v1.5)* | **Mercados longevos** (en contra de Pendle y de vaults nuevas) | Los markets de Pendle vencen, así que nunca acumulan antigüedad. Una vault nueva de un protocolo sin incidentes arranca con `T` = 0 → `λ_max`. Pendle y Beefy quedan con el doble de riesgo que Aave (1,38 % frente a 0,68 %) | **APLICADO en v1.6 (Rod):** los mercados con vencimiento en su contrato cuentan la antigüedad de su familia de implementación (misma factory o versión, en esa cadena), con el mismo alcance para la evidencia negativa (A.3.4). Pendle pasa de 1,38 % a 0,70 %. Los mercados sin vencimiento no cambian |
| C6 | §2: **qué mercado se elige** entre los elegibles | Indefinido: puede favorecer a quien sea | El §2 dice qué pares son elegibles, pero no cuál se elige cuando hay varios. En el A.3.4 se usó "mayor TVL" como ilustración, y eso favorece a los incumbentes. Si se elige a mano el día del congelado, hay margen para un sesgo | **APLICADO en v1.6 (Rod):** todos los elegibles hasta N = 3 por protocolo y cadena, ordenados por `sha256(semilla ‖ dirección)`. La semilla es el hash del primer bloque de Bitcoin minado después de las 00:00 UTC del día del congelado (§2.1) |
| C7 | A.2 y A.3: **T = 0 → `λ_max`** para lo nuevo | Lo antiguo | Sin evidencia, FLP no concede el beneficio de la duda (§3.2). Con v1.5 pesa más, porque se aplica por mercado | **DECLARADO:** es un principio de FLP y es intencional. Queda escrito aquí que, con v1.5, los mercados jóvenes arrancan en el extremo pesimista |
| C8 | §5: **unidades de gas** ("últimas 50 transacciones en 30 días") | Contratos con mucha actividad | Los contratos pequeños o nuevos pueden no tener 50 transacciones en 30 días. La regla queda indefinida o ruidosa justo en los no incumbentes | **APLICADO en v1.6 (Rod); corregido en v1.15 (R5) para que coincida con el §5:** respaldo en tres pasos, **todo dentro de la ventana de 30 días**: (i) todas las transacciones válidas de la ventana (mínimo 10); (ii) la misma acción en contratos hermanos de la misma implementación (bytecode; familia de factory en Curve/Convex, R4), dentro de la ventana, contrato a contrato (R3); (iii) `sin_dato_gas` y el par fuera. Hasta la v1.14 este anexo decía "todas desde el despliegue", que contradecía al §5 y no es medible con nodos públicos |
| C9 | A.2: **cobertura del censo** | **Pequeños** (sentido contrario) | Los incidentes de protocolos grandes tienen más cobertura mediática. Con umbral 0, los incidentes pequeños de protocolos pequeños pueden no aparecer, y eso los favorece | **DECLARADO:** dirección opuesta a las demás; se documenta y no se corrige |
| C10 | §2: **suelo de TVL y fuente DefiLlama** | Protocolos listados | Solo entra lo que DefiLlama sigue, y DefiLlama sigue antes a lo grande | **DECLARADO:** efecto menor; el suelo es bajo (US$250k) |

**Cosas revisadas en las que no se encontró sesgo pro-incumbente:**
- el cálculo de `c_solo` (el gas y el slippage son costes reales de cada pool);
- la banda de indiferencia (simétrica por construcción);
- el tiempo humano (se aplica por acción, igual para todos);
- el umbral de materialidad central en US$0 (no depende de ninguna decisión humana);
- la media vida positiva de 730 días (ahora se aplica por mercado);
- la saturación por `TRUST_SCALE` (declarada intencional en v1.3).

**Resumen:**
- **La cuarta capa existe, y es C2.** Es el mismo patrón del denominador original, que sobrevivía en el caso (c). **Eliminada en la v1.6.**
- **C3 (supervivencia) es un sesgo de clase:** favorece a delegar en general, no a un protocolo frente a otro. **Corregido en la v1.6**, con un residuo declarado.
- **C5 y C6 los creó o destapó la propia v1.5.** **Aplicados en la v1.6.**
- **Estado tras la v1.6:** 6 aplicados (C1, C2, C3, C5, C6 y C8), 4 declarados (C4, C7, C9 y C10) y 1 decisión pendiente (la clase B del A.2.7).
- **v1.7:** la clase B quedó resuelta (muerte relativa, en la banda). Al aplicarla apareció una **quinta capa**: el umbral absoluto de muerte infracuenta las muertes de los protocolos grandes en la clase A (A.2.7, punto 9b). **Decisión pendiente de Rod.**
- **v1.8:** la quinta capa quedó **corregida** (Rod, 9b): la clase A usa la muerte relativa con los mismos parámetros que la B.

---

## Anexo D — Prueba de la objeción contraria (v1.8, 2026-10-06): ANÁLISIS, SIN CAMBIOS

**La duda (Rod).** Llevamos cinco correcciones seguidas en la misma dirección. La tasa base queda alta, y buena parte de la mortalidad sale de protocolos comparables bastante más pequeños que aquellos donde estaría nuestra posición. Si el modelo acaba diciendo "nunca delegues" por construcción, es el mismo error de antes con el signo cambiado.

**Disciplina:** aquí solo se miden números. Cualquier cambio que salga de esto tiene que justificarse por una razón de medición que valga **independientemente del número que arroje**, igual que se hizo con la muerte relativa. Si no hay una razón así, se declara como limitación y no se cambia. Datos: la misma clase del A.2.7 (`c3/clase_tvl.json`, sha256 `20a59145…`) y las mismas reglas de la v1.8.

**Precisión previa sobre la cifra de la duda:** el rango "22 %–59 %" es el de la variante **B · relativa**. La central de la v1.8 (A · relativa) es **9,54 %–50,20 %**. Las dos están en la banda.

### D.1 Reparto de la mortalidad por tamaño (muerte relativa)

| Pico de TVL | Protocolos | Muertes | Años-protocolo | Tasa de muerte | IC 95 % | Muertes bruscas (≥ 50 % del pico en 7 días) |
|---|---|---|---|---|---|---|
| US$0,25–1M | 109 | 35 | 223,4 | 15,67 % | [10,92 %, 21,79 %] | 22/35 (63 %) |
| US$1–10M | 227 | 86 | 491,3 | 17,51 % | [14,00 %, 21,62 %] | 47/86 (55 %) |
| US$10–100M | 196 | 88 | 470,0 | 18,72 % | [15,02 %, 23,07 %] | 42/88 (48 %) |
| US$100M–1,3B | 100 | 43 | 286,7 | 15,00 % | [10,85 %, 20,20 %] | 18/43 (42 %) |
| **≥ US$1,3B (clase A)** | 30 | 6 | 136,3 | **4,40 %** | [1,62 %, 9,58 %] | 3/6 (50 %) |
| — dentro de A: US$1,3–5B | 18 | 6 | 68,1 | 8,81 % | [3,23 %, 19,18 %] | |
| — dentro de A: ≥ US$5B | 12 | 0 | 68,2 | 0,00 % | [0,00 %, 5,41 %] | |

**Lectura:**
- **Los que murieron NO son sistemáticamente los más pequeños.** Por debajo de US$1,3B, la mortalidad es **plana**, entre el 15 % y el 19 % en todos los tramos. La mediana del pico es **mayor** en los muertos (US$12,1M) que en los vivos (US$7,4M).
- **El salto está arriba.** Al pasar de US$1,3B la tasa cae a un cuarto. Dentro de la A, las 6 muertes están en US$1,3–3B, y **ninguno de los 12 protocolos de ≥ US$5B murió**.
- **Incidentes por tramo: no se pueden derivar con los datos actuales.** El censo de incidentes (A.2) solo cubre a los seis, todos en el tramo superior. Sin la API de hacks de DefiLlama, que es de pago, no hay un censo comparable de incidentes para los demás tramos.

### D.2 Tasa base si los comparables se restringen al tamaño de nuestras posiciones

Nuestras posiciones están en los seis protocolos, cuyos picos van de US$1,3B (Beefy) a US$45,8B (Aave). **Ese tramo es exactamente la clase A**, que es la que ya usa la variante central:

| Comparables | Tasa de muerte | Tasa base total (`λ̂` [λ_min, λ_max]) |
|---|---|---|
| Todos los ≥ US$250k (B · relativa, en la banda) | 16,05 % | 35,76 % [22,08 %, 58,75 %] |
| **Tramo de los seis, ≥ US$1,3B (A · relativa, central)** | **4,40 %** | **24,11 % [9,54 %, 50,20 %]** |
| Subtramo US$1,3–5B | 8,81 % | (solo informativo) |
| Subtramo ≥ US$5B | 0,00 % [0, 5,41 %] | (solo informativo) |

### D.3 Peso de cada componente en la tasa base

| Variante | `λ̂` | % incidentes | % muertes | `λ_max` | % incidentes en `λ_max` |
|---|---|---|---|---|---|
| **A · relativa (central)** | 24,11 % | **82 %** | 18 % | 50,20 % | **81 %** |
| A · estricta | 19,71 % | 100 % | 0 % | 43,00 % | 94 % |
| B · estricta | 21,59 % | 91 % | 9 % | 43,16 % | 94 % |
| B · relativa | 35,76 % | 55 % | 45 % | 58,75 % | 69 % |

**Lectura:** en la variante central, **el 82 % de la tasa base sale de los incidentes de los seis, no de la mortalidad.** La mortalidad de los protocolos pequeños solo pesa en la variante B · relativa. Esa variante está en la banda, así que afecta a cuántos días salen en EMPATE, pero no al valor central.

### D.4 Cuánto tendría que rendir de más un protocolo delegado para compensar el término de riesgo

**El coste anual esperado del término de riesgo, en puntos de APY, es `λ(T) × LGD`.** Como `L` = nocional × LGD, **es el mismo para los tres nocionales** (US$1k, US$10k y US$100k). El nocional solo pesa en los costes fijos de gas, que van en `c_ij` y `c_solo`, todavía no medidos. Esos costes **suben** el listón en los nocionales pequeños, encima de lo que muestra esta tabla.

| Posición candidata | `T` | **Listón central** (LGD 0,6) | Peor de la banda (B · relativa, LGD 1,0) | Mejor de la banda (A · estricta, LGD 0,3) | APY actual (DefiLlama) |
|---|---|---|---|---|---|
| Aave · Arbitrum USDC | 0,9896 | **5,98 pp** | 22,46 pp | 2,49 pp | 3,01 % |
| Curve · sUSDai-USDC | 0,9262 | **7,52 pp** | 24,78 pp | 3,15 pp | 4,48 % |
| Yearn · Base USDC | 0,9262 | **7,52 pp** | 24,78 pp | 3,15 pp | 4,88 % |
| Convex · DOLA-sUSDe | 0,9637 | **6,61 pp** | 23,41 pp | 2,76 pp | 6,06 % |
| Pendle · PT-USDai | 0,9935 | **5,88 pp** | 22,31 pp | 2,45 pp | 11,37 % |
| Beefy · Base USDC | 0,7240 | **12,46 pp** | 32,20 pp | 5,28 pp | 4,60 % |

**Lectura:**
- **El listón central (6–12 pp) supera el rendimiento TOTAL actual en cinco de los seis mercados.** Solo Pendle queda por encima.
- **En el peor punto de la banda (22–32 pp), ningún mercado lo supera.**
- Como la banda exige que el ganador sobreviva a **todos** sus puntos, en la puntuación (ii) **es prácticamente imposible que salga "DELEGAR"**: el modelo dice "nunca delegues" **por construcción**. La puntuación (i), de solo costes, no se ve afectada.

### D.5 Dos observaciones de medición *(APLICADAS en v1.9 por decisión de Rod: (a) en el A.2.8, (b) en el A.2.7, punto 11)*

Las dos empujan **hacia delegar**. Por eso se les aplica la misma disciplina: solo valen si su razón de medición se sostiene **sin mirar el número que producen**. Esa razón queda escrita junto a cada una.

**(a) Unidad de `λ_inc`: incidentes por protocolo frente a pérdida por depósito.**
- **Qué mide hoy:** la probabilidad de que el protocolo sufra **algún** incidente en **cualquiera** de sus mercados (19,71 %/año por protocolo).
- **Cómo se usa:** como la probabilidad de que **nuestro** depósito pierda principal, porque se multiplica por `L` = nuestro nocional × LGD.
- **Ponderado por el TVL alcanzado**, el peligro por dólar depositado es, como **cota inferior**, del **0,32 %/año**. Se calcula sumando la fracción del TVL del protocolo que alcanzó cada incidente (Kelp 3,82 %, yUSDT 2,55 %, Vyper 2,13 %, yETH 1,46 %, yDAI 1,16 %, iearn 0,05 %, Harmony 0,05 %), que da 0,11 "protocolos-equivalentes" en 35,51 años. Es cota inferior porque, salvo en Kelp y Harmony, se usó la pérdida como sustituto del TVL alcanzado, y el TVL alcanzado es mayor o igual que la pérdida.
- **Diferencia:** de **hasta ~60 veces** frente a la tasa por protocolo.
- **Razón de medición, independiente del número:** la probabilidad que multiplica a `L` tiene que estar en la **misma unidad** que `L`, que es nuestra posición. Además, desde la v1.3–v1.5 el modelo mide `T_ij` **por mercado**, pero `λ` sigue **por protocolo**: es la misma inconsistencia de alcance que se corrigió en la v1.5, ahora en la tasa base.

**(b) Muerte no es lo mismo que pérdida de principal.**
- A todas las muertes se les aplica un LGD de {0,3, 0,6, 1,0}; es decir, el modelo asume que en cada muerte se pierde al menos el 30 % del principal.
- **Pero en la muerte relativa**, por definición, en el momento de la muerte el protocolo solo retiene **≤ 2,03 % de su pico**: casi todos los depositantes ya habían salido.
- Solo **10 de las 258** muertes de la clase B están marcadas como `rugged`.
- La mitad de las muertes tienen una caída de ≥ 50 % del pico en 7 días, pero **la brusquedad no distingue la pérdida de la salida** (Royco, por ejemplo, cayó un 88 % en una semana al terminar una campaña de puntos).
- El LGD de las muertes **no está medido**.
- **Razón de medición, independiente del número:** el evento que se cuenta (que el TVL de un protocolo se vacíe) no es el evento que modela `L` (que el depositante pierda principal). Hace falta una definición de la pérdida en caso de muerte, o un LGD de muerte medido (por ejemplo, la fracción del TVL que seguía depositada al morir y no se recuperó).

### D.6 Resumen numérico

1. **Mortalidad por tamaño:** plana (15–19 %) por debajo de US$1,3B; 4,4 % en la clase A; 0 % por encima de US$5B. Los muertos **no** son los más pequeños.
2. **Restringida al tramo de nuestras posiciones:** es la clase A, la que ya usa el central: 24,11 % [9,54 %, 50,20 %].
3. **Pesos:** en el central, el 82 % son incidentes y el 18 % muertes. La mortalidad de los pequeños solo pesa en la variante B · relativa (45 %).
4. **Listón en APY**, igual para los tres nocionales: el central (6–12 pp) supera el APY total en 5 de 6 mercados; el peor de la banda (22–32 pp) lo supera en todos.
5. **Dónde está el exceso, si lo hay:** según estos números, **no** en la mortalidad de los pequeños (premisa de la duda), sino en **la unidad de `λ_inc`** (D.5a) y en **el LGD de las muertes** (D.5b).

---

## Anexo E — Pasada simétrica: ¿abrió la v1.9 algún sesgo A FAVOR de delegar? (2026-10-06)

**Por qué existe.** La v1.9 corrige dos sesgos **en contra** de delegar (D.5a y D.5b). Con el mismo rigor que el barrido del Anexo C, se revisa si esas correcciones, o el resto de la metodología, dejan algún punto que favorezca a delegar sin una decisión consciente y declarada.

**Estados:** APLICADO, DECLARADO o DECISIÓN, como en el Anexo C.

| # | Dónde | Qué favorece | Mecanismo | Medida | Estado y propuesta |
|---|---|---|---|---|---|
| **E1** | A.2 y A.2.8: **censo de incidentes solo de los seis** | **Delegar** (subestima `λ_inc`) | Es el residuo de supervivencia ya declarado en el A.2.7, punto 10, pero **la v1.9 lo agrava**. Antes, los protocolos comparables que hicieron perder dinero y después murieron pesaban al menos a través de su muerte (LGD ≥ 0,3). Con D.5b sus muertes son **graduales** y no cuentan. Como sus incidentes nunca entraron en el censo, **ya no cuentan en ningún sitio** (CREAM 2021, Euler 2023…) | **Censo indicativo** de los otros 24 protocolos de la clase A (solo rekt.news, solo pérdidas grandes): **7 incidentes que cuentan** con las reglas del A.2 (CREAM ×2, Balancer ×2, Euler, Venus ×2). Excluidos: Badger 2021 (frontend), Compound 2021 (error de reparto de COMP, sin pérdida de principal) y PancakeBunny (otro protocolo). Probable faltante: Rari/Fuse 2022. Euler perdió el **85 %** de su TVL. Tasa por dólar de la clase A completa: **~0,92 %/año** [0,50 %, 1,55 %], frente a **0,39 %/año** de los seis solos. Es una cota inferior, porque usa `f` en su cota inferior y un censo incompleto | **DECISIÓN.** Ampliar el censo de incidentes a **toda la clase A** (30 protocolos, con las cuatro fuentes y las reglas del A.2), con exposición = años-protocolo de la clase A. **Razón de medición, independiente del número:** es la misma corrección de supervivencia del C3 (muertes), aplicada a los incidentes. La clase de comparables tiene que ser la misma para los dos componentes de la tasa base. **→ APLICADO en la v1.10** (Rod, 2026-10-06): A.2.8, punto 7. Sube la tasa (va en contra de delegar) y sigue siendo una cota inferior. El censo completo se mide en el paso D0·2. *(El IC [0,50 %, 1,55 %] de esta fila usa `f_inf`; el del punto 7 usa `f_sup` = 1 para los externos y llega al 11,19 %.)* |
| E2 | A.2.8: **"por dólar" frente a "por mercado"** | Delegar en **mercados pequeños**, en contra en los grandes | La tasa por dólar promedia sobre todo el TVL del protocolo, y en él pesan mucho los mercados enormes, que rara vez sufren incidentes. Nuestras posiciones se eligen al azar entre los elegibles (§2.1), y pueden estar en mercados pequeños (US$1,5–50M), donde los incidentes del censo se concentran: yETH US$9M, iearn US$0,5M, Harmony US$3M, yDAI US$35M. Para un mercado pequeño, la tasa por dólar podría quedarse corta | No medible con los datos actuales: haría falta el número de mercados de cada protocolo a lo largo del tiempo | **DECLARADO**, con su dirección. Lo compensa en parte `T_ij`, que es por mercado: los mercados jóvenes o pequeños tienen menos evidencia positiva y, por tanto, más riesgo (Beefy 4,21 % frente a Pendle 0,25 %) |
| E3 | A.2.7, punto 11: **muerte gradual = 0 en `L`** | Delegar | (i) La salida forzada tiene coste (gas, slippage en una corrida, rendimiento perdido) que no se modela más allá del gas de salida normal. (ii) La sombra no tiene una **regla de salida**: el modelo supone que la posición delegada se mantiene todo el horizonte, así que quien no sale en una muerte gradual podría perder lo que queda | Las muertes graduales de la clase A son 6 en 136 años-protocolo; en un horizonte de 7 o 30 días, la probabilidad es baja | **DECLARADO.** El coste de salida forzada es de tipo `c`, del orden del gas de salida que ya se cuenta. Añadir una regla de salida queda fuera de alcance |
| E4 | A.2.7, punto 11: **resolución diaria de la curva** | Delegar | Un robo lento (más de un día) o una **congelación de retiradas** con el TVL todavía contabilizado no se ven como una caída brusca de un día. La congelación puede incluso no detectarse como muerte | Las 10 muertes `rugged` se clasifican bruscas por la etiqueta. Las 50 indeterminadas (B) se tratan como sin salida | **DECLARADO**, con la dirección del sesgo. Se mitiga con la etiqueta y con "indeterminada = sin salida" |
| E5 | A.2.8: **`TVL_A` y `TVL_protocolo` de DefiLlama en lending** | Indeterminado | En lending, DefiLlama cuenta la liquidez disponible (suministrado − prestado). El cociente `f` puede salir sesgado en cualquier sentido según la utilización de cada mercado | — | **DECLARADO**, sin dirección conocida |
| E6 | A.2.8: **`TVL_A` sin dato** | No favorece a delegar | La cota superior asume `f` = 1 (todo el protocolo): es pesimista | Es el componente que más ensancha la banda | **DECLARADO.** Conservador; medirlo en la Fase 0 lo estrecha |
| E7 | A.2.7, punto 11: **"indeterminada = sin salida"** | No favorece a delegar | Conservador por construcción | 50 casos en la B | **DECLARADO** |

**Revisado, sin sesgo a favor de delegar encontrado:**
- la propagación conservadora de la incertidumbre (A.2.8, punto 3);
- la banda, que exige robustez frente al peor punto;
- `T` = 0 → `λ_max` en los mercados nuevos (C7);
- las *rugged*, que siguen contando siempre.

**Resumen:**
- La v1.9 **sí deja al descubierto un sesgo a favor de delegar**: el **E1**. No lo crea, porque el censo de incidentes siempre fue solo de los seis, pero lo **agrava**: antes las muertes de los comparables lo compensaban en parte.
- Su efecto indicativo (tasa por dólar ×2,4: 0,39 % → ~0,92 %) **no devuelve el modelo a la tautología**. Con 0,92 % el listón central seguiría en el orden de décimas de punto a ~2 pp. Pero es el mismo tipo de corrección (supervivencia) que ya se aplicó a las muertes, y por coherencia debería aplicarse a los incidentes.
- E2–E5 quedan declarados con su dirección; E6 y E7 son conservadores.
