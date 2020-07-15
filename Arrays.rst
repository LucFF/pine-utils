Arrays
======

.. contents:: :local:
    :depth: 2



Introduction
------------

Arrays can be used to store multiple values in one data structure. Think of them as a better way to handle cases where you would
otherwise need a set of variables named ``price00``, ``price01`` and ``price02``.

Pine arrays are one-dimensional. All elements of any given array are of the same type, which can be *int*, *float*, *bool* or *color*, always of *series* form. 
Arrays are referenced using an array *id*, similar to label and line id's. 
As with other Pine variables, the history-referencing operator can be used to refer to past instances of an array when it is used with the array's id. 
Pine does not use an indexing operator to reference individual array elements;
instead, functions like ``array.get()`` and ``array.set()`` are used to read and write values of array elements. 
Array values can be used in all Pine expressions and functions where a *series* value is allowed.

Elements within an array are referred to using an *index*, which starts at 0 and extends to the number or elements in the array, minus one.
Arrays in Pine can be sized dynamically, so the number of elements in the array can be modified within one iteration of the script on a bar,
and vary across bars. Multiple arrays can be used by the same script. The size of arrays is limited by the runtime resources required for the script,
which are evaluated dynamically, so there are no hard-limits.

Arrays can be declared in a script's global scope, or in local scopes such as function and ``if`` block scopes.
One major distinction between global scope arrays and variables in Pine is that global scope arrays can be modified from within local scopes.
A Pine function, for example, can thus modify an array declared in the script's global scope. 

.. note:: We will use "beginning" of an array to designate index 0, and "end" of an array to designate the array's element with the highest index value. 
We will also extend the meaning of *array* to include array *id's*, for the sake of brevity.



Declaring arrays
----------------

The following syntax is used to declare arrays::

    <type>[] <identifier> = <expression>

The ``[]`` modifier is appended to the type name when declaring arrays. However, since type-specific functions are used to create an array,
the ``<type>[]`` part of the declaration is redundant, except if you initialize the array id to ``na``, as in this example, 
which declares an empty array of *float* values with an array id named ``prices``::

    float[] prices = na

This declaration also creates an array containing zero *float* elements, 
but this time the array id returned by the ``array.new_float(0)`` function call is assigned to ``prices``::

    prices = array.new_float(0)

Similar array creation functions exist for the other types of array elements: ``array.new_int()``, ``array.new_bool()`` and ``array.new_color()``.

You can also initialize array elements when declaring an array. The following declaration creates and array id named ``prices``.
The array is created with two elements, each initialized with the value of the ``close`` built-in variable::

    prices = array.new_float(2, close)



Reading and writing array values
--------------------------------
``array.size()``
``array.get()``
``array.set()``
``array.fill()``



Inserting and removing array elements
-------------------------------------
``array.insert()``
``array.remove()``
``array.clear()``


Using an array as a stack
^^^^^^^^^^^^^^^^^^^^^^^^^

Stacks are LIFO (last in, first out) constructions. They behave somewhat like a vertical pile of books to which books can only be added or removed one at a time,
always from the top. Pine arrays can be used as a stack, in which case you will use the ``array.push()`` and ``array.pop()`` functions to add and remove elements at the end of the array.

``array.push(prices, close)`` will add a new element to the end of the ``prices`` array, increasing the array's size by one.

``array.pop(prices)`` will remove the end element from the ``prices`` array, return its value and decrease the array's size by one.

[[Need a code example]]::

    code



Calculations on arrays
-------------------
``array.min()``
``array.max()``
``array.sum()``



Manipulating arrays
-------------------
``array.sort()``
``array.copy()``
``array.slice()``
``array.concat()``



Runtime errors
--------------



═══════════════════════ JUNK ═════════════════════════

Starting with Pine v4, indicators and strategies can
create *drawing objects* on the chart. Two types of
drawings are currently supported: *label* and *line*.
You will find one instance of each on the following chart:

.. image:: images/label_and_line_drawings.png

.. note:: On TradingView charts, a complete set of *Drawing Tools*
  allows users to create and modify drawings using mouse actions. While they may look similar to
  drawing objects created with Pine code, they are essentially different entities.
  Drawing objects created using Pine code cannot be modified with mouse actions.

The new line and label drawings in Pine v4 allow you to create indicators with more sophisticated
visual components, e.g., pivot points, support/resistance levels,
zig zag lines, labels containing dynamic text, etc.

In contrast to indicator plots (plots are created with functions ``plot``, ``plotshape``, ``plotchar``),
drawing objects can be created on historical bars as well as in the future, where no bars exist yet.

Pine drawing objects are created with the `label.new <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}new>`__
and `line.new <https://www.tradingview.com/pine-script-reference/v4/#fun_line{dot}new>`__ functions.
While each function has many parameters, only the coordinates are mandatory.
This is an example of code used to create a label on every bar::

    //@version=4
    study("My Script", overlay=true)
    label.new(bar_index, high)

.. image:: images/minimal_label.png

The label is created with the parameters ``x=bar_index`` (the index of the current bar,
`bar_index <https://www.tradingview.com/pine-script-reference/v4/#var_bar_index>`__) and ``y=high`` (high price of the current bar).
When a new bar opens, a new label is created on it. Label objects created on previous bars stay on the chart
until the indicator deletes them with an explicit call of the `label.delete <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}delete>`__
function, or until the automatic garbage collection process removes them.

Here is a modified version of the same script that shows the values of the ``x`` and ``y`` coordinates used to create the labels::

    //@version=4
    study("My Script", overlay=true)
    label.new(bar_index, high, style=label.style_none,
              text="x=" + tostring(bar_index) + "\ny=" + tostring(high))

.. image:: images/minimal_label_with_x_y_coordinates.png

In this example labels are shown without background coloring (because of parameter ``style=label.style_none``) but with
dynamically created text (``text="x=" + tostring(bar_index) + "\ny=" + tostring(high)``) that prints label coordinates.

This is an example of code that creates line objects on a chart::

    //@version=4
    study("My Script", overlay=true)
    line.new(x1=bar_index[1], y1=low[1], x2=bar_index, y2=high)

.. image:: images/minimal_line.png


Calculation of drawings on bar updates

Drawing objects are subject to both *commit* and *rollback* actions, which affect the behavior of a script when it executes
in the realtime bar, :doc:`/language/Execution_model`.

This script demonstrates the effect of rollback when running in the realtime bar::

    //@version=4
    study("My Script", overlay=true)
    label.new(bar_index, high)

While ``label.new`` creates a new label on every iteration of the script when price changes in the realtime bar,
the most recent label created in the script's previous iteration is also automatically deleted because of rollback before the next iteration. Only the last label created before the realtime bar's close will be committed, and will thus persist.

.. _drawings_coordinates:

Coordinates

Drawing objects are positioned on the chart according to *x* and *y* coordinates using a combination of 4 parameters: ``x``, ``y``, ``xloc`` and ``yloc``. The value of ``xloc`` determines whether ``x`` will hold a bar index or time value. When ``yloc=yloc.price``, ``y`` holds a price. ``y`` is ignored when ``yloc`` is set to `yloc.abovebar <https://www.tradingview.com/pine-script-reference/v4/#var_yloc{dot}abovebar>`__ or `yloc.belowbar <https://www.tradingview.com/pine-script-reference/v4/#var_yloc{dot}belowbar>`__.

If a drawing object uses `xloc.bar_index <https://www.tradingview.com/pine-script-reference/v4/#var_xloc{dot}bar_index>`__, then
the x-coordinate is treated as an absolute bar index. The bar index of the current bar can be obtained from the built-in variable ``bar_index``. The bar index of previous bars is ``bar_index[1]``, ``bar_index[2]`` and so on. ``xloc.bar_index`` is the default value for x-location parameters of both label and line drawings.

If a drawing object uses `xloc.bar_time <https://www.tradingview.com/pine-script-reference/v4/#var_xloc{dot}bar_time>`__, then
the x-coordinate is treated as a UNIX time in milliseconds. The start time of the current bar can be obtained from the built-in variable ``time``.
The bar time of previous bars is ``time[1]``, ``time[2]`` and so on. Time can also be set to an absolute time point with the
`timestamp <https://www.tradingview.com/pine-script-reference/v4/#fun_timestamp>`__ function.

The ``xloc.bar_time`` mode makes it possible to place a drawing object in the future, to the right of the current bar. For example::

    //@version=4
    study("My Script", overlay=true)
    dt = time - time[1]
    if barstate.islast
        label.new(time + 3*dt, close, xloc=xloc.bar_time)

.. image:: images/label_in_the_future.png

This code places a label object in the future. X-location logic works identically for both label and line drawings.

In contrast, y-location logic is different for label and line drawings.
Pine's *line* drawings always use `yloc.price <https://www.tradingview.com/pine-script-reference/v4/#var_yloc{dot}price>`__,
so their y-coordinate is always treated as an absolute price value.

Label drawings have additional y-location values: `yloc.abovebar <https://www.tradingview.com/pine-script-reference/v4/#var_yloc{dot}abovebar>`__ and
`yloc.belowbar <https://www.tradingview.com/pine-script-reference/v4/#var_yloc{dot}belowbar>`__.
When they are used, the value of the ``y`` parameter is ignored and the drawing object is placed above or below the bar.


The available *setter* functions for label drawings are:

    * `label.set_color <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_color>`__ --- changes color of label
    * `label.set_size <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_size>`__ --- changes size of label
    * `label.set_style <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_style>`__ --- changes :ref:`style of label <drawings_label_styles>`
    * `label.set_text <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_text>`__ --- changes text of label
    * `label.set_textcolor <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_textcolor>`__ --- changes color of text
    * `label.set_x <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_x>`__ --- changes x-coordinate of label
    * `label.set_y <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_y>`__ --- changes y-coordinate of label
    * `label.set_xy <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_xy>`__ --- changes both x and y coordinates of label
    * `label.set_xloc <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_xloc>`__ --- changes x-location of label
    * `label.set_yloc <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_yloc>`__ --- changes y-location of label
    * `label.set_tooltip <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}set_tooltip>`__ --- changes tooltip of label


