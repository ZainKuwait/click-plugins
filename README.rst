=============
click-plugins
=============

.. image:: https://travis-ci.org/click-contrib/click-plugins.svg?branch=master
    :target: https://travis-ci.org/click-contrib/click-plugins?branch=master

.. image:: https://coveralls.io/repos/click-contrib/click-plugins/badge.svg?branch=master&service=github
    :target: https://coveralls.io/github/click-contrib/click-plugins?branch=master

An extension module for `click <https://github.com/mitsuhiko/click>`_ to register
external CLI commands via setuptools entry-points.


What this fork does
----

This fork simply adds the two hanging pull requests of `click-plugins <https://github.com/click-contrib/click-plugins>`_ to the project.

Those two pull requests are:

- `PR #33: Allow \`with_plugins()\` to accept a string <https://github.com/click-contrib/click-plugins/pull/33>`_
- `PR #32: Use entrypoint resolve to not check all the dependencies in the env <https://github.com/click-contrib/click-plugins/pull/32>`_

How to install this fork
----

Use this:

.. code-block:: bash

    pip install git+https://github.com/ZainKuwait/click-plugins.git#egg=click-plugins

To add it to a ``setup.py`` file, add the following:

.. code-block:: python

    from setuptools import find_packages, setup

    setup(
        # ...
        install_requires=[
            "click-plugins @ git+https://github.com/ZainKuwait/click-plugins.git@master#egg=click-plugins"
        ],
        # ...
    )

For a ``requirements.txt`` file, just add this to the list (no version specification required):

.. code-block:: text

    git+https://github.com/ZainKuwait/click-plugins.git@master#egg=click-plugins


Why?
----

Lets say you develop a commandline interface and someone requests a new feature
that is absolutely related to your project but would have negative consequences
like additional dependencies, major refactoring, or maybe its just too domain
specific to be supported directly.  Rather than developing a separate standalone
utility you could offer up a `setuptools entry point <https://pythonhosted.org/setuptools/setuptools.html#dynamic-discovery-of-services-and-plugins>`_
that allows others to use your commandline utility as a home for their related
sub-commands.  You get to choose where these sub-commands or sub-groups CAN be
registered but the plugin developer gets to choose they ARE registered.  You
could have all plugins register alongside the core commands, in a special
sub-group, across multiple sub-groups, or some combination.


Enabling Plugins
----------------

For a more detailed example see the `examples <https://github.com/click-contrib/click-plugins/tree/master/example>`_ section.

The only requirement is decorating ``click.group()`` with ``click_plugins.with_plugins()``
which handles attaching external commands and groups.  In this case the core CLI developer
registers CLI plugins from ``core_package.cli_plugins``.

.. code-block:: python

    import click
    from click_plugins import with_plugins


    @with_plugins('core_package.cli_plugins')
    @click.group()
    def cli():
        """Commandline interface for yourpackage."""

    @cli.command()
    def subcommand():
        """Subcommand that does something."""

Alternatively, an iterable producing one ``pkg_resources.EntryPoint()`` per
interation (like ``pkg_resources.iter_entry_points()``) can be used.

.. code-block:: python

    from pkg_resources import iter_entry_points

    import click
    from click_plugins import with_plugins


    @with_plugins(iter_entry_points('core_package.cli_plugins'))
    @click.group()
    def cli():
        """Commandline interface for yourpackage."""

    @cli.command()
    def subcommand():
        """Subcommand that does something."""


Developing Plugins
------------------

Plugin developers need to register their sub-commands or sub-groups to an
entry-point in their ``setup.py`` that is loaded by the core package.

.. code-block:: python

    from setuptools import setup

    setup(
        name='yourscript',
        version='0.1',
        py_modules=['yourscript'],
        install_requires=[
            'click',
        ],
        entry_points='''
            [core_package.cli_plugins]
            cool_subcommand=yourscript.cli:cool_subcommand
            another_subcommand=yourscript.cli:another_subcommand
        ''',
    )


Broken and Incompatible Plugins
-------------------------------

Any sub-command or sub-group that cannot be loaded is caught and converted to
a ``click_plugins.core.BrokenCommand()`` rather than just crashing the entire
CLI.  The short-help is converted to a warning message like:

.. code-block:: console

    Warning: could not load plugin. See ``<CLI> <command/group> --help``.

and if the sub-command or group is executed the entire traceback is printed.


Best Practices and Extra Credit
-------------------------------

Opening a CLI to plugins encourages other developers to independently extend
functionality independently but there is no guarantee these new features will
be "on brand".  Plugin developers are almost certainly already using features
in the core package the CLI belongs to so defining commonly used arguments and
options in one place lets plugin developers reuse these flags to produce a more
cohesive CLI.  If the CLI is simple maybe just define them at the top of
``yourpackage/cli.py`` or for more complex packages something like
``yourpackage/cli/options.py``.  These common options need to be easy to find
and be well documented so that plugin developers know what variable to give to
their sub-command's function and what object they can expect to receive.  Don't
forget to document non-obvious callbacks.

Keep in mind that plugin developers also have access to the parent group's
``ctx.obj``, which is very useful for passing things like verbosity levels or
config values around to sub-commands.

Here's some code that sub-commands could re-use:

.. code-block:: python

    from multiprocessing import cpu_count

    import click

    jobs_opt = click.option(
        '-j', '--jobs', metavar='CORES', type=click.IntRange(min=1, max=cpu_count()), default=1,
        show_default=True, help="Process data across N cores."
    )

Plugin developers can access this with:

.. code-block:: python

    import click
    import parent_cli_package.cli.options


    @click.command()
    @parent_cli_package.cli.options.jobs_opt
    def subcommand(jobs):
        """I do something domain specific."""


Installation
------------

With ``pip``:

.. code-block:: console

    $ pip install click-plugins

From source:

.. code-block:: console

    $ git clone https://github.com/click-contrib/click-plugins.git
    $ cd click-plugins
    $ python setup.py install


Developing
----------

.. code-block:: console

    $ git clone https://github.com/click-contrib/click-plugins.git
    $ cd click-plugins
    $ pip install -e .\[dev\]
    $ pytest tests --cov click_plugins --cov-report term-missing


Changelog
---------

See ``CHANGES.txt``


Authors
-------

See ``AUTHORS.txt``


License
-------

See ``LICENSE.txt``
