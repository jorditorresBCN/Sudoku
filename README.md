# Tu primer programa de Inteligencia Artificial: resolver un Sudoku

POST: http://jorditorres.org/tu-primer-programa-de-inteligencia-artificial-resolver-un-sudoku/

Hace unos días el periodista [Albert Molins publicaba en La Vanguardia](http://jorditorres.org/inteligencia-artificial-y-poquer/) un excelente artículo titulado “Una máquina gana al póquer a los mejores jugadores del mundo” para el que me llamo para contrastar algunos datos sobre este sistema de inteligencia artificial, Libratus, desarrollado por la Universidad Carnegie Mellon. En el mismo artículo Albert se hacia eco de anteriores duelos entre sistemas de inteligencia artificial y humanos en diferentes juegos como el Ajedrez (en 1997 el ordenador Deeper Blue derrotó a Kaspárov), concurso de preguntas y respuestas de la televisión estadounidense (en 2011 el ordenador Watson ganó a Brad Ruttler y Ken Jennings, los dos mejores concursantes del concurso Jeopardy) o el Go (en el 2016 el sistema AlphaGo desarrollado por la empresa DeepMind de Google ganó a Lee Se-dol, campeón mundial de Go).

A raíz de este artículo algunos me han preguntado si podría explicar un poco más a nivel técnico como funcionan por dentro estos sistemas supuestamente inteligentes. Aunque ciertamente son sistemas complejos, que requieren además mucha computación y no están al alcance de cualquiera, no es menos cierto que son algoritmos que un ingeniero informático sin ninguna duda puede entender, y me atrevería a decir, programar. 

Por ello, he decidido escribir este  post para explicar como es el código de un programa que usa algunas de las técnicas de inteligencia artificial para resolver un juego como puede ser un _Sudoku_. El sistema que permite la resolución automática de este juego utiliza algunas técnicas de las que usan los sistemas que resuelven los anteriores juegos mencionados, aunque son sistemas mucho más sofisticados que ya intentaré explicar algún día. 

Todos sabemos hacer un _Sudoku_ y sus reglas básicas, pero las recuerdo rápidamente pensando ya en el algoritmo que presentaré:

* Un _Sudoku_ es básicamente una cuadrícula de 9x9 casillas dividida en 9 cuadrados de 3x3 casillas.
* Los valores que puede contener una casilla  son `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8` o `9`.
* Si una casilla contiene un determinado valor, entonces ninguna de las casillas en la misma columna, fila o cuadrado de 3x3 al que pertenece dicha casilla puede contener ese mismo valor.
* Si solo hay un valor permitido para una determinada casilla dada su columna, fila o cuadrado al que pertenece, entonces ese valor se asignará a dicha casilla.

Con la programación de estas simples reglas, este algoritmo "inteligente" que les propongo, **siempre** (insisto, siempre) va a resolver cualquier Sudoku más rápido que cualquiera de sus amigos o amigas, suponiendo que ellos o ellas puedan resolverlo. Llévenlo encima en el móvil :-). 

Para presentar el tema sigo el modelo "learn by doing" que presupone que el lector va programando en **Python** (que uso en mis cursos) y experimentando a medida que van avanzando en el post, tal como lo hacemos en los hands-on de los laboratorios de mis clases en la UPC. En realidad les propongo que usen el entorno Anaconda disponible en cualquier sistema operativo actual y que pueden contar con una explicación detallada en [uno de los hands-on](https://github.com/jorditorresBCN/Quick-Start/blob/master/Phyton-Development-Environment-Quick-Start.md) de nuestra asignatura en la UPC). 

1.	[Notación y nomenclatura](#notacion)
2.	[Eliminación de opciones en una casilla](#eliminacion)
3.	[Casillas con una sola opción](#solaopcion)
4.	[Propagación de restricciones](#propagacion)
5.	[Estategia Search](#search)
6.	[Y para acabar](#conclusiones)
7.	[Agradecimientos](#agradecimientos)

<a name="notacion"/>


## 1. Notación y nomenclatura

Antes de empezar a programar debemos acordar una cierta notación. Les propongo la siguiente.


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
 
* A las casillas las llamaremos `boxes`.
* A las columnas, filas o cuadrados 3x3 los llamaremos `units`.  Por tanto, cada elemento de `units` contiene 9 `boxes`, siendo 27 el número total de `units`. 
* Para una casilla (`box`) en particular, sus pares (que llamaremos `peers`) serán todas las otras casillas (`boxes`) que pertenecen a una misma `unit`, es decir, serán todas las otras `box` que pertenecen a cualquier `unit` común (misma columna, fila o cuadrado 3x3). Por tanto para cada casilla, hay 20 pares.  Por ejemplo los pares de `A1` son las casillas de la columna `A2`, `A3`, `A4`, `A5`, `A6`, `A7`, `A8`, `A9`, junto con las casillas de la columna `B1`, `C1`, `D1`, `E1`, `F1`, `G1`, `H1`, `I1` y junto con las casillas del cuadrado 3x3 formado por  `B2`, `B3`, `C2`, `C3` (teniendo en cuenta que `A1`, `A2`, `A3`, `B1`, `C1` ya están contabilizados).



Para facilitar la resolución del problema, vamos a almacenar nuestro tablero o cuadrícula en dos formatos: como `string` y como `dictionary`. 

El formato `string` consiste en la concatenación de todos los dígitos de todas las casillas de las filas desde arriba hacia abajo. Si el _Sudoku_ aún no está solucionado podemos usar '.' para indicar que la casilla aún no tiene valor asignado. 
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

Por otra parte implementaremos el diccionario de tal manera que las *keys* serán los *strings* correspondientes a las casillas (`'A1', 'A2', ..., 'I9'`) y los valores serán o bien el dígito en la casilla o '.'.

Para generar nuestra estructura de datos que contendrá el tablero cuadriculado vamos a empezar programando una función de soporte que llamaremos `cross(a, b)` que dados dos strings `a` y `b` la función retorna la lista  (recordemos que una lista se especifica con `[` `]`) formada por todas las posibles concatenaciones de letras `s` en el string `a` con la `t` en el string `b`.  

```python
def cross(a, b):
      return [s+t for s in a for t in b]
```

Por ejemplo `cross('abc', 'def')` retornará la lista `['ad', 'ae', 'af', 'bd', 'be', 'bf', 'cd', 'ce', 'cf']`. Ahora, para crear todas las etiquetas de las casillas que almacenaremos en `boxes` podemos hacerlo de la siguiente manera:

```python
rows = 'ABCDEFGHI'
cols = '123456789'

boxes = cross(rows, cols)
```
Podemos comprobarlo con `print boxes` o `print (boxes)` (dependiendo si usamos Python 2.x o 3.x) que nos dará la siguiente salida:
```python
['A1', 'A2', 'A3', 'A4', 'A5', 'A6', 'A7', 'A8', 'A9', 'B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B9', 'C1', 'C2', 'C3', 'C4', 'C5', 'C6', 'C7', 'C8', 'C9', 'D1', 'D2', 'D3', 'D4', 'D5', 'D6', 'D7', 'D8', 'D9', 'E1', 'E2', 'E3', 'E4', 'E5', 'E6', 'E7', 'E8', 'E9', 'F1', 'F2', 'F3', 'F4', 'F5', 'F6', 'F7', 'F8', 'F9', 'G1', 'G2', 'G3', 'G4', 'G5', 'G6', 'G7', 'G8', 'G9', 'H1', 'H2', 'H3', 'H4', 'H5', 'H6', 'H7', 'H8', 'H9', 'I1', 'I2', 'I3', 'I4', 'I5', 'I6', 'I7', 'I8', 'I9']

```

En este punto tenemos en formato string el contenido de nuestro tablero y vamos a pasarlo a formato diccionario de Python usando la notación que hemos decidido anteriormente. Es decir, el string `'..3.2.6..9..3.5..1..18.64....81.29..7.......8..67.82....26.95..8..2.3..9..5.1.3..'` lo queremos tener como:

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
def grid_values_original(grid):
    return dict(zip(boxes, grid))
```
Recordemos que el string de entrada a la función que representa el tablero de nuestro sudoku debe ser de 81 caracteres (9x9) compuesto de dígitos `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, o bien de `.` si aún no lo sabemos. Veamos un ejemplo, pero previamente nos dotamos de una función para visualizar más fácilmente nuestro tablero de sudoku:

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
display(grid_values_original(example))
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
<a name="eliminacion"/>

## 2. Eliminación de opciones en una casilla

### Descripción
Como primer paso en nuestra estrategia usaremos lo que llamamos **eliminación** que trata de eliminar posibilidades de los pares.

Empezaremos mirando una casilla y analizando que valores pueden ir allí. Por ejemplo en la posición E6, marcado con una X en el siguiente tablero:
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
vemos que de entre todos los valores `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9` no todos son posibles puesto que ya aparecen o bien en la misma columna, o en la misma fila o en el cuadrado 3x3 correspondiente a la posición E6. En concreto, `1`, `7`, `2` y `8` ya se encuentran en el mismo cuadrado 3x3. Los valores `3`, `5`, `6` y `9` se encuentran en la misma columna. Y los valores `7` y `8` se encuentran en la misma fila, aunque ya los habíamos descartado por encontrarse en el mismo cuadrado. Por tanto, en este caso, solo tenemos el `4` que cumple los requisitos.

Ahora que ya conocemos como eliminar valores, podemos hacerlo para todas las casillas que no tienen valor y eliminar los valores que no pueden aparecer en la casilla al estar ya presentes en su misma columna, fila o cuadrado 3x3.

### función `grid_values()`
Para ello, de momento vamos a reprogramar la función `grid_values()` para que nos devuelva `'123456789'` en vez del `'.'`para las casillas vacías, puesto que los valores iniciales para estas casillas puede ser cualquier valor.

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
Ahora esta función convierte el string que recibe de entrada (con 9x9 carácteres) en un diccionario `{<box>: <value>}` que contiene para cada clave (que indica la posición dentro del tablero) su valor correspondiente o '123456789' si está vacío.

```python
example='483.2.6..9..3.5..1..18.64....81.29..7.......8..67.82....26.95..8..2.3..9..5.1.382'
display(grid_values_original(example))
display(grid_values(example))
```
```
4 8 3 |. 2 . |6 . . 
9 . . |3 . 5 |. . 1 
. . 1 |8 . 6 |4 . . 
------+------+------
. . 8 |1 . 2 |9 . . 
7 . . |. . . |. . 8 
. . 6 |7 . 8 |2 . . 
------+------+------
. . 2 |6 . 9 |5 . . 
8 . . |2 . 3 |. . 9 
. . 5 |. 1 . |3 8 2 
    4         8         3     |123456789     2     123456789 |    6     123456789 123456789 
    9     123456789 123456789 |    3     123456789     5     |123456789 123456789     1     
123456789 123456789     1     |    8     123456789     6     |    4     123456789 123456789 
------------------------------+------------------------------+------------------------------
123456789 123456789     8     |    1     123456789     2     |    9     123456789 123456789 
    7     123456789 123456789 |123456789 123456789 123456789 |123456789 123456789     8     
123456789 123456789     6     |    7     123456789     8     |    2     123456789 123456789 
------------------------------+------------------------------+------------------------------
123456789 123456789     2     |    6     123456789     9     |    5     123456789 123456789 
    8     123456789 123456789 |    2     123456789     3     |123456789 123456789     9     
123456789 123456789     5     |123456789     1     123456789 |    3         8         2     
```


### función `eliminate()`
El siguiente paso consiste en reducir los valores posibles de las casillas de acuerdo a la descripción que hemos hecho anteriormente. 

Para ello primero debemos tener controladas todas las columnas, todas las filas y todos los cuadrados de 3x3 relacionados con una determinada casilla del tablero. Para ello con la misma función `cross('abc', 'def')` vamos a generar todas las columnas, todas las filas y todos los cuadrados de 3x3:

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
Vamos a generar el diccionario `units` que nos dice por cada casilla `s in boxes` que casillas componen la fila, la columna i el cuadrado de 3x3 cuyo valor debemos tener en cuenta: 
```python
units = {s: [u for u in unitlist if s in u] for s in boxes}
 ```
Por otro lado, usando el módulo `sets` (que permite construir y manipular colecciones no ordenadas de elementos únicos) construimos el diccionario `peers` que nos elimina duplicados de casillas por cada casilla:

```python
peers = {s: set(sum(units[s],[]))-set([s]) for s in boxes}
 ```

En resumen, la función `eliminate()` iterará sobre todas las celdas de la cuadrícula que tienen solo un valor asignado, y borrará este valor de todos sus pares. La vamos a implementar de tal manera que reciba como entrada un diccionario y retornará otro diccionario con los valores eliminados.
 
```python
def eliminate(values):
    solved_values = [box for box in values.keys() if len(values[box]) == 1]
    for box in solved_values:
        digit = values[box]
        for peer in peers[box]:
            values[peer] = values[peer].replace(digit,'') "elimina el dígito"
    return values
```

```python
example='483.2.6..9..3.5..1..18.64....81.29..7.......8..67.82....26.95..8..2.3..9..5.1.382'
display(grid_values(example))
display(eliminate(grid_values(example)))
```

```
    4         8         3     |123456789     2     123456789 |    6     123456789 123456789 
    9     123456789 123456789 |    3     123456789     5     |123456789 123456789     1     
123456789 123456789     1     |    8     123456789     6     |    4     123456789 123456789 
------------------------------+------------------------------+------------------------------
123456789 123456789     8     |    1     123456789     2     |    9     123456789 123456789 
    7     123456789 123456789 |123456789 123456789 123456789 |123456789 123456789     8     
123456789 123456789     6     |    7     123456789     8     |    2     123456789 123456789 
------------------------------+------------------------------+------------------------------
123456789 123456789     2     |    6     123456789     9     |    5     123456789 123456789 
    8     123456789 123456789 |    2     123456789     3     |123456789 123456789     9     
123456789 123456789     5     |123456789     1     123456789 |    3         8         2     
   4      8      3   |   9      2      17  |   6     579     57  
   9     267     7   |   3      47     5   |   78     27     1   
   25    257     1   |   8      79     6   |   4    23579   357  
---------------------+---------------------+---------------------
   35    345     8   |   1     3456    2   |   9    34567  34567 
   7    123459   49  |  459   34569    4   |   1    13456    8   
  135   13459    6   |   7     3459    8   |   2     1345   345  
---------------------+---------------------+---------------------
   13    1347    2   |   6     478     9   |   5     147     47  
   8     1467    47  |   2     457     3   |   17    1467    9   
   6     4679    5   |   4      1      47  |   3      8      2   
```

<a name="solaopcion"/>

## 3. Casillas con una sola opción
Ahora estamos en la situación de que nos encontramos en un punto donde hemos marcado casillas que en teoría pueden contener varias opciones pero que dados sus pares solo puede ser una de ellas.  Para ello, vamos a ir a través de todas las `units` y en el caso de que encontremos una `unit` con un valor que solo encaja en una casilla, vamos a asignar este valor en dicha casilla. Por ejemplo, en nuestro ejemplo, el primer cuadrado 3x3 tiene los siguientes valores:

```
+---------------------+
|   4      8      3   |  
|   9     267     7   |   
|   25    257     1   |  
+---------------------+
```
Si miramos la casilla `B2` vemos que puede contener tres valores: `2`,`6` o `7`. Pero resulta que solo el `6` puede aparecer en esta casilla, por ello ya le asignamos este valor a la casilla eliminando las otras opciones.

```python
display(example_after_eliminate)
example_after_only_choice=only_choice(example_after_eliminate)
print (" "), print (" "), print (" ")
display(example_after_only_choice)
```
```
   4      8      3   |   9      2      17  |   6     579     57  
   9     267     7   |   3      47     5   |   78     27     1   
   25    257     1   |   8      79     6   |   4    23579   357  
---------------------+---------------------+---------------------
   35    345     8   |   1     3456    2   |   9    34567  34567 
   7    123459   49  |  459   34569    4   |   1    13456    8   
  135   13459    6   |   7     3459    8   |   2     1345   345  
---------------------+---------------------+---------------------
   13    1347    2   |   6     478     9   |   5     147     47  
   8     1467    47  |   2     457     3   |   17    1467    9   
   6     4679    5   |   4      1      47  |   3      8      2   
 
 
 
  4     8     3   |  9     2     1   |  6    579    57  
  9     6     7   |  3     4     5   |  8     27    1   
  2     5     1   |  8     7     6   |  4   23579  357  
------------------+------------------+------------------
  35   345    8   |  1    3456   2   |  9     7     6   
  7     2     9   |  5   34569   4   |  1   13456   8   
 135  13459   6   |  7    3459   8   |  2    1345  345  
------------------+------------------+------------------
  13   1347   2   |  6     8     9   |  5    147    47  
  8    1467   47  |  2     5     3   |  7     6     9   
  6     9     5   |  4     1     7   |  3     8     2   

```

<a name="propagacion"/>

## 4. Propagación de restricciones
Después de analizar estas dos estrategias para encontrar limitaciones locales que nos permiten encontrar valores a las casillas, lo que intuitivamente se nos ocurre es combinarlas iterativamente para ir resolviendo el *sudoku*, donde en cada iteración, los nuevos valores resueltos nos dan la solución para otras casillas. Por tanto, el código que nos queda podría ser:

```python
def reduce_puzzle(values):
    stalled = False
    while not stalled:
        # Check how many boxes have a determined value
        solved_values_before = len([box for box in values.keys() if len(values[box]) == 1])

        # se the Eliminate Strategy
        values = eliminate(values)

        # Use the Only Choice Strategy
        values = only_choice(values)

        # Check how many boxes have a determined value, to compare
        solved_values_after = len([box for box in values.keys() if len(values[box]) == 1])
        # If no new values were added, stop the loop.
        stalled = solved_values_before == solved_values_after
        # Sanity check, return False if there is a box with zero available values:
        if len([box for box in values.keys() if len(values[box]) == 0]):
            return False
    return values
```
En este código tenemos en cuenta cuando debemos parar, con la condición de `stalled = solved_values_before == solved_values_after`, que nos indica cuando en una nueva iteración no se ha podido resolver ninguna nueva casilla. Es el caso que retorna `false` porque alguna casilla no contenga ningún valor posible (lo veremos en el siguiente apartado).


Si retomamos el ejemplo inicial vemos que con estas dos simples estrategias podemos solucionar el *sudoku*:

```python
example='..3.2.6..9..3.5..1..18.64....81.29..7.......8..67.82....26.95..8..2.3..9..5.1.3..'
display(grid_values_original(example))
display(reduce_puzzle(grid_values(example)))
```

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
 
 
 
4 8 3 |9 2 1 |6 5 7 
9 6 7 |3 4 5 |8 2 1 
2 5 1 |8 7 6 |4 9 3 
------+------+------
5 4 8 |1 3 2 |9 7 6 
7 2 9 |5 6 4 |1 3 8 
1 3 6 |7 9 8 |2 4 5 
------+------+------
3 7 2 |6 8 9 |5 1 4 
8 1 4 |2 5 3 |7 6 9 
6 9 5 |4 1 7 |3 8 2 
```

<a name="search"/>

## 5. Estategia `Search`

¿Pero estamos seguros que únicamente con las dos anteriores estrategias tenemos suficiente para resolver cualquier _sudoku_? Veamos el siguiente ejemplo:


```python
example='2.............62....1....7......8...3...9...7...6..4...4....8....52.............3'
display(grid_values_original(example))
display(reduce_sudoku(grid_values(example)))
```

```
2 . . |. . . |. . . 
. . . |. . 6 |2 . . 
. . 1 |. . . |. 7 . 
------+------+------
. . . |. . 8 |. . . 
3 . . |. 9 . |. . 7 
. . . |6 . . |4 . . 
------+------+------
. 4 . |. . . |8 . . 
. . 5 |2 . . |. . . 
. . . |. . . |. . 3 

   2     356789  346789 |1345789  134578  134579 | 13569  1345689  145689 
 45789   35789   34789  |1345789  134578    6    |   2     134589  14589  
 45689   35689     1    | 34589   23458   23459  |  3569     7     45689  
------------------------+------------------------+------------------------
 145679  125679  24679  | 13457   123457    8    | 13569   123569  12569  
   3     12568    2468  |  145      9      1245  |  156    12568     7    
 15789   125789   2789  |   6     12357   12357  |   4     123589  12589  
------------------------+------------------------+------------------------
  1679     4     23679  | 13579   13567   13579  |   8     12569   12569  
 16789   136789    5    |   2     134678  13479  |  1679    1469    1469  
 16789   126789  26789  | 145789  145678  14579  | 15679   124569    3    
```

¿Qué pasa si tenemos un sudoku  que no sabemos solucionar tan fácilmente? Vamos a presentar otra técnica básica del mundo de la Inteligencia Artificial para solucionar este problema. Se conoce como `Backtracking`. No entraremos en detalle, pero la idea es que en el proceso de resolución de problemas, a menudo llegamos al punto en que existen varias posibilidades. Una forma de atacar el problema es crear un árbol completo de posibilidades y recorrer el árbol hasta encontrar nuestra solución. 
 
Por ejemplo, la casilla `A2` tiene 5 posibilidades: `4`, `5`, `7`, `8` y `9`. Lo que hacemos es considerar que contiene un `4` y resolver el *Sudoku*. Si no lo podemos resolver (nos vendrá indicado por el retorno de `false` en la función `reduce_puzzle()`) probamos con el siguiente, el `5`, y así sucesivamente. Evidentemente es 5 veces más trabajo, pero es la manera de conseguir todas las opciones.

El código que realiza recursivamente esta iteración por todas las opciones es:

```python
def search(values):
    values = reduce_puzzle(values)
    if values is False:
        return False ## Failed earlier
    if all(len(values[s]) == 1 for s in boxes): 
        return values ## Solved!
    
    # Choose one of the unfilled squares with the fewest possibilities
    unfilled_squares= [(len(values[s]), s) for s in boxes if len(values[s]) > 1]
    n,s = min(unfilled_squares)
    
    # recurrence to solve each one of the resulting sudokus
    for value in values[s]:
        nova_sudoku = values.copy()
        nova_sudoku[s] = value
        attempt = search(nova_sudoku)
        if attempt:
            return attempt
```

```python
def solve(grid):
    # Create a dictionary of values from the grid
    values = grid_values(grid)
    return search(values)
```
Veamos ahora si se puede resolver nuestro anterior ejemplo:
```python
example='2.............62....1....7......8...3...9...7...6..4...4....8....52.............3'
display(grid_values_original(example))
display(solve(example))
```

```
2 . . |. . . |. . . 
. . . |. . 6 |2 . . 
. . 1 |. . . |. 7 . 
------+------+------
. . . |. . 8 |. . . 
3 . . |. 9 . |. . 7 
. . . |6 . . |4 . . 
------+------+------
. 4 . |. . . |8 . . 
. . 5 |2 . . |. . . 
. . . |. . . |. . 3 

2 9 6 |3 4 7 |1 5 8 
5 3 7 |8 1 6 |2 4 9 
4 8 1 |9 2 5 |3 7 6 
------+------+------
1 5 9 |4 7 8 |6 3 2 
3 6 4 |1 9 2 |5 8 7 
7 2 8 |6 5 3 |4 9 1 
------+------+------
9 4 3 |7 6 1 |8 2 5 
8 1 5 |2 3 9 |7 6 4 
6 7 2 |5 8 4 |9 1 3 

```
<a name="conclusiones"/>

## 6. Y para acabar
En este post he querido mostrar con la excusa de resolver un _Sudoku_  como es un algoritmo que usa técnicas simples de inteligencia artificial. Ahora bien, debo enfatizar que en realidad los problemas como el Ajedrez, GO o Poker, mencionados anteriormente, tienen una complejidad tal que aplicar un algoritmo de `Backtracking` como el presentado aquí es imposible, pues tardaríamos siglos en computar la solución (el juego del *Sudoku* a pesar de todo tiene muy pocas combinaciones para explorar). En estos casos existen un gran número de técnicas que si el lector le interesa profundizar un poco más le recomiendo el libro [Artificial Intelligence, a modern approach](https://en.wikipedia.org/wiki/Artificial_Intelligence:_A_Modern_Approach) que solo contiene 1132 páginas. Quizás hay otros con menos páginas pero no duden que este está muy bien.

En el [repositorio de github](https://github.com/jorditorresBCN/Sudoku) encontrarán el notebook `.ipynb` que seguro les puede facilitar seguir este post. Suerte!

<a name="agradecimientos"/>
## 7. Agradecimientos
Para este ejercicio me he inspirado en el [fantástico post de Peter Norvig](http://norvig.com/sudoku.html) y parte del programa de [Artificial Intelligence de Udacity](https://www.udacity.com). Añadir mi agradecimiento a [Francesc Sastre Cabot](https://xiscosc.github.io/) y [Alberto Pou Quirós](https://github.com/bertini36) por la revisión que han realizado a la versión [github](https://github.com/jorditorresBCN/Sudoku) de este post antes de su publicación. 
