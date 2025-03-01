.. _otter_assign_v1_usage:

Usage and Output
================

Otter Assign is called using the ``otter assign`` command. This command takes in two required 
arguments. The first is ``master``, the path to the master notebook (the one formatted as described 
above), and the second is ``result``, the path at which output shoud be written. **Otter Assign will 
automatically recognize the language of the notebook** by looking at the kernel metadata; similarly, 
if using an Rmd file, it will automatically choose the language as R. This behavior can be 
overridden using the ``-l`` flag which takes the name of the language as its argument.

The default behavior of Otter Assign is to do the following:

#. Filter test cells from the master notebook and write these to test files
#. Add Otter initialization, export, and ``Notebook.check_all`` cells
#. Clear outputs and write questions (with metadata hidden), prompts, and solutions to a notebook 
   in a new ``autograder`` directory
#. Write *all* tests to ``autograder/tests``
#. Copy autograder notebook with solutions removed into a new ``student`` directory
#. Write *public* tests to ``student/tests``
#. Copy ``files`` into ``autograder`` and ``student`` directories
#. Run all tests in ``autograder/tests`` on the solutions notebook to ensure they pass
#. (If ``generate`` is passed,) generate a Gradescope autograder zipfile from the ``autograder`` 
   directory

The behaviors described in step 2 can be overridden using the optional arguments described in the 
help specification.

**An important note:** make sure that you *run all cells* in the master notebook and save it *with 
the outputs* so that Otter Assign can generate the test files based on these outputs. The outputs 
will be cleared in the copies generated by Otter Assign.


Python Output Additions
-----------------------

The following additional cells are included *only* in Python notebooks.


Export Formats and Flags
++++++++++++++++++++++++

By default, Otter Assign adds an initialization cell at the *top* of the notebook with the contents

.. code-block:: python

    # Initialize Otter
    import otter
    grader = otter.Notebook()

To prevent this behavior, add the `init_cell: false` configuration in your assignment metadata.

Otter Assign also automatically adds a check-all cell and an export cell to the end of the notebook. 
The check-all cells consist of a Markdown cell:

.. code-block:: markdown

    To double-check your work, the cell below will rerun all of the autograder tests.

and a code cell that calls ``otter.Notebook.check_all``:

.. code-block:: python

    grader.check_all()

The export cells consist of a Markdown cell:

.. code-block:: markdown

    ## Submission

    Make sure you have run all cells in your notebook in order before running the cell below, so 
    that all images/graphs appear in the output. **Please save before submitting!**

and a code cell that calls ``otter.Notebook.export`` with HTML comment filtering:

.. code-block:: python

    # Save your notebook first, then run this cell to export.
    grader.export("/path/to/notebook.ipynb")

For R assignments, the export cell looks like:

.. code-block:: r

    # Save your notebook first, then run this cell to export.
    ottr::export("/path/to/notebook.ipynb")

These behaviors can be changed with the corresponding assignment metadata configurations.

**Note:** Otter Assign currently only supports :ref:`HTML comment filtering <pdfs>`. This means 
that if you have other cells you want included in the export, you must delimit them using HTML 
comments, not using cell tags.


Otter Assign Example
--------------------

Consider the directory stucture below, where ``hw00/hw00.ipynb`` is an Otter Assign-formatted 
notebook.

.. code-block::

    hw00
    ├── data.csv
    └── hw00.ipynb

To generate the distribution versions of ``hw00.ipynb`` (after changing into the ``hw00`` 
directory), you would run

.. code-block::

    otter assign hw00.ipynb dist --v1

If it was an Rmd file instead, you would run

.. code-block::

    otter assign hw00.Rmd dist --v1

This will create a new folder called ``dist`` with ``autograder`` and ``student`` as subdirectories, 
as described above.

.. code-block::

    hw00
    ├── data.csv
    ├── dist
    │   ├── autograder
    │   │   ├── hw00.ipynb
    │   │   └── tests
    │   │       ├── q1.(py|R)
    │   │       └── q2.(py|R)  # etc.
    │   └── student
    │       ├── hw00.ipynb
    │       └── tests
    │           ├── q1.(py|R)
    │           └── q2.(py|R)  # etc.
    └── hw00.ipynb

In generating the distribution versions, you can prevent Otter Assign from rerunning the tests using 
the ``--no-run-tests`` flag:

.. code-block::

    otter assign --no-run-tests hw00.ipynb dist --v1

Because tests are not run on R notebooks, the above flag would be ignored if ``hw00.ipynb`` 
had an R kernel.
