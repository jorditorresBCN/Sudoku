# Sudoku

Todo el mundo ha visto un Sudoku en la sección de pasatiempos de su revista o periodico. Pero no siempre es fácil, ¿Verdad? Aquí les dejo un programa para que puedan dejar boquiabiertos a sus amigos. 

Se trata de un simble algoritmo en Python que usa dos técnicas básicas de Inteligencia Artificial:
* **Constraint Propagation**: When trying to solve a problem, you'll find that there are some local constraints to each square. These constraints help you narrow the possibilities for the answer, which can be very helpful. We will learn to extract the maximum information out of these constraints in order to get closer to our solution. Additionally, you'll see how we can repeatedly apply simple constraints to iteratively narrow the search space of possible solutions. Constraint propagation can be used to solve a variety of problems such as calendar scheduling, and cryptographic puzzles.
* **Search**: In the process of problem solving, we may get to the point where two or more possibilities are available. What do we do? What if we branch out and consider both of them? Maybe one of them will lead us to a position in which three or more possibilities are available. Then, we can branch out again. At the end, we can create a whole tree of possibilities and find ways to traverse the tree until we find our solution. This is an example of how search can be used.

Siguiendo el [post fantástico de Peter Norvig](http://norvig.com/sudoku.html) en el que me baso para este ejemplo, Since we're writing an agent to solve the Sudoku puzzle, let's start by labelling rows and columns.

* The rows will be labelled by the letters A, B, C, D, E, F, G, H, I.
* The columns will be labelled by the numbers 1, 2, 3, 4, 5, 6, 7, 8, 9. Here we can see the unsolved and solved puzzles with the labels for the rows and columns.
* The 3x3 squares won't be labelled, but in the diagram, they can be seen with alternating colors of grey and white. ULL AMB AIXO
