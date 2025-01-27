This document describes the detailed steps required to assess the artifact associated with our paper submitted to TACAS'22: Kmclib: Automated Inference and Verification of Session Types from OCaml Programs. 

The artifact comprises of the *kmclib* library -- a library for safe communication programming in OCaml. The *kmclib* library guarantees that threads in well-typed
programs will not get stuck since communication errors are detected statically, i.e at compile-time. 

We would like you to be able to
 
* edit-compile-run the running example from the paper
* edit-compile-run your own programs using the kmclib communication primitives
* observe compile-time detection of communication errors.
 
Additionally, you can try some additional programs implemented with kmclib.
 
## Getting started
 
For the TACAS'22 artifact evaluation, please use the VM available at [TACAS 2022 Artifact Evaluation Virtual Machine](https://doi.org/10.5281/zenodo.5537146)):
 
1. Download the VM from https://doi.org/10.5281/zenodo.5537146
2. Load it in [VirtualBox](https://www.virtualbox.org/) and boot it.
   - We recommend at least 2 CPU cores.
3. Log in with username `tacas22` and password `tacas22`.
   - Adjust the display setting if necessary.
4. We assume you have access to `artifact.iso` (in the submitted zip file and also available from https://drive.google.com/drive/folders/1--_Sq6PD2Zqg9jws3MiGd-0fi_OgQux1?usp=sharing)
5. Mount `artifact.iso` in the VirtualBox (Menu: Devices -> Optical Drives -> Choose a disk file ...)
6. Open the terminal from the bottom-left menu.
   - Either type "terminal" in the search box, or
   - Click the icon "Utilities" and scroll the window down
7. Type `cd /media/tacas22/artifact; ./install.sh`
8. All necessary files should be installed. If the password is asked, type `tacas22`.
   - Close the terminal and reopen, to make the updated PATH effective.
9. Open a terminal and navigate to `/home/tacas22/kmclib`.
10. Follow the instructions below.
 
In the following, we assume that you are in the `kmclib` directory.

## Artifact layout

The artifact is built from this [commit](https://github.com/keigoi/kmclib/tree/d2897c7) in the [kmclib GitHub repository](https://github.com/keigoi/kmclib).
 
In addition to the source code of the library, which is a git clone of [kmclib](https://github.com/keigoi/kmclib/),
the artifact also contains

* The directory [examples/paper](../examples/paper), which includes the running fibonacci example from the paper (Fig.2, Section 2)
* The directory [examples/helloworld](../examples/helloworld) that contains a reference implementation of the simple program explained in Step 2. 
* The directory [examples/miscellaneous](../examples/miscellaneous), which includes various examples you can test and run


## Step 1:  Programming with kmclib

The following instructions guide you how to compile, execute and modify the running example from the paper (Fig.2, Section 2). The program calculates Fibonacci numbers. The implementation is in [examples/paper/fib.ml](../examples/paper/fib.ml)

To start: 
* Open a Unix terminal and type the following commands to compile all examples

   ```
   cd kmclib
   ```
   ```
   dune build 
   ```
   
* Open Visual Studio Code, by typing in the `kmclib` directory:

```
code .
```

  * When it prompts "Do you trust the authors of ...", click "Yes, I trust the authors".
  * In the bottom-right, the following box might apper. Just ignore them and they will disappear.
    - "Ocamlformat_rpc is missing.": Ignore it as it doesn't change the behaviour of the IDE (and [it will not work](https://github.com/ocaml/ocaml-lsp/issues/513) if enabled).
    - "OCaml Platform is using the package manager ...": The environment is designed to work with the default one. Please don't change it.

Then, navigate to `examples/paper/fib.ml` in the left ("EXPLORER") pane and click it. The file will open in the right (main) pane.

Open a terminal in VSCode (*Terminal* menu -> *New Terminal*).

### 1.1. **Compile** the running program

Compile the file `examples/paper/fib.ml` via the following command line:

```
dune build examples/paper/fib.exe
```  

__Note__: Do not get confused by the .exe extension. This is not the Windows executable. It is a dune convention. The file fib.exe does not exist in /examples/paper/ but lies inside the `_build` folder.

Observe that no errors are reported.

### 1.2. **Run** the program

```
dune exec examples/paper/fib.exe
```

Observe the reported result.
The program prints the fibonacci number for 40 (in progress) and 42 (result)).

```
in progress: 102334155
result: 267914296
```

### 1.3. **Edit** the program and observe the reported errors

Next we highlight how concurrency errors are ruled out by static typing (i.e., the ultimate practical purpose of kmclib).

* **Progress errors**

   * Comment out [Line 31](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L31).   
  
       - After the edit [Line 31](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L31) should be: 
       ```ocaml 
       (* let mch = send mch#w#task (x - 2) in *)
       ```
       - Observe the progress violation errors on line 35 and line 24. Note that you do not need to compile again. 
   VS code will underline the wrong interaction(s) and you can see the error(s) if you hover over the line.
	   
	   - Before moving to the next step, uncomment Line 31.

   * If it does not work as above, try the followings:
       - If you have an error in Line 1, (re-)run `dune build examples/paper/fib.exe` at the `kmclib` directory.
       - If no error is highlighted, close and re-open (reload) the Visual Studio Code.

* **Eventual Reception errors**

   * Comment out [Line 35](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L35) and modify [Line 36](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L36) as follows:
       - [Line 35](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L35) should be 
       ```ocaml 
       (* let `result(r2, mch) = receive mch#w in *)
       ```
       - [Line 36](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L36) should be
       ```ocaml 
       loop (send mch#u#result (r1+1))
       ```

   * Observe the eventual reception error reported on Line 36. Similarly to above,  
   VS code will underline the wrong interaction(s) and you will see the error(s) if you hover over the line. 
   
   * Before moving to the next step, undo the changes at Lines 35 and 36.

* **Format (Type mismatch) errors**
 
   Format errors are simpler parse errors. They report possible typos or mismatches in the send/receive signatures, the message payloads, or the message labels that are exchanged. 
   
   Complete the edits suggested below. After each edit, you will be able to observe type mismatch errors.
 
   * Wrong send/receive signatures
       - Modify [Line 9](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L9) by removing one of the parameters of send, for example delete #m. After the edit [Line 9](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L9) should be `let uch = send uch#compute 42 in`

   * Mismatch between send and receive labels.
       - Option 1: Misspell `compute` to `comput` on [Line 9](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L9)
       - Option 2: Misspell `wip` to `wipe` on [Line 12](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L12)

   * Mismatch on payload types
       - Modify 42 on [Line 9](https://github.com/keigoi/kmclib/blob/db4472f24d70ba23a78cee6efd1ed7bd049fd634/examples/paper/fib.ml#L9) to “42”

## STEP 2: Writing your own programs

We explain how to edit, compile, and run a simple "Hello World"
example. Below we assume that VSCode is open in the kmclib directory.


### (1) Setting up the environment

1. Create a new folder called `myhelloworld` in `kmclib/examples` (right
click -> *New Folder*).

2. Create a new file called `dune` in the `myhelloworld` folder you have
 created (right click -> *New File*). Copy/paste the following in this
 file:

```ocaml
(executable
    (name helloworld)
    (modules helloworld)
    (libraries kmclib)
    (preprocess (staged_pps ppx_kmclib)))
```

This file simply declares a module called `helloworld` which will
create an executable called `helloworld.exe`. The module relies on the
`kmclib` library and uses a pre-processor (`ppx_kmclib`) before
compilation.

### (2) Writing your first kmclib program:

1. Create a new file called `helloworld.ml` in the `myhelloworld` folder.

2. Next, we write a kmclib program step-by-step. This program will
consists of two threads (Alice and Bob). Alice sends a string to
Bob, prints out what she sent. Bob receives the string, and prints
it on his end.

* First, we set up the headers:
```ocaml
open Kmclib (* loads the kmclib library *)
```

* At this point, it is a good idea to bootstrap the automatic
background compilation of VSCode. From the `myhelloworld` folder,
execute:

```
dune build
```

At this step, dune will fail because we still have not used the library that we have imported. 
You can safely ignore this error for now and proceed with the next steps.

* Below we will use `ach` (resp. `bch`) for the channel used by Alice
(resp. Bob) to exchange messages with Bob (resp. Alice). Also, atoms
`a` and `b` are role identifiers (used to express to whom/from whom
messages are to be sent/received).

* Next, we implement the thread for Alice:
```ocaml
let alice ach x =
    let ach = send ach#b#msg x in
    Printf.printf "Alice sent: %s\n" x;
    close ach
```

which sends a string `x` to Bob via the channel `ach`, using Bob's
role identifier (`b`). The program terminates by closing
`ach`.

Closing a channel does nothing at run-time. However at compile-time,
it guarantees that the type of the continuation channel `ach` is
`unit` (which signifies the end of Alice's role in the
session). Alternatively, Alice can be implemented without `close` as
follows:

```ocaml
(* Alice: alternative implementation *)
let alice ach x : unit =
	let ach = send ach#b#msg x in
	Printf.printf "Alice sent: %s\n" x;
	ach
```

In this implementation, the compiler infers the type of `ach` from the
return type of `alice` (see type annotation `unit`).


* Next, we implement the thread for Bob:
```ocaml
let bob bch =
    let `msg(txt, bch) = receive bch#a in
    Printf.printf "Bob received: %s\n" txt;
    close bch
```

which receives a string `txt` from the channel `bch` using Alice's
role identifier (`a`). The program terminates by closing `bch`.


Bob can be implemented without `close`, as follows:
```ocaml
(* Bob: alternative implementation *)
let bob bch =
	let `msg(txt, ()) = receive bch#a in
	Printf.printf "Bob received: %s\n" txt
```

In this implementation, the compiler infers that the continuation
channel returned by `receive` is `()` and thus its type is `unit`.


* Next, we initialise a kmclib session:
```ocaml
let KMC (ach, bch) = [%kmc.gen (a, b)]
```

This invocation will take care of checking the compatibility between
Alice and Bob. It also initialises session channels (`ach` and `bch`)
that will be passed to the actual thread instances below.


* Finally, we spawn one instance of each thread (passing a string
argument to Alice), and make the main thread wait for these to
terminate using`join`.

```ocaml
let () =
	let athread = Thread.create (alice ach) "Hello World" in
	let bthread = Thread.create bob bch in
	Thread.join athread;
	Thread.join bthread
```

### (3) Compiling/running your kmclib program:

1. Open a terminal in VSCode (*Terminal* menu -> *New Terminal*).

2. Compile your program (from the `myhelloworld` folder):
```
dune build
```

3. Run the program with:
```
dune exec ./helloworld.exe
```

4. Observe the output printed in the terminal:
```
Alice sent: Hello World
Bob received: Hello World
```

## STEP 3: Setting custom verification bounds

The k-MC checker performs a bounded verification of the session types
inferred via kmclib. By default, the bound is set to `20`. It is
possible to set a custom bound `N` using the phrase `~bound:N` when
setting up a kmclib session.

For instance, consider the program below (follow steps like above to edit it in VSCode):

```ocaml
open Kmclib

let KMC (ach,bch) = [%kmc.gen (a,b) ~bound:2] (* replace 2 by 1 to observe compile error *)

let senderA () =
	let ach = send ach#b#msg () in
	let ach = send ach#b#msg () in
	let `msg((), ach) = receive ach#b in
	let `msg((), ()) = receive ach#b in
	ach


let senderB () =
	let bch = send bch#a#msg () in
	let bch = send bch#a#msg () in
	let `msg((), bch) = receive bch#a in
	let `msg((), ()) = receive bch#a in
	bch
```

The example above is not 1-MC, but it is 2-MC, i.e., the system is
provably safe by exploring executions where there is at most 2 pending
messages in each channel. However, exploring only 1-bounded executions
does not give enough guarantees -- hence kmclib will not compile this
program with a bound < 2.


## STEP 4: Additional Examples (Optional)
The directory [examples/miscellaneous](../examples/miscellaneous) contains a few more examples of concurrent programs implemented with kmclib. 

The interested reader can follow these examples to familiarise themselves with the the kmclib primitives by modifying, compiling and running the programs. 
* Calculator
    - source folder: [examples/miscellaneous/calculator](../examples/miscellaneous/calculator)
    - description: A client-server calculator that can perform addition and multiplication.

* Ring 
    - source folder: [examples/miscellaneous/ring](../examples/miscellaneous/ring)
    - description: A forwarder pattern between three threads where the same message is sent between the threads.

* OAuth 
    - source folder: [examples/miscellaneous/toy_oauth](../examples/miscellaneous/toy_oauth)
    - description: A simplified shared memory implementation of an authentication between a client, a server and an authentication thread. The client requests a login, and the authentication thread grants it. 

    To compile each example: 
    ```
    dune build 
    ```

    To run each example: 
    ```
    dune exec ./example_name.exe 
    ```
    
    You must execute the above commands from the terminal, and must be in the folder corresponding to each example.
