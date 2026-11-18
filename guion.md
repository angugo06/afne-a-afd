# Guion del video de demostración

Duración aproximada: 6 a 8 minutos. Antes de grabar, abre la terminal en la carpeta del proyecto (ver el README), agranda la letra de la terminal y borra la pantalla. Lo que va entre comillas es lo que se dice; lo demás es lo que se muestra o se escribe.

## 1. Presentación (30 s)

- Mostrar: la terminal vacía y, un instante, la carpeta con los archivos.
- Decir: "Hola, somos [nombres del equipo]. Este es nuestro proyecto de Lenguajes Formales y Autómatas: un programa en Python que convierte un autómata finito no determinista con transiciones vacías en un autómata finito determinista."
- Decir: "El programa hace dos etapas: primero quita las transiciones vacías con las ε-cláusulas, y después aplica el algoritmo de subconjuntos para obtener el AFD."

## 2. Arrancar el programa (20 s)

- Escribir: `python afn_a_afd.py` y Enter.
- Decir: "Este es el menú. Podemos capturar un autómata a mano o cargar uno de ejemplo. Primero vamos a capturar uno a mano para mostrar la entrada de datos."

## 3. Captura de datos (1 min 30 s)

- Escribir: `1` y Enter.
- Escribir las respuestas de la tabla y explicar cada una en voz alta:

| Pregunta | Escribir | Qué decir |
|---|---|---|
| Cantidad de estados | `4` | "Cuatro estados, el programa acepta de 2 a 10 y los nombra q0 a q3." |
| Alfabeto | `0,1` | "Alfabeto de 0 y 1. El máximo es 4 símbolos." |
| Estado inicial | `q0` | "Un solo estado inicial." |
| Estados finales | `q3` | "Estado final q3. Se valida que exista en Q." |
| δ(q0, 0), δ(q0, 1), δ(q0, ε) | `q0,q1`, `q0`, Enter | "Desde q0 con 0 vamos a q0 o a q1, eso es el no determinismo. La ε se escribe con la letra e." |
| δ(q1, 0), δ(q1, 1), δ(q1, ε) | Enter, `q2`, Enter | "Enter vacío significa que no hay transición." |
| δ(q2, 0), δ(q2, 1), δ(q2, ε) | Enter, Enter, `q3` | "Aquí hay una transición vacía de q2 a q3." |
| δ(q3, 0), δ(q3, 1), δ(q3, ε) | Enter, Enter, Enter | "q3 no tiene salidas." |

- Opcional (10 s): antes de empezar la captura, escribir algo inválido a propósito (por ejemplo `11` estados o un alfabeto con `e`) para mostrar que el programa valida y vuelve a preguntar.

## 4. Las 4 tablas (2 min)

Al terminar la captura el programa muestra todo. Pausar o desplazar la pantalla en cada tabla.

1. Tabla 1, AFN-ε original: "Aquí está la tabla de transición original. La flecha marca el estado inicial q0 y el asterisco el final q3. La última columna es la de las transiciones ε."
2. Tabla 2, ε-cláusulas: "Estas son las ε-cláusulas. La de q2 es {q2, q3} porque desde q2 llegamos a q3 solo con ε. Los demás estados solo se alcanzan a sí mismos."
3. Tabla 3, AFN sin transiciones vacías: "Con las ε-cláusulas obtenemos el AFN sin transiciones vacías. Además, q2 pasa a ser final porque su ε-cláusula contiene a q3, que era final. Los finales nuevos son {q2, q3}."
4. Tabla 4, AFD final: "Aplicamos subconjuntos empezando por la ε-cláusula del inicial, que es {q0}. Salen tres estados: S0 = {q0}, S1 = {q0, q1} y S2 = {q0, q2, q3}. S0 es el inicial y S2 es el único final, porque contiene a q3. En este autómata no aparece el estado muerto."

## 5. Validación de cadenas (1 min 30 s)

- Escribir: `4` y Enter.
- Cadena aceptada: escribir `1001`. Decir: "Termina en 01, el programa muestra el recorrido S0, S0, S1, S1, S2 y la marca como ACEPTADA porque termina en un estado final."
- Cadena rechazada: escribir `0110`. Decir: "No termina en 01, termina en S1 que no es final, así que es RECHAZADA."
- Cadena vacía (opcional): escribir `e`. Decir: "La cadena vacía también se evalúa, empieza en S0, que no es final, y se rechaza."
- Símbolo inválido (opcional): escribir `012`. Decir: "Si usamos un símbolo fuera del alfabeto, el programa avisa."
- Presionar Enter sin escribir nada para volver al menú.

## 6. Ejemplo con estado muerto (1 min)

- Escribir: `2` y Enter (carga el ejemplo a\*b\*c\*).
- Decir: "Ahora cargamos el ejemplo precargado, que reconoce cadenas con a's, luego b's y luego c's en ese orden."
- Mostrar la Tabla 4: "Aquí sí aparece el estado muerto ∅. Es el que se alcanza cuando la cadena rompe el orden, por ejemplo una a después de una b. No es de aceptación y todas sus transiciones regresan a sí mismo."
- Escribir: `4`, luego `abc` y decir que se acepta. Luego `ba` y decir: "Se rechaza porque cae al estado muerto." Enter para volver.

## 7. Gráfica, punto extra (1 min)

- Escribir: `5` y Enter.
- Decir: "Para el punto extra el programa genera el código DOT de Graphviz y, con Graphviz instalado, la imagen del AFD."
- Mostrar: la imagen `afd.png` que se abre (o el código en el visor en línea si no hay Graphviz).
- Señalar: "La flecha que llega desde un punto es el estado inicial. Los estados con doble círculo son los finales. Las flechas llevan sus símbolos, y aquí se ve el estado muerto ∅."

## 8. Cierre (15 s)

- Escribir: `0` y Enter para salir.
- Decir: "Eso es todo. El código está en un solo archivo de Python y el enlace al repositorio va en la descripción del video. Gracias."

## Lista rápida antes de grabar

- La terminal abre en la carpeta correcta y `python afn_a_afd.py` funciona.
- La letra de la terminal es grande y se leen bien las tablas.
- Graphviz instalado (si no, tener abierto https://dreampuf.github.io/GraphvizOnline).
- Escribir los nombres reales del equipo en la presentación.
- Subir el video a YouTube (no listado o público) y pegar el enlace en la entrega.
