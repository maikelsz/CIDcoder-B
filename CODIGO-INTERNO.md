# El Código Interno (CI) de la CID 201-B

**Documento de estudio, implementación y revisión**

| Dato | Valor |
|---|---|
| Proyecto | CIDcoder-B |
| Versión del emulador | 0.1.6 |
| Fecha | 19 de noviembre de 2026 |
| Estado | Borrador para revisar y corregir |
| Ámbito | El Código Interno de 6 bits, su uso por LEAL-B y por la CID 201-B |
| Implementación | `CIDcoder-B_6.html` |
| Fuentes históricas | `LEAL/CODIGOINTERNO.md`, `CID201B/DOChistorico/Consul260/CODIGOINTERNO.md` |
| Autoridad de comportamiento | `CID201B_17.html` (núcleo 201-B) |

> **Propósito.** Este documento explica qué es el Código Interno (CI), cómo lo usa
> LEAL, cómo lo usa la CID 201-B y qué decisiones tomó CIDcoder-B al respecto.
> No es un manual de usuario: es un documento de trabajo para contrastar la
> implementación con las fuentes históricas. Al final hay una lista explícita de
> puntos que deben revisarse o corregirse.

---

## 0. Cómo leer este documento

El documento distingue tres planos que suelen mezclarse:

1. **Lo histórico**: lo que dicen las tablas y manuales de la CID 201-B y del
   teletipo Consul 260. Se cita como “tabla histórica” o “fuente histórica”.
2. **Lo normativo del emulador**: lo que `CID201B_17.html` implementa y que
   CIDcoder-B adopta como autoridad de comportamiento.
3. **Lo decidido en CIDcoder-B**: extensiones o convenciones de LEAL-B que no
   necesariamente existieron en el hardware original.

Cuando los tres planos no coinciden, se dice explícitamente. Los puntos abiertos
están en la sección 12.

---

## 1. Resumen ejecutivo

El **Código Interno (CI)** es el alfabeto interno de la CID 201-B. Es un código
de **6 bits** con **64 valores posibles** (0–63). Cada valor representa una letra,
un dígito, un signo o un código de control. La computadora no guarda ASCII:
guarda CI.

Como la palabra de la 201-B es de **12 bits**, en una sola palabra caben **dos
caracteres CI**: 6 bits para el primero y 6 bits para el segundo. Eso permite
almacenar texto de forma compacta y es la razón principal de que el CI tenga 6
bits y no 8.

LEAL-B usa el CI para todo el texto:

- `TIT` convierte el texto literal a CI y lo imprime.
- `LIN` emite CR y LF como CI 30 y CI 31.
- `LCA` lee caracteres CI hasta CR y los guarda en un buffer.
- `PCA` recorre el buffer hasta el terminador CI 36 y los imprime.
- `MDI` y `MID` desplazan los caracteres dentro del buffer.
- `ESP` imprime espacios, que son CI 0.
- `PER` no imprime texto libre, pero sí usa CI para el signo, los dígitos, el
  punto decimal y el espacio final.

La CID 201-B tiene instrucciones específicas para mover CI entre la memoria y
los periféricos:

- `TLA` toma un carácter CI del teletipo y lo OR-ea en el acumulador.
- `TAP` toma los 6 bits bajos del acumulador, los imprime y limpia el
  acumulador.
- `SIL`, `LIL`, `SIP`, `LIP` y sus variantes controlan los indicadores de
  entrada y salida.

En CIDcoder-B el teletipo es una **Ricoh 3000 de papel**, sin teclado visible,
siempre en modo computadora. El usuario escribe sobre el papel y cada tecla se
convierte de ASCII a CI en la frontera de entrada; al imprimir, el CI se
convierte de vuelta a ASCII para dibujar el carácter en el papel.

La implementación actual sigue la tabla `CI_TO_ASCII` de `CID201B_17.html`.
Esa tabla coincide con la tabla histórica en casi todos los caracteres
imprimibles, pero difiere en cinco puntos importantes: CI 27, CI 29–31, CI 32,
CI 35 y el tratamiento de CI 36. Esos puntos están documentados y marcados para
revisión en la sección 12.

---

## 2. Qué es el Código Interno

### 2.1 Definición

El CI es un código de carácter de **6 bits**. Sus valores se escriben
indistintamente en decimal (0–63) o en octal (00–77). En este documento se usa
la forma decimal para el valor lógico y la octal para el valor de máquina,
porque el ensamblador LEN-B y los listados de la 201-B trabajan en octal.

| Propiedad | Valor |
|---|---|
| Ancho | 6 bits |
| Valores | 0–63 |
| Representación en memoria | Dos CI por palabra de 12 bits |
| Terminador de texto | CI 36 (octal 44) |
| Espacio | CI 0 |
| CR | CI 30 (octal 36) |
| LF | CI 31 (octal 37) |

El CI no es un código de 8 bits. No hay bit de paridad dentro del CI. Tampoco
hay una tabla de códigos “mayúsculas” y “minúsculas” simultáneas: el juego
principal, llamado **Código A**, cubre mayúsculas, dígitos y puntuación. El
**Código B** es un juego alternativo que se describe en la sección 4.

### 2.2 Por qué 6 bits

La palabra de la 201-B es de 12 bits. Con 6 bits por carácter se guardan dos
caracteres por palabra, sin desperdiciar bits. Con 8 bits solo cabría un
carácter y medio; con 5 bits (como Murray) cabrían dos y sobrarían dos bits,
pero el repertorio de 32 símbolos es demasiado pequeño para mayúsculas, dígitos
y puntuación sin cambios de registro.

El CI de 6 bits es, por tanto, un compromiso de la arquitectura:

- 64 símbolos alcanzan para 26 letras + 10 dígitos + puntuación + controles.
- Dos símbolos por palabra duplican la densidad de texto.
- El costo es que no hay minúsculas en el juego principal y que el repertorio
  es menor que ASCII.

### 2.3 Lo que el CI no es

| Código | Ancho | Valores | Para qué |
|---|---|---|---|
| **CI** | 6 bits | 0–63 | Alfabeto interno de la 201-B |
| **ISO** | 8 bits | 000–377 octal | Intercambio con cintas y otros sistemas |
| **Murray / MTK-2** | 5 bits | 00–37 octal | Teletipo físico, con cambios LTRS/FIGS |
| **ASCII** | 7/8 bits | 0–127 / 0–255 | Terminal moderna del emulador |

El CI **no** es ASCII. La letra `A` es CI 1, no 65. El dígito `0` es CI 48, no
48 decimal de ASCII (aunque casualmente coincida en este caso, no en todos).
El CI **no** es ISO: la columna ISO de las tablas históricas es un código de
8 bits distinto. El CI **no** es Murray: Murray es el código de 5 bits que
entiende el teletipo, y necesita cambios de registro.

### 2.4 El nombre “CI” y su homónimo

En el núcleo de la 201-B, `state.cycle` puede valer `CI`, `CM`, `CD`, `CO` o
`CV`. Esos valores son **ciclos de máquina**, no códigos de carácter:

| Valor | Significado |
|---|---|
| `CI` | Ciclo de Instrucción |
| `CM` | Ciclo de Microinstrucción |
| `CD` | Ciclo de Dirección (indirección) |
| `CO` | Ciclo de Operando |
| `CV` | Ciclo de Salto (`VEA`) |

El nombre `CI` del ciclo de instrucción y el nombre `CI` del Código Interno son
homónimos. En el código conviene no mezclarlos: cuando se hable de carácter,
escribir `CI` o `codigoInterno`; cuando se hable de ciclo, escribir
`CICLO_INSTR` o `cycle: 'CI'` con un comentario. La sección 12.12 propone el
cambio.

---

## 3. La tabla del Código A

### 3.1 Tabla completa

La siguiente tabla combina la transcripción histórica con la implementación
actual. Las columnas son:

- **CI**: valor decimal.
- **Oct**: valor octal.
- **Carácter histórico**: lo que dice la tabla transcrita.
- **ISO**: columna ISO de la tabla histórica, en octal de 3 cifras.
- **Murray**: columna Murray/MTK-2 de la tabla histórica, en octal de 2 cifras.
- **Implementación**: lo que `CI_TO_ASCII` devuelve hoy, con su valor ASCII.

| CI | Oct | Carácter histórico | ISO | Murray | Implementación |
|---:|---:|---|---|---|---|
| 0 | 00 | SP | 240 | 04 | ` ` (32) |
| 1 | 01 | A | 101 | 30(L) | `A` (65) |
| 2 | 02 | B | 102 | 23(L) | `B` (66) |
| 3 | 03 | C | 303 | 16(L) | `C` (67) |
| 4 | 04 | D | 304 | 22(L) | `D` (68) |
| 5 | 05 | E | 305 | 20(L) | `E` (69) |
| 6 | 06 | F | 306 | 26(L) | `F` (70) |
| 7 | 07 | G | 107 | 13(L) | `G` (71) |
| 8 | 10 | H | 110 | 05(L) | `H` (72) |
| 9 | 11 | I | 311 | 14(L) | `I` (73) |
| 10 | 12 | J | 312 | 32(L) | `J` (74) |
| 11 | 13 | K | 113 | 36(L) | `K` (75) |
| 12 | 14 | L | 314 | 11(L) | `L` (76) |
| 13 | 15 | M | 115 | 07(L) | `M` (77) |
| 14 | 16 | N | 116 | 06(L) | `N` (78) |
| 15 | 17 | O | 317 | 03(L) | `O` (79) |
| 16 | 20 | P | 120 | 15(L) | `P` (80) |
| 17 | 21 | Q | 321 | 35(L) | `Q` (81) |
| 18 | 22 | R | 322 | 12(L) | `R` (82) |
| 19 | 23 | S | 123 | 24(L) | `S` (83) |
| 20 | 24 | T | 324 | 01(L) | `T` (84) |
| 21 | 25 | U | 125 | 34(L) | `U` (85) |
| 22 | 26 | V | 126 | 17(L) | `V` (86) |
| 23 | 27 | W | 327 | 31(L) | `W` (87) |
| 24 | 30 | X | 330 | 27(L) | `X` (88) |
| 25 | 31 | Y | 131 | 25(L) | `Y` (89) |
| 26 | 32 | Z | 132 | 21(L) | `Z` (90) |
| 27 | 33 | £ | 333 | — | `$` (36) |
| 28 | 34 | TAB | 011 | 04(sp) | `TAB` (9) |
| 29 | 35 | Cambio A | — | — | `NUL` (0) |
| 30 | 36 | Cambio B | — | 02(cr) | `CR` (13) |
| 31 | 37 | Línea | 215 | 10(lf) | `LF` (10) |
| 32 | 40 | - | 137 | — | `_` (95) |
| 33 | 41 | @ | 300 | — | `@` (64) |
| 34 | 42 | " | 042 | — | `"` (34) |
| 35 | 43 | 1/2 | 243 | — | `«` (171) |
| 36 | 44 | Fin de texto | 003 | 32(f) | `NUL` (0) |
| 37 | 45 | % | 245 | — | `%` (37) |
| 38 | 46 | $ | 246 | — | `$` (36) |
| 39 | 47 | ' | 047 | 24(f) | `'` (39) |
| 40 | 50 | ( | 050 | 36(f) | `(` (40) |
| 41 | 51 | ) | 251 | 11(f) | `)` (41) |
| 42 | 52 | * | 252 | 22(f) | `*` (42) |
| 43 | 53 | + | 053 | 21(f) | `+` (43) |
| 44 | 54 | , | 254, 336 | 06(f) | `,` (44) |
| 45 | 55 | - | 055 | 30(f) | `-` (45) |
| 46 | 56 | . | 056, 335 | 07(f) | `.` (46) |
| 47 | 57 | / | 257 | 27(f) | `/` (47) |
| 48 | 60 | 0 | 060 | 15(f) | `0` (48) |
| 49 | 61 | 1 | 261 | 35(f) | `1` (49) |
| 50 | 62 | 2 | 262 | 31(f) | `2` (50) |
| 51 | 63 | 3 | 063 | 20(f) | `3` (51) |
| 52 | 64 | 4 | 264 | 12(f) | `4` (52) |
| 53 | 65 | 5 | 065 | 01(f) | `5` (53) |
| 54 | 66 | 6 | 066 | 25(f) | `6` (54) |
| 55 | 67 | 7 | 267 | 34(f) | `7` (55) |
| 56 | 70 | 8 | 270 | 14(f) | `8` (56) |
| 57 | 71 | 9 | 071 | 03(f) | `9` (57) |
| 58 | 72 | : | 072 | 16(f) | `:` (58) |
| 59 | 73 | ; | 273 | — | `;` (59) |
| 60 | 74 | $ (variante) | 044 | — | `$` (36) |
| 61 | 75 | = | 275 | 17(f) | `=` (61) |
| 62 | 76 | ! | 276 | — | `!` (33) |
| 63 | 77 | ? | 077 | 23(f) | `?` (63) |

**Advertencia sobre la tabla histórica.** La tabla anterior es una
transcripción de Tabla 1 de `CODIGOINTERNO.md`. Las tablas originales son
escaneos; la transcripción pudo confundir glifos parecidos. La columna ISO, en
particular, no es ASCII estándar: muchos valores tienen el bit 8 puesto
(por ejemplo, `C` = 303 octal = 195 decimal = 67 + 128). Eso puede ser paridad,
una variante de código o un error de transcripción. No use la columna ISO como
ASCII sin verificarla contra el manual original.

### 3.2 Lectura de la tabla

La tabla no es arbitraria. Tiene una estructura clara:

| Rango | Contenido | Comentario |
|---|---|---|
| 0 | Espacio | Único símbolo de separación |
| 1–26 | A–Z | Orden alfabético, 26 letras |
| 27 | £ / moneda | Punto dudoso (ver 12.1) |
| 28 | TAB | Tabulador |
| 29–31 | Controles | Cambio A, Cambio B, Línea (ver 12.2) |
| 32–47 | Puntuación | `_ @ " ½ END % $ ' ( ) * + , - . /` |
| 48–57 | Dígitos 0–9 | Orden consecutivo, fácil de convertir |
| 58–63 | Puntuación final | `: ; $ = ! ?` |

Los dígitos ocupan el rango 48–57. Eso es cómodo para el compilador: para
imprimir un dígito, se suma el valor del dígito a CI 48. El runtime de `PER`
hace exactamente eso: `CI 0o60 + dígito`.

### 3.3 Los códigos de control 28–31

La tabla histórica dice:

| CI | Etiqueta histórica | Murray histórico | Lectura actual |
|---:|---|---|---|
| 28 | TAB | 04(sp) | TAB (ASCII 9) |
| 29 | Cambio A | — | NUL (ASCII 0) |
| 30 | Cambio B | 02(cr) | CR (ASCII 13) |
| 31 | Línea | 10(lf) | LF (ASCII 10) |

Hay dos lecturas posibles:

1. **Lectura “teletipo”**: 29 y 30 son los códigos de cambio de registro
   (letras/figuras) del teletipo, y 31 es avance de línea. El teletipo los
   interpreta como controles, no como caracteres imprimibles.
2. **Lectura “201-B”**: 29 no se usa, 30 es retorno de carro y 31 es salto de
   línea. Es la lectura que implementa `CID201B_17.html` y la que usa
   CIDcoder-B.

La anotación Murray de la tabla (`02(cr)` para 30 y `10(lf)` para 31) apoya la
segunda lectura: 30 produce CR y 31 produce LF en el teletipo. Pero la etiqueta
“Cambio A/Cambio B” apoya la primera. Es un punto que debe resolverse con el
manual original del Consul 260 o con una prueba en hardware. Mientras tanto,
CIDcoder-B mantiene la lectura 201-B por compatibilidad con el emulador y con
los programas `.lstb` existentes.

### 3.4 Los códigos 32–47

Es el bloque de puntuación. Dos observaciones:

- **CI 32**: la tabla histórica muestra un guion `-`, pero su ISO es 137 octal
  = 95 decimal = `_` (guion bajo). La implementación usa `_`. Probablemente el
  glifo histórico es un guion bajo o un guion alto, no el menos de CI 45.
- **CI 35**: la tabla muestra `1/2`, pero la implementación guarda el valor
  **171**. Ese valor corresponde a `½` en la página de códigos CP437, pero
  JavaScript lo interpreta como Unicode U+00AB, es decir, `«`. Por eso el papel
  muestra `«` y no `½`. Es una discrepancia real entre la intención histórica y
  el resultado actual; se trata en 12.6.
- **CI 36**: “Fin de texto”, con ISO 003. La implementación lo usa como
  centinela de fin de cadena; al imprimirlo con `TAP` produce un espacio en
  blanco, no un carácter visible.
- **CI 38 y CI 60**: ambos son variantes de `$`. La implementación los dibuja
  como `$`, pero el texto fuente `$` se convierte a CI 27, no a 38 ni a 60.

### 3.5 Los dígitos 48–57

Los dígitos están en orden consecutivo:

```
CI 48 = '0'
CI 49 = '1'
...
CI 57 = '9'
```

Para convertir un dígito a CI: `CI = 48 + dígito`. Para convertir un CI a
dígito: `dígito = CI - 48`. El runtime `RT_PER_DIGITS` usa esta relación para
imprimir la parte entera y la fracción.

### 3.6 Los códigos 58–63

Son los últimos signos: `: ; $ = ! ?`. El `$` de CI 60 es otra variante; el
`=` de CI 61 se usa en comparaciones y en el ensamblador, pero no en `PER`.
El `!` y el `?` son imprimibles.

---

## 4. El Código B

### 4.1 Tabla resumida

El Código B es un juego alternativo. No reemplaza al Código A: la computadora
cambia de juego según el contexto. La tabla histórica dice:

| Rango | Código B | Comentario |
|---|---|---|
| 0 | espacio | Igual que A |
| 1–26 | a–z | Minúsculas |
| 27 | — | No definido |
| 28 | tab | Igual que A |
| 29 | Cambio A | Volver a mayúsculas |
| 30 | Cambio B | Quedarse en minúsculas |
| 31 | Línea | Igual que A |
| 32 | Carácter no permitido | — |
| 33 | — | No definido |
| 34 | Start of reading | Control de lectora de cinta |
| 35 | Start of text | Control de texto |
| 36 | End of text | Fin de texto |
| 37 | End of transm | Fin de transmisión |
| 38–39 | — | No definidos |
| 40 | Retroespacio | Backspace |
| 41–42 | — | No definidos |
| 43 | Cancel | Cancelar |
| 44–45 | — | No definidos |
| 46 | Punch on 2 | Perforadora |
| 47 | Reader excn | Excepción de lectora |
| 48 | Pare el lector | Detener lectora |
| 49 | Print rest 2 | Imprimir resto |
| 50 | Prog ind 2 | Indicador de programa |
| 51 | Prog ind | Indicador de programa |
| 52 | File separat | Separador de archivo |
| 53 | Print restore | Restaurar impresión |
| 54 | Unit separat | Separador de unidad |
| 55 | Synchron Cicle | Ciclo de sincronización |
| 56 | End of transm block | Fin de bloque |
| 57 | Prefix code | Prefijo |
| 58 | Skip restore | Restaurar salto |
| 59 | Punch on | Encender perforadora |
| 60 | Punch off | Apagar perforadora |
| 61 | Non print | No imprimir |
| 62 | Data select | Selección de datos |
| 63 | Record separator | Separador de registro |

### 4.2 Por qué no se implementa

CIDcoder-B implementa solo el Código A. Las razones:

- LEAL-B no tiene una instrucción para cambiar de juego de caracteres.
- El teletipo Ricoh 3000 del emulador es de papel y no tiene minúsculas
  mecánicas; forzar mayúsculas es coherente con el hardware modelado.
- Los códigos de control de cinta (34–63) no tienen periférico asociado en
  CIDcoder-B; `SEP` solo acepta la Ricoh.

Si en el futuro se quiere fidelidad histórica con minúsculas o con cinta
perforada, hay que añadir un modo de Código B y una tabla de conversión
específica. No es una tarea de esta versión.

---

## 5. CI, ISO y Murray: tres códigos distintos

### 5.1 Tres códigos, tres anchos

| Código | Ancho | Rango | Uso |
|---|---:|---|---|
| CI | 6 bits | 0–63 | Alfabeto interno de la 201-B |
| ISO | 8 bits | 000–377 octal | Cinta, intercambio, periféricos |
| Murray/MTK-2 | 5 bits | 00–37 octal | Teletipo, con LTRS/FIGS |
| ASCII | 7/8 bits | 0–127 / 0–255 | Terminal moderna del emulador |

No son intercambiables. Una `A` es CI 1, ISO 101 octal (65 decimal) y Murray
30(L) según la tabla histórica. Si se confunden, se imprime basura.

### 5.2 Ejemplo completo: la letra A

| Representación | Valor | Comentario |
|---|---|---|
| CI | 1 | Lo que guarda la 201-B |
| ISO | 101 octal = 65 decimal | `A` en ASCII/ISO de 7 bits |
| Murray | 30(L) | 5 bits, estado Letras |
| ASCII del emulador | 65 | Para dibujar en la pantalla/papel |

El camino completo en CIDcoder-B es:

```
Tecla 'A' (ASCII 65)
   │  ASCII_TO_CI[65]
   ▼
CI 1
   │  TLA / memoria / TAP
   ▼
CI_TO_ASCII[1] = 65
   │
   ▼
Papel: 'A'
```

### 5.3 Ejemplo completo: el dígito 1

| Representación | Valor | Comentario |
|---|---|---|
| CI | 49 | 48 + 1 |
| ISO | 261 octal = 177 decimal | El bit 8 está puesto |
| Murray | 35(f) | 5 bits, estado Figuras |
| ASCII del emulador | 49 | Para dibujar `1` |

El ISO 261 octal no es el ASCII `1` (que es 061 octal). Eso confirma que la
columna ISO no es ASCII puro: tiene el bit 8 modificado. No la use para
comparar con ASCII sin normalizar.

### 5.4 Por qué el ISO de la tabla no es ASCII puro

En la tabla histórica, algunos caracteres tienen ISO 101, 102, 107, 110, 113,
etc. (sin bit 8), y otros tienen 303, 304, 305, 306, 311, etc. (con bit 8).
Ejemplos:

| Carácter | ISO octal | Decimal | ¿ASCII? |
|---|---:|---:|---|
| A | 101 | 65 | Sí |
| B | 102 | 66 | Sí |
| C | 303 | 195 | No: 67 + 128 |
| D | 304 | 196 | No: 68 + 128 |
| G | 107 | 71 | Sí |
| H | 110 | 72 | Sí |
| I | 311 | 201 | No: 73 + 128 |
| 0 | 060 | 48 | Sí |
| 1 | 261 | 177 | No: 49 + 128 |

La alternancia no parece aleatoria: puede ser paridad, una convención de cinta
o un error de transcripción. Si el bit 8 es paridad, el valor sin paridad se
obtiene con `ISO & 0o177`; si es una variante, hay que consultar el manual.
**No asuma que `ISO & 0o177` siempre da ASCII**; verifíquelo carácter por
carácter.

### 5.5 El cambio de registro LTRS/FIGS

Murray/MTK-2 solo tiene 32 combinaciones. Para imprimir más de 32 símbolos, el
teletipo usa dos registros:

- **LTRS** (letras): letras y algunos signos.
- **FIGS** (figuras): dígitos y otros signos.

El mismo código físico produce una letra o un dígito según el registro activo.
Por eso la tabla histórica anota `(L)` o `(f)` en la columna Murray. La
computadora debe enviar un cambio de registro antes de cambiar de tipo de
carácter.

En la tabla histórica, CI 29 y CI 30 se llaman “Cambio A” y “Cambio B”. Es
tentador identificarlos con LTRS y FIGS. Pero la implementación 201-B los usa
como NUL y CR. Es el punto 12.2.

**Nota importante:** la columna Murray de la tabla histórica **no coincide** con
la tabla ITA2 que usa LEAL-A (`MURR_L`/`MURR_F`). Por ejemplo:

| Letra | Murray histórico | ITA2 de LEAL-A |
|---|---:|---:|
| A | 30(L) | 03(L) |
| B | 23(L) | 31(L) |
| C | 16(L) | 16(L) |
| D | 22(L) | 11(L) |
| E | 20(L) | 01(L) |

Solo `C` coincide en este pequeño ejemplo. Eso sugiere que la tabla histórica
usa la variante **MTK-2 soviética** (o una transcripción con otro orden), no
ITA2. Antes de usar los valores Murray para emular el Consul 260, hay que
verificar cuál de las dos tablas es la correcta. La implementación de LEAL-B no
usa Murray para texto: usa CI directamente. La tabla Murray solo aparece en la
documentación histórica y en el modo LEAL-A legado.

---

## 6. Cómo se guarda el CI en la 201-B

### 6.1 Palabra de 12 bits

La memoria de la 201-B tiene palabras de 12 bits. Un carácter CI ocupa 6 bits.
Por tanto, una palabra puede guardar:

```
bits 11 ───── 6   bits 5 ───── 0
   carácter 2        carácter 1
```

El carácter de la mitad baja (bits 0–5) es el **primero** de la cadena. El de
la mitad alta (bits 6–11) es el **segundo**. La fórmula es:

```
palabra = bajo | (alto << 6)
```

donde `bajo` y `alto` son valores CI de 0 a 63.

### 6.2 Empaquetado 6+6

El modo **packed** (empaquetado) guarda dos caracteres por palabra. El modo
**unpacked** (desempaquetado) guarda un carácter por palabra, en los 6 bits
bajos, y deja los 6 bits altos en cero.

| Modo | Caracteres por palabra | Uso |
|---|---:|---|
| UNPK | 1 | Depuración, buffers pequeños, compatibilidad |
| PKD | 2 | Texto compacto, `TIT` y buffers de LEAL-B |

El botón `STRPK` del editor controla el modo. En LEAL-B, el compilador genera
las palabras según ese modo.

### 6.3 Terminador CI 36

Las cadenas de LEAL-B terminan con **CI 36** (octal `0o44`). El terminador no
es un carácter imprimible: `PCA` y `RT_PUTSTR` lo detectan y se detienen.
`TAP` lo imprimiría como espacio en blanco, pero los recorridos de cadena lo
saltan.

En modo empaquetado, el terminador puede caer en la mitad baja o en la mitad
alta de una palabra. Por eso `PCA` comprueba las dos mitades antes de imprimir.

### 6.4 Ejemplos verificados

#### `TIT ABC`

Secuencia CI: `A=1`, `B=2`, `C=3`, `LF=31`, `CR=30`, `END=36`.

| Palabra | Cálculo | Octal | Comentario |
|---|---:|---:|---|
| 1 | `1 OR (2 << 6)` | `0o201` | `A` en bajo, `B` en alto |
| 2 | `3 OR (31 << 6)` | `0o3703` | `C` en bajo, `LF` en alto |
| 3 | `30 OR (36 << 6)` | `0o4436` | `CR` en bajo, `END` en alto |

Salida en papel: `ABC` seguido de un salto de línea (LF+CR contados como uno).

#### `TIT HOLA`

Secuencia CI: `H=8`, `O=15`, `L=12`, `A=1`, `LF=31`, `CR=30`, `END=36`.

| Palabra | Cálculo | Octal | Comentario |
|---|---:|---:|---|
| 1 | `8 OR (15 << 6)` | `0o1710` | `H` en bajo, `O` en alto |
| 2 | `12 OR (1 << 6)` | `0o0114` | `L` en bajo, `A` en alto |
| 3 | `31 OR (30 << 6)` | `0o3637` | `LF` en bajo, `CR` en alto |
| 4 | `36` | `0o0044` | `END` en bajo |

Salida en papel: `HOLA` seguido de un salto de línea (LF+CR contados como uno).

#### `LCA U` con entrada `USER` + CR

Secuencia CI: `U=21`, `S=19`, `E=5`, `R=18`, `END=36`. El CR no se guarda.

| Palabra | Cálculo | Octal | Comentario |
|---|---:|---:|---|
| 1 | `21 OR (19 << 6)` | `0o2325` | `U` en bajo, `S` en alto |
| 2 | `5 OR (18 << 6)` | `0o2205` | `E` en bajo, `R` en alto |
| 3 | `36` | `0o0044` | `END` en bajo |

Salida de `PCA U`: `USER`.

### 6.5 Modo desempaquetado

En modo UNPK, cada carácter ocupa una palabra:

| Carácter | CI | Palabra octal |
|---|---:|---:|
| `A` | 1 | `0001` |
| `B` | 2 | `0002` |
| `C` | 3 | `0003` |
| `LF` | 31 | `0037` |
| `END` | 36 | `0044` |

Una cadena `ABC` ocupa 4 palabras: `0001 0002 0003 0037`, más el terminador
`0044`. Es más simple de leer en el visor de memoria, pero ocupa el doble.

### 6.6 Comparación con LEAL-A

LEAL-A usaba el código Murray de 5 bits y empaquetaba dos caracteres en 5+5
bits, con terminador 0. LEAL-B usa CI de 6 bits y empaqueta 6+6, con terminador
CI 36. Las diferencias son:

| Aspecto | LEAL-A | LEAL-B |
|---|---|---|
| Código de carácter | Murray/ITA2 de 5 bits | CI de 6 bits |
| Empaquetado | 5+5 | 6+6 |
| Terminador | 0 | CI 36 (`0o44`) |
| `TIT` | Sin salto automático | Con LF+CR automático (un solo salto visible) |
| `LIN` | CR+LF | CR+LF |
| Mayúsculas/minúsculas | Cambios LTRS/FIGS | Solo Código A; minúsculas se pliegan |
| `PER` entero | Formato propio | `+`/`-` opcional + dígitos + 2 espacios; cero sin signo |
| `PER` flotante | `DIG m,n` con 3 espacios | `+`/`-` opcional + Entero.Fracción + 3 espacios; cero sin signo |

Esta tabla es la razón por la que un programa LEAL-A no se puede ejecutar
directamente en CIDcoder-B sin recompilarlo: el formato de texto cambió.

---

## 7. Instrucciones de E/S

### 7.1 Tabla

| Mnemónico | Octal | Función | Papel del CI |
|---|---:|---|---|
| `SIL` | `6101` | Salta si hay entrada lista | Consulta el indicador IL |
| `LIL` | `6102` | Limpia el indicador de entrada | — |
| `TLA` | `6104` | Toma un CI de entrada y lo OR-ea en AC | Lee un CI |
| `SIF` | `6111` | Salta si fin de línea | En Ricoh: no-op |
| `SIE` | `6122` | Salta si error | En Ricoh: no-op |
| `SITE` | `6141` | Salta si tecla | En Ricoh: no-op |
| `SIP` | `6221` | Salta si impresora lista | Consulta el indicador de impresión |
| `LIP` | `6222` | Limpia el indicador de impresión | — |
| `TAP` | `6224` | Imprime los 6 bits bajos de AC y limpia AC | Escribe un CI |

### 7.2 `TLA`: entrada

`TLA` toma el siguiente carácter de la cola de entrada, lo convierte de ASCII a
CI y lo OR-ea en el acumulador:

```js
const asciiCode = UNIDAD.inBuf.shift();
let ci = ASCII_TO_CI[asciiCode];
if (ci === undefined) ci = 0;
S.AC = mask12(S.AC | ci);
```

Consecuencias:

- `TLA` **no** borra el acumulador. Si no se limpia antes, el CI anterior se
  mezcla con el nuevo mediante OR.
- El CI ocupa los 6 bits bajos; los bits altos del acumulador no se tocan.
- Si el carácter ASCII no tiene CI (por ejemplo, `ñ`), se usa CI 0 (espacio).

La secuencia típica de lectura es:

```
SIL             / ¿hay tecla?
VEA esperar     / no: volver
LAC             / limpiar AC
TLA             / AC = CI de la tecla
```

### 7.3 `TAP`: salida

`TAP` toma los 6 bits bajos del acumulador, los imprime y limpia el acumulador:

```js
const ci = S.AC & 0x3F;
S.AC = 0;
let asciiCode = CI_TO_ASCII[ci];
// casos especiales: TAB, NUL, CR, LF, END
```

Consecuencias:

- `TAP` **sí** limpia el acumulador. Si se necesita el valor después, hay que
  guardarlo antes.
- Solo se imprimen los 6 bits bajos; los bits 6–11 se ignoran.
- El CI 36 (END) se imprime como espacio, no como carácter visible.

La secuencia típica de escritura es:

```
LAC
SUM K_A       / AC = CI de 'A'
TAP           / imprime 'A' y limpia AC
SIP           / ¿impresora lista?
VEA esperar   / no: volver
```

### 7.4 Indicadores

| Indicador | Significado | Instrucciones |
|---|---|---|
| IL | Hay entrada disponible | `SIL`, `LIL`, `TLA` |
| IP | Impresora ocupada | `SIP`, `LIP` |
| IF | Fin de línea | `SIF` (no-op en Ricoh) |
| IE | Error | `SIE` (no-op en Ricoh) |
| IT | Tecla | `SITE` (no-op en Ricoh) |

En el Ricoh 3000 del emulador, `SIF`, `SIE` y `SITE` no hacen nada porque el
teletipo de papel no modela esos indicadores. Si en el futuro se modela un
Consul 260 con cinta, habrá que implementarlos.

### 7.5 Secuencia típica de lectura

```
*0000
loop,   SIL             / ¿hay una tecla?
        VEA loop        / no: esperar
        LAC             / limpiar AC
        TLA             / AC = CI de la tecla
        DEL TMP         / guardar el CI
        ...
```

### 7.6 Secuencia típica de escritura

```
        LAC
        SUM K_A         / AC = CI de 'A'
        TAP             / imprimir
wait,   SIP             / ¿impresora lista?
        VEA wait        / no: esperar
```

### 7.7 `SIF`, `SIE`, `SITE`

En el emulador 201-B, estas instrucciones existen y tienen códigos octales,
pero en modo Ricoh son no-ops. En modo Consul 260 (no implementado en
CIDcoder-B) consultarían los indicadores IF, IE e IT. La decisión de
mantenerlas como no-ops es deliberada: el Ricoh de papel no tiene cinta ni
teclado mecánico.

---

## 8. El Ricoh 3000 en CIDcoder-B

### 8.1 Modo computadora

El Ricoh 3000 del emulador está siempre en **modo computadora**. No hay modo
local ni modo de cinta. El usuario escribe sobre el papel; cada carácter se
ecoa localmente y se entrega a la Unidad de Enlace. `Enter` envía CR (ASCII 13),
que se convierte a CI 30.

### 8.2 Papel y eco local

| Propiedad | Valor |
|---|---|
| Ancho de línea | 60 columnas |
| Salida | Papel (texto en pantalla) |
| Entrada | Teclado de la computadora, sin teclado visible |
| Eco | Local: lo tecleado se imprime en el papel |
| `Enter` | CR (ASCII 13) |
| `Backspace` | Borra el último carácter de la línea |

El eco local es importante para las pruebas: si se teclea `USER` y luego se
ejecuta `PCA U`, el papel muestra primero el eco `USER` y después la salida de
`PCA`. Para comparar solo la salida del programa, hay que desactivar el eco o
recortar el texto.

### 8.3 Entrada

La entrada del Ricoh es ASCII. En la frontera se convierte a CI:

```
Tecla ASCII  ──ASCII_TO_CI──>  CI  ──TLA──>  AC
```

Las teclas válidas son las que tienen CI en la tabla inversa. Las que no
(fuera del repertorio) se convierten a CI 0 o se ignoran, según el camino.
`Enter` es CR, que tiene CI 30. `Tab` es CI 28.

### 8.4 Salida

La salida del `TAP` es CI. En la frontera se convierte a ASCII:

```
AC (6 bits)  ──CI_TO_ASCII──>  ASCII  ──papel──>  carácter
```

Casos especiales:

| CI | ASCII | Efecto en papel |
|---:|---:|---|
| 28 | 9 | Tabulador (avanza la línea) |
| 29 | 0 | Espacio en blanco |
| 30 | 13 | Nueva línea |
| 31 | 10 | Nueva línea si la línea no está vacía |
| 36 | 0 | Espacio en blanco |

### 8.5 `SEP 7,7`

`SEP J,K` selecciona el periférico de entrada y el de salida. CIDcoder-B solo
tiene la Ricoh 3000, que es el periférico 7. Por eso:

- `SEP 7,7` es válido y abre la Ricoh.
- Cualquier otro valor numérico produce `E:PERIF` en compilación.
- Un valor variable produce `E:PERIF` en ejecución y detiene el programa.

Esta restricción es deliberada: no hay una segunda unidad de cinta ni un
Consul 260 físico en el emulador.

### 8.6 Diferencias con el Consul 260

| Aspecto | Consul 260 histórico | Ricoh 3000 de CIDcoder-B |
|---|---|---|
| Código físico | Murray/MTK-2 de 5 bits | ASCII en la frontera, CI en la máquina |
| Registros | LTRS/FIGS | Solo Código A |
| Teclado | Mecánico, con teclas cirílicas | Teclado de computadora, sin teclado visible |
| Cinta | Lectora/perforadora | No modelada |
| Indicadores | IF, IE, IT | No modelados (no-ops) |
| Ancho | Depende del modelo | 60 columnas |

La Ricoh es una simplificación moderna del teletipo histórico. Sirve para
probar el camino CI, pero no pretende ser un Consul 260 exacto.

---

## 9. El CI en LEAL-B

### 9.1 `TIT`

`TIT` convierte un texto literal a CI, le añade LF (CI 31), CR (CI 30) y un
terminador CI 36, y lo imprime con `RT_PUTSTR`.

```
TIT HOLA
```

Genera:

| Paso | CI | Comentario |
|---|---|---|
| H | 8 | `H` |
| O | 15 | `O` |
| L | 12 | `L` |
| A | 1 | `A` |
| LF | 31 | Primera mitad del salto |
| CR | 30 | Segunda mitad del salto |
| END | 36 | Terminador |

En modo empaquetado: `0o1710`, `0o0114`, `0o3637`, `0o0044`.

Salida: `HOLA` y un salto de línea. La Ricoh trata LF+CR como un solo salto:
el segundo control de un par sobre una línea aún vacía no añade otra línea, y
el siguiente carácter se imprime en la columna 0 de la línea nueva.

**Diferencias con LEAL-A:**

- LEAL-A no añade salto automático; hay que usar `LIN`.
- LEAL-A usa Murray de 5 bits; LEAL-B usa CI de 6 bits.
- LEAL-A empaqueta 5+5; LEAL-B empaqueta 6+6.
- LEAL-B pliega minúsculas a mayúsculas.

**Caracteres no representables:** si el texto contiene un carácter sin CI
(por ejemplo, `ñ`, `á`, `ü`), el compilador emite un warning y lo omite. No es
un error fatal; el resto del texto se imprime.

### 9.2 `LIN`

`LIN` emite CR (CI 30) y LF (CI 31), en ese orden:

```
LAC
SUM K_CR       / AC = CI 30
TAP            / CR
SIP
VEA esperar
LAC
SUM K_LF       / AC = CI 31
TAP            / LF
```

En el papel, CR y LF son dos mitades del mismo salto. `LIN` emite CR+LF y
`TIT` emite LF+CR; en ambos casos la Ricoh cuenta un solo salto y deja el
cursor al inicio de la línea nueva. Por eso `TIT HOLA` + `LIN` produce **dos**
saltos de línea: el de `TIT` y el de `LIN`. Es el comportamiento esperado si
`TIT` se considera un título con salto propio.

### 9.3 `LCA`

`LCA v` lee caracteres CI del Ricoh hasta CR (CI 30) y los guarda en el buffer
`v`. No guarda el CR; al final escribe CI 36.

```
LCA U
```

Con entrada `USER` + Enter:

| Paso | Acción |
|---|---|
| 1 | Espera `SIL` |
| 2 | `TLA` lee `U` (CI 21) |
| 3 | Guarda `U` |
| 4 | Repite con `S`, `E`, `R` |
| 5 | Lee CR (CI 30), no lo guarda, termina |
| 6 | Escribe CI 36 en la posición siguiente |

En modo empaquetado, las palabras son `0o2325`, `0o2205`, `0o0044`.

**Tamaño del buffer:** el buffer debe tener espacio para los caracteres más el
terminador. Si se llena, `LCA` ignora los caracteres extra (no desborda). El
terminador se escribe en la última posición reservada.

### 9.4 `PCA`

`PCA v` recorre el buffer desde el inicio y emite cada CI con `TAP` hasta
encontrar CI 36. En modo empaquetado comprueba las dos mitades de cada palabra.

```
PCA U
```

Con el buffer anterior, imprime `USER`.

**No imprime el terminador.** Si el buffer está vacío (primer CI es 36), no
imprime nada.

### 9.5 `MDI`

`MDI v` desplaza la cadena una posición a la **izquierda**: se pierde el primer
carácter y se inserta un espacio (CI 0) justo antes del terminador.

```
"ABC"  ──MDI──>  "BC "
```

El terminador CI 36 no se mueve de su posición final; el espacio ocupa el
hueco que deja el primer carácter. En modo empaquetado se trabaja a nivel de
palabra, pero el resultado lógico es el mismo.

### 9.6 `MID`

`MID v` desplaza la cadena una posición a la **derecha**: se inserta un espacio
(CI 0) al inicio y se pierde el último carácter.

```
"ABC"  ──MID──>  " AB"
```

El terminador CI 36 se conserva. En modo empaquetado se trabaja a nivel de
palabra; el último carácter se pierde.

**Estado de verificación:** las cuatro variantes están verificadas con un
buffer llenado por `LCA` (no por `TIT`, que no llena buffer):

| Variante | Entrada | Salida de `PCA` | Buffer resultante |
|---|---|---|---|
| `MDI` UNPK | `ABC` + CR | `BC ` | `2 3 0 44` |
| `MDI` PKD | `ABC` + CR | `BC ` | `0o302 0o4400` |
| `MID` UNPK | `ABC` + CR | ` AB` | `0 1 2 44` |
| `MID` PKD | `ABC` + CR | ` AB` | `0o100 0o4402` |

Durante la verificación se corrigieron dos defectos:

- `K_77` y `K_7700` no se emitían para `RT_MDI_PACKED` ni `RT_MID_PACKED`; el
  ensamblador fallaba con “etiqueta o dirección no reconocida: K_7700”.
- `RT_MDI_PACKED` entraba en `mdp_begin` con el acumulador sucio (`K_END −
  primer CI`) y `SUM TLO` calculaba un `TPTR` 35 palabras más allá del buffer.
  Se añadió un `LAC` antes de `SUM TLO`.

### 9.7 `ESP`

`ESP n` imprime `n` espacios. Cada espacio es CI 0. El compilador emite la
constante `n` y llama a `RT_ESP`, que imprime CI 0 repetidamente.

### 9.8 `PER` y `DIG`

`PER` no imprime texto libre: imprime un número. Pero internamente usa CI para
cada símbolo:

| Símbolo | CI | Cómo se obtiene |
|---|---:|---|
| `+` | 43 | Constante; se imprime en positivos distintos de cero |
| `-` | 45 | Constante; se imprime si el valor es negativo |
| `0`–`9` | 48–57 | `48 + dígito` |
| `.` | 46 | Constante |
| espacio | 0 | `LAC` + `TAP` |

#### `PER` entero

Formato: **signo + dígitos significativos + 2 espacios**. `+` en positivos
distintos de cero, `-` en negativos; el cero no lleva signo. No hay relleno por
la izquierda (ni ceros ni espacios) y el cero se imprime como `0`.

| Valor | Salida |
|---:|---|
| `579` | `+579␠␠` |
| `0` | `0␠␠` |
| `1234567` | `+1234567␠␠` |
| `-579` | `-579␠␠` |

#### `PER` flotante

Formato: **signo + parte entera + punto + F dígitos fraccionarios + 3 espacios**.
La parte entera se imprime completa, sin relleno por la izquierda; si no tiene
enteros, se imprime el cero de las unidades. La fracción se rellena con ceros
a la derecha hasta `F` dígitos. El signo `+` aparece en positivos distintos de
cero, `-` en negativos y el cero no lleva signo. Por defecto `F=3`.

| Valor | Salida |
|---:|---|
| `-3.5` | `-3.500␠␠␠` |
| `3.5` | `+3.500␠␠␠` |
| `0.0` | `0.000␠␠␠` |
| `-0.25` | `-0.250␠␠␠` |
| `-123.75` | `-123.750␠␠␠` |
| `DIG 2,4` + `-3.5` | `-3.5000␠␠␠` |
| `1234567.25` | `+1234560.000␠␠␠` |

El último caso no es un error de formato: el flotante de 17 bits de la 201-B
tiene una granularidad de 16 en esa magnitud, así que el valor almacenado es
`1234560`, no `1234567.25`. El `PER` imprime el valor real, no el literal.

#### `DIG m,n`

- `n` = dígitos de la fracción (0–5). Es el único parámetro que cambia la
  salida: controla los ceros de la fracción.
- `m` = ancho declarado de la parte entera (0–8). Se acepta y se satura, pero
  **no rellena**: la parte entera siempre sale completa y sin espacios a la
  izquierda. Se conserva por compatibilidad con la sintaxis histórica.
- `DIG m` (un solo argumento) es el formato exponencial: **no soportado**.
  El compilador lo rechaza con `E:DIGEXP`.
- Sin `DIG`, el formato es `6,3` (el `m=6` tampoco rellena).

### 9.9 Limitaciones del juego de caracteres

| Limitación | Efecto |
|---|---|
| Solo mayúsculas | `tit hola` se compila como `HOLA` |
| Sin acentos | `á`, `é`, `í`, `ó`, `ú`, `ñ` se omiten con warning |
| Sin `£` | El carácter libra no se puede teclear; `$` va a CI 27 |
| Sin minúsculas | El Código B no está implementado |
| Sin cinta | `SEP` solo acepta 7,7; no hay lectora/perforadora |
| `½` (CI 35) | Se almacena 171 y el papel muestra `«` (U+00AB), no `½`; ver 12.6 |

### 9.10 Ejemplo integrador

```
TIT NOMBRE:
LCA U
TIT HOLA,
PCA U
LIN
ALT
```

Con entrada `USER` + Enter, la secuencia es:

1. `TIT NOMBRE:` imprime `NOMBRE:` y un salto (LF+CR).
2. `LCA U` lee `USER` y guarda `USER` + END.
3. `TIT HOLA,` imprime `HOLA,` y un salto (LF+CR).
4. `PCA U` imprime `USER`.
5. `LIN` imprime CR + LF.
6. `ALT` detiene.

Salida (sin contar el eco local): `NOMBRE:\nHOLA,\nUSER\n`.

---

## 10. Implementación en CIDcoder-B

### 10.1 Constantes y tablas

En `CIDcoder-B_6.html`, cerca del inicio del compilador:

```js
const CI_END = 0o44;   // fin de texto (CI 36)
const CI_CR  = 0o36;   // retorno de carro (CI 30)
const CI_LF  = 0o37;   // salto de línea (CI 31)
const CI_SP  = 0;      // espacio (CI 0)
```

La tabla `CI_ASCII` contiene los 64 valores ASCII. La tabla inversa
`ASCII_CI` se construye recorriendo `CI_ASCII` y tomando la **primera**
aparición de cada valor no cero. Luego se añaden las minúsculas:

```js
for (let ci = 0; ci < 64; ci++) {
  const code = CI_ASCII[ci];
  if (code && ASCII_CI[code] === undefined) ASCII_CI[code] = ci;
}
for (let i = 65; i <= 90; i++) {
  if (ASCII_CI[i] !== undefined) ASCII_CI[i + 32] = ASCII_CI[i];
}
```

Consecuencia: `$` (ASCII 36) aparece en CI 27, 38 y 60; la primera aparición es
27, así que el texto fuente `$` se convierte a CI 27. Las otras dos variantes
solo se alcanzan con constantes numéricas.

### 10.2 Conversión

```js
function ciCodeOfChar(ch) {
  if (ch === "\r") return CI_CR;
  if (ch === "\n") return CI_LF;
  const up = ch.toUpperCase();
  const code = ASCII_CI[ch] ?? ASCII_CI[up] ?? ASCII_CI[ch.charCodeAt(0)];
  return code === undefined ? null : code;
}
```

`encodeCiText(text)` recorre el texto, convierte cada carácter y omite los que
no tienen CI, avisando con un callback. `TIT` usa esta función.

El núcleo `CORE201B` tiene su propia copia de la tabla:

```js
const CI_TO_ASCII = [ ... ];
const ASCII_TO_CI = {};
function asciiToCi(code) { ... }
function ciToAscii(ci) { return CI_TO_ASCII[ci & 0x3F]; }
```

Las dos copias deben coincidir. Si se cambia una, hay que cambiar la otra.

### 10.3 Emisores del compilador

| Método | Qué emite | Observación |
|---|---|---|
| `tit(t)` | Texto CI + LF + CR + END | `targetB` usa 6 bits |
| `_emitLin()` | CR + LF | Dos `TAP` |
| `_emitLcaUnpacked()` | Lectura CI, 1 por palabra | Termina en CI 36 |
| `_emitLcaPacked()` | Lectura CI, 2 por palabra | 6+6 |
| `_emitPcaUnpacked()` | Impresión CI, 1 por palabra | Hasta CI 36 |
| `_emitPcaPacked()` | Impresión CI, 2 por palabra | Comprueba ambas mitades |
| `_emitMdiUnpacked()` | Desplazamiento izquierdo | 1 por palabra |
| `_emitMdiPacked()` | Desplazamiento izquierdo | 2 por palabra |
| `_emitMidUnpacked()` | Desplazamiento derecho | 1 por palabra |
| `_emitMidPacked()` | Desplazamiento derecho | 2 por palabra |
| `_emitPerInt()` | Signo + dígitos significativos + 2 espacios | Usa `RT_PER_DIGITS` |
| `_emitPerFlt()` | Signo + Entero.Fracción + 3 espacios | Usa `RT_PER_DIGITS` |
| `_emitPerCore()` | Signo + entero de 31 bits + fracción | Para magnitudes grandes |

### 10.4 Rutinas runtime

| Rutina | Función |
|---|---|
| `RT_PUTSTR` | Imprime una cadena CI empaquetada o desempaquetada |
| `RT_GETCHAR` | Lee un carácter CI del Ricoh |
| `RT_LCA_UNPACKED` | Implementa `LCA` en modo UNPK |
| `RT_LCA_PACKED` | Implementa `LCA` en modo PKD |
| `RT_PCA_UNPACKED` | Implementa `PCA` en modo UNPK |
| `RT_PCA_PACKED` | Implementa `PCA` en modo PKD |
| `RT_MDI_UNPACKED` | Implementa `MDI` en modo UNPK |
| `RT_MDI_PACKED` | Implementa `MDI` en modo PKD |
| `RT_MID_UNPACKED` | Implementa `MID` en modo UNPK |
| `RT_MID_PACKED` | Implementa `MID` en modo PKD |
| `RT_PER_DIGITS` | Imprime los dígitos significativos, sin signo ni relleno |
| `RT_PER_INT` | Imprime `+`/`-` (cero sin signo) + `RT_PER_DIGITS` |
| `RT_PER_FLT` | Imprime flotante con signo y punto fijo |
| `RT_PER_CORE` | Imprime flotante grande con signo y entero de 31 bits |

### 10.5 Núcleo y terminal

`CORE201B` implementa las instrucciones de E/S en `handleIO`:

- `TLA` lee de `UNIDAD.inBuf` y convierte con `ASCII_TO_CI`.
- `TAP` toma `AC & 0x3F`, convierte con `CI_TO_ASCII` y emite por el callback
  `onPaper`; la interfaz conecta ese callback a `RICOH3000.printAscii`.
- `SIL`, `LIL`, `SIP`, `LIP` consultan o limpian los indicadores.
- `SIF`, `SIE`, `SITE` son no-ops en modo Ricoh.

`RICOH3000` implementa el papel:

- `printAscii(code)`: dibuja el carácter, maneja CR, LF, TAB y backspace.
- `typeAscii(code)`: eco local + `core.feedInput(code)`.
- `getText()`: devuelve el papel como texto.
- `reset()`: limpia el papel.

### 10.6 Archivos y funciones de referencia

| Archivo | Contenido |
|---|---|
| `CIDcoder-B_6.html` | Implementación completa |
| `DOC/PROYECTO-CIDcoder-B.md` | Decisiones y fases del proyecto |
| `TEST/lealb-helpers.js` | Helpers de compilación/ejecución aislada |
| `TEST/lenb.test.js` | Suite del ensamblador LEN-B |
| `TEST/run201b.test.js` | Suite de ejecución 201-B |
| `LEAL/CODIGOINTERNO.md` | Tablas CI de LEAL-F |
| `CID201B/DOChistorico/Consul260/CODIGOINTERNO.md` | Tablas CI de la 201-B |
| `CID201B/CID201B_17.html` | Emulador de referencia |

Las líneas exactas cambian con las ediciones. Busque por nombre de función:
`tit(`, `_emitLcaPacked`, `_emitPcaPacked`, `RT_MID_PACKED`, `CI_TO_ASCII`,
`RICOH3000`.

---

## 11. Vectores de prueba

Los siguientes vectores están verificados en CIDcoder-B 0.1.6, salvo los
marcados como pendientes. Sirven para comprobar que una modificación no rompe
el camino CI.

### 11.1 Texto

| Programa | Salida esperada | Estado |
|---|---|---|
| `TIT HOLA` | `HOLA` + salto (LF+CR) | Verificado |
| `TIT ABC` | `ABC` + salto (LF+CR) | Verificado |
| `TIT 123` | `123` + salto (LF+CR) | Verificado |
| `TIT HOLA` + `LIN` | CI: `HOLA` + LF + CR + CR + LF; papel: `HOLA` + dos líneas nuevas | Verificado |
| `TIT` con `ñ` | Texto sin `ñ` + warning | Verificado |

### 11.2 Buffers

En estas pruebas, la “salida esperada” es la que produce `PCA`, **sin contar el
eco local** del teclado. Si el eco está activo, el papel muestra primero lo
tecleado y después la salida de `PCA`.

| Programa | Entrada | Salida esperada | Estado |
|---|---|---|---|
| `LCA U` + `PCA U` | `USER` + CR | `USER` | Verificado |
| `LCA U` + `PCA U` | `A` + CR | `A` | Verificado |
| `LCA U` + `PCA U` | `ABCDE` + CR | `ABCDE` | Verificado |
| `LCA U` + `MDI U` + `PCA U` | `ABC` + CR | `BC ` | Verificado UNPK y PKD |
| `LCA U` + `MID U` + `PCA U` | `ABC` + CR | ` AB` | Verificado UNPK y PKD |

### 11.3 `PER` entero

Las tablas de esta sección abrevian `PER n` para el par `n = A` + `PER A`.
El compilador exige una variable o un elemento de arreglo en `PER`; un literal
suelto produce `Referencia inválida`.

| Programa | Salida esperada |
|---|---|
| `PER 579` | `+579␠␠` |
| `PER 0` | `0␠␠` |
| `PER 1234567` | `+1234567␠␠` |
| `PER -579` | `-579␠␠` |

### 11.4 `PER` flotante

| Programa | Salida esperada |
|---|---|
| `PER -3.5` | `-3.500␠␠␠` |
| `PER 3.5` | `+3.500␠␠␠` |
| `PER 0.0` | `0.000␠␠␠` |
| `PER -0.25` | `-0.250␠␠␠` |
| `PER -123.75` | `-123.750␠␠␠` |
| `DIG 2,4` + `PER -3.5` | `-3.5000␠␠␠` |
| `PER 1234567.25` | `+1234560.000␠␠␠` |

### 11.5 Errores

| Programa | Resultado esperado |
|---|---|
| `DIG 4` | `E:DIGEXP` |
| `SEP 4,4` | `E:PERIF` |
| `TIT` con carácter no representable | Warning, no error |
| `LCA` sin buffer declarado | Error de compilación |

### 11.6 Monitor de teletipo 201-B

`MONITOR_TELETIPO_201B` usa el CI directamente, sin shifts Murray/ITA2:
`R`=`022`, `A`=`001`, `C`=`003`, `?`=`077`, `:`=`072`, CR=`036`, LF=`037`
y dígitos `060`–`067`. La suite `TEST/monitor.test.js` verifica:

| Vector | Resultado esperado |
|---|---|
| Compilación | 205 palabras; entrada `7400`; 104 en página 29 y 101 en página 30 |
| `R 1000` con `[1000]=1234` | `1000:1234` + CRLF |
| `R 1000 1003` | Cuatro líneas, una por dirección |
| `A 2000 7777` + `R 2000` | `2000:7777` y confirmación |
| `C 3000` con `[3000]=7201` (`ALT`) | Salto indirecto y alto `AP` |
| `R 0076 0101` | Cruce de página 0 a 1 sin perder direcciones |
| `.lstb` de 205 palabras | 0 diferencias contra la compilación |

El listado se cargó en `CID201B_17.html` con 205 palabras, 0 duplicadas,
0 ignoradas y 0 diferencias. La auditoría `LAC` que redujo el monitor de 228 a
205 palabras está documentada en `DOC/GUIA-AUDITORIA-LAC.md`.

### 11.7 Cómo reproducirlos

En el editor:

1. Abra `CIDcoder-B_6.html`.
2. Pulse `New LEAL`.
3. Escriba el programa.
4. Compile y ejecute.
5. Lea el papel del Ricoh.

En las pruebas automatizadas:

```js
const r = LEALBTEST.run("TIT HOLA\nALT");
console.log(r.paper); // "HOLA\n"
```

`LEALBTEST.run` compila con `target: "lenb"`, ensambla con `compileLenB` y
ejecuta en `CORE201B` con `RICOH3000`, sin tocar el editor visible.

---

## 12. Puntos para revisar y corregir

Esta es la parte más importante del documento. Los puntos están ordenados por
impacto. Cada uno indica qué dice la fuente, qué hace la implementación y qué
conviene decidir.

### 12.1 CI 27: `£` o `$`

| | |
|---|---|
| **Fuente histórica** | CI 27 = `£` (libra), ISO 333 octal |
| **Implementación** | `CI_TO_ASCII[27] = 36` (`$`) |
| **Efecto** | El texto fuente `$` se convierte a CI 27 y se imprime `$`; el `£` no se puede teclear |
| **Riesgo** | Pérdida del símbolo histórico; confusión con CI 38 y CI 60 |
| **Recomendación** | Decidir si CI 27 debe ser `£` o `$`. Si es `£`, cambiar `CI_TO_ASCII[27]` a 163 y hacer que `$` se convierta a CI 60 (o 38). Si se mantiene `$`, documentar que es una simplificación deliberada |

### 12.2 CI 29–31: controles o CR/LF

| | |
|---|---|
| **Fuente histórica** | 29 = Cambio A, 30 = Cambio B, 31 = Línea |
| **Implementación** | 29 = NUL, 30 = CR, 31 = LF |
| **Efecto** | `TIT` y `LIN` usan 30/31 como CR/LF; 29 no se usa |
| **Riesgo** | Si 29/30 son cambios de registro del teletipo, la implementación pierde esa función; si 30/31 son CR/LF, la etiqueta “Cambio B” es engañosa |
| **Recomendación** | Verificar contra el manual del Consul 260. Si 29/30 son shifts, añadir un modo de compatibilidad o renombrar las constantes. Si 30/31 son CR/LF, actualizar la tabla histórica con una nota |

### 12.3 CI 32: `-` o `_`

| | |
|---|---|
| **Fuente histórica** | Glifo `-`, ISO 137 octal |
| **Implementación** | `_` (ASCII 95) |
| **Efecto** | El guion bajo se imprime como `_`; el menos está en CI 45 |
| **Riesgo** | Bajo: ISO 137 = 95 = `_`, así que la implementación es coherente con el ISO |
| **Recomendación** | Confirmar el glifo en el escaneo original. Si es un guion bajo, dejar `_`. Si es un guion alto, documentarlo |

### 12.4 CI 36: fin de texto

| | |
|---|---|
| **Fuente histórica** | “Fin de texto”, ISO 003 |
| **Implementación** | Centinela 0; `TAP` lo imprime como espacio |
| **Efecto** | No se puede imprimir un carácter visible con CI 36 |
| **Riesgo** | Bajo: es un control, no un imprimible |
| **Recomendación** | Mantener como centinela y documentar que `TAP` lo trata como espacio. Si se quiere un “fin de texto” visible, usar otro código |

### 12.5 CI 60: variante de `$`

| | |
|---|---|
| **Fuente histórica** | `$ (variante)`, ISO 044 |
| **Implementación** | `$` (ASCII 36), pero el texto fuente `$` va a CI 27 |
| **Efecto** | CI 60 solo se alcanza con constantes numéricas |
| **Riesgo** | Bajo, pero confuso |
| **Recomendación** | Si CI 27 es `£`, hacer que `$` vaya a CI 60. Si CI 27 es `$`, documentar que 60 es una variante no usada |

### 12.6 CI 35: `½`, `«` o `1/2`

| | |
|---|---|
| **Fuente histórica** | `1/2`, ISO 243 |
| **Implementación** | Valor 171; JavaScript lo dibuja como `«` (U+00AB) |
| **Efecto** | El papel muestra `«`, no `½` ni la secuencia `1/2` |
| **Causa** | 171 es el byte de `½` en CP437, pero no es el punto de código Unicode de `½` (U+00BD = 189). `String.fromCharCode(171)` produce `«` |
| **Riesgo** | Bajo para la lógica (el valor CI es estable), alto para la fidelidad visual |
| **Recomendación** | Decidir una de tres: (a) mantener 171 como identidad de byte CP437 y dibujar `½` con una tabla explícita; (b) cambiar el valor a 189 para que JavaScript dibuje `½`; (c) imprimir la secuencia `1/2`. La opción (a) conserva mejor el valor histórico; la (b) es la más simple si solo importa el glifo |

### 12.7 Código B no implementado

| | |
|---|---|
| **Fuente histórica** | Código B con minúsculas y comandos de cinta |
| **Implementación** | Solo Código A; minúsculas se pliegan |
| **Efecto** | `tit hola` imprime `HOLA`; no hay comandos de cinta |
| **Riesgo** | Medio si se quiere fidelidad histórica o texto en minúsculas |
| **Recomendación** | Mantener fuera de alcance por ahora y documentarlo. Si se implementa, añadir un modo de Código B y una tabla de conversión |

### 12.8 `TIT` añade LF+CR (resuelto)

| | |
|---|---|
| **Fuente histórica** | LEAL-A: `TIT` no añade salto |
| **Implementación** | LEAL-B: `TIT` añade LF (CI 31) y CR (CI 30) antes de CI 36 |
| **Efecto** | `TIT HOLA` produce un salto visible; `TIT HOLA` + `LIN` produce dos |
| **Riesgo** | Bajo: la Ricoh colapsa el par LF+CR en un solo salto |
| **Recomendación** | Mantener; el vector está en `lealb.test.js` y en `TEST/pruebas.html` |

### 12.9 `PER`: signo y relleno (resuelto)

| | |
|---|---|
| **Fuente histórica** | `LEALtexto.txt`: sin relleno izquierdo, cola de espacios |
| **Implementación** | `+` en positivos distintos de cero, `-` en negativos, cero sin signo; dígitos significativos sin ceros ni espacios a la izquierda; 2 espacios de cola en enteros y 3 en flotantes |
| **Efecto** | `PER 579` → `+579␠␠`; `PER -579` → `-579␠␠`; `PER 3.5` → `+3.500␠␠␠`; `PER 0` → `0␠␠` |
| **Riesgo** | Bajo: el formato está alineado con la decisión del 18/Nov/2026 |
| **Recomendación** | Mantener los vectores de `lealb.test.js` (positivo, negativo, cero, `DIG 8,5` y carril grande) como regresión |

### 12.10 Ancho de `PER` (resuelto)

| | |
|---|---|
| **Fuente histórica** | `DIG m,n` con `m` enteros y `n` decimales; la parte entera no se rellena |
| **Implementación** | Entero: signo + dígitos + 2 espacios. Flotante: signo + Entero.Fracción + 3 espacios. `m` se acepta pero no rellena; `n` controla la fracción |
| **Efecto** | `+579` ocupa 5 columnas (`+579␠␠`); `-3.5` con `6,3` ocupa 9 (`-3.500␠␠␠`) |
| **Riesgo** | Bajo; el ancho ya no depende de `m` |
| **Recomendación** | Documentar que `m` es declarativo y añadir vectores con `DIG 0,2`, `DIG 8,5` y valores negativos |

### 12.11 Precisión del flotante

| | |
|---|---|
| **Fuente histórica** | Flotante de 17 bits de mantisa |
| **Implementación** | `PER 1234567.25` → `+1234560.000␠␠␠` |
| **Efecto** | El valor se redondea a la granularidad de 16 en esa magnitud |
| **Riesgo** | Bajo: es el valor real almacenado, no un error de formato |
| **Recomendación** | Documentarlo en el manual de usuario y en los mensajes de `PER` si se detecta un literal grande |

### 12.12 Homónimo `CI` (ciclo vs código)

| | |
|---|---|
| **Fuente** | `state.cycle` usa `CI`, `CM`, `CD`, `CO`, `CV` |
| **Implementación** | Son ciclos de máquina, no códigos de carácter |
| **Efecto** | Confusión al leer el código y la documentación |
| **Riesgo** | Medio para mantenibilidad |
| **Recomendación** | Renombrar los valores a `CICLO_INSTR`, `CICLO_MICRO`, `CICLO_DIR`, `CICLO_OPER`, `CICLO_SALTO`, o añadir un comentario claro en `CORE201B.state.cycle` |

### 12.13 Ayuda de LEAL actualizada

| | |
|---|---|
| **Fuente** | La ayuda de `CIDcoder-B_1.html` decía “`TIT` no añade CR+LF” y “terminador 0” en la sección LEAL |
| **Implementación** | La ayuda se actualizó el 18/Nov/2026: LEAL-B añade LF+CR, usa CI 36 y `PER` lleva `+`/`-` |
| **Efecto** | La tabla de instrucciones y las notas de E/S describen el comportamiento real |
| **Riesgo** | Bajo, si se mantiene sincronizada con el compilador |
| **Recomendación** | Revisar la ayuda en cada cambio de formato; los vectores están en `TEST/lealb.test.js` |

### 12.14 Tablas Murray históricas

| | |
|---|---|
| **Fuente** | La columna Murray de la tabla histórica no coincide con ITA2 |
| **Implementación** | LEAL-B no usa Murray para texto; LEAL-A legado usa `MURR_L`/`MURR_F` (ITA2) |
| **Efecto** | Dos tablas Murray distintas en el proyecto |
| **Riesgo** | Medio si se intenta emular el Consul 260 con los valores históricos |
| **Recomendación** | Verificar cuál es la tabla correcta del Consul 260. Si es MTK-2, añadir una tabla separada y no reutilizar `MURR_L`/`MURR_F` |

### 12.15 Columna ISO histórica

| | |
|---|---|
| **Fuente** | Valores ISO con bit 8 alternante |
| **Implementación** | No se usa ISO; se usa ASCII en la frontera |
| **Efecto** | La columna ISO es solo documental |
| **Riesgo** | Bajo para el emulador, alto si se exportan datos a un formato ISO real |
| **Recomendación** | No usar la columna ISO como ASCII. Si se necesita intercambio, crear una tabla ISO→CI verificada |

### 12.16 `MURR_L`/`MURR_F` legado

| | |
|---|---|
| **Fuente** | Tablas ITA2 usadas por LEAL-A |
| **Implementación** | Permanecen en `CIDcoder-B_1.html`, pero el camino LEAL-B no las usa |
| **Efecto** | Código muerto que puede confundir |
| **Riesgo** | Bajo |
| **Recomendación** | Marcarlas como legado o eliminarlas cuando ya no se necesite compatibilidad con LEAL-A |

### 12.17 Capacidad del campo 0

| | |
|---|---|
| **Fuente** | El corpus de CIDcoder-A ya iba justo de página 0; la 201-B reserva `0010`–`0017` para autoíndices y las escalares empiezan en `0020` |
| **Implementación** | La tabla de punteros debe terminar antes de `0200`; si no cabe, el compilador emite `Página 0 desborda` con el rango ocupado. No se fuerza el ajuste recortando el runtime |
| **Efecto** | 14 programas del corpus no se ejecutan en campo 0 y se omiten de la paridad; otros 3 dependen de constantes ITA2 del 201-A |
| **Riesgo** | Medio: la omisión podría ocultar una regresión si no se revisa el motivo |
| **Recomendación** | Mantener el error explícito y el motivo visible en `TEST/corpus.test.js`; si se reduce el runtime o se reubican datos, mover los programas a la lista de paridad |

### 12.18 Resumen de severidad

| Punto | Severidad | Acción |
|---|---|---|
| 12.1 CI 27 £/$ | Alta | Decidir y corregir tabla |
| 12.2 CI 29–31 | Alta | Verificar manual |
| 12.3 CI 32 | Baja | Confirmar glifo |
| 12.4 CI 36 | Baja | Documentar |
| 12.5 CI 60 | Baja | Documentar o corregir |
| 12.6 CI 35 | Baja | Decidir 171 / 189 / `1/2` |
| 12.7 Código B | Media | Fuera de alcance |
| 12.8 `TIT` + LF+CR | Resuelto | LF+CR; la Ricoh cuenta un salto |
| 12.9 Signo `PER` | Resuelto | `+`/`-`; cero sin signo |
| 12.10 Ancho `PER` | Resuelto | Sin relleno izquierdo; 2/3 espacios de cola |
| 12.11 Precisión flotante | Baja | Documentar |
| 12.12 Homónimo CI | Media | Renombrar o comentar |
| 12.13 Ayuda LEAL | Resuelto | Actualizada el 18/Nov/2026 |
| 12.14 Murray histórico | Media | Verificar |
| 12.15 ISO histórico | Baja | No usar como ASCII |
| 12.16 Tablas legado | Baja | Marcar/eliminar |
| 12.17 Campo 0 | Media | Mantener `Página 0 desborda`; omisiones visibles en el corpus |

---

## 13. Referencias

### 13.1 Fuentes históricas

| Documento | Contenido |
|---|---|
| `LEAL/CODIGOINTERNO.md` | Tablas CI de LEAL-F, Código A y B, análisis ISO/Murray |
| `CID201B/DOChistorico/Consul260/CODIGOINTERNO.md` | Tablas CI de la CID 201-B y relación con el Consul 260 |
| `CID201B/DOChistorico/Consul260/*.png` | Escaneos originales de las tablas |
| `CID201B/CID201B_17.html` | Emulador de referencia: `CI_TO_ASCII`, `handleIO`, ciclos |
| `LEAL/LEALshort.md` | Referencia del lenguaje LEAL |
| `LEAL/LEAL_general.pdf` | Manual general de LEAL |
| `LEAL/LEAL-B.pdf` | Manual de LEAL-B |

### 13.2 Documentos del proyecto

| Documento | Contenido |
|---|---|
| `DOC/PROYECTO-CIDcoder-B.md` | Fases, decisiones y tareas diferidas |
| `DONE/CHANGELOG.md` | Cambios por versión |
| `README.md` | Presentación del proyecto |
| `TEST/README.md` | Arquitectura de pruebas |
| `CIDcoder-A/DOC/MANUAL_LEAL.md` | Línea base de LEAL-A |

### 13.3 Código

| Archivo | Funciones |
|---|---|
| `CIDcoder-B_6.html` | `CI_ASCII`, `ciCodeOfChar`, `encodeCiText`, `LEALCompiler.tit`, `_emitLca*`, `_emitPca*`, `_emitMdi*`, `_emitMid*`, `CORE201B`, `RICOH3000` |
| `TEST/lealb-helpers.js` | `compile`, `run`, `runRows`, `addr`, `readInt`, `readFloat`, `text`, `lines` |
| `TEST/lenb.test.js` | Ensamblador LEN-B |
| `TEST/run201b.test.js` | Ejecución 201-B |

---

## Apéndice A. Tabla inversa ASCII → CI

La siguiente tabla es la que usa `ASCII_CI` en CIDcoder-B. Solo incluye los
caracteres que tienen CI. Las minúsculas se pliegan a mayúsculas.

| ASCII | Código | CI | Octal |
|---|---:|---:|---:|
| `TAB` | 9 | 28 | 34 |
| `LF` | 10 | 31 | 37 |
| `CR` | 13 | 30 | 36 |
| ` ` | 32 | 0 | 00 |
| `!` | 33 | 62 | 76 |
| `"` | 34 | 34 | 42 |
| `$` | 36 | 27 | 33 |
| `%` | 37 | 37 | 45 |
| `'` | 39 | 39 | 47 |
| `(` | 40 | 40 | 50 |
| `)` | 41 | 41 | 51 |
| `*` | 42 | 42 | 52 |
| `+` | 43 | 43 | 53 |
| `,` | 44 | 44 | 54 |
| `-` | 45 | 45 | 55 |
| `.` | 46 | 46 | 56 |
| `/` | 47 | 47 | 57 |
| `0` | 48 | 48 | 60 |
| `1` | 49 | 49 | 61 |
| `2` | 50 | 50 | 62 |
| `3` | 51 | 51 | 63 |
| `4` | 52 | 52 | 64 |
| `5` | 53 | 53 | 65 |
| `6` | 54 | 54 | 66 |
| `7` | 55 | 55 | 67 |
| `8` | 56 | 56 | 70 |
| `9` | 57 | 57 | 71 |
| `:` | 58 | 58 | 72 |
| `;` | 59 | 59 | 73 |
| `=` | 61 | 61 | 75 |
| `?` | 63 | 63 | 77 |
| `@` | 64 | 33 | 41 |
| `A` | 65 | 1 | 01 |
| `B` | 66 | 2 | 02 |
| `C` | 67 | 3 | 03 |
| `D` | 68 | 4 | 04 |
| `E` | 69 | 5 | 05 |
| `F` | 70 | 6 | 06 |
| `G` | 71 | 7 | 07 |
| `H` | 72 | 8 | 10 |
| `I` | 73 | 9 | 11 |
| `J` | 74 | 10 | 12 |
| `K` | 75 | 11 | 13 |
| `L` | 76 | 12 | 14 |
| `M` | 77 | 13 | 15 |
| `N` | 78 | 14 | 16 |
| `O` | 79 | 15 | 17 |
| `P` | 80 | 16 | 20 |
| `Q` | 81 | 17 | 21 |
| `R` | 82 | 18 | 22 |
| `S` | 83 | 19 | 23 |
| `T` | 84 | 20 | 24 |
| `U` | 85 | 21 | 25 |
| `V` | 86 | 22 | 26 |
| `W` | 87 | 23 | 27 |
| `X` | 88 | 24 | 30 |
| `Y` | 89 | 25 | 31 |
| `Z` | 90 | 26 | 32 |
| `_` | 95 | 32 | 40 |
| `«` (171; `½` solo en CP437) | 171 | 35 | 43 |

**No están en la tabla:** `£` (163), `ñ`, `á`, `é`, `í`, `ó`, `ú`, `ü`, y
cualquier carácter fuera del repertorio. Se omiten con warning en `TIT` y se
convierten a CI 0 en `TLA` si llegan por la entrada.

---

## Apéndice B. Glosario

| Término | Definición |
|---|---|
| **CI** | Código Interno de 6 bits de la 201-B |
| **Código A** | Juego principal: mayúsculas, dígitos, puntuación |
| **Código B** | Juego alternativo: minúsculas y controles de cinta |
| **ISO** | Código de 8 bits de la tabla histórica, no ASCII puro |
| **Murray / MTK-2** | Código de 5 bits del teletipo, con LTRS/FIGS |
| **LTRS** | Registro de letras del teletipo |
| **FIGS** | Registro de figuras (dígitos/símbolos) |
| **TAP** | Instrucción que imprime los 6 bits bajos de AC |
| **TLA** | Instrucción que lee un CI y lo OR-ea en AC |
| **LCA** | Instrucción LEAL que lee una cadena hasta CR |
| **PCA** | Instrucción LEAL que imprime una cadena hasta CI 36 |
| **MDI** | Desplazamiento de cadena a la izquierda |
| **MID** | Desplazamiento de cadena a la derecha |
| **Ricoh 3000** | Teletipo de papel modelado por CIDcoder-B |
| **Consul 260** | Teletipo histórico soviético, base de las tablas |
| **Packed** | Dos caracteres por palabra (6+6) |
| **Unpacked** | Un carácter por palabra |

---

## Apéndice C. Notas históricas

1. **La palabra de 12 bits** de la 201-B es la razón del CI de 6 bits. Dos
   caracteres por palabra era una ventaja decisiva en una máquina con memoria
   de ferrita/diodos.

2. **El Código A y el Código B** no son “mayúsculas y minúsculas” en el
   sentido moderno. Son dos juegos que la computadora selecciona según el
   contexto. El Código B también contiene comandos de cinta, lo que sugiere
   que se usaba para controlar periféricos, no solo para texto.

3. **El Consul 260** es un teletipo, no una impresora. Tiene teclado, cinta de
   papel y un juego de caracteres cirílico/latino. Las tablas CI describen cómo
   la 201-B se comunica con él.

4. **La columna ISO** de las tablas parece ser un código de cinta, no ASCII.
   Los valores con bit 8 puesto pueden ser paridad o una convención de la
   máquina. No se debe asumir que `ISO & 0o177` es ASCII.

5. **La columna Murray** de las tablas no coincide con ITA2. Es probable que
   sea MTK-2 soviético o una transcripción con otro orden. La implementación
   de LEAL-B no depende de ella porque usa CI directamente.

6. **El término “Código Interno”** también aparece en la documentación de
   LEAL-F. La tabla es esencialmente la misma, con las diferencias de
   transcripción ya señaladas.

---

## Apéndice D. Checklist de revisión

Use esta lista para revisar el documento y la implementación:

- [ ] Confirmar el glifo de CI 27: `£` o `$`.
- [ ] Confirmar el significado de CI 29, 30 y 31.
- [ ] Confirmar el glifo de CI 32: `-` o `_`.
- [x] Confirmar el uso de CI 36 como terminador.
- [ ] Confirmar si CI 60 debe ser alcanzable desde el texto fuente.
- [ ] Decidir CI 35: conservar 171 y dibujar `½` con tabla, cambiar a 189, o imprimir `1/2` (ver 12.6).
- [ ] Decidir si el Código B entra en el alcance.
- [x] Confirmar el salto de `TIT`: LF+CR.
- [x] Confirmar el signo y ancho de `PER`: `+`/`-`, sin relleno, 2/3 espacios.
- [ ] Confirmar la precisión del flotante y documentarla.
- [ ] Renombrar o comentar el homónimo `CI` de los ciclos.
- [x] Actualizar la ayuda de LEAL-B (sección 7b).
- [ ] Verificar la tabla Murray contra el manual del Consul 260.
- [ ] Verificar la tabla ISO contra el manual.
- [ ] Marcar o eliminar `MURR_L`/`MURR_F` como legado.
- [ ] Añadir los vectores de prueba de la sección 11 a la suite unificada.

---

*Fin del documento. Las correcciones deben reflejarse en este archivo y, cuando
correspondan, en `DOC/PROYECTO-CIDcoder-B.md` y en la ayuda de
`CIDcoder-B_6.html`.*
