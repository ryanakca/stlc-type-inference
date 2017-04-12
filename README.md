### hacking

To run or hack on this, ensure that you have SML/NJ installed. Then, open an SML REPL:

    rlwrap sml

Here's an example session:

    Standard ML of New Jersey...
    - CM.make "src/stlc.cm";
    - Test.test ();

This should print the following:

    (lam nil) ==> (-> #1 unit)
    (lam (var 0)) ==> (-> #4 #4)
    (lam (lam (lam (var 2)))) ==> (-> #7 (-> #9 (-> #11 #7)))

There is no parser yet, but I can write one if we need it.


### editors?

I like to edit SML code using [Visual Studio
Code](http://code.visualstudio.com), because we have made a [very nice SML
mode](https://marketplace.visualstudio.com/items?itemName=freebroccolo.sml)
that has mostly correct highlighting (a herculean feat!) and integration with
SML/NJ for type error reporting. (This is what the `sml.json` file is for in
the root directory of this repository.)

### dataset

The file `dataset` contains 1,031,993 randomly generated term/type pairs. You
can obtain it by extracting `dataset.xz` using `xz -d dataset.xz`. The first
column is the term. The second column is its type. The two columns are separated
by a single space. Example data points include:

    (\0)(\(\\0)(0)) (1>(1>1))
    (\0)(\(\0)((\\*)(0))) (1>(1>1)
    * 1
    (\*)(\\\\\*) 1
    (\*)(\\\\3) 1

I tried to minimise syntax to simplify things for our neural network. The syntax
used for terms is

    `NIL`        = *
    `LAM t`      = \`t`
    `AP (t1,t2)` = (`t1`)(`t2`)

The syntax used for types is

    `UNIT`       = 1
    `AR (t1,t2)` = (`t1`>`t2`)
    `META v`     = 1

I replaced the type variables `META v` by `UNIT` to give us the simplest
possible types. Again, our initial goal is to get ML to handle the simplest type
inferences first, and then generalise.

The longest term is 308 characters long:

    (\(\\(\\*)((\*)(*)))((\(\(\(\\\\\\*)(\3))(\\4))((\(\\*)(\\\\\\\\\\\\10))((\\\\\\\\4)(\\\\\\\\*))))(\(\\\(\\\*)(*))((\\\\\\\\7)(\\\\\\\\4)))))((\(\\(\(\\\*)(\(\\1)(\\*)))(\(\\\\5)(\\\\1)))((\(\\\\\\\\\\\*)(\\*))((\\(\\\\\\\\\\*)(\\\\\\\\\\\\5))((\\(\\\\0)(\\\\\\\\\\\\*))(\(\*)(\\\\\\\\\\\\\\15))))))((\(\\\\0)((\\*)((\*)(\\\\\\\\3))))((\(\\*)((\\\\0)(\\\\\\\\\\\\\\7)))((\\1)((\\\\\\\\\\\\\3)(\\\\\\\\*))))))

The longest type is 198 characters long:

    (1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>(1>1)))))))))))))))))))))))))))))))))))))))))))))))))

One can straightforwardly embed these into R^n (say, R^400 and R^300,
respectively) by taking the vectors of ASCII character codes, and padding with
zeros as needed.

#### Generating the dataset

First compile the random lambda term generator using mlton. Modify the suffix
`x86-linux` to match the file generated by SML/NJ in the instructions below:

    $ make -C cm2mlb
    $ cd src
    $ sml @SMLload=../cm2mlb/cm2mlb.x86-linux rando.cm > rando.mlb
    $ mlton rando.mlb

Then to generate N random lambda terms, go

    $ ./rando N

For example,

     $ ./rando 3
     \(\\0)((\(\(\\2)(*))(\1))((\\\\\*)(\\\*))) (1>(1>1))
     \\(\(\3)(\\\*))(\\(\\\6)(\\\\\\\3)) (1>(1>1))
     \(\1)((\\\\2)(\(\\\\\6)((\\1)(\*)))) (1>1)

Because these won't necessarily be unique, you'll probably want to go

    $ ./rando N | sort -u

The code in `rando.sml` uses hardcoded probabilities to decide what kind of term
to generate. Edit these probabilities to change the complexity of the random
terms we generate.
