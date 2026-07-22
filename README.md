# INFORME TÉCNICO

## PROTOTIPO DE ANALIZADOR SINTÁCTICO LL(1) PARA CALCULADORA ARITMÉTICA

---

**Asignatura:** Compiladores y Lenguajes de Programación  
**Evaluación:** Segundo Bimestre  
**Fecha:** 22 de julio de 2025  
**Estudiantes:**   
**Grupo:** 

---

## ÍNDICE

1. Resumen Ejecutivo
2. Marco Teórico
3. Desarrollo del Prototipo
4. Pruebas y Resultados
5. Conclusiones
6. Anexos

---

## 1. RESUMEN EJECUTIVO

El presente informe documenta el desarrollo de un prototipo de analizador sintáctico tipo LL(1) para una calculadora aritmética que procesa expresiones con números enteros, operaciones básicas (+, -, *, /), raíz cuadrada (RAIZ) y paréntesis.

El sistema implementa:
- Una **Gramática Independiente del Contexto (GIC)** transformada a forma LL(1)
- Un **analizador léxico** (lexer) que tokeniza la entrada
- Un **parser descendente predictivo** basado en tabla M[A,a]
- Un **árbol de derivación** visual e interactivo
- Manejo de errores sintácticos con mensajes descriptivos

**Palabras clave:** Parser LL(1), Gramática Independiente del Contexto, Análisis Sintáctico, Árbol de Derivación, Compiladores.

---

## 2. MARCO TEÓRICO

### 2.1 Gramática Independiente del Contexto (GIC)

Una GIC es una cuádrupla G = (N, T, P, S) donde:
- **N**: Conjunto de símbolos no terminales
- **T**: Conjunto de símbolos terminales
- **P**: Conjunto de producciones
- **S**: Símbolo inicial

Para nuestro proyecto, la gramática original contenía recursividad izquierda, la cual fue eliminada mediante transformaciones algebraicas para obtener una gramática LL(1).

### 2.2 Parser LL(1)

Un parser LL(1) (Left-to-right, Leftmost derivation, 1 symbol lookahead):
- Procesa la entrada de izquierda a derecha
- Construye una derivación por la izquierda
- Usa **un solo símbolo de lookahead** para decidir qué producción aplicar

Requiere que la gramática cumpla:
1. No tener recursividad izquierda
2. Estar factorizada (sin prefijos comunes en alternativas)
3. Conjuntos PRIMERO y SIGUIENTE calculados

### 2.3 Tabla de Análisis Sintáctico M[A,a]

Es una matriz donde:
- Filas = No terminales (A ∈ N)
- Columnas = Terminales (a ∈ T ∪ {$})
- Celdas = Producción a aplicar o Error

La tabla se construye usando:
- **PRIMERO(α)**: Terminales que pueden iniciar α
- **SIGUIENTE(A)**: Terminales que pueden seguir a A

---

## 3. DESARROLLO DEL PROTOTIPO

### 3.1 Arquitectura del Sistema
```
+-----------------+     +------------------       +-----------------+
|   INTERFAZ UI   |---->|   PARSER LL(1)    |---->|    ÁRBOL AST    |
|    (React)      |     |  (Lógica Pila)    |     |    (Visual)     |
+-----------------+     +------------------       +-----------------+
         |                       |                       
         v                       v                       
+-----------------+     +------------------              
|     ENTRADA     |     |   TABLA M[A,a]   |              
|     (Texto)     |     |  (Diccionario)   |              
+-----------------+     +------------------              
         |                                               
         v                                               
+-----------------+                                      
|     LEXER/      |                                      
|    TOKENIZER    |                                      
+-----------------+
```
### 3.2 Gramática Implementada

**No terminales:** N = {S, E, E', T, T', U, F, num, num', dig}

**Terminales:** T = {RAIZ, +, -, *, /, (, ), 0-9, $}

**Producciones (transformadas a LL(1)):**

| Producción | Significado |
|------------|-------------|
| S → E | Inicio de la expresión |
| E → T E' | Expresión = Término + Resto de E |
| E' → + T E' \| - T E' \| λ | Suma/resta encadenada |
| T → U T' | Término = Factor + Resto de T |
| T' → * U T' \| / U T' \| λ | Multiplicación/división encadenada |
| U → RAIZ F \| F | Raíz cuadrada o Factor simple |
| F → ( E ) \| num | Paréntesis o número |
| num → dig num' | Número = dígito + resto |
| num' → dig num' \| λ | Más dígitos o fin |
| dig → 0\|1\|2\|3\|4\|5\|6\|7\|8\|9 | Dígito individual |

### 3.3 Implementación del Lexer

El analizador léxico convierte la cadena de entrada en tokens mediante expresiones regulares:

```javascript
const tokenize = (text) => {
    const regex = /\s*(RAIZ|\(|\)|\+|-|\*|\/|\d)\s*/g;
    // Identifica: RAIZ, paréntesis, operadores, dígitos
    // Retorna array de objetos {type, value}
    // Añade $ al final (EOF)
};
Tokens reconocidos:

RAIZ: Palabra clave para raíz cuadrada
(, ): Paréntesis
+, -, *, /: Operadores aritméticos
dig: Cualquier dígito 0-9
$: Fin de entrada (EOF)
3.4 Implementación del Parser
El parser utiliza una pila explícita que almacena pares (símbolo, nodo_del_árbol):

Algoritmo:

1. Inicializar pila con [$ , S]
2. Crear nodo raíz S en el árbol
3. Mientras pila no vacía:
   a. Sacar tope (X)
   b. Si X es terminal:
      - Debe coincidir con token actual
      - Si coincide: consumir token, avanzar
      - Si no: ERROR
   c. Si X es no-terminal:
      - Buscar M[X, token] en tabla
      - Si existe producción: expandir (sacar X, meter producción inversa)
      - Si no existe: ERROR
   d. Si X = $ y token = $: ACEPTAR
Ejemplo de traza:

Paso	Pila	Token	Acción
1	$ S	(	M[S,(] = S→E
2	$ E	(	M[E,(] = E→TE'
3	$ E' T	(	M[T,(] = T→UT'
...	...	...	...
###3.5 Construcción del Árbol de Derivación
Cada vez que se expande un no-terminal, se crean nodos hijos en el árbol:

javascript
// Al expandir A → X Y Z:
const newNodes = prod.map(s => ({ name: s, children: [] }));
nodoActual.children = newNodes;  // Conectar al árbol
// Apilar en orden inverso: Z, Y, X
El árbol se visualiza recursivamente con líneas de conexión estilo árbol de directorios.

###3.6 Manejo de Errores
El sistema detecta dos tipos de errores:

Error léxico: Caracteres no reconocidos (ej: @, #, letras)
Error sintáctico: Secuencia inválida de tokens
No existe entrada en tabla M[A,a]
Terminal en pila no coincide con token actual
Fin de entrada inesperado
Mensajes de error incluyen:

Símbolo no terminal y terminal involucrados
Posición en la entrada (cuando aplica)
##4. PRUEBAS Y RESULTADOS
###4.1 Casos de Prueba Válidos
Prueba 1: Expresión con paréntesis y operaciones mixtas
Entrada: ( 3 + 5 ) * 2

Resultado esperado: Aceptada
Árbol generado: Expresión con precedencia correcta (paréntesis primero, luego multiplicación)

[Insertar captura de pantalla]

Prueba 2: Raíz cuadrada
Entrada: RAIZ 9 + 5

Resultado esperado: Aceptada
Árbol generado: U → RAIZ F, luego E' → + T E'

[Insertar captura de pantalla]

Prueba 3: Asociatividad izquierda
Entrada: 3 + 5 + 2

Resultado esperado: Aceptada
Árbol generado: (3 + 5) + 2 (asociatividad izquierda demostrada)

[Insertar captura de pantalla]

###4.2 Casos de Prueba con Error
Prueba 4: Sintaxis inválida (operador donde no corresponde)
Entrada: 3 + * 2

Error detectado: Error sintáctico: No hay producción M[T, *]
Explicación: Después de un +, el parser espera un Término (T), pero encuentra * que no inicia ningún término válido.

[Insertar captura de pantalla]

Prueba 5: Paréntesis no balanceado
Entrada: ( 3 + 5

Error detectado: Error sintáctico: No hay producción M[E', $]
Explicación: Se llega al final de la entrada ($) esperando un ) que nunca aparece.

[Insertar captura de pantalla]

###4.3 Tabla de Resumen de Pruebas
ID	Entrada	Tipo	Resultado	Observación
1	( 3 + 5 ) * 2	Válida	✅ Aceptada	Precedencia correcta
2	RAIZ 9 + 5	Válida	✅ Aceptada	Operador unario
3	3 + 5 + 2	Válida	✅ Aceptada	Asociatividad
4	3 + * 2	Error	❌ Rechazada	Token inesperado
5	( 3 + 5	Error	❌ Rechazada	Paréntesis abierto
5. CONCLUSIONES
Transformación de gramática: Se logró transformar exitosamente la gramática original con recursividad izquierda a una forma LL(1) mediante la introducción de no-terminales auxiliares (E', T', num'), permitiendo el uso de un parser predictivo no recursivo.

Eficiencia del parser LL(1): El algoritmo de análisis sintáctico descendente con tabla de análisis M[A,a] demostró ser eficiente y fácil de implementar, requiriendo solo O(n) tiempo para analizar una cadena de n tokens.

Visualización didáctica: La interfaz gráfica que muestra la traza paso a paso (Pila, Token, Acción) y el árbol de derivación resultó ser una herramienta efectiva para comprender el funcionamiento interno del parser.

Manejo de errores: El sistema detecta y reporta errores sintácticos con precisión, indicando qué símbolo se esperaba y qué se encontró, facilitando la depuración.

Preparación para integración: El diseño modular separa claramente el lexer, el parser y el generador del árbol, facilitando la futura integración con un analizador léxico real desarrollado en semestres anteriores.

###6. RECOMENDACIONES
Extensión de operadores: Incluir operadores adicionales como potencia (^) y módulo (%), requiriendo ajustes en la gramática y la tabla de análisis.

Análisis semántico: Agregar una fase posterior que evalúe la expresión numéricamente o genere código intermedio.

Recuperación de errores: Implementar estrategias de recuperación (modo pánico, frases de sincronización) para continuar el análisis después de un error.

Integración con AL real: Reemplazar el lexer simplificado por el analizador léxico completo desarrollado en las semanas 5-7, manteniendo la interfaz de tokens.

ANEXOS
Anexo A: Código Fuente Completo
[El código HTML/JS completo va aquí o como archivo adjunto]

Anexo B: Tabla LL(1) Completa
No Terminal	RAIZ	(	)	+	-	*	/	dig	$
S	E	E	-	-	-	-	-	E	-
E	TE'	TE'	-	-	-	-	-	TE'	-
E'	-	-	λ	+TE'	-TE'	-	-	-	λ
T	UT'	UT'	-	-	-	-	-	UT'	-
T'	-	-	λ	λ	λ	*UT'	/UT'	-	λ
U	RAIZ F	F	-	-	-	-	-	F	-
F	-	(E)	-	-	-	-	-	num	-
num	-	-	-	-	-	-	-	dig num'	-
num'	-	-	λ	λ	λ	λ	λ	dig num'	λ
dig	-	-	-	-	-	-	-	VAL	-
Notación: - indica celda vacía (error), λ es la cadena vacía

Anexo C: Glosario
AST: Abstract Syntax Tree (Árbol de Sintaxis Abstracta)
GIC: Gramática Independiente del Contexto
LL(1): Left-to-right, Leftmost derivation, 1 symbol lookahead
Lexer: Analizador léxico (tokenizador)
Parser: Analizador sintáctico
Token: Símbolo terminal reconocido por el lexer
