# Sudoku

Posar aquí el tema de article de Albert Molins a LV i que la gent em preguntava una mica com va això. Doncs comencem amb un exemple senzill. 
(posar links a article Albert LV)
Todo el mundo ha visto un Sudoku en la sección de pasatiempos de su revista o periodico. Pero no siempre es fácil, ¿Verdad? Aquí les dejo un programa para que puedan dejar boquiabiertos a sus amigos. 

 
## Estrategia
 
Una estratégia para solucionar el sudoky del domingo de La Vanguardia que la mayoría de mis amigos usa consiste en seguir dos pasos:

* If a box has a value, then all the boxes in the same row, same column, or same 3x3 square cannot have that same value.
* If there is only one allowed value for a given box in a row, column, or 3x3 square, then the box is assigned that value.

## Algoritmo

### Introducción
Se trata de un simble algoritmo en Python que usa dos técnicas básicas de Inteligencia Artificial:
* **Constraint Propagation**: When trying to solve a problem, you'll find that there are some local constraints to each square. These constraints help you narrow the possibilities for the answer, which can be very helpful. We will learn to extract the maximum information out of these constraints in order to get closer to our solution. Additionally, you'll see how we can repeatedly apply simple constraints to iteratively narrow the search space of possible solutions. Constraint propagation can be used to solve a variety of problems such as calendar scheduling, and cryptographic puzzles.
* **Search**: In the process of problem solving, we may get to the point where two or more possibilities are available. What do we do? What if we branch out and consider both of them? Maybe one of them will lead us to a position in which three or more possibilities are available. Then, we can branch out again. At the end, we can create a whole tree of possibilities and find ways to traverse the tree until we find our solution. This is an example of how search can be used.

### Notación y nomenclatura
En nuestro código las filas las etiquetaremos con las letras `A`, `B`, `C`, `D`, `E`, `F`, `G`, `H`, `I`. Y las columnas con los números `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`. Es decir, las posiciones de nuestro tablero quedarán etiquetadas como:

 ```
 A1 A2 A3| A4 A5 A6| A7 A8 A9     
 B1 B2 B3| B4 B5 B6| B7 B8 B9    
 C1 C2 C3| C4 C5 C6| C7 C8 C9    
--------+---------+---------    
 D1 D2 D3| D4 D5 D6| D7 D8 D9    
 E1 E2 E3| E4 E5 E6| E7 E8 E9    
 F1 F2 F3| F4 F5 F6| F7 F8 F9    
--------+---------+---------    
 G1 G2 G3| G4 G5 G6| G7 G8 G9    
 H1 H2 H3| H4 H5 H6| H7 H8 H9   
 I1 I2 I3| I4 I5 I6| I7 I8 I9    
 
 ```
 En el código vamos a usar la siguiente nomenclatura:
 
* The individual squares at the intersection of rows and columns will be called `boxes` . These boxes will have labels 'A1', 'A2', ..., 'I9'.
* The complete rows, columns, and 3x3 squares, will be called `units`. Thus, each unit is a set of 9 boxes, and there are 27 units in total.
* For a particular box (such as 'A1'), sus pares ( `peers`)  will be all other boxes that belong to a common unit (namely, those that belong to the same row, column, or 3x3 square). Por tanto para cada casilla, hay 20 pares.  Por ejemplo los pares de 'A1' son :  row: A2, A3, A4, A5, A6, A7, A8, A9 column: B1, C1, D1, E1, F1, G1, H1, I1 3x3 square: B2, B3, C2, C3 (since A1, A2, A3, B1, C1 are already counted).



### Implementación del tablero en Python

Para facilitar la resolución del problema, vamos a almacenar nuestro tablero en dos formatos: como `string` y como `dictionary`. 

#### String
The string will consist of a concatenation of all the readings of the digits in the rows, taking the rows from top to bottom. If the puzzle is not solved, we can use a **.** para indicar que la casilla aun no tiene valor asignado. 
 Por ejemplo el tablero: 
 ```
. . 3 |. 2 . |6 . . 
9 . . |3 . 5 |. . 1 
. . 1 |8 . 6 |4 . . 
------+------+------
. . 8 |1 . 2 |9 . . 
7 . . |. . . |. . 8 
. . 6 |7 . 8 |2 . . 
------+------+------
. . 2 |6 . 9 |5 . . 
8 . . |2 . 3 |. . 9 
. . 5 |. 1 . |3 . . 
```
lo almacenaremos con el `string`:
```
'..3.2.6..9..3.5..1..18.64....81.29..7.......8..67.82....26.95..8..2.3..9..5.1.3..'
```
#### En formato `dictionary`

We'll implement the dictionary as follows. The *keys* will be strings corresponding to the boxes — namely, `'A1', 'A2', ..., 'I9'`. The values will either be the digit in each box (if there is one) or a '.' (if not).

Para generar nuestra estructura de datos que contendrá el tablero vamos a empezar programando una función de soporte que llamaremos `cross(a, b)` que dados dos strings `a` y `b` la función retorna la lista  (recordemos que una lista se especifica con `[` `]`) formada por todas las posibles concatenaciones de letras `s`en el string `a` con la `t` en el string `b`.  

```python
def cross(a, b):
      return [s+t for s in a for t in b]
```

Por ejemplo `cross('abc', 'def')` retornara la lista `['ad', 'ae', 'af', 'bd', 'be', 'bf', 'cd', 'ce', 'cf']`. Ahora, para crear todas las etiquetas de las casillas que almacenaremos en `boxes` podemos hacerlo de la siguiente manera:

```python
rows = 'ABCDEFGHI'
cols = '123456789'

boxes = cross(rows, cols)
```
Podemos comprobarlo con `print boxes` o `print (boxes)` (dependiendo si usamos Pyhon 2.x o 3.x) que nos dará la siguiente salida:
```python
[['A1', 'A2', 'A3', 'A4', 'A5', 'A6', 'A7', 'A8', 'A9'], ['B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B9'], ['C1', 'C2', 'C3', 'C4', 'C5', 'C6', 'C7', 'C8', 'C9'], ['D1', 'D2', 'D3', 'D4', 'D5', 'D6', 'D7', 'D8', 'D9'], ['E1', 'E2', 'E3', 'E4', 'E5', 'E6', 'E7', 'E8', 'E9'], ['F1', 'F2', 'F3', 'F4', 'F5', 'F6', 'F7', 'F8', 'F9'], ['G1', 'G2', 'G3', 'G4', 'G5', 'G6', 'G7', 'G8', 'G9'], ['H1', 'H2', 'H3', 'H4', 'H5', 'H6', 'H7', 'H8', 'H9'], ['I1', 'I2', 'I3', 'I4', 'I5', 'I6', 'I7', 'I8', 'I9']]

```

En este punto tenemos en formato string el contenido de nuestro tablero y vamos a pasarlo en formato diccionario de Python usando la notación que hemos decidido anteriormente. Es decir, el string `'..3.2.6..9..3.5..1..18.64....81.29..7.......8..67.82....26.95..8..2.3..9..5.1.3..'` lo queremos tener como:

```
{
  'A1': '.'
  'A2': '.',
  'A3': '3',
  'A4': '.',
  'A5': '2',
  
  ...
  
  'I9': '.'
}
```

Para ello implementamos una nueva función que llamaremos `grid_values()` basada en la librería de Python [`zip`] (https://docs.python.org/3.3/library/functions.html) que nos facilitará esta tarea de repartir este string de entrada que contiene los valores de las casillas (`boxes`):


```python
def grid_values(grid):
    return dict(zip(boxes, grid))
```
Recordemos que el string de entrada a la función que representa el tablero de nuestro sudoku debe ser de 81 carácteres (9x9) compuesto de dígitos `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, o bien de `.` si aun no lo sabemos. Veamos un ejemplo, pero previamente nos dotamos de una función para visualizar más facilmente nuestro tablero de sudoku:

```python
def display(values):
    """
    Visualiza nuestro tablero de sudoku
    """
    width = 1+max(len(values[s]) for s in boxes)
    line = '+'.join(['-'*(width*3)]*3)
    for r in rows:
        print(''.join(values[r+c].center(width)+('|' if c in '36' else '')
                      for c in cols))
        if r in 'CF': print(line)
    return
```
Con esta función podemos visualizar  el tablero en el formato que estamos acostumbrados. Aplicando esta función a nuestro ejemplo anterior:

```python
example='483.2.6..9..3.5..1..18.64....81.29..7.......8..67.82....26.95..8..2.3..9..5.1.382'
display(grid_values(example))
```
el resultado de `display` será:
```
. . 3 |. 2 . |6 . . 
9 . . |3 . 5 |. . 1 
. . 1 |8 . 6 |4 . . 
------+------+------
. . 8 |1 . 2 |9 . . 
7 . . |. . . |. . 8 
. . 6 |7 . 8 |2 . . 
------+------+------
. . 2 |6 . 9 |5 . . 
8 . . |2 . 3 |. . 9 
. . 5 |. 1 . |3 . . 
```

## Primer paso: Eliminación

Como primer paso en nuestra estratégia usaremos lo que llamamos **eliminación**. Empezaremos mirando una casilla y analizando que valores pueden ir allí. Por ejemplo en la posición E6, marcado con una X en el siguiente tablero:
```
. . 3 |. 2 . |6 . . 
9 . . |3 . 5 |. . 1 
. . 1 |8 . 6 |4 . . 
------+------+------
. . 8 |1 . 2 |9 . . 
7 . . |. . X |. . 8 
. . 6 |7 . 8 |2 . . 
------+------+------
. . 2 |6 . 9 |5 . . 
8 . . |2 . 3 |. . 9 
. . 5 |. 1 . |3 . . 
```
Vemos de entre los posibles valores `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9` no todos son posibles puesto que ya aparecen o bien en la misma columna, misma fila o en el cuadrado 3x3 correspondiente a la posición E6. En concreto, el `1`, `7`, `2` y `8` ya se encuentran en el mismo cuadrado 3x3. Los valores `3`, `5`, `6` y `9` se encuentran en la misma columna. Y los `7` y `8` se encuentran en la misma fila, aunque ya los habiamos descartado por encontrarse en el mismo cuadrado. Por tanto, en este caso, solo tenemos el `4` que cumple los requisitos.

Now that we know how to eliminate values, we can take one pass, go over every box that has a value, and eliminate the values that can't appear on the box si ya aparecen en su misma columna, fila o cuadrado.

Vamos a incorporar en la anterior función `grid_values()` y añadir información con los valores posibles para una determinada casilla. Por ejemplo en `B5`pondremos el valor `47`(dado que `4` y `7`son los dos únicos valores posibles para esta casilla). Para ello, de momento vamos a reprogramar la función `grid_values()` para que nos devuelva `'123456789'` en vez del `'.'`para las casillas vacias, puesto que los valores iniciales para estas casillas puede ser cualquier valor.

```python
def grid_values(grid):
    values = []
    for c in grid:
        if c == '.':
            values.append('123456789')
        elif c in '123456789':
            values.append(c)
    return dict(zip(boxes, values))
 ```
Ahora esta función convierte el string que recibe de entrada (con 9x9 carácteres) en un diccionario `{<box>: <value>}` que contiene para cada clave (que indica la posiciónc dentro del tablero) su valor correspondiente o '123456789' si está vacio.

## Segundo paso: Eliminación

Para ello primero debemos tener controladas todas las columnas, todas las filas y todos los cuadrados de 3x3 relacionados con una determinada casilla del tablero.  Para ello con la misma función `cross('abc', 'def')` vamos a generar todas las columnas, todas las filas y todos los cuadrados de 3x3:

```python
row_units = [cross(r, cols) for r in rows]
column_units = [cross(rows, c) for c in cols]
square_units = [cross(rs, cs) for rs in ('ABC','DEF','GHI') for cs in ('123','456','789')]
```
Lo podemos comprobar:
```python
print (row_units)
[['A1', 'A2', 'A3', 'A4', 'A5', 'A6', 'A7', 'A8', 'A9'], ['B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B9'], ['C1', 'C2', 'C3', 'C4', 'C5', 'C6', 'C7', 'C8', 'C9'], ['D1', 'D2', 'D3', 'D4', 'D5', 'D6', 'D7', 'D8', 'D9'], ['E1', 'E2', 'E3', 'E4', 'E5', 'E6', 'E7', 'E8', 'E9'], ['F1', 'F2', 'F3', 'F4', 'F5', 'F6', 'F7', 'F8', 'F9'], ['G1', 'G2', 'G3', 'G4', 'G5', 'G6', 'G7', 'G8', 'G9'], ['H1', 'H2', 'H3', 'H4', 'H5', 'H6', 'H7', 'H8', 'H9'], ['I1', 'I2', 'I3', 'I4', 'I5', 'I6', 'I7', 'I8', 'I9']]
print(column_units)
[['A1', 'B1', 'C1', 'D1', 'E1', 'F1', 'G1', 'H1', 'I1'], ['A2', 'B2', 'C2', 'D2', 'E2', 'F2', 'G2', 'H2', 'I2'], ['A3', 'B3', 'C3', 'D3', 'E3', 'F3', 'G3', 'H3', 'I3'], ['A4', 'B4', 'C4', 'D4', 'E4', 'F4', 'G4', 'H4', 'I4'], ['A5', 'B5', 'C5', 'D5', 'E5', 'F5', 'G5', 'H5', 'I5'], ['A6', 'B6', 'C6', 'D6', 'E6', 'F6', 'G6', 'H6', 'I6'], ['A7', 'B7', 'C7', 'D7', 'E7', 'F7', 'G7', 'H7', 'I7'], ['A8', 'B8', 'C8', 'D8', 'E8', 'F8', 'G8', 'H8', 'I8'], ['A9', 'B9', 'C9', 'D9', 'E9', 'F9', 'G9', 'H9', 'I9']]
print (square_units)
[['A1', 'A2', 'A3', 'B1', 'B2', 'B3', 'C1', 'C2', 'C3'], ['A4', 'A5', 'A6', 'B4', 'B5', 'B6', 'C4', 'C5', 'C6'], ['A7', 'A8', 'A9', 'B7', 'B8', 'B9', 'C7', 'C8', 'C9'], ['D1', 'D2', 'D3', 'E1', 'E2', 'E3', 'F1', 'F2', 'F3'], ['D4', 'D5', 'D6', 'E4', 'E5', 'E6', 'F4', 'F5', 'F6'], ['D7', 'D8', 'D9', 'E7', 'E8', 'E9', 'F7', 'F8', 'F9'], ['G1', 'G2', 'G3', 'H1', 'H2', 'H3', 'I1', 'I2', 'I3'], ['G4', 'G5', 'G6', 'H4', 'H5', 'H6', 'I4', 'I5', 'I6'], ['G7', 'G8', 'G9', 'H7', 'H8', 'H9', 'I7', 'I8', 'I9']]
```

Juntamos todo en una sola lista:

```python
unitlist = row_units + column_units + square_units

```
Con el constructor `dict()` que construye diccionarios directamente de secuencias de pares clave-valor vamos a generar el diccionario `units` que nos dice por cada casilla `s in boxes` que casillas componen la fila, la columna i el cuadrado de 3x3 cuyo valor debemos tener en cuenta: 
```python
units = dict((s, [u for u in unitlist if s in u]) for s in boxes)
 ```
Por otro lado, usando el módulo `sets` (que provides classes for constructing and manipulating unordered collections of unique elements) construimos el diccionario `peers` que nos elimina duplicados de casillas por cada casilla:

```python
peers = dict((s, set(sum(units[s],[]))-set([s])) for s in boxes)
 ```
 
Now, let's finish the code for the function `eliminate()`. The function will iterate over all the boxes in the puzzle that only have one value assigned to them, and it will remove this value from every one of its peers. `eliminate()` Go through all the boxes, and whenever there is a box with a single value, eliminate this value from the set of values of all its peers.
will take as input a puzzle in dictionary form y retornará un nuevo diccionario con los valores eliminados.
 
```python
def eliminate(values):
    solved_values = [box for box in values.keys() if len(values[box]) == 1]
    for box in solved_values:
        digit = values[box]
        for peer in peers[box]:
            values[peer] = values[peer].replace(digit,'') "elimina el dígito"
    return values
```


# Strategy 2: Only Choice
ARA VAIG PER AQUÍ no he continuat per tenir més temps
# Constraint Propagation


### Agradecimientos

Para este ejercicio nos hemos inspirado en el [post fantástico de Peter Norvig](http://norvig.com/sudoku.html) y parte de los ejercicios del nanodregree de AI de Udacity). 
