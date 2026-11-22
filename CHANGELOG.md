# Cambios de CIDcoder-B

Historial único de cambios, ordenado de **más reciente a más antiguo**.
Las entregas congeladas están en `DONE/`; la próxima versión de trabajo será
`_9`.

## 0.1.8 — Vector en `7776` y página 0 densa

**Fecha:** 21/Nov/2026 · **Estado:** congelada en `DONE/CIDcoder-B_8.html`

**Entrega:** `CIDcoder-B_8.html` · **771 518 bytes** ·
SHA-256 `767B5C3F69604AE33D43CCB3AAF6F027037CC2178839BDBD97C5622F33BC533E`

### Arranque

- El vector de arranque del programa compilado pasa de `7777` a **`7776`** y contiene `bodyStart`. `0000`–`0001` quedan libres para escalares; `7777` queda libre para datos.
- Se elimina el bootstrap `*0` / `VEA I 0001` / `*1`. El compilador emite una sola palabra en `*7776`.
- `loadFromResult()` usa el vector solo si coincide con `entryPoint`; si no, respeta `entryPoint` (compatibilidad con LEN-B escrito a mano). `CORE201B.reset()` lee `memory[7776]`.
- La comprobación de capacidad pasa a `arrayEnd ≤ 0o7776`.

### Página 0

- Layout denso: hasta 4 escalares en `0000`–`0007`, resto en `0020`+; `0010`–`0017` solo autoíndices y constantes de lectura directa.
- `emitScalars()` emite en la dirección asignada, no desde `0002`; se corrige el defecto de `_7` que colocaba escalares en `0010`–`0017`.
- **Regla dura:** ninguna variable accedida indirectamente puede caer en `0010`–`0017`; el hardware la incrementaría antes de leerla.
- Las constantes directas llenan las celdas sobrantes de `0010`–`0017`; no se emite basura.

### Capacidad

- 11 de los 14 programas del corpus que antes no cabían en el campo 0 ahora compilan: `EVALFX`, `INTEGRALSIMPSON`, `MANDELBROT`, `MATIXTRANSP`, `MEDIADESV`, `PLOT3D`, `POLINOMIOHORNER`, `SUMAPOSCUADNEGCONT`, `TANGENTHIPER`, `TEST_ALL` y `TOTALCOVERAGE`.
- `EVALFX` se ejecuta desde `7776` con entrada `2` y produce `+0.500`; `TANGENTHIPER` produce los cinco valores esperados.
- Siguen sin caber `CALENDAR`, `KINEMA` y `MULTMATRIX` por desbordamiento de la tabla de punteros antes de `0200`.

### Autoíndices y futuro

- No se convierten todas las rutinas: se priorizan `RT_PCA_UNPACKED`, `RT_MDI_UNPACKED`, `RT_LCA_UNPACKED` y `RT_PUTSTR` *packed*. `CIC`/`REP` no es objetivo inmediato.
- El análisis completo, con semántica, candidatos, ahorro estimado y riesgos, está en `DOC/AUTOINDICES-ANALISIS.md`.
- Los flotantes de 36 bits quedan para una versión posterior; el layout de escalares se deja parametrizado (`PAGE0_LOW`, `PAGE0_HIGH`, `PAGE0_AI`).
- La cinta sigue bloqueada hasta que se defina el formato; se mantienen 8 canales, palabra 12 bits en 8+5, enlace en canal 5, canales 6–8 vacíos y LEAL-B en ISO-8.

### Pruebas

- Suite completa contra `_8`: **299/305 OK · 6 omitidas · 0 fallos · 13/13**.
- Nueva prueba `vector de arranque: 7776 = bodyStart y 7777 libre`.
- `TEST/pruebas.html`, `TEST/lealb-helpers.js`, `TEST/lac-audit-kit.js` y `TEST/corpus.test.js` actualizados al vector `7776` y al layout denso.
- `_7` y `DONE/CIDcoder-B_7.html` no se modifican.
- **Entrega `_8` congelada** en `DONE/CIDcoder-B_8.html`, idéntica byte a byte a `CIDcoder-B_8.html`. Los próximos cambios se harán sobre un `_9`.

## 0.1.7 — `TIT` sin salto y ejemplo definitivo

**Fecha:** 19/Nov/2026

### LEAL-B

- `TIT` vuelve al comportamiento original de LEAL: emite solo el texto y el terminador CI 36, sin LF+CR. Para pasar a otra línea se usa `LIN`.
- La ayuda integrada actualiza `TIT` y `LIN`: `TIT` solo imprime el texto y `LIN` hace el salto.
- El ejemplo de bienvenida y el de la sección de ejemplos son la suma de dos números con `TIT`, `LEC`, `PER`, `LIN` y `VEA`, sin `LIN` intermedios.

### Pruebas

- `lealb.test.js`, `auditoria.test.js`, `corpus.test.js` y `TEST/README.md` ajustan las expectativas: `TIT HOLA` → `HOLA`; `TIT HOLA` + `LIN` → `HOLA\n`.
- Suite completa: **287/304 OK · 17 omitidas · 13/13 suites**, sin errores de consola.
- Verificación manual: el ejemplo por defecto compila en **545 palabras** y, con entradas `3` y `4`, imprime `A?3`, `B?4`, `A+B=+7` y el salto de `LIN`.

### Documentación

- `README.md`, `DONE/README.md`, `changelog.md`, `DOC/PROYECTO-CIDcoder-B.md`, `DOC/CODIGO-INTERNO.md` y `TEST/README.md` actualizados a 0.1.7.

### Entrega

- **Release `_7` congelada** en `DONE/CIDcoder-B_7.html`; `_1`–`_6` se conservan sin cambios.
- SHA-256: `C3E8CCD3C6EDCDA99D1C6D3A29ECB705243B4CDA37FDECCE98739E5B3CA77CE9` (767 103 bytes).

## 0.1.6 — Limpieza del motor 201-A y cinta de 8 canales

**Fecha:** 19/Nov/2026

### Limpieza

- Eliminados `CORE201`, `TT51`, `compileLen1`/`analyzeLen1` y los helpers exclusivos del motor 201-A. `RUN`/`RUNMEM` usan solo `CORE201B`/`RICOH3000`.
- El panel RT de LEAL se conecta a `CORE201B.state.memory` y muestra los badges **PAPEL** y **R3000**.
- Se conservan `resolveOp`, `splitLabel`, `.len1`/`.len` como legado y el Ricoh siempre en COMPUTER con eco software.

### Cinta perforada de 8 canales

- Restaurados el panel **Cinta** (`#tapePane`), la tira `#tapeStrip`, el estado `#tapeStatus`, el guardado `#btnSaveRib` y el botón **🎞 Tape**; el guardado usa `baseOut() + ".rib"`.
- **LEN-B:** cada palabra de 12 bits ocupa **2 marcos**: 8 bits bajos y 5 bits (4 datos altos + **enlace** en el canal 5). El enlace (bit 13) marca palabra completa para el futuro algoritmo de lectura; los canales 6–8 quedan vacíos.
- **LEAL-B:** el fuente se perfora como caracteres **ISO-8**, un marco por carácter.
- `frame8`/`frame8Char` generan los marcos; `renderTape` se llama al compilar (LEAL con `{sourceText}`, LEN con `{values}`) y se limpia en errores, `newCode` y `loadF`.
- `recomputeLayout` incluye la columna de la cinta solo si está visible y no hay ejecutor ni ayuda/About (`0.55fr` dinámica; `0.45fr` estática en LEN y `0.53fr` en LEAL).

### Interfaz

- Botones **`📄 LEN`** y **`📄 LEAL`** (sin «New»); la fila de ayuda pasa a **LEAL / LEN**.
- Ayuda y About actualizados a LEN-B/201-B y a la cinta de 8 canales.

### Pruebas

- Suite completa: **287/304 OK · 17 omitidas · 13/13 suites**, sin errores de consola.
- Verificación manual: LEN de ejemplo (8 palabras) → 16 marcos, 8 enlaces y primera palabra `7300` → `0000|0011` y `0111|1000`; LEAL de ejemplo (471 caracteres) → 477 filas y ~121.2 cm.
- Comprobado que el ejecutor, la ayuda y About ocultan la cinta, y que el layout incluye o excluye su columna correctamente.

### Documentación

- `README.md`, `DONE/README.md`, `changelog.md`, `DOC/PROYECTO-CIDcoder-B.md`, `DOC/CODIGO-INTERNO.md` y `TEST/README.md` actualizados a 0.1.6.

### Entrega

- **Release `_6` congelada** en `DONE/CIDcoder-B_6.html`; `_1`–`_5` se conservan sin cambios.
- SHA-256: `6610C865AA5D69A1F8AF18214162EF9B8BBFAE563BFEAB0AAEAA75F9A3FFBE14` (766 800 bytes).

## 0.1.5 — Eco del teletipo en software

**Fecha:** 19/Nov/2026

### Eco

- `RICOH3000` queda **siempre en modo COMPUTER**: se elimina `localEcho` y `typeAscii()` solo entrega el carácter a la CPU. El emulador histórico `CID201B_17.html` no se modifica y conserva su conmutador LOCAL/COMPUTER.
- `MONITOR_TELETIPO_201B`: `GETCHAR` conserva sus **7 palabras** y llama a `ECHO_CH` con `VAS I P_ECHO`. `ECHO_CH` (**11 palabras**, `7313`–`7325`) imprime el carácter con `TAP`, espera con `SIP` y lo devuelve en AC; el `LIL` de entrada limpia el indicador de lectura.
- Monitor: **165 → 176 palabras** (86 en página 29 y 90 en página 30), rango `7200`–`7325`; quedan **42 libres** en página 29 y **38** en página 30.
- `RT_GETCHAR` del runtime: eco con `DEL GCH`/`SUM GCH`/`TAP`/`SIP`/`SUM GCH`; pasa de **6 a 13 palabras**. No se añade `LIL` tras el eco: `TLA` conserva `IL` si quedan caracteres en la cola, igual que antes.
- Runtime: **3289/3301 → 3296 palabras UNPK / 3308 PKD** (+7 por el eco en ambos modos).
- FFD: `_placeRoutines()` reubica las rutinas por tamaño y `RT_PAGES` solo desempata; no fue necesario mover rutinas a mano.

### Pruebas

- `TEST/lealb-helpers.js`: `dynamicEcho` opcional y desactivado por defecto; el arnés ya no imprime la entrada por su cuenta.
- `monitor.test.js`: **176** palabras, distribución **86/90**, rango `7200`–`7325` y `.lstb` de 176.
- `lecper.test.js` y `lealb.test.js` incluyen el eco en las expectativas de papel; `core201b.browser.test.js` y `run201b.test.js` comprueban que teclear no imprime.
- `rutinas.test.js`: `RT_GETCHAR` de 13 palabras; totales **3296/3308**.
- Suite completa: **287/304 OK · 17 omitidas · 13/13 suites**; LEAL-B **51/51**, LEC/PER **29/29**, Rutinas **6/6**, Monitor **12/12**; corpus **20/37** con 17 omitidas.

### Documentación

- Nuevo `DOC/GUIA-ECO-TELETIPO.md` con la decisión, la implementación y la validación.
- `README.md`, `TEST/README.md`, `DOC/PROYECTO-CIDcoder-B.md` y `DOC/CODIGO-INTERNO.md` actualizados a 0.1.5.

### Entrega

- **Release `_5` congelada** en `DONE/CIDcoder-B_5.html`; `_1`, `_2`, `_3` y `_4` se conservan sin cambios.
- SHA-256: `46222F81280A627D7D1672C28C23CFA3E282869F632D4D272D845443219D5719` (829 347 bytes).
- Verificado sin errores de consola y con el `.lstb` de 176 palabras cargado en `CID201B_17.html`.

## 0.1.4 — Optimización de los monitores 201-B

**Fecha:** 19/Nov/2026

### Monitores

- `MONITOR_TELETIPO_201B`: **205 → 165 palabras** (−40, −19,5 %). Página 29 de 104 a **75** (53 libres en `7313`–`7377`) y página 30 de 101 a **90** (38 libres en `7532`–`7577`).
  - `READ_OCT4` de 37 a 20 palabras y 10 menos de constantes: decodificación aritmética de los CI contiguos `060`–`067`, sin `TAB_DIG`, `TD1`, `PTR2` ni `K_TABD`.
  - Despachador de 20 a 12: comparación por diferencias `K_AR` = `7757` (`K_A-K_R`) y `K_AC` = `0002` (`K_C-K_A`), sin `TMP` ni recargas.
  - `PRINT_DIG` de 8 a 6: recibe el dígito 0–7 en AC y conserva solo `SUM K_DIG0` + `TAP`.
  - `CMD_R` de 13 a 11 y `r_range` de 22 a 20 (`ISC DIR` + `NOP`).
- `MONITOR_Ultima_Pagina_201B`: **128 → 107 palabras** (−21, −16,4 %). Código hasta `7746`, cola fija `7774`–`7777` intacta y **21 libres** en `7747`–`7773`.
  - `display` de 32 a 22: cuatro `TAR` de ceros, `RD9`/`RD6`/`RD3` y una sola máscara `K_7`; se eliminan `mask2`, `mask3` y `mask4`.
  - `validar` de 11 a 8: sin bloque `invalido` separado ni `LAC` previo a `TRA`.
  - `input`, `RECupera`+`saveAdr`, `readAdr`, `ALMacena`+`getAdd` y `write` bajan una palabra cada uno con `ISC` y `LAC` redundantes eliminados.
- Total de los dos monitores: **333 → 272 palabras** (−61, −18,3 %); **112 libres** en las páginas 29–31.
- Comparación A/B: **19** casos del teletipo y **10** secuencias del CODEC, todos idénticos en papel, RDP, variables y memoria. El teletipo añade un caso con `8` y `9` intercalados en comando y dirección.
- La comparación con la 201-A queda documentada: teletipo 227 → 165 (−27,3 %); Última Página 85 → 107 (+25,9 %) por el RDP serie y la validación octal.

### Pruebas

- Fixtures `TEST/MONITOR_TELETIPO_201B.lenb` y `TEST/MONITOR_Ultima_Pagina_201B.lenb` sustituidas por las versiones optimizadas; ambos `.lstb` regenerados palabra por palabra.
- `monitor.test.js`: 205→165 palabras; distribución 104/101→**75/90**; rangos `7200`–`7312` y `7400`–`7531`; constantes CI reubicadas; `.lstb` 165; el caso de caracteres ignorados incluye `8` y `9`.
- `lenb.test.js`: monitor de **107** palabras exactas.
- `run201b.test.js`: **107** palabras, entrada `7600`, primera `7140` y última `7600`.
- `TEST/pruebas.html` apunta a `CIDcoder-B_4.html`.
- Suites afectadas: `monitor` **12/12**, `lenb` **7/7**, `run201b` **7/7**. Total: **287/304 OK · 17 omitidas · 13/13 suites**.

### Documentación

- Nuevo `DOC/ANALISIS-OPTIMIZACION-MONITORES.md`: desglose por rutina, comparación 201-A, mecánica de los campos (`CCDN`/`CCIN`, buzón, `RCI`/`RCD`), presupuesto y opciones O0–O5.
- `README.md`, `TEST/README.md`, `DONE/README.md`, `CODE/LEN CID201B/MONITOR_TELETIPO_201B_README.md` y `CODE/LEN CID201B/MONITOR_Ultima_Pagina_201B_README.md` actualizados.
- El README de Última Página se reescribió: ya no describe `.len1` ni un monitor de 128 palabras, e incluye el aviso de que el `VEA end` de `validar` es el retorno de `display`.

### Entrega

- **Release `_4` congelada** en `DONE/CIDcoder-B_4.html`; `_1`, `_2` y `_3` se conservan sin cambios.
- SHA-256: `CC8E4020921EA8ECBD51D547678E375998F6F1EFFFF4358E51C4EC9826646852` (828 614 bytes).
- Verificado sin errores de consola: página de trabajo, release de `DONE/` y ejecutor unificado.

### Diferido

- **Gestión de los ocho campos (0.2.0):** decidir el selector del CODEC, el comando `F` del teletipo, la política del buzón y el alcance de `C`. El informe recomienda un núcleo compartido en las páginas 29/30, con el teletipo como hospedador y Última Página como cliente.
- **Dataflow global entre etiquetas:** sigue pendiente para eliminar los `LAC` conservados en `CIC`.
- **Capacidad del campo 0:** los 14 programas del corpus que no caben siguen documentados.

## 0.1.3 — Auditoría LAC del compilador y del cuerpo

**Fecha:** 19/Nov/2026

### Auditoría del runtime

- Pasada completa sobre los **66 emisores** del runtime, en UNPK y PKD, con `TEST/lac-audit-kit.js` y el analizador de flujo `__LAC`.
- 19 rutinas reducidas: `RT_PER_INT` 36→34, `RT_PER_CORE` 77→69, `RT_PER_DIG` 78→72, `RT_PER_DIGITS` 79→77, `RT_PER_FLT` 122→118, `RT_PER_DISPATCH` 15→14, `RT_LIN` 12→11, `RT_NEG` 18→17, `RT_SUBT` 40→39, `RT_NORM24` 35→34, `RT_UNPACK_FLT` 38→37, `RT_NOR` 28→27, `RT_LEC_FF` 52→51, `RT_LCA_UNPACKED` 39→38, `RT_LCA_PACKED` 87→86, `RT_MUL_INT` 109→104, `RT_PEN` 83→82, `RT_F2I` 47→46 y `RT_FDIV` 61→60.
- Runtime: **3329/3341 → 3289 palabras UNPK / 3301 PKD** (40 menos en cada modo).
- Patrones eliminados: `LAC` tras `DEL`/`TAP`, tras `VAS I` de rutinas con retorno `AC=0`, guardas de entrada muertas cuando la primera instrucción ya escribe `AC`, y segundas ramas de condicionales con cero probado por `SAC`/`SAD`.
- Conservados: `LAC` antes de `DEL` (almacena el cero), `LAC LEN` (funcional), retornos con `AC≠0` (`RT_EAB`, `RT_GETCHAR`, `RT_NORM24`, `RT_PER_INT`, `RT_PER_DIGITS`, `RT_EPERIF`) y etiquetas con llegadas mixtas.
- Emisor: nuevo estado `_acAfter` (`0` cero, `1` distinto de cero, `2` desconocido), actualizado en cada `emit`; `emitL` lo reinicia a desconocido. `LEAL_RT_AC_ZERO` reúne 58 rutinas con retorno cero verificado con `__LAC.exits`.

### Auditoría del cuerpo

- Revisados stubs de función, `_parmSetVal`, `_parmSetConst`, `_emitStringShift`, `condJump`, `repT1`, `repT2F`, `repT3F`, `repT3Int`, `dig`, `esp`, `sep`, `emitVas`, `loadVar`/`storeVar`/`loadItemInto`/`intoT`/`intoR`/`intoParm`/`loadParm`/`binop`/`intoParmF`/`_intoParmFloatAuto`, `FAB`, `PEN`, `FRA` y el stub de `evalT`.
- Eliminados los `LAC` que seguían a `DEL` (incluido el segundo `LAC` del stub tras `DEL PARM+2`), a rutinas con retorno cero y a tramos de condicional ya probados.
- No se tocó `startCic` ni los `LAC` de etiquetas de `CIC` (`L1`, `L2`, …) y joins con `AC=0` local: sin dataflow global entre etiquetas, borrarlos podría cambiar el comportamiento. Queda documentado como patrón pendiente.
- `lac-audit-kit.js` ampliado con `KEEP_DEL`, `LAC LEN`, `LEN`, `flowOf` con mapa `jptrMap`, `lacRows`, `exitStates`, `bodyLines` y `__LAC.body`/`__LAC.exits`. El mapa `JPTR_n → símbolo` elimina falsos `DEAD_ZERO` en etiquetas alcanzadas por punteros.

### Pruebas

- Nuevos casos en `lealb.test.js`: tabla completa de condiciones enteras, `VEA (n)`, `REP` tipos 1/2/3 y anidado, asignaciones encadenadas, subrutinas (dos entradas, doble llamada, etiqueta `FRU`, espacios, constantes y anidadas) y `TITULO` en declaraciones.
- Nuevos casos en `flotantes.test.js`: tabla de condiciones flotantes, `REP` flotante (lista, rangos y arranque), expresiones y condiciones mixtas, cadenas flotantes y `LNE`/`LOG` de negativos.
- Nuevos casos en `arreglos.test.js`: índice anidado `X(I(1))` y Facilidad Común `X(4)=Y(0)` con aviso.
- Nuevos casos en `auditoria.test.js`: errores de asignación encadenada (`2+C` y `==`).
- `rutinas.test.js`: tallas nuevas y totales 3289/3301.
- `TEST/pruebas.html` apunta a `../CIDcoder-B_3.html`.
- Suites propias: `lealb` 51/51, `div` 21/21, `lecper` 29/29, `flotantes` 57/57, `arreglos` 24/24 y `auditoria` 21/21: **203/203**.
- Suite completa: **287/304 pruebas OK · 17 omitidas · 13/13 suites**.
- Las expectativas nuevas se capturaron en la referencia congelada `DONE/CIDcoder-B_2.html` y se confirmaron idénticas en 0.1.3.

### Documentación

- `DOC/GUIA-AUDITORIA-LAC.md`: nueva sección 9 con alcance, contrato de `AC`, patrones eliminados y conservados, resultados por rutina, auditoría del cuerpo y límite del análisis local.
- `DONE/README.md` y `changelog.md` actualizados con la entrega `_3`.

### Entrega

- **Release `_3` congelada** en `DONE/CIDcoder-B_3.html`; `_1` y `_2` se conservan sin cambios.
- SHA-256: `05BC516DA2480233A4097A7DF6D49A062069B3D688FC7A5DB22D728C0559654B` (828 614 bytes).
- Verificado sin errores de consola: página de trabajo, release de `DONE/` y ejecutor unificado.

### Diferido

- **Dataflow global entre etiquetas:** permitiría eliminar los `LAC` que hoy se conservan en etiquetas de `CIC` y joins con llegadas indirectas. Requiere que el emisor propague el estado de `AC` por todas las aristas, no solo en línea recta.
- **Monitor de toda la memoria:** versión para los ocho campos y versión del «Última Página» para toda la memoria.
- **Capacidad del campo 0:** los 14 programas del corpus que no caben siguen documentados.

## 0.1.2 — Monitor de teletipo 201-B y auditoría LAC

**Fecha:** 18/Nov/2026

### Monitor de teletipo

- `MONITOR_TELETIPO_201B`: port de `MONITOR_TELETIPO_v2.1` (201-A) al campo 0 de la 201-B. Ocupa **205 palabras**: 104 en la página 29 (`7200`–`7347`) y 101 en la página 30 (`7400`–`7544`); entrada `7400`; página 31 libre para el monitor CODEC y los vectores de interrupción.
- Comandos `R dddd`, `R dddd dddd`, `A dddd vvvv` y `C dddd`; direcciones y valores octales de 4 dígitos; terminación con `Enter` (CR); caracteres no octales ignorados en silencio.
- E/S directa por CI: `R`=`022`, `A`=`001`, `C`=`003`, `?`=`077`, `:`=`072`, CR=`036`, LF=`037`, dígitos `060`–`067`. Sin shifts Murray/ITA2.
- Puentes indirectos de una palabra (página + offset) en lugar de los punteros de dos palabras de la 201-A. `C` usa `VEA I DIR`; `R`/`A` usan `SUM I PTR`/`DEL I PTR`.
- `R dddd` usa una sola dirección; `A` confirma con `dddd:vvvv`; `C` imprime la dirección y salta; el retorno al prompt deja una línea en blanco.
- Listado `MONITOR_TELETIPO_201B.lstb` de **205 palabras**, cargable en `CID201B_17.html`.
- Etiquetas globales entre páginas: `K_CR2`, `K_LF2`, `P_GETC2`, `TMP2` y `PTR2` evitan duplicar nombres de la página 30.

### Auditoría LAC

- Pasada completa sobre el monitor: **33 LAC → 10**; **23 eliminados** (10 en la página 30 y 13 en la página 29).
- Primera pasada (21 eliminados): tras `DEL`/`DEL I` (AC=0), tras `TAP` (AC=0), tras `VAS I` a rutinas con contrato AC=0 (`PRINT_DIG`, `PRINT_CRLF`, `PRINT_OCT4`), en etiquetas con todas las entradas AC=0 (`DISP`, `r_cr`, `ro_find`, `ro_ok`) y tras el bucle `rs_w` que sigue a un `TAP`.
- Segunda pasada (2 eliminados): la guarda de entrada de `READ_OCT4`, porque la primera instrucción (`DEL ACC`) ya limpia AC, y el `LAC` posterior a `SAC` en `ro_find`, porque `SAC` salta precisamente cuando AC=0.
- Conservados: entrada de `START` (alimenta a `DISP`), dos comparaciones de `DISP_LP` tras `SAD`, entrada de `r_range` y su `LAC` tras `SAD`, `CMD_A`/`CMD_C` tras `VAS I P_GETC` (contrato de `GETCHAR`), `GETCHAR` antes de `TLA` (que hace OR), entrada de `PRINT_CRLF` (alimenta a `SUM K_CR2`) y `ro_next` (llegadas mixtas).
- Sin cambios de comportamiento: mismas salidas `R`/`A`/`C`, suite **12/12** y `.lstb` con **0 diferencias**.
- Guía de referencia: `DOC/GUIA-AUDITORIA-LAC.md`.

### Pruebas y documentación

- Nueva suite `TEST/monitor.test.js`: **12/12** pruebas de compilación, distribución 104/101, constantes CI, paridad `.lstb`, `R` simple y por rango, `A`, `C` a `ALT` con alto `AP`, caracteres ignorados, cruce de página 0→1 y retorno a `7400`.
- `TEST/pruebas.html`: **13 suites**, una pestaña por suite y resumen global.
- **224/241 pruebas OK · 17 omitidas · 13/13 suites.**
- `.lstb` cargado en `CID201B_17.html`: **205 palabras, 0 duplicadas, 0 ignoradas y 0 diferencias**.
- `README.md`, `TEST/README.md`, `DONE/README.md`, `changelog.md`, `DOC/PROYECTO-CIDcoder-B.md`, `DOC/CODIGO-INTERNO.md`, `DOC/GUIA-AUDITORIA-LAC.md` y `CODE/LEN CID201B/MONITOR_TELETIPO_201B_README.md` actualizados.

### Entrega

- **Release `_2` congelada** en `DONE/CIDcoder-B_2.html`; `_1` se conserva sin cambios.
- SHA-256: `638CD94CA395DE1482433A71D32DD331DB2CA5ACA31BAC8CBBC2E9A5C03D436C` (823 844 bytes).
- Verificado sin errores de consola: la página de trabajo, la release de `DONE/` y el ejecutor unificado.
- **Numeración corregida:** la serie del proyecto es `0.1.x`; `_1` conserva el rótulo interno `0.2.1` por estar congelada.

### Diferido

- **Monitor de toda la memoria:** versión para los ocho campos y versión del «Última Página» de la consola CODEC para toda la memoria.
- **Autoíndices:** optimización aplazada hasta medir el runtime real.
- **Capacidad del campo 0:** los 14 programas del corpus que no caben quedan documentados; no se reduce el runtime para forzarlos.

## 0.1.1 — LEAL-B, paridad y entrega de pruebas

> **Nota de numeración:** el archivo `DONE/CIDcoder-B_1.html` conserva el
> rótulo interno `0.2.1` de la primera numeración. La serie correcta del
> proyecto es `0.1.x`; esta entrada documenta la entrega `_1` como `0.1.1`.

**Fecha:** 18/Nov/2026

### Núcleo y teletipo

- `CORE201B`: memoria de 32 768 palabras (8 campos × 4096), CP de 15 bits, `RCI`/`RCD`, autoíndices `0010`–`0017`, puente indirecto de 12 bits, ciclo CI/CM/CD/CO/CV, IRM, microinstrucciones, rotaciones, saltos, interrupciones, CODEC y E/S.
- `RICOH3000`: papel de 60 columnas, eco local, `Enter` = CR (13), siempre en modo computadora, sin teclado visible.
- `CIDB`: utilidades de la fase 0, `selfTest()` y `selfTestLenB()`.

### Ensamblador LEN-B

- `LENB` / `compileLenB()`: IRM, micros del grupo 0 y 1, saltos, rotaciones, `PUE`/`POE`, `CCIN`/`CCDN`, `CIN`/`DIN`/`VPP`/`LCI`/`LCD`, E/S de la 201-B, compuestos `SIF LIF`/`SIE LIE`/`SITE LITE`, cadenas CI, datos, etiquetas, `*ORG`, `$`, decimal/hexadecimal, etiqueta+offset, rango estricto de 12 bits y detección de solapamientos.
- Codificaciones canónicas: `NOP` = `7200`, `LAC` = `7300`, `LEN` = `7440`, `CEN` = `7420`, `ALT` = `7201`, `LAC LEN` = `7140`, `RI3` = `7503`, `RD3` = `7403`.
- Listado `.lstb` compatible con el cargador de `CID201B_17.html`.
- Editor: modo LEN-B con resaltado propio, estimación de palabras, mensajes de ensamblado y guardado `.lenb`/`.lstb`.

### Compilador LEAL-B

- `LEALCompiler` con destino `lenb`: enteros, flotantes, arreglos, cadenas CI, ciclos `CIC`/`REP`, subrutinas `VAS`/`CRU`/`FRU`, todas las funciones y condiciones, `;` y `TIT`.
- `PER` sin relleno por la izquierda: entero = signo + dígitos + 2 espacios; flotante = signo + `Entero.Fracción` + 3 espacios. Los positivos llevan `+`; el cero no lleva signo.
- `TIT` emite texto + LF + CR antes del terminador CI 36; `LIN` emite CR + LF. La Ricoh cuenta el par como un solo salto visible.
- `LCA`, `PCA`, `MDI`, `MID`, `ESP`, `LEC`, `PER`, `SEP` y `DIG`. `DIG 6,3` implícito; `DIG m` se rechaza con `E:DIGEXP`; otro periférico en `SEP` produce `E:PERIF`.
- `LEC` entero de 24 bits; flotante con magnitud menor que 2²³. Entradas mayores o iguales que `8388608` producen `E:DESB`.
- Layout de campo 0: escalares desde `0020`, autoíndices en `0010`–`0017`, runtime desde `0200`. La tabla de punteros debe caber antes de `0200`; si no, se informa **`Página 0 desborda`** con la causa, en lugar de un solapamiento interno.
- Post-paso que convierte `VEA`/`VAS` indirectos por `JPTR` en directos cuando el destino está en página 0 o en la misma página.
- Runtime medido con `LEALCompiler._C` y `_measureLines()`: **3329 palabras UNPK / 3341 PKD**.

### Ejecutor

- LEAL-B compila a LEN-B y se ejecuta en el **campo 0** sobre `CORE201B`/`RICOH3000`; el visor permite inspeccionar los 8 campos.
- `CONT`/`INSTR`, CP de 15 bits, breakpoints, traza y visor de memoria de 8 campos × 32 páginas.
- Teclado de papel del Ricoh: `PAPEL:ACT`/`PAPEL:INAC`, `Enter` → CR, aviso de caracteres no válidos.
- Panel de cinta ITA2 oculto en LEN-B.

### Archivos y ayudas

- Guardado/carga `.lenb`, `.lstb` y `.leal`; `.len1` se acepta como texto legado con aviso.
- Ayuda integrada actualizada para `+`/`-` de `PER`, `TIT` LF+CR, límites de `LEC`, `DIG`, `SEP` y layout de página 0.
- `DOC/CODIGO-INTERNO.md` ampliado con el CI, su uso por LEAL y la 201-B, vectores de prueba y puntos de revisión 12.8–12.13.

### Pruebas y documentación

- `TEST/pruebas.html`: ejecutor unificado con **12 suites**, una pestaña por suite, resumen global, repetición por suite/todo y enlace `#suite`.
- **212/229 pruebas OK · 17 omitidas · 12/12 suites.**
- `TEST/rutinas.test.js`: 6/6 con tamaños reales de 66 rutinas.
- `TEST/corpus.test.js`: 20 programas con paridad semántica contra `CODE/LEAL`; 14 omitidos por capacidad del campo 0 y 3 por constantes ITA2 del 201-A.
- Monitor `MONITOR_Ultima_Pagina_201B`: **128/128** palabras exactas; `.lstb` cargado en `CID201B_17.html` con **0 diferencias**.
- `README.md`, `TEST/README.md`, `changelog.md` y `DOC/PROYECTO-CIDcoder-B.md` actualizados.

### Entrega

- **Release `_1` congelada** en `DONE/CIDcoder-B_1.html`, idéntica byte a byte al archivo de trabajo.
- SHA-256: `D0D4106FF0F730A7E70E774C2A6660D89EF3BECFEFCED87B4EB79438DDCF4CCD` (823 844 bytes).
- `DONE/README.md` con el índice de la entrega, contenido, limitaciones y pasos de verificación.
- Verificado sin errores de consola: la página de trabajo, la release de `DONE/` y el ejecutor unificado.

### Diferido

- **`MONITOR_TELETIPO_201B`:** portar `MONITOR_TELETIPO_v2.1.len1` al campo 0, con los comandos `R`, `A` y `C`. Se hará cuando el usuario lo apruebe.
- **Monitor de toda la memoria:** versión para los 8 campos y versión del «Última Página» de la consola CODEC.
- **Autoíndices:** optimización aplazada hasta medir el runtime real.
- **Capacidad del campo 0:** los 14 programas del corpus que no caben quedan documentados; no se reduce el runtime para forzarlos.

## 0.1.0 — fase 0

**Fecha:** 17/Nov/2026

- Copia de trabajo `CIDcoder-B_1.html` a partir de `LEN-1ASSMB_52.html`.
- Marca, modos, extensiones `.lenb`/`.lstb` y textos iniciales de la 201-B.
- `CORE201B`, `RICOH3000` y utilidades `CIDB`.
- Documento de proyecto con decisiones y plan por fases.
