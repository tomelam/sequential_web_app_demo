# Sequential Web Application Demo

This project demonstrates a web application programmed sequentially.
This means its structure mirrors its execution. In this README, we give
some background on how web applications are typically programmed,
then compare the structures of a traditional web application and
a Node.js-style event-driven web application. Then we point the reader
to a working example of an application written for a continuation-based
web server and important ideas about the usefulness of such applications.
Finally we present a continuation-based web application that runs entirely
in the web browser (a single-page web application).


## How Asynchronous Events Are Handled in Web Applications

Any program, simple or complex, that has I/O (user input or a disk read or
write operation or transfer of data to or from another computer) must have a
way of synchronizing with the completion of that I/O. For desktop applications,
the operating system provides system calls, a scheduler, and library functions
that allow the programmer to create a structure for his program that mirrors
the flow of the program's execution. For desktop applications the waiting
for completion of I/O can usually be neatly hidden within some function
like read(), write(), getchar(), etc., but in most web applications written
up to 2019, this mirroring is not possible because of the stateless nature of
the web. Even most web applications that are written to run entirely in the
web browser (a single-page web application) cannot mirror program flow
because of the event-driven nature of JavaScript in the web browser.

In 2019, web servers can be classified as event-driven (like Node.js) and
non-event-driven (the traditional web server). An event-driven web server
can react to many asynchronous events in real time, without holding up
the main event loop.

Web applications written for an event-driven web server and single-page
web applications must be structured using callbacks, deferreds, promises,
or some form of continuation-passing style. Thus, their struction cannot
mirror their flow. However, a web application written using *true*
continuations *can* be structured to mirror its flow.


## Traditional Web Applications vs. Continuation-Based Web Applications

Very often, web applications interact with the user by building request
pages that pass information from web page to web page in cookies or
hidden form fields, something like this:

```
    (define (sum query)
      (build-request-page "First number:" "/one" ""))
     
    (define (one query)
      (build-request-page "Second number:"
			  "/two"
			  (cdr (assq 'number query))))
     
    (define (two query)
      (let ([n (string->number (cdr (assq 'hidden query)))]
	    [m (string->number (cdr (assq 'number query)))])
	`(html (body "The sum is " ,(number->string (+ m n))))))
     
    (hash-set! dispatch-table "sum" sum)
    (hash-set! dispatch-table "one" one)
    (hash-set! dispatch-table "two" two)
```

This typical programming style of a web application is more
complicated and unwieldy than just writing the same application
straightforwardly like this:

```
    (define (sum2 query)
      (define m (get-number "First number:"))
      (define n (get-number "Second number:"))
      `(html (body "The sum is " ,(number->string (+ m n)))))
```

This latter version of the same web application was written using Racket's
server-side web continuations. See the section "11 Continuations" of
"More: Systems Programming with Racket" (https://docs.racket-lang.org/more/).


## Server-Side Web Continuations

A continuation-based web server and web application that run in Racket
Scheme are described at https://docs.racket-lang.org/more/ . (Racket is
downloadable from https://download.racket-lang.org/ .) The finished
Scheme code is at https://docs.racket-lang.org/more/step9.txt . It can
be loaded and run in Racket something like this (where 8080 is the port
number the server responds to):

```
    $ racket 
    Welcome to Racket v7.5.
    > (enter! "step9.txt")
    "step9.txt"> (serve 8080)
    #<procedure:...webcon/step9.txt:17:2>
    "step9.txt"> 
```

After starting the program, you can use the web application locally
by typing "http://localhost:8080/sum2" into your web browser's address
bar. The program gets one number, jumps to a new web page (let's call it
the second page)  where it asks for a second number, then jumps to a
new web page (let's call it the third page) where it sums the two
numbers. It stores continuations so it remembers the first number in
case you press your browser's back button once or twice or retype the
URL of the third or second page.

Section 5.2 of Christian Queinnec's paper 'Inverting back the inversion
of control or, Continuations versus page-centric programming' describes
a very similar web application but does not detail its implementation.


## Client-Side Web Continuations

Server-side web continuations are interesting because of the simplification
they provide to web applications. However, the memory that they consume
can easily be a problem. Since each user of a website comes with his own
web browser, it would be convenient to put the continuations in the browser.
This way, a web application with 10,000 or 500,000 users is not burdened
by a heavy price of memory for continuations.
This Git project demonstrates how a way to create a web application using
client-side continuations. It includes a submodule,
https://github.com/tomelam/jsScheme, which provides the necessary code,
and a test program, ```sequential.txt```.  Here are instructions
to run the application:

1. Clone the Git repository https://github.com/tomelam/sequential_web_app_demo .

2. The Git submodule contains a file ```scheme.html```. Open it in a
web browser. You will see jsScheme's input textarea and its Log
textarea. To the right of the input textarea are buttons ```eval```,
```clear input```, etc.

3. Press the 'library' button. The library will be copied into the input
textarea.

4. Press the 'eval' button. The library will be added to jsScheme's
work environment and lots of stuff will be printed in the Log.

5. Clear the input and log.

6. The cloned repository contains a file ```sequential.txt```. Copy it
into the input textarea.

7. Press the 'eval' button. Lots of stuff will be printed in the Log.

8. Clear the input and log.

9. Eval (reset (test)). In the Log the input should be 
echoed and ```=> #lambda``` should be printed.

10. Click on the TABLE element with ID 'symbols' (below the
```work environment``` and ```core environment``` links).

11. Output from on-click-handler should return a value (probably a simple
list) containing a symbol indicating which handler was triggered and
any data from the JavaScript handler.

12. Click on the TABLE element with ID 'symbols'. Confirm that no code is
executed.
