# JS.LA Presentation on PureScript
## SLIDE 1
What is _PURE_ functional coding:
This is a technical post series about pure functional programming. The intended audience is general programmers who are familiar with closures and some functional programming.

We're going to be seeing how pure functional programming differs from regular "functional programming", in a significant way.

We're going to be looking at a little language theory, type theory, and implementation and practice of pure functional programming.

We'll look at correctness, architecture, and performance of pure programs.

We're going to look at code samples in C#, JavaScript and SQL. We're going to look at Haskell code. We're going to look at assembly language.

In closing we'll conclude that typed purely functional programming matters, and in a different way to what is meant by "functional programming".

Hold onto yer butts.

In this post
Firstly, we'll start with a pinch of theory, but not enough to bore you. We'll then look at how functional programming differs from "pure" functional programming. I'll establish what we mean by "side-effects". Finally, I'll try to motivate using pure languages from a correctness perspective.

Types and Functions
A function is a relation between terms. Every input term has exactly one output term. That's as simple as it gets.


In type theory, it's described in notation like this:

f : A → B

Inputs to f are the terms in type A, and outputs from f are the terms in type B. The type might be Integer and the terms might be all integers .., -2, -1, 0, 1, 2, ...

The type might be Character and the terms might be Homer, Marge, SideShowBob, FrankGrimes, InanimateCarbonRod, etc.

This is the core of functional programming, and without type theory, it's hard to even talk formally about the meaning of functional programs.

Get it? I thought so. We'll come back to this later.

Functional programming isn't
Functional programming as used generally by us as an industry tends to mean using first-class functions, closures that capture their environment properly, immutable data structures, trying to write code that doesn't have side-effects, and things like that.

That's perfectly reasonable, but it's distinct from pure functional programming. It's called "pure functional", because it has only functions as defined in the previous section. But why aren't there proper functions in the popular meaning of functional programming?


It's something of a misnomer that many popular languages use this term function. If you read the language specification for Scheme, you may notice that it uses the term procedure. Some procedures may implement functions like cosine, but not all procedures are functions, in fact most aren't. Any procedure can do cheeky arbitrary effects. But for the rest of popular languages, it's a misnomer we accept.

Let's look into why this is problematic in the next few sections.

Side-effects, mutation, etc.
What actually are side-effects, really? What's purity? I'll establish what I am going to mean in this series of blog posts. There are plenty of other working definitions.

These are some things you can do in source code:

Mutate variables.
Make side-effects (writing to standard output, connecting to a socket, etc.)
Get the current time.
These are all things you cannot do in a pure function, because they're not explicit inputs into or explicit outputs of the function.

In C#, the expression DateTime.Now.TimeOfDay has type TimeSpan, it's not a function with inputs. If you put it in your program, you'll get the time since midnight when evaluating it. Why?

In contrast, these are side-effects that pure code can cause:

Allocate lots of memory.
Take lots of time.
Loop forever.
But none of these things can be observed during evaluation of a pure functional language. There isn't a meaningful pure function that returns the current time, or the current memory use, and there usually isn't a function that asks if the program is currently in an infinite loop.

So I am going to use a meaning of purity like this: an effect is something implicit that you can observe during evaluation of your program.

Which languages are purely functional
So, a pure functional language is simply one in which a function cannot observe things besides its inputs. Given that, we can split the current crop of Pacman-complete languages up into pure and impure ones:

Pure languages	Impure languages
	
Haskell (we'll use Haskell in this series), PureScript and Idris are purely functional in this sense. You can't change variables. You can't observe time. You can't observe anything not constructed inside Haskell's language. Side effects like the machine getting hotter, swap space being used, etc. are not in Haskell's ability to care about. (But we'll explore how you can straight-forwardly interact with the real world to get this type of information safely in spite of this fact.)

We'll look at the theoretical and practical benefits of programming in a language that only has functions during this series.

Reasoning and function composition
In a pure language, every expression is pure. That means you can move things around without worrying about implicit dependencies. Function composition is a basic property that lets you take two functions and make a third out of them.

Calling back to the theory section above, you can readily understand what function composition is by its types. Lets say we have two functions, f : X → Y and g : Y → Z:


The functions f : X → Y and g : Y → Z can be composed, written f ∘ g, to yield a function which maps x in X to g(f(x)) in Z:


In JavaScript, f ∘ g is:

function(x){ return f(g(x)); }
In Haskell, f ∘ g is:

\x -> f (g x)
There's also an operator that provides this out of the box:

f . g
You can compare this notion of composition with shell scripting pipes:

sed 's/../../' | grep -v ^[0-9]+ | uniq | sort | ...
Each command takes an input and produces an output, and you compose them together with |.

We'll use this concept to study equational reasoning next.

A really simple example
Let's look at a really simple example of composition and reasoning about it in the following JavaScript code and its equivalent in Haskell:

JavaScript	Haskell
> [1, 4, 9].map(Math.sqrt)
[1, 2, 3]
> [1, 4, 9].map(Math.sqrt).map(function(x){ return x + 1 });
[2, 3, 4]
> map sqrt [1,4,9]
[1.0,2.0,3.0]
> map (\x -> x + 1) (map sqrt [1,4,9])
6.0
There's a pattern forming here. It looks like this:

xs.map(first).map(second)
map second (map first xs)
The f and g variables represent any function you might use in their place.

Let's do equational reasoning
We know that map id ≣ id, that is, applying the identity function across a list yields the original list's value. What does that tell us? Mapping preserves the structure of a list. In JavaScript, this implies that, given id

var id = function(x){ return x; }
Then xs.map(id) ≣ xs.

Map doesn't change the length or shape of the data structure.

By extension and careful reasoning, we can further observe that:

map second . map first   ≣ map (second . first)
In JavaScript, that's

xs.map(first).map(second) ≣ xs.map(function(x){ return second(first(x)); })
For example, applying a function that adds 1 across a list, and then applying a function that takes the square root, is equivalent to applying a function across a list that does both things in one, i.e. adding one and taking the square root.

So you can refactor your code into the latter!

Actually, whether the refactor was a good idea or a bad idea depends on the code in question. It might perform better, because you put the first and the second function in one loop, instead of two separate loops (and two separate lists). On the other hand, the original code is probably easier to read. "Map this, map that ..."

You want the freedom to choose between the two, and to make transformations like this freely you need to be able to reason about your code.

But it doesn't work in JavaScript
Turning back to the JavaScript code, is this transformation really valid? As an exercise, construct in your head a definition of first and second which would violate this assumption.

xs.map(first).map(second)
→
x.map(function(x){ return second(first(x)); })
The answer is: no. Because JavaScript's functions are not mathematical functions, you can do anything in them. Imagine your colleague writes something like this:

var state = 0;
function first(i){ return state += i; }
function second(i){ return i `mod` state; }
You come along and refactor x.map(first).map(second), only to find the following results:

> var state = 0;
  function first(i){ return state += i; }
  function second(i){ return i % state; }
  [1,2,3,4].map(first).map(second)
[1, 3, 6, 0]
> var state = 0;
  function first(i){ return state += i; }
  function second(i){ return i % state; }
  [1,2,3,4].map(function(x){ return second(first(x)); })
[0, 0, 0, 0]
Looking at a really simple example, we see that basic equational reasoning, a fundamental "functional" idea, is not valid in JavaScript. Even something simple like this!

So we return back to the original section "Functional programming isn't", and why the fact that some procedures are functions doesn't get you the same reliable reasoning as when you can only define functions.

A benefit like being able to transform and reason about your code translates to practical pay-off because most people are changing their code every day, and trying to understand what their colleagues have written. This example is both something you might do in a real codebase and a super reduced down version of what horrors can happen in the large.

Summary
In this post I've laid down some terms and setup for follow up posts.

We've looked at what purity is and what it isn't.
We've looked at functions and function composition.
We've looked at how reasoning is easier if you only have functions. We'll circle back to this in coming posts when we get to more detailed reasoning and optimizations.
In the next post, we'll explore:

SQL as a pure language, which is a familiar language to almost everybody.
How pure languages deal with the real-world and doing side-effects, which are obviously practical things to do.
Evaluation vs execution, the generalization of pure vs impure languages.
After that, we'll explore the performance and declarative-code-writing benefits of pure functional languages in particular detail, looking at generated code, assembly, performance, etc.

In the last post, we covered the following:

What purity is and what it isn't.
We looked at functions and function composition.
We've looked at how reasoning is easier if you only have functions.
In this post, we'll explore:

SQL as a pure language, which is a familiar language to almost everybody.
How pure languages deal with the real-world and doing side-effects, which are obviously practical things to do.
Evaluation vs execution, the generalization of pure vs impure languages.
In the next post, we'll explore the performance and declarative-code-writing benefits of pure functional languages in particular detail, looking at generated code, assembly, performance, etc.

In a familiar setting: SQL
In the last post, we established that pure languages (like Haskell or PureScript) are unable to make observations about the real world directly.

So, then, the question is: How do pure languages interact with the real world? Let's look at SQL first, because everyone knows SQL, and then use that as a bridge towards how it's done in Haskell and other general purpose pure languages.

Imagine a subset of SQL which is pure. It's not a big leap: SQL is mostly pure anyway. I don't like it when people write an article and ask me to imagine a syntax that's in their heads, so here is trivial a PEG grammar, which you can try online here.

Select = 'SELECT ' Fields ' FROM ' Names Where?
Names = Name (', ' Name)*
Fields = Exp (', ' Exp)*
Where = ' WHERE ' Exp
Exp = Prefix? (Funcall / Condition / Name) (Connective Exp)?
Funcall = Name '(' Exp ')'
Condition = (Funcall / Name) Op Exp
Prefix = 'NOT '
Connective = ' AND ' / ' OR '
Op = ' = ' / ' <> ' / ' < ' / ' > ' / ' * '
Name = [a-zA-Z0-9]+
The following is an example of the above grammar.

SELECT sin(foo), bar * 3 FROM table1, table2 WHERE foo > 1 AND NOT bar = floor(foo)
This language is pure. The result of evaluating one of these expressions is the same every time. You can swap the order of commutative expressions like floor(foo) = bar and it doesn't change the meaning of the program. floor(x) * sin(y) = sin(y) * floor(x).

You get a description of the work you want to be done, and then the database engine (PostgreSQL, MySQL, SQL Server, etc.), usually creating an optimized query plan based on what it knows about your query and the database schema and contents, executes the query on the database, returning a collection of results.

You didn't write instructions about how to get the database, where to look on the database, how to map indices across tables and whether to do an index scan or a sequence scan, allocating a buffer, etc.

And yet, above, we clearly are giving the SQL engine a little program (not a turing complete one), that can compute conditionals and even functions (floor and sin) that we specified freely.

Because our little program is pure, SQL is capable of basically rewriting it completely, reducing it into a more normalized form without redundancy, and executing something far more efficient.

Look at what PostgreSQL is able to do with your queries. Here I do a query which works on an indexed field on a table with 37,626,086 rows. PostgreSQL knows both the query and my data, and plans accordingly, able to estimate the cost of how much work we'll have to do.

ircbrowse=> explain select * from event where id > 123 and id < 200*2;
                                    QUERY PLAN
----------------------------------------------------------------------------------
 Index Scan using event_unique_id on event  (cost=0.56..857.20 rows=309 width=87)
   Index Cond: ((id > 123) AND (id < 400))
(The 200*2 was evaluated statically.)

Simply based on a static analysis of the SQL program.

Evaluation vs execution
The SQL example is a specific case of a more general way of separating two concerns: evaluation and execution (or interpretation). On the one side you have your declarative language that you can evaluate, pretty much in terms of substition steps (and that's how you do your reasoning, in terms of the denotational semantics), and on the other you have a separate program which interprets that language in "the real world" (associated with operational semantics).

I'm sure any programmer reading this easily understands the informal denotational semantics of the SQL mini language above, and would feel comfortable implementing one or more ways of executing it.

In the same way, Haskell and other pure languages, construct declarative descriptions of work which a separate program can execute.

Evaluation
What's evaluation, as differentiated from execution? To do your language's regular evaluation steps, as you might do on paper. If you've read your Structured and Interpretation of Computer Programs by MIT, you'll know this as the substitution model. Here's an example.

If you want to count the length of a linked list, you might do:

length [1, 2, 3]
And length is implemented like this:

length [] = 0
length (x:xs) = 1 + length xs
So let's evaluate the expression, step-by-step:

length [1, 2, 3]

1 + length [2, 3]

1 + (1 + length [3])

1 + (1 + (1 + length []))

1 + (1 + (1 + 0))

1 + (1 + 1)

1 + 2

3
Even if you don't know Haskell, I think the simple substitution steps to evaluate the expression are straight-forward and make sense. That's evaluation.

A Terminal mini-language
Let's look at a data type that models a list of terminal actions to perform. I realise that if you don't know Haskell or ML, the below is mostly nonsense. I'll also bring in JavaScript as a "lingua franca", so that the idea translates nicely.

(If it looks concise in Haskell and verbose in JavaScript, it's because JavaScript was designed for imperative, OO-style programming, and Haskell was designed for pure functional programming.)

data Terminal a
  = Print String (Terminal a)
  | GetLine (String -> Terminal a)
  | Return a
function Print(string, next) {
  this.line = string;
  this.next = next;
}
function GetLine(cont) {
  this.f = cont;
}
function Return(a) {
  this.a = a;
}
It can describe printing to stdout, getting a line from stdin, or returning a result. An example might be:

Print
  "Please enter your name: "
  (GetLine
     (\name ->
        Print
           ("Hello, " ++ name ++ "!")
           (Return ())))
new Print(
  "Please enter your name: ",
  new GetLine(function(name){
    return new Print("Hello, " + name + "!",
                     new Return(null));}))
Think of this like an abstract syntax tree. It's the description of what to do. When there is a function in the AST, that's a callback. The callback is just another pure function that returns a new action to be interpreted.

The expressions above can be evaluated, but they perform no side-effects. It's a piece of purely functional code which generates a data structure. Our substitution model above still works.

Hammering the point home, consider a greeting action like this:

greeting :: String -> Terminal ()
greeting msg =
  Print
    msg
    (GetLine
       (\name ->
          if name == "Chris"
            then Print "That's my name too." (Return ())
            else Print "Hello, stranger!" (Return ())))
The greeting function itself is pure. It takes a message string, and constructs a Print using that. That's done by evaluation. Also, the sub-expression (\name -> ...) is another pure function that takes a name and conditionally produces one action or another.

Now, onto execution.

Execution
We can write a number of ways to interpret this AST. Here is one pure implementation. It takes as input (1) some action list to run and (2) lines of input, then it returns a value and some output lines:

data Result a = Success a | Failure String deriving Show
interpretPure :: Terminal a -> [String] -> Result (a,[String])
interpretPure action stdin =
  case action of
    Return a -> Success (a, [])
    Print line next ->
      case interpretPure next stdin of
        Success (a, stdout) -> Success (a, line : stdout)
        Failure e -> Failure e
    GetLine f ->
      case stdin of
        [] -> Failure "stdin closed"
        (line:stdin') -> interpretPure (f line) stdin'
In diagram form:


Note the part,

    GetLine f ->
      case stdin of
        [] -> Failure "stdin closed"
        (line:stdin') -> interpretPure (f line) stdin'
-------------------------------------- ^
Above is where we call the pure function from GetLine to produce another action for us to interpret.

In JavaScript:

function Success(x){ this.success = x; }
function Failure(e){ this.error = e; }
function interpretPure(action, stdin) {
  if (action instanceof Return) {
    return new Success({ result: action.a, stdout: [] });
  } else if (action instanceof Print) {
    var result = interpretPure(action.next, stdin);
    if (result instanceof Success) {
      return new Success({
        result: result.success.result,
        stdout: [action.line].concat(result.success.stdout)
      });
    } else return result;
  } else if (action instanceof GetLine) {
    if (stdin == "") {
      return new Failure("stdin closed");
    } else {
      var line = stdin[0];
      var stdin_ = stdin.slice(1);
      return interpretPure(action.f(line), stdin_);
    }
  }
}
Which we can run like this:

> interpretPure demo ["Dave"]
Success ((),["Please enter your name: ","Hello, Dave!"])
> interpretPure demo []
Failure "stdin closed"
> JSON.stringify(interpretPure(demo, ["Dave"]));
"{"success":{"result":null,"stdout":["Please enter your name: ","Hello, Dave!"]}}"
> JSON.stringify(interpretPure(demo, []));
"{"error":"stdin closed"}"
If we liked, we could interpret this as real I/O that talks to the world directly.

In Haskell, we translate it to a different type called IO. In JavaScript, let's do a good olde-fashioned 90's-style alert/prompt user interaction.

interpretIO :: Terminal a -> IO a
interpretIO terminal =
  case terminal of
    Return a -> return a
    Print str next -> do
      putStrLn str
      interpretIO next
    GetLine f -> do
      line <- getLine
      interpretIO (f line)
function interpretIO(action) {
  if (action instanceof Return) {
    return action.a;
  } else if (action instanceof Print) {
    alert(action.line);
    interpretIO(action.next);
  } else if (action instanceof GetLine) {
    var line = prompt();
    interpretIO(action.f(line));
  }
}
In this case, we're just translating from one model to another. The IO a type is interpreted by the Haskell runtime either in a REPL prompt, or in a compiled program. JavaScript is capable of executing things directly during evaluation, so we were able to implement something similar to what Haskell's runtime does for IO.

In SICP, they talk about programs as being spells, and the interpreter being like a spirit that does your bidding. Here is a formal description:


(In practice, Haskell's IO type is not a closed set of operations, but an open set that can be extended with e.g. bindings to C libraries and such, and it isn't interpreted at runtime but rather compiled down to bytecode or machine code. The strict evaluation/execution separation remains in place, however.)

Summary
We've covered an informal comparison between "evaluating" and "executing" code, which is a distinction often discussed in the functional programming community.

We've explored making a declarative data type that represents terminal interaction, and implemented two different ways of interpreting it: as a pure function, or executing it directly. There may be other interpretations, such as talking to a server. We could write an interpreter which logs every action performed. We could also write a test suite which feeds randomized or specific inputs and demands consistent outputs.

We looked at how SQL is such an example of an evaluation (1 + 1 is evaluated before executing the query) vs execution separation. In SELECT customer.purchases + 1 FROM customer, the customer.purchases + 1 expression is evaluated by the interpreter for each row in the table.

In the next post, we'll explore:

Performance of pure code.
Optimization of pure code, in particular: Inlining, deforestation, common sub-expression elimination, fusion, rewrite rules, etc.
And how these optimizations allow for low- or zero-cost abstractions.
We may further explore different types of interpreters (software transactional memory, local mutation, parsers, etc).

Slide 2:
History of Haskell
