# Conversión de AFN-ε a AFD

Programa de consola en Python que recibe un autómata finito no determinista con transiciones vacías (AFN-ε), le quita las transiciones vacías y lo convierte a un autómata finito determinista (AFD). Proyecto de Lenguajes Formales y Autómatas.

## Qué hace

1. Captura el autómata M = (Q, Σ, δ, q0, F): de 2 a 10 estados, máximo 4 símbolos, un solo estado inicial y al menos un estado final.
2. Calcula la ε-cláusula de cada estado.
3. Obtiene el AFN equivalente sin transiciones vacías (con sus estados finales nuevos).
4. Construye el AFD por subconjuntos, mostrando el estado muerto ∅ cuando existe.
5. Muestra en pantalla las 4 tablas pedidas: AFN-ε original, ε-cláusulas, AFN sin ε y AFD final.
6. Valida cadenas con el AFD y muestra el recorrido de estados (ACEPTADA o RECHAZADA).
7. Punto extra: genera el código DOT de Graphviz y, si Graphviz está instalado, la imagen PNG y SVG del AFD.

## Qué necesitas

- Python 3.8 o más nuevo. Para revisar si lo tienes, abre una terminal y escribe `python --version` (en Mac o Linux, `python3 --version`).
- No se instala ninguna biblioteca. Solo se usa la biblioteca estándar de Python.
- Graphviz es opcional y solo sirve para dibujar la imagen. El programa funciona sin él.

## Cómo correrlo (Windows)

1. Descarga o copia esta carpeta a tu computadora.
2. Abre la carpeta en el Explorador de archivos.
3. Haz clic en la barra de dirección (donde dice la ruta de la carpeta), escribe `cmd` y presiona Enter. Se abre una ventana negra ya ubicada en esa carpeta.
4. Escribe este comando y presiona Enter:

   ```
   python afn_a_afd.py
   ```

5. Aparece un menú. Escribe el número de la opción y presiona Enter.

En Mac o Linux, abre la terminal en la carpeta y usa `python3 afn_a_afd.py`.

Si en Windows los símbolos `ε`, `∅` o `→` se ven como signos de pregunta, usa Windows Terminal o PowerShell en lugar de `cmd`.

## Cómo usarlo

El menú tiene estas opciones:

| Opción | Qué hace |
|---|---|
| 1 | Captura un autómata a mano |
| 2 | Carga el autómata de ejemplo (el lenguaje a\*b\*c\*) y muestra todo al instante |
| 3 | Vuelve a mostrar las 4 tablas |
| 4 | Prueba cadenas con el AFD |
| 5 | Genera la gráfica del AFD |
| 0 | Sale del programa |

Las opciones 3, 4 y 5 aparecen cuando ya hay un autómata cargado.

Si es la primera vez, elige la opción 2: carga el ejemplo y muestra las 4 tablas sin escribir nada. Luego elige la 4 y prueba, por ejemplo, `abc` (aceptada) y `ba` (rechazada).

### Captura manual (opción 1)

- Los estados se llaman `q0`, `q1`, `q2`, etc. Solo escribes cuántos son.
- Alfabeto: símbolos de un carácter separados por coma, por ejemplo `a,b` o `0,1`.
- La transición vacía se escribe con la letra `e` (también se acepta `@`). Por eso `e`, `@` y `ε` no se pueden usar en el alfabeto.
- Para el estado inicial escribe uno solo, por ejemplo `q0`. Para los finales, uno o varios separados por coma, por ejemplo `q2,q3`.
- Para cada pareja estado-símbolo (incluida ε) escribe los estados destino separados por coma, por ejemplo `q0,q1`. Si no hay transición, solo presiona Enter.
- Si escribes algo inválido, el programa avisa y vuelve a preguntar.

Ejemplo corto para probar la captura manual (cadenas que terminan en `01`):

| Pregunta | Escribe |
|---|---|
| Cantidad de estados | `4` |
| Alfabeto | `0,1` |
| Estado inicial | `q0` |
| Estados finales | `q3` |
| δ(q0, 0) / δ(q0, 1) / δ(q0, ε) | `q0,q1` / `q0` / Enter |
| δ(q1, 0) / δ(q1, 1) / δ(q1, ε) | Enter / `q2` / Enter |
| δ(q2, 0) / δ(q2, 1) / δ(q2, ε) | Enter / Enter / `q3` |
| δ(q3, 0) / δ(q3, 1) / δ(q3, ε) | Enter / Enter / Enter |

### Probar cadenas (opción 4)

Escribe una cadena formada con los símbolos del alfabeto y presiona Enter. El programa muestra cada paso, el recorrido de estados y si la cadena es ACEPTADA o RECHAZADA. Escribe `e` para probar la cadena vacía. Presiona Enter sin escribir nada para volver al menú.

### Gráfica (opción 5)

El programa siempre crea el archivo `afd.dot` y muestra su contenido en pantalla.

- Si Graphviz está instalado, también crea `afd.png` y `afd.svg` en la misma carpeta (en Windows abre el PNG solo).
- Si no lo está, copia el código que aparece en pantalla y pégalo en https://dreampuf.github.io/GraphvizOnline para ver el dibujo.
- Para instalar Graphviz en Windows: descárgalo de https://graphviz.org/download y marca la opción de agregarlo al PATH durante la instalación. Después cierra y vuelve a abrir la terminal.

En la gráfica, la flecha que viene de un punto señala el estado inicial, los estados finales tienen doble círculo y cada flecha lleva los símbolos de su transición.

## Archivos

| Archivo | Contenido |
|---|---|
| `afn_a_afd.py` | Programa completo (un solo archivo) |
| `afn_a_afd.txt` | Copia idéntica del código en texto plano, para entregar en la plataforma |
| `guion.md` | Guion para grabar el video de demostración |
| `afd.dot`, `afd.png`, `afd.svg` | Resultado de ejemplo de la opción 5 con el autómata de ejemplo |

Si cambias algo en `afn_a_afd.py`, vuelve a copiarlo a `afn_a_afd.txt`.
