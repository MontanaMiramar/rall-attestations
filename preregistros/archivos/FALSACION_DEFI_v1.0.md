# Condición de falsación de la apuesta del proyecto en DeFi — v1.0 (PRE-REGISTRO)

Aprobado por Rod el 2026-10-23: el esquema y los umbrales de la v0.1, las enmiendas 1 y 2 en la v0.2, y D-23a (mayoría simple). Este documento se sella con la hora real del servidor, junto a su criterio en formato máquina y al script que lo evaluará. **Después del sellado no se cambia:** cualquier cambio sería un documento nuevo, con fecha nueva, declarado como tal.

## 0. La apuesta y por qué necesita esta condición

- **La apuesta del proyecto:** «los agentes autónomos se encontrarán con desconocidos y necesitarán decidir si cooperan, cada vez más, en terrenos donde hay dinero».
- **Sin condición de falsación, ningún resultado podría contradecirla.**
- **Este documento fija, antes de los datos que la deciden:**
  - qué resultado diría que **no se sostiene en DeFi**;
  - cuándo se mira, **una sola vez**;
  - qué elecciones del diseño favorecen al autor (§4).

## 1. Qué se mide: el CENSO DEL TERRENO, no el recuento de pares

- **Por qué no los pares:** 7 de los 8 pares del ensayo están muertos porque nuestra medición no pudo medirlos: R6 excluye en cascada las 67 retiradas de Yearn, hay constantes solo de hermanos y ventanas poco cubiertas. Contar pares mediría nuestras reglas de gas, no el mercado.
- **Lo que se mide:** mercado a mercado, la fracción de la actividad que mueve capital que hace **automatización de terceros** frente a la que decide y firma **el titular**, con `n` absolutos.
- **Independiente de nuestras reglas de gas:** se toman todas las transacciones que producen el evento (R1), sin R6, sin hermanos y sin medianas. Solo se clasifican con la lectura R1-B': ¿la posición acaba en quien firma, en la misma transacción?

## 2. Cómo se mide

**Unidad:** el mercado (contrato).

**Acciones que cuentan:** las que asignan o retiran capital:
- `yearn_deposit/withdraw`, `erc4626_deposit/redeem` (y peticiones R2), `beefy_deposit/withdraw`;
- `pendle_buy_pt/sell_pt`;
- `curve_add_liq/remove_one`, `curve_gauge_deposit/withdraw`, `convex_deposit/withdraw`;
- `aero_add_liq/remove_liq`, `aero_gauge_deposit/withdraw`.

**Fuera:** `approve`, el `swap` del pool genérico del router y los cobros de recompensas (ver §4).

**Fuentes:** solo on-chain: `eth_getLogs` y recibos de nodos públicos, en los almacenes `gas_v2` y `gas_censo`, con la clasificación R1-B' guardada en cada fila.

**Universo:**
- los mercados con buscadores en el universo de la observación y en el del censo, durante la ventana;
- la lista se congela en la salida con su sha256;
- solo transacciones propias de cada mercado, nunca de hermanos.

**Clases:**

| Clase | Qué significa |
|---|---|
| **autodirigida** | La posición acaba en quien firma: directa, vía router o reenviada por un zap |
| **de terceros** | Un contrato se queda la posición: estrategia, keeper, vault que asigna a otro vault |
| **sin lectura** | No se pudo leer el beneficiario |

**Fracción de intermediación:** `Î = terceros / (autodirigidas + terceros)`, la estimación directa. Al lado se publica el intervalo de Wilson al 95 %.

**Mercado medible** si cumple todo:
- (a) `n = autodirigidas + terceros ≥ 50`;
- (b) `sin lectura ≤ 20 %` del total observado;
- (c) los datos de su cadena cubren la ventana: la primera fila tiene como mucho 1 día de margen tras el inicio y la última, como mucho 1 día antes del final.

## 3. Veredicto

**Por mercado (medible):**

| Veredicto | Condición |
|---|---|
| **INTERMEDIADO** | `Î ≥ 0,80`. La igualdad cuenta como intermediado |
| **AUTODIRIGIDO** | `Î < 0,80` |
| **NO CONCLUYENTE** | Un AUTODIRIGIDO en el que una sola dirección firma más del 50 % de sus transacciones autodirigidas: la apuesta habla de desconocidos, y un solo actor no lo es |

El intervalo de Wilson se publica pero **no decide**: la cota inferior hacía más difícil declarar intermediación, que es lo que refuta. Es la enmienda 1 de Rod.

**Global** (`M` = mercados medibles; `C` = cadenas con al menos uno):

| Resultado | Condición |
|---|---|
| **FALSADA EN DEFI** | `M ≥ 6`, `C ≥ 2` e `INTERMEDIADOS > M/2` (**mayoría simple**) |
| **NO FALSADA** | `M ≥ 6`, `C ≥ 2` e `INTERMEDIADOS + NO CONCLUYENTES ≤ M/2` |
| **INDETERMINADA** | Cualquier otro caso. Se publica con los números, sin forzar conclusión |

**Por qué mayoría simple y no 2/3 (D-23a).**
- Al aplicar la enmienda 1 de Rod a todo el diseño, apareció que exigir 2/3 para falsar es **el mismo mecanismo que la cota de Wilson**: pedir más evidencia de la necesaria para refutarse a uno mismo.
- Lo detectó el propio autor al revisar su diseño con el criterio de Rod, y Rod lo aprobó.
- Queda escrito porque es el tipo de corrección que tiene que poder verse.

## 4. DECISIONES DE DISEÑO QUE FAVORECEN LA HIPÓTESIS DEL AUTOR

**Cada una con su resultado alternativo, publicado al lado del veredicto principal. Cualquiera puede recalcular con las otras elecciones y ver si el veredicto cambia.**

| # | Decisión que se mantiene | Por qué favorece la hipótesis | Por qué se mantiene | Resultado alternativo que se publica |
|---|---|---|---|---|
| F1 | **Los cobros de recompensas no cuentan** (`merkl_claim`, `curve_mint_crv`, `convex_claim`, `aero_claim`) | Los hacen sobre todo keepers y compounders; incluirlos **subiría** la intermediación | Un cobro no es la decisión de cooperar con capital: es mantenimiento | Veredicto global **con los cobros incluidos** |
| F2 | **Los zaps que reenvían la posición al firmante cuentan como autodirigidos** | Cuentan como decisión del titular algo que pasa por un contrato de terceros; contarlos como terceros **subiría** la intermediación | El titular firma y recibe la posición en la misma transacción: usar una herramienta no es delegar | Veredicto global **con las autodirigidas reenviadas contadas como de terceros** |
| F3 | **Las transacciones sin lectura no entran en la fracción** (solo pueden invalidar el mercado si superan el 20 %) | **Dirección desconocida.** Si las ilegibles fueran mayoritariamente transacciones complejas de terceros, excluirlas favorecería la hipótesis | No se puede clasificar lo que no se pudo leer | Veredicto global **con las sin lectura contadas como de terceros** (el peor caso para la hipótesis) |
| F4 | **El universo son los mercados que nuestra selección observa** (3 cadenas, 6 protocolos, estables con TVL ≥ 250 k, regla de medibilidad del D0) | **Dirección desconocida.** La selección puede sobrerrepresentar mercados con actividad directa, o lo contrario | Es el universo que medimos; no hay otro con esta lectura | **No hay alternativa calculable.** Se declara en el alcance (§6) |

**Decisiones que NO favorecen la hipótesis, también visibles:**
- **La estimación directa en vez de la cota de Wilson.** Era a favor de la hipótesis y se cambió (enmienda 1). Alternativa publicada: el veredicto con la cota inferior de Wilson.
- **La mayoría simple en vez de 2/3.** Era a favor de la hipótesis y se cambió (D-23a). Alternativa publicada: el veredicto con 2/3.
- **El empate `Î = 0,80` cuenta como intermediado.** Va hacia la refutación.
- **La regla de actor único solo rebaja los AUTODIRIGIDOS.** Va hacia la refutación.
- **n ≥ 50, sin lectura ≤ 20 %, M ≥ 6 y C ≥ 2.** Son neutras: hacen más difícil cualquier conclusión, en los dos sentidos.

**Sensibilidad al umbral (informativa):** veredicto global con T = 0,70 y con T = 0,90.

**Orden de escritura en la salida:** primero el veredicto principal, después las alternativas. Las alternativas **no cambian** el veredicto principal; existen para que el lector juzgue si depende de una elección del autor.

**Datos para recalcular:** por cada mercado se publican `n`, autodirigidas, autodirigidas reenviadas, de terceros, sin lectura, cobros (autodirigidos y de terceros), `Î`, el intervalo de Wilson y la cuota del primer firmante. Con eso, cualquier combinación de elecciones se puede recalcular a mano.

## 5. CUÁNDO se evalúa: UNA sola vez, en un momento pre-declarado

- **Corte: 2026-12-11 a las 00:00:00 UTC**, el cierre del mes de observación si el D0 es el 2026-11-10.
  - Día 1 = 2026-11-11 (primer día UTC completo tras el D0); la posición de 30 días se abre el día 1 y se cierra el día 30.
  - **Ventana evaluada:** [2026-11-11 00:00, 2026-12-11 00:00) UTC.
- **Si el D0 es otro día:** el corte es **D0 + 31 días a las 00:00 UTC** y la ventana son los 30 días anteriores.
  - El D0 se toma de la entrada `methodology_freeze` del paper_log, que es pública e inmutable; nadie lo elige después.
  - Si no hay D0, no se evalúa y se declara así.
- **Ejecución:**
  - una sola vez, entre el corte y el corte + 72 h;
  - el script se niega a correr fuera de esa ventana, si existe la marca `FALSACION_DEFI_APLICADA.json` o si su sha no es el sellado;
  - **no tiene vista previa sobre datos reales**, solo una autoprueba con datos sintéticos.
- **Si ese día no se alcanza el mínimo** (`M < 6` o `C < 2`): **INDETERMINADA, y es definitivo** para este mes de observación. No hay prórroga ni segunda evaluación. Cualquier evaluación posterior sería un pre-registro nuevo, que no sustituye a este.
- **Requisito operativo:** la acumulación de gas y el censo siguen corriendo hasta el corte. Una cadena que pierda cobertura queda fuera por la regla (c), sin rellenar a posteriori.

## 6. ALCANCE: esto falsa LA APUESTA EN DEFI, no la apuesta entera

- **Si se cumple, queda falsado:** que en DeFi (en los mercados observados) los agentes autónomos estén decidiendo, cada vez más, cooperar con desconocidos.
- **No queda falsado:** lo que ocurra cuando los agentes tengan **su propia infraestructura económica**: sus mercados, sus contrapartes, protocolos de agente a agente.
- **Tampoco es «todo DeFi»:** es nuestra selección (F4).
- **Frase obligatoria en cualquier publicación:** «Esta condición falsa la apuesta en DeFi, en los mercados observados. No dice nada sobre los terrenos en que los agentes tengan su propia infraestructura económica.»

## 7. Qué NO cae aunque la condición se cumpla

1. **La confianza vive en el protocolo, no en la inteligencia.** Se demostró puertas adentro (identidad y atestación FLP; la puerta de bias_audit).
2. **La auto-auditoría detecta cuándo una medición se inclina.** Se demostró contra el propio aparato y contra este mismo documento, cuya prudencia apuntaba a favor de la hipótesis (Wilson y 2/3) y se corrigió.

Ninguna de las dos depende de una contraparte externa.

## 8. La Fase 1 y la Fase 2 NO son réplicas independientes

- **Preguntan cosas distintas:** demanda de interacción bilateral entre agentes en un terreno social (Fase 1), frente a quién decide los movimientos de capital en DeFi (Fase 2).
- **Lo que comparten es más débil:** en los dos terrenos, la decisión bilateral autónoma que esperábamos es rara o no aparece.
- **No son dos confirmaciones:** un mismo sesgo nuestro puede estar detrás de las dos, y la Fase 2 depende mucho de nuestro aparato de medición.
