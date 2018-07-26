Interacting with Owl
=================================================

There are several ways to interact with Owl system. The most classic one is to write an OCaml application, compile the code, then run it natively on a computer. You can also skip the compilation step, and use Zoo system to run the code as a script.

However, the easiest way for a beginner to try out Owl is REPL (Read–Eval–Print Loop), or an interactive toplevel. The toplevel offers a convenient way to play with small code snippets. The code run in the toplevel is compiled into bytecode rather than native code. Bytecode often runs much slower than native code. However, this has very little impact on Owl's performance because all its performance-critical functions are implemented in C language.

In the following, I will introduce two options to set up an interactive environment for Owl.



Using Toplevel
-------------------------------------------------

OCaml language has bundled with a simple toplevel, but I recommend to use *utop* as a more advance replacement. Installing *utop* is straightforward in OPAM, simply run the following command in the system shell.


.. code-block:: bash

  opam install utop


After installation, you can load Owl in *utop* with the following commands. ``owl-toplevel`` is Owl's toplevel library which will automatically load several dependent libraries (including ``owl-zoo``, ``owl-base``, and ``owl`` core library) to set up a complete numerical environment.

.. code-block:: ocaml

  #require "owl-top";;
  open Owl;;


If you do not want to type these commands every time you start *toplevel*, you can add them to ``.ocamlinit`` file. The toplevel reads ``.ocamlinit`` file when it starts and uses it to initialise the environment. This file is often stored in the home directory on your computer.



Using Notebook
-------------------------------------------------

Jupyter Notebook is a popular way to mix presentation with interactive code execution. It originates in Python world but is widely supported by various languages. One attractive feature of notebook is that it uses client/server architecture and runs in a browser.

If you want to know how to use a notebook and its technical details, please read `Jupyter Documentation <http://jupyter.org/documentation>`_. Here let me show you how to set up a notebook to run Owl step by step.

Run the following commands in the shell will install all the dependency for you. This includes Jupyter Notebook and its `OCaml language extension <https://github.com/akabe/ocaml-jupyter>`_.


.. code-block:: bash

  pip install jupyter
  opam install jupyter
  jupyter kernelspec install --name ocaml-jupyter "$(opam config var share)/jupyter"


To start a Jupyter notebook, you can run this command. The command starts a local server running on ``http://127.0.0.1:8888/``, then opens a tab in your browser as the client.


.. code-block:: bash

  jupyter notebook


When everything is up and running, you can start a new notebook in the web interface. In the new notebook, you must run the following OCaml code in the first input field to load Owl environment.


.. code-block:: bash

  #use "topfind";;
  #require "owl-top, jupyter.notebook";;


At this point, a complete Owl environment is set up in the Jupyter Notebook, and you are free to go with any experiments you like.



Notebook Examples
-------------------------------------------------

You can simply copy & paste the whole `lazy_mnist.ml <https://github.com/owlbarn/owl/blob/master/examples/lazy_mnist.ml>`_ to train a convolutional neural network in the notebook. But here, let us just use the following code.


.. code-block:: ocaml

  #use "topfind";;
  #require "owl-top, jupyter.notebook";;

  open Owl
  open Neural.S
  open Neural.S.Graph
  open Neural.S.Algodiff


  let make_network input_shape =
    input input_shape
    |> lambda (fun x -> Maths.(x / F 256.))
    |> conv2d [|5;5;1;32|] [|1;1|] ~act_typ:Activation.Relu
    |> max_pool2d [|2;2|] [|2;2|]
    |> dropout 0.1
    |> fully_connected 1024 ~act_typ:Activation.Relu
    |> linear 10 ~act_typ:Activation.(Softmax 1)
    |> get_network
    ;;


  make_network [|28;28;1|];;


Jupyter notebook should nicely print out the structure of the neural network.


.. figure:: ../figure/jupyter_example_01.png
   :scale: 50 %
   :align: center
   :alt: jupyter example 01


Second example demonstrates how to plot figures in notebook. Because Owl's Plot module does not support in-memory plotting, the figure needs to be written into a file first then read back as a string. In-memory plotting is not difficult to implement at all so I will certainly add this in future, at the moment we can just use ``Owl_io.read_file_string``.


.. code-block:: ocaml

  #use "topfind";;
  #require "owl-top, jupyter.notebook";;
  open Owl;;

  (* Plot a normal figure using Plot *)

  let f x = Maths.sin x /. x in
  let h = Plot.create "plot_003.png" in
  Plot.set_foreground_color h 0 0 0;
  Plot.set_background_color h 255 255 255;
  Plot.set_title h "Function: f(x) = sine x / x";
  Plot.set_xlabel h "x-axis";
  Plot.set_ylabel h "y-axis";
  Plot.set_font_size h 8.;
  Plot.set_pen_size h 3.;
  Plot.plot_fun ~h f 1. 15.;
  Plot.output h;;

  (* Load into memory and display in Jupyter *)

  let data = Owl_io.read_file_string "plot_003.png" in
  Jupyter_notebook.display ~base64:true "image/png" data


Then we can see the plot is correctly rendered in the notebook running in your browser. Plotting capability greatly enriches the content of an interactive presentation.


.. figure:: ../figure/jupyter_example_02.png
   :scale: 50 %
   :align: center
   :alt: jupyter example 02