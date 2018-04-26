# Tutorial `pdb`


El propósito de este tutorial es enseñar las bases de `pdb`, el **P**ython **D** e**B**ugger
(Depurador de Python) para [Python2](https://docs.python.org/2/library/pdb.html)
y [Python3](https://docs.python.org/3/library/pdb.html).
También incluirá algunos trucos útiles para que tus sesiones de debugging sean
menos estresantes. El tutorial funcionará de la mejor manera si usas Python 2.7
o Python 3.4 y yo iré marcando las diferencias entre las dos versiones si
algún comando de `pdb` cambie.

Para ver qué version de Python estás usando, escribí lo siguiente en tu terminal:


```shell
python --version
```

Ahora que sabés qué versión de python tenés, comencemos!


## Cuál es el propósito de un debugger?

Antes de que saltemos al código, deberíamos tener una breve discusión sobre
la importancia de el debugging y el uso de una herramienta para hacerlo. Para mi,
estos 3 puntos resaltan la importancia de un debugger.

Con un debugger, uno puede:
* Explorar el estado de un programa en ejecución
* Probar la implementación antes de ponerla operativa
* Seguir la lógica de ejecución del programa

Usando un debugger, podés agregar un [breakpoint](https://en.wikipedia.org/wiki/Breakpoint)
en cualquier punto de tu programa para frenarlo y llevar a cabo los 3 puntos
que mencionamos antes. Los debuggers son herramientas poderosas y ellos pueden
acelerar mucho el proceso de debugging que usando los típicos `print()` por
todos lados.

Para aquellos de ustedes que son programadores veteranos, estarán de acuerdo
conmigo en que hay una correlación entre los mejores programadores y aquellos
que saben como debuggean correctamente. Por debuggear correctamente, me refiero
a ser capaz de diagnosticar un problema y tratar el error con facilidad.

Usando un debugger y aprendiendo a cómo usarlo apropiadamente te ayudará a combertirte
en "buen depurador". Tomará algo de tiempo hasta que te sientas cómodo navegando
en un entorno de debugging pero el propósito de este tutorial es ensuciarse antes
de comenzar a usar `pdb` en tu propio código!


## Playing the Game

So we already talked about the purpose of a debugger and now it's time to see it in action. First, you
should clone this repo if you haven't already done so. If you don't have `git` installed, I recommend using
it (or some version of source control) and you can find out details on how to install `git` [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
Once you have `git` installed, clone the repo by entering the following in your terminal:

```shell
git clone https://github.com/spiside/pdb-tutorial
```

**NB**: If that didn't work for you, you should follow Github's [cloning tutorial](https://help.github.com/articles/cloning-a-repository/).

Now that you have the repo cloned, let's navigate to the root of the project and take a look at the instructions given:

```shell
cd /path/to/pdb-tutorial
```

`file: instructions.txt`
```
Your boss has given you the following project to fix for a client. It's supposed to be a simple dice
game where the object of the game is to correctly add up the values of the dice for 6 consecutive turns.

The issue is that a former programmer worked on it and didn't know how to debug effectively.
It's now up to you to fix the errors and finally make the game playable.

To play the game you must run the main.py file.
```

Seems easy enough! To begin, let's try playing the game to see what's wrong. To run the program, type the following in your
terminal:

```shell
python main.py
```

You should see something like this:

```
Introduce los valores del dado
Es así de fácil
Qué es de tu vida
Round 1

---------
|*      |
|       |
|      *|
---------
---------
|*      |
|       |
|      *|
---------
---------
|*     *|
|       |
|*     *|
---------
---------
|*     *|
|       |
|*     *|
---------
---------
|*     *|
|   *   |
|*     *|
---------
Sigh. What is your guess?:
```

Seems like the previous programmer had a sense of...humor? Nonetheless, let's enter 17 (since that is the total value of the dice).

```
Sigh. What is your guess?: 17
Sorry that's wrong
The answer is: 5
Like seriously, how could you mess that up
Wins: 0 Loses 1
Would you like to play again?[Y/n]:
```

Weird. It said the answer is 5 but that's clearly wrong... Alright, maybe the dice addition is wrong but let's play the game again to
figure it out. Looks like the prompt to play again is `'Y'` so let's enter that now.

```
Would you like to play again?[Y/n]: Y
Traceback (most recent call last):
  File "main.py", line 12, in <module>
    main()
  File "main.py", line 8, in main
    GameRunner.run()
  File "/Users/Development/pdb-tutorial/dicegame/runner.py", line 62, in run
    i_just_throw_an_exception()
  File "/Users/Development/pdb-tutorial/dicegame/utils.py", line 13, in i_just_throw_an_exception
    raise UnnecessaryError("You actually called this function...")
dicegame.utils.UnnecessaryError: You actually called this function...
```

Ok weird, there was an exception that was thrown even though we used what was supposed to be a valid input. I think it's safe to
say that the program is broken so let's start the debugging process!


## PDB 101: Intro to `pdb`

It's time to finally work with python's very own debugger `pdb`. The debugger is included in python's standard library and we
use it the same way we would with any python library. First, we have to import the `pdb` module and then call one of its methods
to add a debugging breakpoint in the program. The conventional way to do this is to add the import **and** call the method at the same line you
would like to stop at. This is the full statement you would want to include:

```python
import pdb; pdb.set_trace()
```

The method [`set_trace()`](https://docs.python.org/3/library/pdb.html#pdb.set_trace) hard codes a breakpoint at the line you place
it the command. Let's try it now by opening up the `main.py` file and adding the breakpoint on line 8:

`file: main.py`
```python
from dicegame.runner import GameRunner


def main():
    print("Introduce los valores del dado")
    print("Es así de fácil")
    print("Qué es de tu vida")
    import pdb; pdb.set_trace() # add pdb here
    GameRunner.run()


if __name__ == "__main__":
    main()
```

Cool, now let's try to run `main.py` again and see what happens.

```shell
python main.py
```
```
Introduce los valores del dado
Es así de fácil
Qué es de tu vida
> /Users/Development/pdb-tutorial/main.py(9)main()
-> GameRunner.run()
(Pdb)
```

There we go! We are now in the middle of the running program and we can start poking around. I think the first issue we should
solve is the proper summation of the dice values.

If you are familiar with Python's interpreter, a lot of that knowledge can be transferred to the `pdb` debugger. However, there will be
a couple gotchas that we will get to in the advanced section. Regardless, let's learn a couple commands that will help us solve the
addition issue.


## The 5 `pdb` commands that will leave you "speechless"

Taken directly from the `pdb` documentation, these are the five commands that, once you learn them, you won't know how you lived
without them.

1. `l(ist)` - Displays 11 lines around the current line or continue the previous listing.
2. `s(tep)` - Execute the current line, stop at the first possible occasion.
3. `n(ext)` - Continue execution until the next line in the current function is reached or it returns.
4. `b(reak)` - Set a breakpoint (depending on the argument provided).
5. `r(eturn)` - Continue execution until the current function returns.

Notice that there are brackets around the last part of every keyword. The brackets indicate that the rest of the word is _optional_ when
using the command prompt for `pdb`. This saves typing but a major gotcha is if you have a variable name such as `l` or `n`, then the
`pdb` command takes precedence. That is, say you have a variable named `c` in your program and you want to know the value of `c`. Well,
if you type `c` in `pdb`, you will actually be issuing the `c(ontinue)` keyword which executes the program and only stops if it encounters
a break point!

**NB**: I, and many other programmers, discourage the use of short variable names such as `a`, `b`, `gme`, etc. These carry no meaning
and will confuse other people reading your code. I'm only demonstrating the issues you may encounter with `pdb` in the presence of
shortened variable names.

**NNB**: Another helpful tool is the following:
`h(elp) - Without argument, print the list of available commands. With a command as an argument, print help about that command.`

For the rest of the tutorial, I will be using the shortened version of the commands and if I use a command that I have not introduced
here, I will explain what it does. So, let's begin with the first one.

### 1. l(ist) a.k.a. I'm too lazy to open the file containing the source code

```
l(ist) [first [,last]]
    List source code for the current file. Without arguments, list 11 lines around the current line
    or continue the previous listing. With one argument, list 11 lines starting at that line.
    With two arguments, list the given range; if the second argument is less than the first, it is a count.
```

**NB**: The above description was generated by calling `help` on `list`. To get the same output, in the `pdb` REPL type `help l`.

Using `list`, we can examine the source code of the current file we are in. The arguments for `list` lets you specify a given range
of lines you wish to see which can be helpful if you are in some weird 3rd party package and you are trying to figure out why they
can't get string encoding working _true story_.

**NB**: In Python 3.2 and above, you can type `ll` (long list) which shows you source code for the current function or frame. I use
this all the time instead of `l` since it's much better knowing which function you are in than an arbitrary 11 lines around your
current position.

Let's try using `l` now. In your already open `pdb` prompt, type in `l` and look at the output:

```
(Pdb) l
  4     def main():
  5         print("Introduce los valores del dado")
  6         print("Es así de fácil")
  7         print("Qué es de tu vida")
  8         import pdb; pdb.set_trace()
  9  ->     GameRunner.run()
 10     
 11     
 12     if __name__ == "__main__":
 13         main()
[EOF]
```

If we want to see the whole file, we can call the list function with the range 1 to 13 like so:

```
(Pdb) l 1, 13
  1     from dicegame.runner import GameRunner
  2     
  3     
  4     def main():
  5         print("Introduce los valores del dado")
  6         print("Es así de fácil")
  7         print("Qué es de tu vida")
  8         import pdb; pdb.set_trace()
  9  ->     GameRunner.run()
 10     
 11     
 12     if __name__ == "__main__":
 13         main()
```

Unfortunately, we don't get that much information from this file alone but we do see that it is calling the `run()` method of the `GameRunner`
class. At this point, you might be thinking, "Awesome, I'll just set a `pdb` in the run method in the `dicegame/runner.py` file !" That will
work, but there's an even easier way using the `step` command we will discuss next.

### 2. `s(tep)` a.k.a let's see what this method does...

```
s(tep)
    Execute the current line, stop at the first possible occasion
    (either in a function that is called or in the current
    function).
```

Your current line of execution should still be on `:9` and you can tell the current line by looking at the `->` outputted by the `list` command.
Let's call the `step` command and see what happens.

```
(Pdb) s
--Call--
> /Users/Development/pdb-tutorial/dicegame/runner.py(22)run()
-> @classmethod
```

Nice! We're currently in the `runner.py` file on line 22 which we can tell from this line:
`> /Users/Development/pdb-tutorial/dicegame/runner.py(21)run()`.
The problem is, we don't have much context so run the `list` command to checkout the method.

```
(Pdb) l
 16             total = 0
 17             for die in self.dice:
 18                 total += 1
 19             return total
 20     
 21  ->     @classmethod
 22         def run(cls):
 23             # Probably counts wins or something.
 24             # Great variable name, 10/10.
 25             c = 0
 26             while True:
```

Awesome! Now we have some more context on the `run()` method but we are currently on `:22`. Let's step in one more time so that we enter the method itself and
then run the list command to see our current position.

```
(Pdb) s
> /Users/Development/pdb-tutorial/dicegame/runner.py(25)run()
-> c = 0
(Pdb) l
 20     
 21         @classmethod
 22         def run(cls):
 23             # Probably counts wins or something.
 24             # Great variable name, 10/10.
 25  ->         c = 0
 26             while True:
 27                 runner = cls()
 28     
 29                 print("Round {}\n".format(runner.round))
 30  
```

As we can see, we are on a terribly named `c` variable that will cause us a major issue if we try to call it (remember the comment from earlier regarding the
`c(ontinue)` command). We are just before the `while` loop so let's enter the loop and see what else we can uncover.

### 3. `n(ext)` a.k.a I hope this current line doesn't throw an exception

```
n(ext)
    Continue execution until the next line in the current function
    is reached or it returns.
```

From the current line, type the `n(ext)` command followed by `list` (notice a pattern) and let's observe what happens.

```
(Pdb) n
> /Users/Development/pdb-tutorial/dicegame/runner.py(27)run()
-> while True:
(Pdb) l
 21         @classmethod
 22         def run(cls):
 23             # Probably counts wins or something.
 24             # Great variable name, 10/10.
 25             c = 0
 26  ->         while True:
 27                 runner = cls()
 28     
 29                 print("Round {}\n".format(runner.round))
 30     
 31                 for die in runner.dice:
```

Now our current line on the `while True` statement! We can keep calling `next` indefinitely until the program throws an exception or terminates. Call `next` 3 more
times to get to the `for` loop and then follow up `next` with `list`.

```
(Pdb) n
> /Users/Development/pdb-tutorial/dicegame/runner.py(27)run()
-> runner = cls()
(Pdb) n
> /Users/Development/pdb-tutorial/dicegame/runner.py(29)run()
-> print("Round {}\n".format(runner.round))
(Pdb) n
Round 1

> /Users/Development/pdb-tutorial/dicegame/runner.py(31)run()
-> for die in runner.dice:
(Pdb) l
 26             while True:
 27                 runner = cls()
 28     
 29                 print("Round {}\n".format(runner.round))
 30     
 31  ->             for die in runner.dice:
 32                     print(die.show())
 33     
 34                 guess = input("Sigh. What is your guess?: ")
 35                 guess = int(guess)
```

At this current point, if you continue to type the `next` command you will then iterate through the `for` loop for the length of the `runner.dice`
attribute. We can take a look at the length of the `runner.dice` by calling the `len()` function around it in the `pdb` REPL which should return 5.

```
(Pdb) len(runner.dice)
5
```

Since the length is _only_ 5 items, we could iterate through the loop by calling `next` 5 times, but let's say there were 50 items to iterate over, or even 10,000!
A better option would be to set a break point and then `continue` to that break point instead.


### 4. `b(reak)` a.k.a I don't want to type `n` anymore

```
b(reak) [ ([filename:]lineno | function) [, condition] ]
    Without argument, list all breaks.

    With a line number argument, set a break at this line in the
    current file.  With a function name, set a break at the first
    executable line of that function.  If a second argument is
    present, it is a string specifying an expression which must
    evaluate to true before the breakpoint is honored.

    The line number may be prefixed with a filename and a colon,
    to specify a breakpoint in another file (probably one that
    hasn't been loaded yet).  The file is searched for on
    sys.path; the .py suffix may be omitted.
```

We're only going to pay attention to the first two paragraphs of `b(reak)`'s description in this tutorial. Like I mentioned in the previous section, we want
to set a break point past the `for` loop so we can continue to navigate through the `run()` method. Let's stop on `:34` since this has the input function
which will break and wait for a user input anyways. To do this, we can type `b 34` and then `continue` to the break point.

```
(Pdb) b 34
Breakpoint 1 at /Users/Development/pdb-tutorial/dicegame/runner.py(34)run()
(Pdb) c

[...] # prints some dice

> /Users/Development/pdb-tutorial/dicegame/runner.py(34)run()
-> guess = input("Sigh. What is your guess?: ")
```

We can also take a look at the break points that we have set by calling `break` without any arguments.

```
(Pdb) b
Num Type         Disp Enb   Where
1   breakpoint   keep yes   at /Users/Development/pdb-tutorial/dicegame/runner.py:34
    breakpoint already hit 1 time
```

To clear your break points, you can use the `cl(ear)` command followed by the breakpoint number which is found in the leftmost column of the above
output. Let's clear the breakpoint now by calling the `clear` command followed by 1.

**NB**: You can also clear all the breakpoints if you don't provide any arguments to the `clear` command.

```
(Pdb) cl 1
Deleted breakpoint 1 at /Users/Development/pdb-tutorial/dicegame/runner.py:34
```

From here we can call `next` and execute the `input()` function. Let's just type 10 for our guess and once we are back in the `pdb` REPL, call `list` so we can see
the next few lines.

```
(Pdb) n
Sigh. What is your guess?: 10
> /Users/Development/pdb-tutorial/dicegame/runner.py(35)run()
-> guess = int(guess)
(Pdb) l
 30     
 31                 for die in runner.dice:
 32                     print(die.show())
 33     
 34                 guess = input("Sigh. What is your guess?: ")
 35  ->             guess = int(guess)
 36     
 37                 if guess == runner.answer():
 38                     print("Congrats, you can add like a 5 year old...")
 39                     runner.wins += 1
 40                     c += 1


```

Remember that we are trying to find out why our guess wasn't correct on our first playthrough. It seemed like there was an error with the `guess == runner.answer`
equality condition. We should check to see what the `runner.answer()` method is doing in case there might be an error there. Call `next` and then let's call `step`
to _step_ into the `runner.answer()` method.

```
(Pdb) s
--Call--
> /Users/spiro/Development/mobify/engineering-meeting/pdb-tutorial/dicegame/runner.py(15)answer()
-> def answer(self):
(Pdb) l
 10         def reset(self):
 11             self.round = 1
 12             self.wins = 0
 13             self.loses = 0
 14     
 15  ->     def answer(self):
 16             total = 0
 17             for die in self.dice:
 18                 total += 1
 19             return total
 20  
```

I think I found the issue! On line 18, it doesn't look like the `total` variable is adding up the values of the dice like we want it to. Let's see if we can fix that by
checking whether a `die` has an attribute which would contain its value. To get to line 18, you can either set a break point or just call `next` until you
hit the first iteration. Once you're on `:18`, let's call the `dir()` function on the `die` instance and check what methods and attributes it has.

```
-> total += 1
(Pdb) dir(die)
['__class__', '__delattr__', [...], 'create_dice', 'roll', 'show', 'value']
```

There is a `value` attribute after all! Let's call that and see what returns (remember, this value will probably be different than mine). And just for fun,
let's make sure it is equal to the value that the die is showing by calling the `show()` method as well.

```
(Pdb) die.value
2
(Pdb) die.show()
'---------\n|*      |\n|       |\n|      *|\n---------'
```

**NB**: If you want the newline character `\n` to print as a newline, call `print()` with `die.show()` as its argument.

It looks like it works as expected and we're ready to fix the answer method. However, some of us may want to continue with the debugging process and catch all the
errors in one go. Unfortunately, we are once again stuck in this for loop. You might think to set a break point at `:19` and then call `continue` but there is actually
a better way in this case.

### 5. `r(eturn)` a.k.a. I want to get out of this function

```
r(eturn)
    Continue execution until the current function returns.
```

The `return` is a great _power user_ command that let's you examine the final outcome of a function. While you could set a breakpoint at the return call, the
`return` pdb command will help if there are multiple return statements in a single function since it only follows the path of execution for a single return. Let's
call the `return` command and get to the end of the function.

```
(Pdb) r
--Return--
> /Users/Development/pdb-tutorial/dicegame/runner.py(19)answer()->5
-> return total
(Pdb) l
 14     
 15         def answer(self):
 16             total = 0
 17             for die in self.dice:
 18                 total += 1
 19  ->         return total
 20     
 21         @classmethod
 22         def run(cls):
 23             # Probably counts wins or something.
 24             # Great variable name, 10/10.
(Pdb)
```

To check the value of the returned `total` variable, you can call `total` here or look at the final value in line below the `--Return--` output. Now, to return back
to the `run()` method, call the `next` command and you'll be back in your happy place.

At this point, you can exit the `pdb` debugger by calling `exit()` **OR** `CTRL+D` (same as the Python REPL). With these five commands, you should be able to figure
out a couple other bugs and then follow along with a bit more advanced `pdb` examples.


## Advanced `pdb` topics

Here are a couple advanced `pdb` commands that you can also use.

### The `!` (bang) command

```
!
  Execute the (one-line) statement in the context of the current stack frame.
```

The bang command (`!`) lets `pdb` know that the following statement will be a Python command and not a `pdb` command. Where this is helpful is in the `run()` method
with the `c` variable. Like I mentioned in the beginning of the tutorial, calling `c` in `pdb` will issue the `continue` command. Navigating in your `pdb` REPL, stop
at `:26` in the `runner.py` file and from that point you can prefix `c` with the `!` command and see what happens.

```
(Pdb) !c
0
```

We get the intended result, since `:25` assigned `c = 0`!


### `pdb` Post Mortem

```
pdb.post_mortem(traceback=None)
    Enter post-mortem debugging of the given traceback object. If no traceback is given, it uses the one of the exception that is currently being handled
    (an exception must be being handled if the default is to be used).

pdb.pm()
    Enter post-mortem debugging of the traceback found in sys.last_traceback.
```

While both methods may look the same, `post_mortem() and pm()` differ by the traceback they are given. I commonly use `post_mortem()` in the `except` block.
However, we will cover the `pm()` method since I find it to be a bit more powerful. Let's try and see how this works in practice.

Open up the python REPL by typing `python` in your shell in the root of this project. From there, let's import the `main` method from the `main` module and import `pdb`
as well. Play the game until the we get the exception after trying to type `Y` to continue the game.

```
>>> import pdb
>>> from main import main
>>> main()
[...]
Would you like to play again?[Y/n]: Y
Traceback (most recent call last):
  File "main.py", line 12, in <module>
    main()
  File "main.py", line 8, in main
    GameRunner.run()
  File "/Users/Development/pdb-tutorial/dicegame/runner.py", line 62, in run
    i_just_throw_an_exception()
  File "/Users/Development/pdb-tutorial/dicegame/utils.py", line 13, in i_just_throw_an_exception
    raise UnnecessaryError("You actually called this function...")
dicegame.utils.UnnecessaryError: You actually called this function...
```

Now, let's call the `pm()` method from the `pdb` module and see what happens.

```
>>> pdb.pm()
> /Users/Development/pdb-tutorial/dicegame/utils.py(13)i_just_throw_an_exception()
-> raise UnnecessaryError("You actually called this function...")
(Pdb)
```

Look at that! We recover from the point where the last exception was thrown and are placed in the `pdb` prompt. From here, we can examine the state the program was in
before it crashed which will help you in your investigation.

**NB**: You can also start the `main.py` script using `python -m pdb main.py` and `continue` until an exception is thrown. Python will automatically enter `post_mortem`
mode at the uncaught exception.


## The End

Congrats on making it to the end and thank you for following along in this tutorial! If you have any comments, critiques, or additional advanced examples, I'm open to pull requests.
