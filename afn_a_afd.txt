import os
import shutil
import subprocess
import sys

EPSILON = "e"
ALIAS_EPSILON = ("e", "@", "ε")
MIN_ESTADOS = 2
MAX_ESTADOS = 10
MAX_SIMBOLOS = 4
CARPETA = os.path.dirname(os.path.abspath(__file__))
ARCHIVO_DOT = os.path.join(CARPETA, "afd.dot")
ARCHIVO_PNG = os.path.join(CARPETA, "afd.png")
ARCHIVO_SVG = os.path.join(CARPETA, "afd.svg")


def separar(texto):
    return texto.replace(",", " ").split()


def texto_conjunto(conjunto, estados):
    if not conjunto:
        return "∅"
    return "{" + ",".join(sorted(conjunto, key=estados.index)) + "}"


def simbolo_visible(simbolo):
    return "ε" if simbolo == EPSILON else simbolo


def leer_entero(mensaje, minimo, maximo):
    while True:
        texto = input(mensaje).strip()
        if texto.isdigit() and minimo <= int(texto) <= maximo:
            return int(texto)
        print(f"  Error: escribe un número entero entre {minimo} y {maximo}.")


def leer_alfabeto():
    while True:
        simbolos = separar(input(f"Alfabeto Σ (1 a {MAX_SIMBOLOS} símbolos separados por coma, ejemplo a,b): "))
        if not 1 <= len(simbolos) <= MAX_SIMBOLOS:
            print(f"  Error: el alfabeto debe tener de 1 a {MAX_SIMBOLOS} símbolos.")
        elif any(len(s) != 1 for s in simbolos):
            print("  Error: cada símbolo debe ser un solo carácter y separarse con coma.")
        elif any(s in ALIAS_EPSILON for s in simbolos):
            print("  Error: 'e', '@' y 'ε' están reservados para la transición vacía.")
        elif len(set(simbolos)) != len(simbolos):
            print("  Error: hay símbolos repetidos.")
        else:
            return simbolos


def interpretar_estados(texto, estados):
    resultado = set()
    for token in separar(texto):
        if token in estados:
            resultado.add(token)
        elif "q" + token in estados:
            resultado.add("q" + token)
        else:
            return None
    return resultado


def leer_estados(mensaje, estados, minimo=0, maximo=None):
    while True:
        conjunto = interpretar_estados(input(mensaje), estados)
        if conjunto is None:
            print("  Error: usa solo estados existentes (" + ", ".join(estados) + ").")
        elif len(conjunto) < minimo:
            print(f"  Error: debes indicar al menos {minimo} estado(s).")
        elif maximo is not None and len(conjunto) > maximo:
            print(f"  Error: solo se permite {maximo} estado(s).")
        else:
            return conjunto


def crear_delta(estados, alfabeto, transiciones):
    delta = {(q, s): set() for q in estados for s in alfabeto + [EPSILON]}
    for clave, destinos in transiciones.items():
        delta[clave] = set(destinos)
    return delta


def capturar_automata():
    print("\n--- Captura del AFN-ε ---")
    cantidad = leer_entero(f"Cantidad de estados ({MIN_ESTADOS} a {MAX_ESTADOS}): ", MIN_ESTADOS, MAX_ESTADOS)
    estados = [f"q{i}" for i in range(cantidad)]
    print("Estados Q = " + ", ".join(estados))
    alfabeto = leer_alfabeto()
    print(f"La transición vacía se escribe con '{EPSILON}' (también se acepta '@').")
    inicial = leer_estados("Estado inicial (uno solo, ejemplo q0): ", estados, 1, 1).pop()
    finales = leer_estados("Estados finales (separados por coma, ejemplo q2,q3): ", estados, 1)
    print("\nTransiciones: escribe los estados destino separados por coma.")
    print("Si no hay transición, solo presiona Enter.")
    delta = {}
    for estado in estados:
        for simbolo in alfabeto + [EPSILON]:
            delta[(estado, simbolo)] = leer_estados(f"  δ({estado}, {simbolo_visible(simbolo)}) = ", estados)
    return {"estados": estados, "alfabeto": alfabeto, "delta": delta, "inicial": inicial, "finales": finales}


def automata_ejemplo():
    estados = ["q0", "q1", "q2"]
    alfabeto = ["a", "b", "c"]
    transiciones = {
        ("q0", "a"): {"q0"},
        ("q0", EPSILON): {"q1"},
        ("q1", "b"): {"q1"},
        ("q1", EPSILON): {"q2"},
        ("q2", "c"): {"q2"},
    }
    return {
        "estados": estados,
        "alfabeto": alfabeto,
        "delta": crear_delta(estados, alfabeto, transiciones),
        "inicial": "q0",
        "finales": {"q2"},
    }


def calcular_cerradura(estado, delta):
    visitados = {estado}
    pendientes = [estado]
    while pendientes:
        actual = pendientes.pop()
        for destino in delta[(actual, EPSILON)]:
            if destino not in visitados:
                visitados.add(destino)
                pendientes.append(destino)
    return frozenset(visitados)


def calcular_cerraduras(automata):
    return {q: calcular_cerradura(q, automata["delta"]) for q in automata["estados"]}


def eliminar_vacias(automata, cerraduras):
    delta = {}
    for estado in automata["estados"]:
        for simbolo in automata["alfabeto"]:
            alcanzados = set()
            for p in cerraduras[estado]:
                alcanzados |= automata["delta"][(p, simbolo)]
            resultado = set()
            for r in alcanzados:
                resultado |= cerraduras[r]
            delta[(estado, simbolo)] = frozenset(resultado)
    finales = {q for q in automata["estados"] if cerraduras[q] & automata["finales"]}
    return {
        "estados": automata["estados"],
        "alfabeto": automata["alfabeto"],
        "delta": delta,
        "inicial": automata["inicial"],
        "finales": finales,
    }


def construir_afd(afn, cerraduras):
    inicial = cerraduras[afn["inicial"]]
    subconjuntos = [inicial]
    delta = {}
    indice = 0
    while indice < len(subconjuntos):
        actual = subconjuntos[indice]
        indice += 1
        for simbolo in afn["alfabeto"]:
            destino = frozenset().union(*[afn["delta"][(q, simbolo)] for q in actual])
            delta[(actual, simbolo)] = destino
            if destino not in subconjuntos:
                subconjuntos.append(destino)
    nombres = {}
    contador = 0
    for subconjunto in subconjuntos:
        if subconjunto:
            nombres[subconjunto] = f"S{contador}"
            contador += 1
        else:
            nombres[subconjunto] = "∅"
    finales = {s for s in subconjuntos if s & afn["finales"]}
    return {
        "estados_afn": afn["estados"],
        "alfabeto": afn["alfabeto"],
        "subconjuntos": subconjuntos,
        "nombres": nombres,
        "delta": delta,
        "inicial": inicial,
        "finales": finales,
    }


def imprimir_tabla(encabezados, filas):
    columnas = list(zip(encabezados, *filas))
    anchos = [max(len(celda) for celda in columna) for columna in columnas]
    separador = "+" + "+".join("-" * (ancho + 2) for ancho in anchos) + "+"

    def linea(celdas):
        return "| " + " | ".join(c.ljust(a) for c, a in zip(celdas, anchos)) + " |"

    print(separador)
    print(linea(encabezados))
    print(separador)
    for fila in filas:
        print(linea(fila))
    print(separador)


def marcar(nombre, es_inicial, es_final):
    return ("→" if es_inicial else " ") + ("*" if es_final else " ") + " " + nombre


def titulo(texto):
    print("\n" + "=" * 64)
    print(texto)
    print("=" * 64)


def mostrar_quintupla(automata):
    estados = automata["estados"]
    print("M = (Q, Σ, δ, q0, F)")
    print("  Q  = " + texto_conjunto(estados, estados))
    print("  Σ  = " + texto_conjunto(automata["alfabeto"], automata["alfabeto"]))
    print("  q0 = " + automata["inicial"])
    print("  F  = " + texto_conjunto(automata["finales"], estados))
    print(f"  ε  = '{EPSILON}' (transición vacía)")


def mostrar_tabla_afne(automata):
    titulo("TABLA 1. Tabla de transición del AFN-ε original")
    estados = automata["estados"]
    simbolos = automata["alfabeto"] + [EPSILON]
    encabezados = ["Estado"] + [simbolo_visible(s) for s in simbolos]
    filas = []
    for q in estados:
        fila = [marcar(q, q == automata["inicial"], q in automata["finales"])]
        fila += [texto_conjunto(automata["delta"][(q, s)], estados) for s in simbolos]
        filas.append(fila)
    imprimir_tabla(encabezados, filas)
    print("→ = estado inicial     * = estado final")


def mostrar_cerraduras(automata, cerraduras):
    titulo("TABLA 2. Cálculo de las ε-cláusulas")
    estados = automata["estados"]
    filas = []
    for q in estados:
        directos = texto_conjunto(automata["delta"][(q, EPSILON)], estados)
        filas.append([q, directos, texto_conjunto(cerraduras[q], estados)])
    imprimir_tabla(["Estado", "Destinos con ε directos", "ε-cláusula"], filas)
    print("La ε-cláusula de un estado incluye al propio estado y todos los que se")
    print("alcanzan desde él usando solo transiciones ε (una o varias seguidas).")


def mostrar_tabla_afn(automata, afn, cerraduras):
    titulo("TABLA 3. Tabla de transición del AFN sin transiciones vacías")
    estados = afn["estados"]
    encabezados = ["Estado"] + afn["alfabeto"]
    filas = []
    for q in estados:
        fila = [marcar(q, q == afn["inicial"], q in afn["finales"])]
        fila += [texto_conjunto(afn["delta"][(q, s)], estados) for s in afn["alfabeto"]]
        filas.append(fila)
    imprimir_tabla(encabezados, filas)
    print("→ = estado inicial     * = estado final")
    print("Estado inicial: " + afn["inicial"])
    print("Estados finales F': " + texto_conjunto(afn["finales"], estados))
    nuevos = sorted(afn["finales"] - automata["finales"], key=estados.index)
    for q in nuevos:
        alcanzados = cerraduras[q] & automata["finales"]
        print(f"  {q} es final nuevo: su ε-cláusula contiene al final {texto_conjunto(alcanzados, estados)}")
    if not nuevos:
        print("  No se agregaron estados finales nuevos.")


def mostrar_tabla_afd(afd):
    titulo("TABLA 4. Tabla de transición del AFD final (determinista)")
    estados = afd["estados_afn"]
    nombres = afd["nombres"]
    encabezados = ["Estado", "Subconjunto"] + afd["alfabeto"]
    filas = []
    for s in afd["subconjuntos"]:
        fila = [marcar(nombres[s], s == afd["inicial"], s in afd["finales"]), texto_conjunto(s, estados)]
        fila += [nombres[afd["delta"][(s, a)]] for a in afd["alfabeto"]]
        filas.append(fila)
    imprimir_tabla(encabezados, filas)
    print("→ = estado inicial     * = estado final (de aceptación)")
    print(f"Estado inicial: {nombres[afd['inicial']]} = {texto_conjunto(afd['inicial'], estados)}")
    finales = [s for s in afd["subconjuntos"] if s in afd["finales"]]
    print("Estados finales:")
    for s in finales:
        print(f"  {nombres[s]} = {texto_conjunto(s, estados)}")
    if not finales:
        print("  (ninguno)")
    if frozenset() in afd["nombres"]:
        print("Estado muerto: ∅ (no es de aceptación; todas sus transiciones regresan a ∅)")
    else:
        print("Estado muerto: no existe, ninguna transición llega al conjunto vacío.")


def convertir(automata):
    cerraduras = calcular_cerraduras(automata)
    afn = eliminar_vacias(automata, cerraduras)
    afd = construir_afd(afn, cerraduras)
    return cerraduras, afn, afd


def mostrar_resultados(automata, cerraduras, afn, afd):
    titulo("AUTÓMATA CAPTURADO")
    mostrar_quintupla(automata)
    mostrar_tabla_afne(automata)
    mostrar_cerraduras(automata, cerraduras)
    mostrar_tabla_afn(automata, afn, cerraduras)
    mostrar_tabla_afd(afd)


def describir(subconjunto, afd):
    nombre = afd["nombres"][subconjunto]
    if not subconjunto:
        return nombre
    return f"{nombre} {texto_conjunto(subconjunto, afd['estados_afn'])}"


def evaluar_cadena(afd, cadena):
    actual = afd["inicial"]
    recorrido = [actual]
    for simbolo in cadena:
        actual = afd["delta"][(actual, simbolo)]
        recorrido.append(actual)
    return actual in afd["finales"], recorrido


def probar_cadenas(afd):
    titulo("VALIDACIÓN DE CADENAS CON EL AFD")
    print("Símbolos permitidos: " + ", ".join(afd["alfabeto"]))
    print("Escribe una cadena y presiona Enter. Escribe 'e' para la cadena vacía.")
    print("Presiona Enter sin escribir nada para volver al menú.")
    while True:
        texto = input("\nCadena: ").strip()
        if texto == "":
            return
        cadena = "" if texto in ALIAS_EPSILON else texto
        invalidos = sorted({c for c in cadena if c not in afd["alfabeto"]})
        if invalidos:
            print("  Error: la cadena tiene símbolos que no están en Σ: " + ", ".join(invalidos))
            continue
        aceptada, recorrido = evaluar_cadena(afd, cadena)
        print(f"  Cadena: {cadena if cadena else 'ε (vacía)'}")
        print("  Paso 0: empieza en " + describir(recorrido[0], afd))
        for numero, simbolo in enumerate(cadena, start=1):
            print(f"  Paso {numero}: lee '{simbolo}' y pasa a {describir(recorrido[numero], afd)}")
        nombres = [afd["nombres"][s] for s in recorrido]
        print("  Recorrido: " + " -> ".join(nombres))
        final = describir(recorrido[-1], afd)
        if aceptada:
            print(f"  Resultado: ACEPTADA (termina en {final}, que es un estado final)")
        else:
            print(f"  Resultado: RECHAZADA (termina en {final}, que no es un estado final)")


def generar_dot(afd):
    nombres = afd["nombres"]
    estados = afd["estados_afn"]
    lineas = [
        "digraph AFD {",
        "  rankdir=LR;",
        '  node [shape=circle, fontname="Arial"];',
        '  edge [fontname="Arial"];',
        '  inicio [shape=point, label=""];',
    ]
    for s in afd["subconjuntos"]:
        forma = "doublecircle" if s in afd["finales"] else "circle"
        if s:
            etiqueta = nombres[s] + "\\n" + texto_conjunto(s, estados)
        else:
            etiqueta = "∅"
        lineas.append(f'  "{nombres[s]}" [shape={forma}, label="{etiqueta}"];')
    lineas.append(f'  inicio -> "{nombres[afd["inicial"]]}";')
    aristas = {}
    for (origen, simbolo), destino in afd["delta"].items():
        aristas.setdefault((origen, destino), []).append(simbolo)
    for (origen, destino), simbolos in aristas.items():
        etiqueta = ",".join(simbolos)
        lineas.append(f'  "{nombres[origen]}" -> "{nombres[destino]}" [label="{etiqueta}"];')
    lineas.append("}")
    return "\n".join(lineas) + "\n"


def generar_grafica(afd):
    titulo("GRÁFICA DEL AFD FINAL (Graphviz)")
    codigo = generar_dot(afd)
    with open(ARCHIVO_DOT, "w", encoding="utf-8") as archivo:
        archivo.write(codigo)
    print(codigo)
    print("Se guardó el código DOT en: " + ARCHIVO_DOT)
    if shutil.which("dot") is None:
        print("\nGraphviz no está instalado, así que no se creó la imagen.")
        print("Para verla, copia el código de arriba y pégalo en:")
        print("  https://dreampuf.github.io/GraphvizOnline")
        print("O instala Graphviz (https://graphviz.org/download) y vuelve a elegir esta opción.")
        return
    try:
        subprocess.run(["dot", "-Tpng", ARCHIVO_DOT, "-o", ARCHIVO_PNG], check=True)
        subprocess.run(["dot", "-Tsvg", ARCHIVO_DOT, "-o", ARCHIVO_SVG], check=True)
    except (subprocess.CalledProcessError, OSError):
        print("Graphviz no pudo crear la imagen. Usa el código DOT con el visor en línea.")
        return
    print("Imagen PNG guardada en: " + ARCHIVO_PNG)
    print("Imagen SVG guardada en: " + ARCHIVO_SVG)
    if sys.platform == "win32":
        try:
            os.startfile(ARCHIVO_PNG)
        except OSError:
            print("Abre el archivo PNG manualmente para verlo.")


def mostrar_menu(hay_automata):
    print("\n" + "-" * 64)
    print("CONVERSIÓN DE AFN-ε A AFD")
    print("-" * 64)
    print("1) Capturar un autómata AFN-ε a mano")
    print("2) Cargar el autómata de ejemplo (a*b*c*)")
    if hay_automata:
        print("3) Ver de nuevo las 4 tablas")
        print("4) Probar cadenas con el AFD")
        print("5) Generar la gráfica del AFD (punto extra)")
    print("0) Salir")


def cargar(automata):
    cerraduras, afn, afd = convertir(automata)
    mostrar_resultados(automata, cerraduras, afn, afd)
    return cerraduras, afn, afd


def main():
    try:
        sys.stdout.reconfigure(encoding="utf-8")
    except (AttributeError, ValueError):
        pass
    automata = cerraduras = afn = afd = None
    try:
        while True:
            mostrar_menu(automata is not None)
            opcion = input("Elige una opción: ").strip()
            if opcion == "1":
                automata = capturar_automata()
                cerraduras, afn, afd = cargar(automata)
            elif opcion == "2":
                automata = automata_ejemplo()
                cerraduras, afn, afd = cargar(automata)
            elif opcion == "3" and automata is not None:
                mostrar_resultados(automata, cerraduras, afn, afd)
            elif opcion == "4" and automata is not None:
                probar_cadenas(afd)
            elif opcion == "5" and automata is not None:
                generar_grafica(afd)
            elif opcion == "0":
                break
            else:
                print("Opción no válida.")
    except (EOFError, KeyboardInterrupt):
        print()
    print("Programa terminado.")


if __name__ == "__main__":
    main()
