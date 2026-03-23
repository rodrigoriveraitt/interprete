## 📝 Código completo

```python
import tkinter as tk
from tkinter import Toplevel, PhotoImage
import re

# =========================
# COLORES
# =========================

color_fondo = "#1F1147"
color_panel = "#46178F"
color_boton = "#864CBF"
color_texto = "white"
color_correcto = "#2ECC71"
color_error = "#E74C3C"

historial = []

# =========================
# FUNCIONES DEL LENGUAJE
# =========================

alfabeto = set("0123456789+-*/() ")

def pertenece_alfabeto(cadena):
    return all(c in alfabeto for c in cadena)

patron = r'^[0-9+\-*/() ]+$'

def es_expresion_valida(cadena):
    return re.match(patron, cadena) is not None

def parentesis_balanceados(cadena):
    contador = 0
    for c in cadena:
        if c == "(":
            contador += 1
        elif c == ")":
            contador -= 1
        if contador < 0:
            return False
    return contador == 0

def detectar_error(cadena):

    cadena = cadena.replace(" ", "")

    if cadena == "":
        return "Expresión vacía"

    if cadena[0] in "+*/":
        return "No puede iniciar con operador"

    if cadena[-1] in "+-*/":
        return "No puede terminar con operador"

    if "++" in cadena or "--" in cadena or "**" in cadena or "//" in cadena:
        return "Operadores duplicados"

    return None

def evaluar(cadena):
    try:
        # Agregar multiplicación implícita entre: número y paréntesis, paréntesis y número, paréntesis y paréntesis
        cadena_mejorada = ""
        for i, c in enumerate(cadena):
            cadena_mejorada += c
            
            # Si es un dígito y el siguiente es paréntesis de apertura
            if c.isdigit() and i + 1 < len(cadena) and cadena[i + 1] == "(":
                cadena_mejorada += "*"
            
            # Si es paréntesis de cierre y el siguiente es dígito
            elif c == ")" and i + 1 < len(cadena) and cadena[i + 1].isdigit():
                cadena_mejorada += "*"
            
            # Si es paréntesis de cierre y el siguiente es paréntesis de apertura
            elif c == ")" and i + 1 < len(cadena) and cadena[i + 1] == "(":
                cadena_mejorada += "*"
        
        return eval(cadena_mejorada)
    except:
        return None

# =========================
# TOKENIZADOR
# =========================

def tokenizar(cadena):

    tokens = re.findall(r'\d+|[+\-*/()]', cadena)

    resultado = []

    for t in tokens:

        if t.isdigit():
            tipo = "NUMERO"

        elif t in "+-*/":
            tipo = "OPERADOR"

        elif t in "()":
            tipo = "PARENTESIS"

        resultado.append((t, tipo))

    return resultado

# =========================
# SECCION ANALIZADOR (MEJORADA)
# =========================

def abrir_analizador():

    ventana = Toplevel(root)
    ventana.title("Analizador de Expresiones")
    ventana.geometry("600x850")
    ventana.configure(bg=color_panel)

    # =========================
    # TITULO
    # =========================

    titulo = tk.Label(
        ventana,
        text="🔍 ANALIZADOR DE EXPRESIONES",
        font=("Arial", 16, "bold"),
        bg=color_panel,
        fg=color_correcto
    )
    titulo.pack(pady=10)

    # =========================
    # ENTRADA DE TEXTO
    # =========================

    frame_entrada = tk.Frame(ventana, bg=color_panel)
    frame_entrada.pack(pady=10)

    tk.Label(
        frame_entrada,
        text="Expresión:",
        font=("Arial", 10, "bold"),
        bg=color_panel,
        fg="white"
    ).pack(side="left", padx=5)

    entrada = tk.Entry(
        frame_entrada,
        font=("Arial", 18),
        width=25,
        justify="center",
        bg="white",
        fg="black"
    )
    entrada.pack(side="left", padx=5)

    # =========================
    # RESULTADO
    # =========================

    resultado = tk.Label(
        ventana,
        text="",
        font=("Arial", 14, "bold"),
        bg=color_panel,
        fg="black"
    )
    resultado.pack(pady=10)

    # =========================
    # PANEL DE VALIDACIONES
    # =========================

    frame_validaciones = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_validaciones.pack(pady=10, padx=10, fill="both", expand=False)

    tk.Label(
        frame_validaciones,
        text="📋 VALIDACIONES",
        font=("Arial", 12, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=5)

    validaciones_label = tk.Label(
        frame_validaciones,
        text="",
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    )
    validaciones_label.pack(pady=10, padx=10)

    # =========================
    # PANEL DE TOKENS
    # =========================

    frame_tokens = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_tokens.pack(pady=10, padx=10, fill="both", expand=True)

    tk.Label(
        frame_tokens,
        text="🔤 TOKENS",
        font=("Arial", 12, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=5)

    # Canvas con Scrollbar para tokens
    canvas = tk.Canvas(frame_tokens, bg="#2C1550", highlightthickness=0, height=150)
    scrollbar = tk.Scrollbar(frame_tokens, orient="vertical", command=canvas.yview)
    scrollable_frame = tk.Frame(canvas, bg="#2C1550")

    scrollable_frame.bind(
        "<Configure>",
        lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
    )

    canvas.create_window((0, 0), window=scrollable_frame, anchor="nw")
    canvas.configure(yscrollcommand=scrollbar.set)

    tokens_label = tk.Label(
        scrollable_frame,
        text="",
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    )
    tokens_label.pack(pady=10, padx=10)

    canvas.pack(side="left", fill="both", expand=True, padx=5, pady=5)
    scrollbar.pack(side="right", fill="y")

    # =========================
    # FUNCIONES
    # =========================

    def agregar(valor):
        entrada.insert(tk.END, valor)
        actualizar_preview()

    def limpiar():
        entrada.delete(0, tk.END)
        resultado.config(text="")
        validaciones_label.config(text="")
        tokens_label.config(text="")

    def actualizar_preview():
        """Actualiza validaciones en tiempo real"""
        cadena = entrada.get()

        validaciones_texto = ""

        # Validación 1: Alfabeto
        if cadena:
            if pertenece_alfabeto(cadena):
                validaciones_texto += "✅ Alfabeto válido\n"
            else:
                validaciones_texto += "❌ Contiene símbolos inválidos\n"

            # Validación 2: Estructura
            if es_expresion_valida(cadena):
                validaciones_texto += "✅ Estructura válida\n"
            else:
                validaciones_texto += "❌ Estructura inválida\n"

            # Validación 3: Paréntesis
            if parentesis_balanceados(cadena):
                validaciones_texto += "✅ Paréntesis balanceados\n"
            else:
                validaciones_texto += "❌ Paréntesis desbalanceados\n"

            # Validación 4: Errores sintácticos
            error = detectar_error(cadena)
            if error:
                validaciones_texto += f"❌ {error}\n"
            else:
                validaciones_texto += "✅ Sin errores sintácticos\n"

        validaciones_label.config(text=validaciones_texto)

    def calcular():

        cadena = entrada.get()

        if not pertenece_alfabeto(cadena):
            resultado.config(text="❌ Símbolos inválidos", fg=color_error)
            return

        if not es_expresion_valida(cadena):
            resultado.config(text="❌ Expresión inválida", fg=color_error)
            return

        if not parentesis_balanceados(cadena):
            resultado.config(text="❌ Paréntesis incorrectos", fg=color_error)
            return

        error = detectar_error(cadena)

        if error:
            resultado.config(text=f"❌ {error}", fg=color_error)
            return

        valor = evaluar(cadena)

        if valor is None:
            resultado.config(text="❌ Error al evaluar", fg=color_error)
            return

        # Mostrar resultado
        resultado.config(text=f"✅ Resultado = {valor}", fg=color_correcto)

        # Agregar al historial
        historial.append(f"{cadena} = {valor}")

        # GENERAR TOKENS
        tokens = tokenizar(cadena)

        texto = f"{'Token':<15} {'Tipo':<15}\n"
        texto += "─" * 30 + "\n"

        colores_token = {
            "NUMERO": "🔢",
            "OPERADOR": "➕",
            "PARENTESIS": "🟠"
        }

        for t in tokens:
            icono = colores_token.get(t[1], "•")
            texto += f"{icono} {t[0]:<12} {t[1]:<15}\n"

        tokens_label.config(text=texto)

    # =========================
    # CALCULADORA - PARTE 1
    # =========================

    frame_calc = tk.Frame(ventana, bg=color_panel)
    frame_calc.pack(pady=10)

    tk.Label(
        frame_calc,
        text="🧮 CALCULADORA",
        font=("Arial", 12, "bold"),
        bg=color_panel,
        fg=color_correcto
    ).pack(pady=5)

    botones = [
        ['7', '8', '9', '/'],
        ['4', '5', '6', '*'],
        ['1', '2', '3', '-'],
        ['0', '+', '(', ')']
    ]

    frame_botones = tk.Frame(frame_calc, bg=color_panel)
    frame_botones.pack()

    for fila in botones:
        fila_frame = tk.Frame(frame_botones, bg=color_panel)
        fila_frame.pack()

        for b in fila:
            boton = tk.Button(
                fila_frame,
                text=b,
                font=("Arial", 14, "bold"),
                width=5,
                height=2,
                bg=color_boton,
                fg="black",
                activebackground="#A855D8",
                command=lambda x=b: agregar(x)
            )
            boton.pack(side="left", padx=3, pady=3)

    # =========================
    # BOTONES DE CONTROL
    # =========================

    frame_botones_control = tk.Frame(ventana, bg=color_panel)
    frame_botones_control.pack(pady=10)

    tk.Button(
        frame_botones_control,
        text="🔍 Calcular",
        font=("Arial", 11, "bold"),
        bg="#00C2FF",
        fg="black",
        width=15,
        command=calcular
    ).pack(side="left", padx=5)

    tk.Button(
        frame_botones_control,
        text="🗑️ Limpiar",
        font=("Arial", 11, "bold"),
        bg=color_error,
        fg="black",
        width=15,
        command=limpiar
    ).pack(side="left", padx=5)

    # =========================
    # BOTON VOLVER
    # =========================

    tk.Button(
        ventana,
        text="◀ Volver al menú",
        font=("Arial", 10),
        bg="#555555",
        fg="black",
        command=ventana.destroy
    ).pack(pady=10)

    # Actualizar preview al escribir
    entrada.bind("<KeyRelease>", lambda e: actualizar_preview())

# =========================
# SECCION VALIDADOR (MEJORADA)
# =========================

def abrir_validador():

    ventana = Toplevel(root)
    ventana.title("Validador de Lenguaje")
    ventana.geometry("700x800")
    ventana.configure(bg=color_panel)

    # =========================
    # TITULO
    # =========================

    titulo = tk.Label(
        ventana,
        text="✅ VALIDADOR DE LENGUAJE FORMAL",
        font=("Arial", 16, "bold"),
        bg=color_panel,
        fg=color_correcto
    )
    titulo.pack(pady=15)

    subtitulo = tk.Label(
        ventana,
        text="Verifica si tu expresión pertenece al lenguaje",
        font=("Arial", 11),
        bg=color_panel,
        fg="#A0A0A0"
    )
    subtitulo.pack(pady=5)

    # =========================
    # ENTRADA DE EXPRESIÓN
    # =========================

    frame_entrada = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_entrada.pack(pady=15, padx=20, fill="x")

    tk.Label(
        frame_entrada,
        text="📝 Ingresa tu expresión:",
        font=("Arial", 11, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10)

    entrada = tk.Entry(
        frame_entrada,
        font=("Arial", 16, "bold"),
        width=30,
        justify="center",
        bg="white",
        fg="black",
        relief="flat",
        bd=0
    )
    entrada.pack(pady=10, padx=10, fill="x", ipady=8)

    tk.Label(
        frame_entrada,
        text="Ejemplo: 3+5*2  |  (10-2)*3  |  5(2)",
        font=("Arial", 9, "italic"),
        bg="#2C1550",
        fg="#888888"
    ).pack(pady=(0, 10), padx=10)

    # =========================
    # PANEL DE ALFABETO
    # =========================

    frame_alfabeto = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_alfabeto.pack(pady=10, padx=20, fill="x")

    tk.Label(
        frame_alfabeto,
        text="🔤 ALFABETO DEL LENGUAJE",
        font=("Arial", 11, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5))

    alfabeto_texto = tk.Label(
        frame_alfabeto,
        text="{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, +, -, *, /, (, )}",
        font=("Arial", 10),
        bg="#2C1550",
        fg="white"
    )
    alfabeto_texto.pack(pady=(0, 10))

    # =========================
    # PANEL DE VERIFICACIÓN
    # =========================

    frame_verificacion = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_verificacion.pack(pady=10, padx=20, fill="x")

    tk.Label(
        frame_verificacion,
        text="🔍 VERIFICACIONES",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5))

    verificaciones_label = tk.Label(
        frame_verificacion,
        text="",
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    )
    verificaciones_label.pack(pady=10, padx=15, anchor="w")

    # =========================
    # PANEL DE RESULTADO PRINCIPAL
    # =========================

    frame_resultado = tk.Frame(ventana, bg=color_fondo, relief="solid", bd=3)
    frame_resultado.pack(pady=15, padx=20, fill="x")

    resultado_label = tk.Label(
        frame_resultado,
        text="",
        font=("Arial", 18, "bold"),
        bg=color_fondo,
        fg="white"
    )
    resultado_label.pack(pady=20)

    detalles_label = tk.Label(
        frame_resultado,
        text="",
        font=("Arial", 11),
        bg=color_fondo,
        fg="#A0A0A0",
        justify="left",
        wraplength=600
    )
    detalles_label.pack(pady=(0, 15), padx=15)

    # =========================
    # ANÁLISIS DETALLADO
    # =========================

    frame_analisis = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_analisis.pack(pady=10, padx=20, fill="both", expand=True)

    tk.Label(
        frame_analisis,
        text="📊 ANÁLISIS DETALLADO",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5))

    # Canvas con scrollbar
    canvas = tk.Canvas(frame_analisis, bg="#2C1550", highlightthickness=0, height=120)
    scrollbar = tk.Scrollbar(frame_analisis, orient="vertical", command=canvas.yview)
    scrollable_frame = tk.Frame(canvas, bg="#2C1550")

    scrollable_frame.bind(
        "<Configure>",
        lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
    )

    canvas.create_window((0, 0), window=scrollable_frame, anchor="nw")
    canvas.configure(yscrollcommand=scrollbar.set)

    analisis_label = tk.Label(
        scrollable_frame,
        text="",
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    )
    analisis_label.pack(pady=10, padx=10, anchor="w")

    canvas.pack(side="left", fill="both", expand=True, padx=5, pady=5)
    scrollbar.pack(side="right", fill="y")

    # =========================
    # FUNCIONES
    # =========================

    def actualizar_verificaciones():
        """Actualiza las verificaciones mientras escribes"""
        cadena = entrada.get()

        if not cadena:
            verificaciones_label.config(text="Escribe una expresión para verificar...")
            return

        verificaciones_texto = ""

        # Verificación 1: Alfabeto
        pertenece = pertenece_alfabeto(cadena)
        verificaciones_texto += "✅ Caracteres válidos\n" if pertenece else "❌ Contiene caracteres inválidos\n"

        # Verificación 2: Estructura
        valida = es_expresion_valida(cadena)
        verificaciones_texto += "✅ Estructura válida\n" if valida else "❌ Estructura inválida\n"

        # Verificación 3: Paréntesis
        parentesis_ok = parentesis_balanceados(cadena)
        verificaciones_texto += "✅ Paréntesis balanceados\n" if parentesis_ok else "❌ Paréntesis desbalanceados\n"

        # Verificación 4: Sin errores sintácticos
        error = detectar_error(cadena)
        verificaciones_texto += "✅ Sin errores sintácticos\n" if not error else f"❌ {error}\n"

        verificaciones_label.config(text=verificaciones_texto)

    def validar():
        """Valida la expresión completamente"""
        cadena = entrada.get()

        if not cadena:
            resultado_label.config(text="⚠️ Campo vacío", fg=color_error)
            detalles_label.config(text="Por favor escribe una expresión")
            analisis_label.config(text="")
            return

        # Validación 1: Alfabeto
        if not pertenece_alfabeto(cadena):
            resultado_label.config(text="❌ NO pertenece al lenguaje", fg=color_error)
            detalles_label.config(text="Contiene símbolos que no están en el alfabeto del lenguaje.")
            
            # Encontrar caracteres inválidos
            caracteres_invalidos = set()
            for c in cadena:
                if c not in alfabeto:
                    caracteres_invalidos.add(c)
            
            analisis = f"Caracteres inválidos encontrados: {', '.join(sorted(caracteres_invalidos))}\n\n"
            analisis += f"Alfabeto válido: {{{', '.join(sorted(list(alfabeto)))}}}"
            analisis_label.config(text=analisis)
            return

        # Validación 2: Estructura
        if not es_expresion_valida(cadena):
            resultado_label.config(text="❌ NO pertenece al lenguaje", fg=color_error)
            detalles_label.config(text="La estructura de la expresión no es válida según las reglas del lenguaje.")
            
            analisis = "Regla de estructura: [número operador número]\n"
            analisis += "Patrón válido: ^[0-9+\\-*/() ]+$\n\n"
            analisis += f"Tu expresión: {cadena}"
            analisis_label.config(text=analisis)
            return

        # Validación 3: Paréntesis
        if not parentesis_balanceados(cadena):
            resultado_label.config(text="❌ NO pertenece al lenguaje", fg=color_error)
            detalles_label.config(text="Los paréntesis no están correctamente balanceados.")
            
            # Contar paréntesis
            abiertos = cadena.count("(")
            cerrados = cadena.count(")")
            analisis = f"Paréntesis abiertos: {abiertos}\n"
            analisis += f"Paréntesis cerrados: {cerrados}\n\n"
            analisis += "Deben ser iguales y estar correctamente ordenados."
            analisis_label.config(text=analisis)
            return

        # Validación 4: Errores sintácticos
        error = detectar_error(cadena)
        if error:
            resultado_label.config(text="❌ NO pertenece al lenguaje", fg=color_error)
            detalles_label.config(text=f"Error: {error}")
            
            analisis = "Reglas sintácticas:\n"
            analisis += "✓ No puede iniciar con operador\n"
            analisis += "✓ No puede terminar con operador\n"
            analisis += "✓ No puede tener operadores duplicados (++, --, **, //)\n"
            analisis += "✓ No puede estar vacía"
            analisis_label.config(text=analisis)
            return

        # ✅ VALIDACIÓN EXITOSA
        resultado_label.config(text="✅ SI PERTENECE AL LENGUAJE", fg=color_correcto)
        detalles_label.config(
            text="¡Excelente! Esta expresión es válida según las reglas del lenguaje formal."
        )

        # Generar análisis detallado
        tokens = tokenizar(cadena)
        analisis = "TOKENS IDENTIFICADOS:\n"
        analisis += "─" * 40 + "\n"

        colores_token = {
            "NUMERO": "🔢",
            "OPERADOR": "➕",
            "PARENTESIS": "🟠"
        }

        for t in tokens:
            icono = colores_token.get(t[1], "•")
            analisis += f"{icono} '{t[0]}' → {t[1]}\n"

        analisis += "\n" + "─" * 40
        analisis += "\n✓ Expresión válida y lista para evaluarse"

        analisis_label.config(text=analisis)

        # Agregar al historial
        historial.append(f"Validación: {cadena} ✓")

    def limpiar():
        """Limpia todos los campos"""
        entrada.delete(0, tk.END)
        resultado_label.config(text="")
        detalles_label.config(text="")
        verificaciones_label.config(text="Escribe una expresión para verificar...")
        analisis_label.config(text="")

    # =========================
    # BOTONES DE CONTROL
    # =========================

    frame_botones = tk.Frame(ventana, bg=color_panel)
    frame_botones.pack(pady=15)

    tk.Button(
        frame_botones,
        text="✅ Validar",
        font=("Arial", 11, "bold"),
        bg=color_correcto,
        fg="black",
        width=15,
        command=validar
    ).pack(side="left", padx=5)

    tk.Button(
        frame_botones,
        text="🗑️ Limpiar",
        font=("Arial", 11, "bold"),
        bg=color_error,
        fg="black",
        width=15,
        command=limpiar
    ).pack(side="left", padx=5)

    # =========================
    # BOTÓN VOLVER
    # =========================

    tk.Button(
        ventana,
        text="◀ Volver al menú",
        font=("Arial", 10),
        bg="#555555",
        fg="black",
        command=ventana.destroy
    ).pack(pady=10)

    # Actualizar verificaciones mientras escribes
    entrada.bind("<KeyRelease>", lambda e: actualizar_verificaciones())

# =========================
# SECCION TEORIA (MEJORADA)
# =========================

def abrir_teoria():

    ventana = Toplevel(root)
    ventana.title("Teoría del Lenguaje")
    ventana.geometry("800x900")
    ventana.configure(bg=color_panel)

    # =========================
    # TITULO
    # =========================

    titulo = tk.Label(
        ventana,
        text="📚 TEORÍA DEL LENGUAJE FORMAL",
        font=("Arial", 16, "bold"),
        bg=color_panel,
        fg=color_correcto
    )
    titulo.pack(pady=15)

    subtitulo = tk.Label(
        ventana,
        text="Conceptos fundamentales de Lenguajes y Autómatas",
        font=("Arial", 11),
        bg=color_panel,
        fg="#A0A0A0"
    )
    subtitulo.pack(pady=5)

    # =========================
    # FRAME CON SCROLL
    # =========================

    canvas = tk.Canvas(ventana, bg=color_panel, highlightthickness=0)
    scrollbar = tk.Scrollbar(ventana, orient="vertical", command=canvas.yview)
    scrollable_frame = tk.Frame(canvas, bg=color_panel)

    scrollable_frame.bind(
        "<Configure>",
        lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
    )

    canvas.create_window((0, 0), window=scrollable_frame, anchor="nw")
    canvas.configure(yscrollcommand=scrollbar.set)

    # =========================
    # CONTENIDO EDUCATIVO
    # =========================

    # SECCIÓN 1: ALFABETO
    frame_seccion1 = tk.Frame(scrollable_frame, bg="#2C1550", relief="solid", bd=2)
    frame_seccion1.pack(pady=10, padx=15, fill="x")

    tk.Label(
        frame_seccion1,
        text="1️⃣ ALFABETO (Σ)",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10, anchor="w")

    texto_alfabeto = """Un alfabeto es un conjunto finito de símbolos.

En nuestro lenguaje:
Σ = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, +, -, *, /, (, )}

Características:
✓ Conjunto finito
✓ No vacío
✓ Símbolos distintos
✓ Define qué se puede usar"""

    tk.Label(
        frame_seccion1,
        text=texto_alfabeto,
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    ).pack(pady=10, padx=15, anchor="w")

    # SECCIÓN 2: CADENA
    frame_seccion2 = tk.Frame(scrollable_frame, bg="#2C1550", relief="solid", bd=2)
    frame_seccion2.pack(pady=10, padx=15, fill="x")

    tk.Label(
        frame_seccion2,
        text="2️⃣ CADENA (String)",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10, anchor="w")

    texto_cadena = """Una cadena es una secuencia finita de símbolos del alfabeto.

Ejemplos válidos:
✓ 3+5
✓ (10-2)*3
✓ 5(2)
✓ ((8/2)+1)

Ejemplos inválidos:
✗ +3 (inicia con operador)
✗ 5- (termina con operador)
✗ 3++5 (operadores duplicados)
✗ (5 (paréntesis sin cerrar)"""

    tk.Label(
        frame_seccion2,
        text=texto_cadena,
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    ).pack(pady=10, padx=15, anchor="w")

    # SECCIÓN 3: LENGUAJE
    frame_seccion3 = tk.Frame(scrollable_frame, bg="#2C1550", relief="solid", bd=2)
    frame_seccion3.pack(pady=10, padx=15, fill="x")

    tk.Label(
        frame_seccion3,
        text="3️⃣ LENGUAJE (L)",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10, anchor="w")

    texto_lenguaje = """Un lenguaje es un conjunto de cadenas válidas.

L = {cadenas | cadena ∈ Σ* y cadena es válida}

Nuestro lenguaje:
L = {expresiones aritméticas válidas}

Características:
✓ Conjunto de cadenas válidas
✓ Pueden ser infinitas
✓ Definidas por reglas (gramática)
✓ Pueden ser reconocidas por autómatas"""

    tk.Label(
        frame_seccion3,
        text=texto_lenguaje,
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    ).pack(pady=10, padx=15, anchor="w")

    # SECCIÓN 4: TOKEN
    frame_seccion4 = tk.Frame(scrollable_frame, bg="#2C1550", relief="solid", bd=2)
    frame_seccion4.pack(pady=10, padx=15, fill="x")

    tk.Label(
        frame_seccion4,
        text="4️⃣ TOKEN",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10, anchor="w")

    texto_token = """Un token es cada elemento identificado en una cadena.

Ejemplo: 3 + 5 * 2

Tokens:
🔢 3        →  NUMERO
➕ +        →  OPERADOR
🔢 5        →  NUMERO
➕ *        →  OPERADOR
🔢 2        →  NUMERO

Tipos de tokens en nuestro lenguaje:
• NUMERO: Dígitos 0-9
• OPERADOR: +, -, *, /
• PARENTESIS: (, )"""

    tk.Label(
        frame_seccion4,
        text=texto_token,
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    ).pack(pady=10, padx=15, anchor="w")

    # SECCIÓN 5: REGLAS DE VALIDACIÓN
    frame_seccion5 = tk.Frame(scrollable_frame, bg="#2C1550", relief="solid", bd=2)
    frame_seccion5.pack(pady=10, padx=15, fill="x")

    tk.Label(
        frame_seccion5,
        text="5️⃣ REGLAS DE VALIDACIÓN",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10, anchor="w")

    texto_reglas = """Una cadena es válida si cumple:

1. PERTENENCIA AL ALFABETO
   ✓ Solo contiene símbolos del alfabeto
   ✗ No contiene caracteres especiales

2. ESTRUCTURA VÁLIDA
   ✓ Patrón: ^[0-9+\\-*/() ]+$
   ✓ Comienza con número o (
   ✓ Termina con número o )

3. PARÉNTESIS BALANCEADOS
   ✓ Igual cantidad de ( y )
   ✓ Cada ( tiene su )
   ✓ Ningún ) sin su (

4. SIN ERRORES SINTÁCTICOS
   ✓ No inicia con operador
   ✓ No termina con operador
   ✓ No hay operadores duplicados
   ✓ No está vacía"""

    tk.Label(
        frame_seccion5,
        text=texto_reglas,
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    ).pack(pady=10, padx=15, anchor="w")

    # SECCIÓN 6: AUTÓMATA FINITO
    frame_seccion6 = tk.Frame(scrollable_frame, bg="#2C1550", relief="solid", bd=2)
    frame_seccion6.pack(pady=10, padx=15, fill="x")

    tk.Label(
        frame_seccion6,
        text="6️⃣ AUTÓMATA FINITO DETERMINISTA (AFD)",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10, anchor="w")

    texto_afd = """Un AFD es una máquina que reconoce si una cadena pertenece a un lenguaje.

Componentes:
• Estados: Puntos en el proceso de reconocimiento
• Transiciones: Movimientos entre estados
• Alfabeto: Símbolos que puede leer
• Estado inicial: Por donde comienza
• Estados finales: Indica cadena válida

Funcionamiento:
1. Lee un símbolo de la entrada
2. Cambia de estado según la transición
3. Repite hasta terminar la entrada
4. Si termina en estado final → VÁLIDA
5. Si no → INVÁLIDA"""

    tk.Label(
        frame_seccion6,
        text=texto_afd,
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    ).pack(pady=10, padx=15, anchor="w")

    # SECCIÓN 7: EJEMPLO COMPLETO
    frame_seccion7 = tk.Frame(scrollable_frame, bg="#2C1550", relief="solid", bd=2)
    frame_seccion7.pack(pady=10, padx=15, fill="x")

    tk.Label(
        frame_seccion7,
        text="7️⃣ EJEMPLO COMPLETO",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10, anchor="w")

    texto_ejemplo = """Análisis de: (3+5)*2

PASO 1: Verificar alfabeto
✓ ( ∈ Σ, 3 ∈ Σ, + ∈ Σ, 5 ∈ Σ, ) ∈ Σ, * ∈ Σ, 2 ∈ Σ
✓ Pertenece al alfabeto

PASO 2: Verificar estructura
✓ Cumple patrón: ^[0-9+\\-*/() ]+$

PASO 3: Verificar paréntesis
✓ Abiertos: 1, Cerrados: 1, Balanceados: SÍ

PASO 4: Verificar sintaxis
✓ No inicia con operador
✓ No termina con operador
✓ Sin operadores duplicados

PASO 5: Tokenizar
( → PARENTESIS
3 → NUMERO
+ → OPERADOR
5 → NUMERO
) → PARENTESIS
* → OPERADOR
2 → NUMERO

RESULTADO: ✅ VÁLIDA
Puede evaluarse: (3+5)*2 = 16"""

    tk.Label(
        frame_seccion7,
        text=texto_ejemplo,
        font=("Courier", 16),
        bg="#2C1550",
        fg="white",
        justify="left",
    ).pack(pady=10, padx=15, anchor="w")

    # SECCIÓN 8: REFERENCIAS
    frame_seccion8 = tk.Frame(scrollable_frame, bg="#2C1550", relief="solid", bd=2)
    frame_seccion8.pack(pady=10, padx=15, fill="x")

    tk.Label(
        frame_seccion8,
        text="📖 REFERENCIAS Y FUENTES",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 5), padx=10, anchor="w")

    texto_referencias = """Conceptos basados en:
• Teoría de Autómatas y Lenguajes Formales
• Introducción a la Teoría de Computación - Michael Sipser
• Compiladores: Principios, técnicas y herramientas
• ISO/IEC 14977 - Notación de Forma de Backus-Naur (BNF)

Aplicaciones en:
• Compiladores e intérpretes
• Análisis léxico y sintáctico
• Expresiones regulares
• Procesamiento de lenguajes"""

    tk.Label(
        frame_seccion8,
        text=texto_referencias,
        font=("Arial", 16),
        bg="#2C1550",
        fg="white",
        justify="left"
    ).pack(pady=10, padx=15, anchor="w")

    # =========================
    # CANVAS Y SCROLLBAR
    # =========================

    canvas.pack(side="left", fill="both", expand=True, padx=0, pady=10)
    scrollbar.pack(side="right", fill="y")

    # =========================
    # BOTONES INFERIORES
    # =========================

    frame_botones = tk.Frame(ventana, bg=color_panel)
    frame_botones.pack(fill="x", padx=20, pady=10)

    tk.Button(
        frame_botones,
        text="🔝 Ir al inicio",
        font=("Arial", 10),
        bg="#00C2FF",
        fg="black",
        command=lambda: canvas.yview_moveto(0)
    ).pack(side="left", padx=5)

    tk.Button(
        frame_botones,
        text="📋 Copiar contenido",
        font=("Arial", 10),
        bg="#2ECC71",
        fg="black",
        command=lambda: ventana.clipboard_clear() or ventana.clipboard_append(
            "ALFABETO: {0-9,+,-,*,/,(,)}\n"
            "LENGUAJE: Expresiones aritméticas válidas\n"
            "Ver ventana de teoría para más detalles."
        ) or tk.messagebox.showinfo("Copiar", "Contenido copiado al portapapeles")
    ).pack(side="left", padx=5)

    tk.Button(
        frame_botones,
        text="◀ Volver al menú",
        font=("Arial", 10),
        bg="#555555",
        fg="black",
        command=ventana.destroy
    ).pack(side="left", padx=5)

# =========================
# SECCION GENERADOR (NUEVA)
# =========================

import random

def abrir_generador():

    ventana = Toplevel(root)
    ventana.title("Generador de Expresiones")
    ventana.geometry("700x900")
    ventana.configure(bg=color_panel)

    # =========================
    # TITULO
    # =========================

    titulo = tk.Label(
        ventana,
        text="🎲 GENERADOR DE EXPRESIONES",
        font=("Arial", 16, "bold"),
        bg=color_panel,
        fg=color_correcto
    )
    titulo.pack(pady=15)

    subtitulo = tk.Label(
        ventana,
        text="Genera expresiones válidas e inválidas para aprender",
        font=("Arial", 11),
        bg=color_panel,
        fg="#A0A0A0"
    )
    subtitulo.pack(pady=5)

    # =========================
    # OPCIONES DE GENERACIÓN
    # =========================

    frame_opciones = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_opciones.pack(pady=15, padx=20, fill="x")

    tk.Label(
        frame_opciones,
        text="⚙️ OPCIONES DE GENERACIÓN",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 10))

    # Frame para dificultad
    frame_dificultad = tk.Frame(frame_opciones, bg="#2C1550")
    frame_dificultad.pack(fill="x", padx=10, pady=5)

    tk.Label(
        frame_dificultad,
        text="Dificultad:",
        font=("Arial", 10),
        bg="#2C1550",
        fg="white"
    ).pack(side="left", padx=5)

    dificultad_var = tk.StringVar(value="media")

    tk.Radiobutton(
        frame_dificultad,
        text="Fácil",
        variable=dificultad_var,
        value="facil",
        bg="#2C1550",
        fg="white",
        selectcolor="#864CBF"
    ).pack(side="left", padx=5)

    tk.Radiobutton(
        frame_dificultad,
        text="Media",
        variable=dificultad_var,
        value="media",
        bg="#2C1550",
        fg="white",
        selectcolor="#864CBF"
    ).pack(side="left", padx=5)

    tk.Radiobutton(
        frame_dificultad,
        text="Difícil",
        variable=dificultad_var,
        value="dificil",
        bg="#2C1550",
        fg="white",
        selectcolor="#864CBF"
    ).pack(side="left", padx=5)

    # Frame para tipo de expresión
    frame_tipo = tk.Frame(frame_opciones, bg="#2C1550")
    frame_tipo.pack(fill="x", padx=10, pady=5)

    tk.Label(
        frame_tipo,
        text="Tipo:",
        font=("Arial", 10),
        bg="#2C1550",
        fg="white"
    ).pack(side="left", padx=5)

    tipo_var = tk.StringVar(value="ambas")

    tk.Radiobutton(
        frame_tipo,
        text="Válida",
        variable=tipo_var,
        value="valida",
        bg="#2C1550",
        fg="white",
        selectcolor="#864CBF"
    ).pack(side="left", padx=5)

    tk.Radiobutton(
        frame_tipo,
        text="Inválida",
        variable=tipo_var,
        value="invalida",
        bg="#2C1550",
        fg="white",
        selectcolor="#864CBF"
    ).pack(side="left", padx=5)

    tk.Radiobutton(
        frame_tipo,
        text="Ambas (Reto)",
        variable=tipo_var,
        value="ambas",
        bg="#2C1550",
        fg="white",
        selectcolor="#864CBF"
    ).pack(side="left", padx=5)

    # =========================
    # GENERADORES DE EXPRESIONES
    # =========================

    def generar_valida(dificultad):
        """Genera una expresión válida"""
        numeros = list(range(1, 100))

        if dificultad == "facil":
            # Simple: numero + numero
            a = random.choice(numeros)
            op = random.choice(["+", "-", "*", "/"])
            b = random.choice(numeros)
            return f"{a}{op}{b}"

        elif dificultad == "media":
            # Con paréntesis: (numero op numero) op numero
            a = random.choice(numeros)
            b = random.choice(numeros)
            c = random.choice(numeros)
            op1 = random.choice(["+", "-", "*", "/"])
            op2 = random.choice(["+", "-", "*", "/"])

            tipos = [
                f"({a}{op1}{b}){op2}{c}",
                f"{a}{op1}({b}{op2}{c})",
                f"{a}{op1}{b}{op2}{c}"
            ]
            return random.choice(tipos)

        else:  # dificil
            # Complejo con múltiples paréntesis
            a = random.choice(numeros)
            b = random.choice(numeros)
            c = random.choice(numeros)
            d = random.choice(numeros)
            op1 = random.choice(["+", "-", "*", "/"])
            op2 = random.choice(["+", "-", "*", "/"])
            op3 = random.choice(["+", "-", "*", "/"])

            tipos = [
                f"(({a}{op1}{b}){op2}{c}){op3}{d}",
                f"{a}{op1}({b}{op2}({c}{op3}{d}))",
                f"({a}{op1}{b}){op2}({c}{op3}{d})"
            ]
            return random.choice(tipos)

    def generar_invalida(dificultad):
        """Genera una expresión inválida"""
        numeros = list(range(1, 20))

        if dificultad == "facil":
            # Errores simples
            errores = [
                f"+{random.choice(numeros)}",  # Inicia con operador
                f"{random.choice(numeros)}-",  # Termina con operador
                f"{random.choice(numeros)}++{random.choice(numeros)}",  # Operadores duplicados
                f"({random.choice(numeros)}",  # Paréntesis sin cerrar
                f"{random.choice(numeros)})",  # Paréntesis sin abrir
            ]
            return random.choice(errores)

        elif dificultad == "media":
            a = random.choice(numeros)
            b = random.choice(numeros)
            c = random.choice(numeros)
            op = random.choice(["+", "-", "*", "/"])

            errores = [
                f"{a}*/{b}",  # Operadores duplicados
                f"({a}{op}{b}",  # Paréntesis sin cerrar
                f"{a}{op}{b})",  # Paréntesis sin abrir
                f"({a}{op}({b}",  # Múltiples paréntesis sin cerrar
                f"{op}{a}+{b}",  # Inicia con operador
            ]
            return random.choice(errores)

        else:  # dificil
            a = random.choice(numeros)
            b = random.choice(numeros)
            c = random.choice(numeros)
            d = random.choice(numeros)

            errores = [
                f"(({a}+{b})({c}+{d}))",  # Falta operador entre paréntesis
                f"{a}+++{b}",  # Múltiples operadores
                f"({a}+{b}",  # Anidación sin cerrar
                f"{a}+-{b}",  # Operadores consecutivos
                f"(({a}+{b}))*{c}"   # Paréntesis mal balanceados
            ]
            return random.choice(errores)

    def generar():
        """Genera una expresión según las opciones seleccionadas"""
        dificultad = dificultad_var.get()
        tipo = tipo_var.get()

        if tipo == "valida":
            expresion = generar_valida(dificultad)
            es_valida = True
        elif tipo == "invalida":
            expresion = generar_invalida(dificultad)
            es_valida = False
        else:  # ambas (reto)
            if random.choice([True, False]):
                expresion = generar_valida(dificultad)
                es_valida = True
            else:
                expresion = generar_invalida(dificultad)
                es_valida = False

        entrada_expr.config(state="normal")
        entrada_expr.delete(0, tk.END)
        entrada_expr.insert(0, expresion)
        entrada_expr.config(state="readonly")

        # Guardar la respuesta correcta
        generar.respuesta_correcta = es_valida
        generar.expresion_actual = expresion

        resultado_label.config(text="")
        explicacion_label.config(text="")
        razon_label.config(text="")

    # =========================
    # EXPRESIÓN GENERADA
    # =========================

    frame_expresion = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_expresion.pack(pady=15, padx=20, fill="x")

    tk.Label(
        frame_expresion,
        text="📐 EXPRESIÓN GENERADA",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 10))

    entrada_expr = tk.Entry(
        frame_expresion,
        font=("Arial", 16, "bold"),
        width=30,
        justify="center",
        bg="white",
        fg="white",
        state="readonly"
    )
    entrada_expr.pack(pady=10, padx=10, fill="x", ipady=8)

    tk.Label(
        frame_expresion,
        text="Presiona 'Generar' para obtener una nueva expresión",
        font=("Arial", 9, "italic"),
        bg="#2C1550",
        fg="#888888"
    ).pack(pady=(0, 10))

    # =========================
    # RETO (SOLO SI TIPO == AMBAS)
    # =========================

    frame_reto = tk.Frame(ventana, bg="#2C1550", relief="solid", bd=2)
    frame_reto.pack(pady=15, padx=20, fill="x")

    tk.Label(
        frame_reto,
        text="🎯 RETO: ¿Es válida o inválida?",
        font=("Arial", 14, "bold"),
        bg="#2C1550",
        fg=color_correcto
    ).pack(pady=(10, 10))

    frame_respuestas = tk.Frame(frame_reto, bg="#2C1550")
    frame_respuestas.pack(fill="x", padx=10, pady=10)

    respuesta_var = tk.StringVar()

    tk.Radiobutton(
        frame_respuestas,
        text="✅ VÁLIDA",
        variable=respuesta_var,
        value="valida",
        bg="#2C1550",
        fg=color_correcto,
        selectcolor="#2C1550",
        font=("Arial", 11, "bold")
    ).pack(side="left", padx=10, ipady=5)

    tk.Radiobutton(
        frame_respuestas,
        text="❌ INVÁLIDA",
        variable=respuesta_var,
        value="invalida",
        bg="#2C1550",
        fg=color_error,
        selectcolor="#2C1550",
        font=("Arial", 11, "bold")
    ).pack(side="left", padx=10, ipady=5)

    # =========================
    # RESULTADO DEL RETO
    # =========================

    resultado_label = tk.Label(
        ventana,
        text="",
        font=("Arial", 14, "bold"),
        bg=color_panel,
        fg="white"
    )
    resultado_label.pack(pady=10)

    explicacion_label = tk.Label(
        ventana,
        text="",
        font=("Arial", 16),
        bg=color_panel,
        fg="#A0A0A0",
        justify="left",
        wraplength=600
    )
    explicacion_label.pack(pady=5, padx=20)

    razon_label = tk.Label(
        ventana,
        text="",
        font=("Arial", 16),
        bg=color_panel,
        fg="white",
        justify="left",
        wraplength=600
    )
    razon_label.pack(pady=5, padx=20)

    # =========================
    # FUNCIÓN VERIFICAR RESPUESTA
    # =========================

    def verificar_respuesta():
        """Verifica la respuesta del usuario"""
        if not hasattr(generar, 'respuesta_correcta'):
            resultado_label.config(text="⚠️ Genera una expresión primero", fg=color_error)
            return

        respuesta_usuario = respuesta_var.get() == "valida"
        respuesta_correcta = generar.respuesta_correcta

        expresion = generar.expresion_actual

        if respuesta_usuario == respuesta_correcta:
            resultado_label.config(text="🎉 ¡CORRECTO!", fg=color_correcto)

            if respuesta_correcta:
                explicacion_label.config(text=f"'{expresion}' SÍ es una expresión válida")
            else:
                explicacion_label.config(text=f"'{expresion}' NO es una expresión válida")

            # Mostrar por qué
            if respuesta_correcta:
                razon_label.config(text="✓ Pertenece al alfabeto\n✓ Tiene estructura válida\n✓ Paréntesis balanceados\n✓ Sin errores sintácticos")
            else:
                error = detectar_error(expresion)
                razon_label.config(text=f"Razón: {error if error else 'Estructura inválida'}")

        else:
            resultado_label.config(text="❌ INCORRECTO", fg=color_error)

            if respuesta_correcta:
                explicacion_label.config(text=f"'{expresion}' SÍ es una expresión válida (Respondiste inválida)")
                razon_label.config(text="✓ Pertenece al alfabeto\n✓ Tiene estructura válida\n✓ Paréntesis balanceados")
            else:
                explicacion_label.config(text=f"'{expresion}' NO es una expresión válida (Respondiste válida)")
                error = detectar_error(expresion)
                razon_label.config(text=f"Razón: {error if error else 'Estructura inválida'}")

    # =========================
    # BOTONES DE CONTROL
    # =========================

    frame_botones = tk.Frame(ventana, bg=color_panel)
    frame_botones.pack(pady=15)

    tk.Button(
        frame_botones,
        text="🎲 Generar",
        font=("Arial", 16, "bold"),
        bg="#00C2FF",
        fg="black",
        width=15,
        command=generar
    ).pack(side="left", padx=5)

    tk.Button(
        frame_botones,
        text="✔️ Verificar",
        font=("Arial", 16, "bold"),
        bg=color_correcto,
        fg="black",
        width=15,
        command=verificar_respuesta
    ).pack(side="left", padx=5)

    tk.Button(
        frame_botones,
        text="🔄 Limpiar",
        font=("Arial", 16, "bold"),
        bg=color_error,
        fg="black",
        width=15,
        command=lambda: (entrada_expr.config(state="normal"), entrada_expr.delete(0, tk.END), 
                        entrada_expr.config(state="readonly"), resultado_label.config(text=""), 
                        explicacion_label.config(text=""), razon_label.config(text=""), 
                        respuesta_var.set(""))
    ).pack(side="left", padx=5)

    # =========================
    # BOTÓN VOLVER
    # =========================

    tk.Button(
        ventana,
        text="◀ Volver al menú",
        font=("Arial", 14),
        bg="#555555",
        fg="black",
        command=ventana.destroy
    ).pack(pady=10)

# =========================
# VENTANA PRINCIPAL (REDISEÑADA)
# =========================

root = tk.Tk()
root.title("Intérprete de Operaciones Aritméticas")
root.geometry("750x650")
root.configure(bg=color_fondo)

# =========================
# HEADER CON GRADIENTE SIMULADO
# =========================

header = tk.Frame(root, bg="#0D0520", height=120)
header.pack(fill="x", pady=0)
header.pack_propagate(False)

# Línea decorativa superior
linea_top = tk.Frame(header, bg="##864CBF", height=3)
linea_top.pack(fill="x")
# Frame interno del header
header_interno = tk.Frame(header, bg="#0D0520")
header_interno.pack(fill="both", expand=True, padx=20, pady=15)

# Logos
try:
    logo1 = PhotoImage(file="logo_tecnm.png").subsample(4, 4)
    tk.Label(header_interno, image=logo1, bg="#0D0520").pack(side="left", padx=10)
except:
    tk.Label(header_interno, text="🏫", font=("Arial", 24), bg="#0D0520").pack(side="left", padx=10)

# Título principal
frame_titulo = tk.Frame(header_interno, bg="#0D0520")
frame_titulo.pack(side="left", padx=20, expand=True, fill="both")

titulo_principal = tk.Label(
    frame_titulo,
    text="INTÉRPRETE DE",
    font=("Arial", 20, "bold"),
    bg="#0D0520",
    fg="#00C2FF"
)
titulo_principal.pack()

titulo_secundario = tk.Label(
    frame_titulo,
    text="OPERACIONES ARITMÉTICAS",
    font=("Arial", 18, "bold"),
    bg="#0D0520",
    fg="#864CBF"
)
titulo_secundario.pack()

subtitulo = tk.Label(
    frame_titulo,
    text="Lenguajes y Autómatas",
    font=("Arial", 11, "italic"),
    bg="#0D0520",
    fg="#A0A0A0"
)
subtitulo.pack()

# Logo derecha
try:
    logo2 = PhotoImage(file="logo_tijuana.png").subsample(4, 4)
    tk.Label(header_interno, image=logo2, bg="#0D0520").pack(side="right", padx=10)
except:
    tk.Label(header_interno, text="🌆", font=("Arial", 24), bg="#0D0520").pack(side="right", padx=10)

# Línea decorativa inferior
linea_bottom = tk.Frame(header, bg="#864CBF", height=2)
linea_bottom.pack(fill="x")

# =========================
# DESCRIPCIÓN
# =========================

frame_descripcion = tk.Frame(root, bg="#1F1147")
frame_descripcion.pack(fill="x", padx=20, pady=15)

descripcion = tk.Label(
    frame_descripcion,
    text="📚 Analiza, valida y evalúa expresiones matemáticas con análisis de tokens",
    font=("Arial", 11),
    bg="#1F1147",
    fg="#00C2FF"
)
descripcion.pack()

# =========================
# MENÚ PRINCIPAL
# =========================

menu_titulo = tk.Label(
    root,
    text="🎯 SELECCIONA UNA OPCIÓN",
    font=("Arial", 14, "bold"),
    bg=color_fondo,
    fg=color_correcto
)
menu_titulo.pack(pady=15)

menu_frame = tk.Frame(root, bg=color_fondo)
menu_frame.pack(pady=10, padx=20, fill="both", expand=True)

botones_datos = [
    ("🔍 Analizador de Expresión", abrir_analizador, "#00C2FF"),
    ("✅ Validador de Lenguaje", abrir_validador, "#2ECC71"),
    ("🎲 Generador de Expresiones", abrir_generador, "#F39C12"),
    ("📖 Teoría del Lenguaje", abrir_teoria, "#E74C3C")
]

for i, (texto, funcion, color) in enumerate(botones_datos):
    
    # Frame para cada botón (contenedor)
    frame_boton = tk.Frame(menu_frame, bg=color_fondo)
    frame_boton.pack(pady=8, fill="x")
    
    # Botón con estilo mejorado
    boton = tk.Button(
        frame_boton,
        text=texto,
        width=40,
        height=2,
        font=("Arial", 16, "bold"),
        bg=color,
        fg="black",
        activebackground=color,
        activeforeground="white",
        relief="raised",
        bd=2,
        cursor="hand2",
        command=funcion
    )
    boton.pack(fill="x", ipady=5)
    
# =========================
# FOOTER
# =========================

footer_frame = tk.Frame(root, bg="#0D0520", height=50)
footer_frame.pack(fill="x", side="bottom")
footer_frame.pack_propagate(False)

linea_footer_top = tk.Frame(footer_frame, bg="#864CBF", height=1)
linea_footer_top.pack(fill="x")

footer_texto = tk.Label(
    footer_frame,
    text="© 2026 - Instituto Tecnológico de Tijuana | Teoría de Lenguajes y Autómatas",
    font=("Arial", 9),
    bg="#0D0520",
    fg="#888888"
)
footer_texto.pack(pady=12)

root.mainloop()
