---
navhome: /docs/
next: false
sort: 13
title: Network messages
---

Network messages
================

Now we've learned enough Hoon to do more interesting things. Let's get
our planets to talk to each other.

All we've written up until now are just shell commands that produce a value and
then disappear. To listen for and receive messages from other planets, we'll
need an app. Let's look at a very simple one, `echo.hoon`:

    ::  Accepts any noun from dojo and prints it out
    ::
    ::::  /===/app/echo/hoon
      ::
    !:                          :: enables debug printfs
    |%                          :: forms a core with subject as payload
    ++  move  {bone card}       :: defines echo app's arvo move mold
    ++  card  $%  $~            :: defines card (data) mold as null tagged union 
              ==                :: end of mold definition; see $%
    --                          :: end of card arm definition
    ::                          ::
    |_  {bow/bowl $~}           :: forms a core with bowl (app state) as sample
    ++  poke-noun               :: begin definition of poke-noun arm
      |=  non/*                 :: forms a gate which accepts any noun
      ^-  {(list move) _+>.$}   :: casts product to mold of moves and context
      ~&  echo+noun+non         :: debug printf, i.e. '[%echo %noun *]'
      [~ +>.$]                  :: produce a cell of empty list and our state
    --                          :: end of poke-noun arm definition

This is a very simple app that does only one thing. If you poke it with
a value, it prints that value out. To try this out, you have to start
the app, then you can poke it from the command line with the following
commands:

    ~fintud-macrep:dojo> |start %examples %echo
    >=
    ~fintud-macrep:dojo> :echo 5
    [%echo %noun 5]
    >=
    ~fintud-macrep:dojo> :echo [1 2]
    [%echo %noun [1 2]]
    >=

> There is currently a bug where the `[%echo %noun *]` lines are printed *above*
> the line you entered, so your output may not look exactly like this. `>=` 
> means that a command was successfully received and executed.

Most of the app code should be simple enough to guess its function. The
important part of this code is the definition of `++poke-noun`.

Once an app starts, it's always running in the background and you interact with
it by sending it messages. The most straightforward way to do that is to poke it
from the command line, which we we did with `:echo 5`
(`:[app-name] [argument(s)]`).

In this case, `++poke-noun` takes an argument (*sample*) `arg` and 
prints it to dojo with `~&` ([sigpam](../../hoon/rune/sig/pam/)). 
This is an unusual rune that formally "does nothing", but the interpreter 
detects it and printfs the first child, before executing the second as if the
first didn't exist. This is a slightly hacky way of printing to the console, 
but we'll get to the correct way later on.

But what does `++poke-noun` produce? Recall that `^-` casts to a type. In this
case, it's declaring that the end result (*product*) of the function 
(`++poke-noun`'s *gate*) will be of type `{(list) _+>.$}`. But what does this 
mean?

The phrase to remember is "a list of moves and our state". Urbit is a message
passing system, so whenever we want to do something that interacts with the rest
of the system, we send a message. Thus, a move is Arvo's equivalent of a
syscall. The first thing that `++poke-noun` produces is a list of messages,
called "moves". In this case, we don't actually want the system to do anything,
so we produce the empty list, `~` (in the `[~ +>.$]` line).

The second thing `++poke-noun` produces is our state. `+>.$` refers to a
particular address in our subject where our formal app state is stored.
It'll become clear why this is later on, but for now pretend that `+>.$`
is a magic invocation that means "app state".

Let's look at another example (edit your code in
`/app/examples/echo/hoon` to reflect the code below). Say we want to
only accept a number, and then print out the square of that number.

    ::  Accepts an atom from the dojo and squares it.
    ::
    ::::  /===/app/square/hoon
      ::
    !:                          
    |%                                                  ::>  no moves in :square
    ++  move  {bone card}                               ::>  no cards in :square
    ++  card  $%  $~            
              ==                
    --                          
    ::                                                  ::<  stateless
    |_  {bow/bowl $~}           
    ++  poke-atom
      |=  tom/@                 
      ^-  {(list move) _+>.$}   
      ~&  square+(mul tom tom)
      [~ +>.$]                  
    --                          

A few things have changed. Firstly, we no longer accept arbitrary nouns because
we can only square atoms (integers, in this case an unsigned one). Thus, our
argument is now `tom/@`. Secondly, it's `++poke-atom` rather than `++poke-noun`.

Intro to marks
==============

Are there other `++poke`s? Yes. In fact, `noun` and `atom` are just two
of arbitrarily many "marks". A mark is fundamentally a type definition,
but accessible at the Arvo level. Each mark is defined in the `/mar`
directory. Some marks have conversion routines to other marks, and some
have diff, patch, and merge algorithms. None of these are required for a
mark to exist, however.

`noun` and `atom` are two predefined marks. In your `/mar` directory,
there are already many more, and you may add more at will. The type
associated with `noun` is `*`, and the type associated with `atom` is
`@`.

When we poke an app from anywhere, we do so with a mark that searches
for the corresponding (`++poke-[mark]`) arm. Data constructed on the
command line is by default marked with `noun`. In this case, the app is
expecting an atom, so we have to explicitly mark the data with `atom`
using `&[mark]`. Try the following commands:

    ~fintud-macrep:dojo> |start %square
    >=
    ~fintud-macrep:dojo> :square 6
    gall: %square: no poke arm for noun
    ~fintud-macrep:dojo> :square &atom 6
    [%square 36]
    >=

> Recall the bug where `%square` may get printed above the input line.

Marks are powerful, and they're the backbone of Urbit's data pipeline,
so we'll be getting quite used to them.

**Exercise**:

-   Write an app that computes fizzbuzz on its input (as in the previous
    section).

Sending a message to another urbit
==================================

Let's write our first network message! Here's `examples/app/pong.hoon`:

    ::  Allows one urbit to send the string 'Pong' to 
    ::  another urbit.
    ::
    ::::  /===/app/pong/hoon
      ::
    !:
    |%
    ++  move  {bone card}
    ++  card  $%  {$poke wire dock poke-contents}
              ==
    ++  poke-contents  $%  {$atom @}
                       ==
    --
    |_  {bow/bowl $~}                                       ::<  stateless
    ::
    ++  poke-urbit
      |=  to/ship
      ^-  {(list move) _+>.$}
      ~&  pong+'Outgoing pong!'
      :_  +>.$
      ~[[ost.bow %poke /sending [to dap.bow] %atom 'Pong']]
    ::
    ++  poke-atom
      |=  tom/@
      ^-  {(list move) _+>.$}
      ~&  pong+'Incoming pong!'
      ~&  pong+received+`@t`tom
      [~ +>.$]
    ::
    ++  coup  |=(* [~ +>.$])
    ::
    --
 
Run it with these commands:

    ~fintud-macrep:dojo> |start %pong
    >=
    ~fintud-macrep:dojo> :pong &urbit ~sampel-sipnym
    >=

Replace `~sampel-sipnym` with another urbit. The easiest thing to do is to start
a comet, a free and disposable Urbit identity. If you don't know how to start a
comet, see [the user setup section](/docs/using/setup/). Don't forget to start
the `%pong` app on that urbit, too. You should see, on the foreign
urbit, this output:

    [%pong 'Incoming pong!']
    [%pong %received 'Pong']

Most of the code should be straightforward. In `++poke-atom`, the only new thing
is the expression `` `@t`tom ``, which is *casting* the argument `tom` to type 
`@t`. As we already know, `@t` is the type of "cord" (text string). In Hoon, 
another way to perform a kind of casting is to use a type as a function, like 
this: `(@t tom)`. When types are called like this, they are called "clams", 
and they serve as validator functions-- that is, functions whose domain is all 
nouns, and range is the given type (in this case, `@t`). Clams will "bunt" the 
default value of the mold whereas casts (`@t`tom) may `nest-fail` if the 
argument passed to it does not fit. An example of such a failure case is 
passing a cell to an atom: `` `@`[1 2] ``. Using a clam here instead 
(`(@ [1 2])`) will bunt the default value of `@`, which is `0`. Try it in dojo!

The more interesting part is in `++poke-urbit`. The `urbit` mark is an urbit
identity, and the Hoon type associated with it is `ship` or `@p` (the "p" 
stands for "phonetic base").

Recall that in a `++poke` arm we produce "a list of moves and our state". Until
now, we've left the list of moves empty, since we haven't wanted to tell Arvo to
do anything in particular. Now we want to send a message to another urbit. Thus,
we produce a list with one element:

    ~[[ost.bow %poke /sending [to dap.bow] %atom 'Pong]]

### Moves

The general form of a move is `[bone term wire *]`.

Or, in pseudo-code:

`["cause" sys-call/action tack-new-layer-on-cause action-specific information]`

Let's walk through each of these elements step by step.

#### Bones ("cause")

If you look up `++bone` in `hoon.hoon`, you'll see that it's a number (`@ud`),
and that it's an opaque reference to a duct. `++duct` in `hoon.hoon` is a list 
of `wire`s, where `++wire` is an alias for `++path`. `++path` is a list of
`++knot`s, which are ASCII text. Thus, a duct is a list of paths, and a bone is
an opaque reference to it (in the same way that a Unix file descriptor is an
opaque reference to a file structure). Thus, to truly understand bones, we must
understand ducts.

A duct is a stack of causes, again, represented as paths, which are called wires.
At the bottom of every duct is a unix event, such as a keystroke, network
packet, file change, or timer event. When Arvo is given this event, it routes
the event to appropriate kernel module for handling.

Sometimes, the module can immediately handle the event and produce any necessary
results. For example, when we poked the `++poke-atom` arm above, we poked %gall,
our application server, which was able to respond to our poke directly. When the
module cannot service the request itself, it sends instructions to another
kernel module or application (through the %gall module) to do a specified
action, and produces the result from that.

Furthermore, when one module sends a message to another kernel module or
application, it also sends along the duct it was given with its new wire tacked
onto the top. Now the duct has two entries, with the unix event on the bottom
and the kernel module that handled it on top. This process can continue
indefinitely, pushing more and more wires onto the top of the duct. When an 
entity finally produces a result, a wire is popped off the duct, and the result 
is passed all the way back down, repeating the process of wire popping 
sequentially until the bottom of the duct is reached.

In effect, a duct is an Arvo-level call stack. It's worth noting that while in
traditional call stacks a function call happens synchronously and returns
exactly once. In Arvo, multiple moves can be sent at once, are evaluated
asynchronously, and each one may be responded to zero or more times.

The point to take home is that whatever caused `++poke-urbit` to be called is
also the root cause for the network message we're trying to send. Thus, we say
to send the network message along the given bone `ost`.

##### Wire (path)

Of course, we have to push a new wire onto our duct before passing it along (or
responding to it directly) anywhere. This wire can have any data we want in it,
but we don't need anything specific here, so we just use the wire `/sending`
(`/elem1/elem2/elemN` is one syntax used to create `++path`s and `++wire`s of N
elements). If we were expecting a response (which we're not), it would come back
along the `/sending` wire, meaning that the path `/sending` will be passed back
to the response handler as an argument. Although it's not required, it's
generally a good idea to make the wire human-readable for bug-handling purposes.

##### Term (sys-call)

Each move also has a `term`, composed of lowercase ASCII and/or `-`. This `term`
has the sign `@tas`. In this case, our `term` is `%poke`, which is the name of
the particular kind of move we're sending. You can always use `%poke` to message
an app. Other common names include `%warp`, to read from the filesystem;
`%wait`, to set a timer; and `%them`, to send an http request.

The move ends with `*` (that is, any noun) since each type of move takes
different data. In our case, a `%poke` move takes a target (urbit and app) and
marked data, then pokes the arm of the corresponding mark on that app on that
urbit with that data. `[to-urbit-address %pong]` is the target urbit and app,
`%atom` is the `mark`, and`'Pong'` is the data.

When Arvo receives a `%poke` move, it calls the appropriate `++poke`.
The same mechanism is used for sending messages between apps on the same
urbit as for sending messages between apps on different urbits.

> We said earlier that we're not expecting a response. This is not
> entirely true: the `++coup` is called when we receive acknowledgment
> that the `++poke` was called. We don't do anything with this
> information right now, but we could.

**Exercises**:

-   Extend either of the apps in the first two exercises to accept input
    over the network in the same way as `pong`.

-   Modify `pong.hoon` to print out a message when it receives
    acknowledgement.

-   Write two apps, `even` and `odd`. When you pass an atom to `even`,
    check whether it's even. If so, divide it by two and recurse;
    otherwise, poke `odd` with it. When `odd` receives an atom, check
    whether it's equal to one. If so, terminate, printing "%success".
    Otherwise, check whether it's odd. If so, multiply it by three, add
    one, and recurse; otherwise, poke `even` with it. When either app
    receives a number, print it out along with the name of the app. In
    the end, you should be able to watch Collatz's conjecture play out
    between the two apps. Sample output:

```
~fintud-macrep:dojo> :even &atom 18
[%even 18]
[%odd 9]
[%even 28]
[%even 14]
[%odd 7]
[%even 22]
[%odd 11]
[%even 34]
[%odd 17]
[%even 52]
[%even 26]
[%odd 13]
[%even 40]
[%even 20]
[%even 10]
[%odd 5]
[%even 16]
[%even 8]
[%even 4]
[%even 2]
%success
```

-   Put `even` and `odd` on two separate urbits and pass the messages
    over the network. Post a link to a working solution in :talk to
    receive a cookie.
