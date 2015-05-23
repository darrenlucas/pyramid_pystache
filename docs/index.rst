================
pyramid_pystache
================

Overview
========

:mod:`pyramid_pystache` is a set of bindings that make templates written for
the :term:`Mustache` templating system work under the Pyramid web framework.

Installation
============

Install using setuptools, e.g. (within a virtualenv)::

  $ $myvenv/bin/easy_install pyramid_pystache

Setup
=====

There are several ways to make sure that :mod:`pyramid_pystache` is active.
They are completely equivalent:

#) Add pyramid_pystache to the `pyramid.includes` section of your applications
   main configuration section::

    [app:main]
    ...
    pyramid.includes = pyramid_pystache


#) Use the ``includeme`` function via ``config.include``::

    config.include('pyramid_pystache')

Once activated, files with the ``.mustache`` extension are considered to be
:term:`Mustache` templates.

.. _using_mustache_templates:

Using Mustache Templates
=========================

Once :mod:`pyramid_pystache` been activated ``.mustache`` templates can be
loaded either by looking up names that would be found on the :term:`Mustache`
search path or by looking up an absolute asset specification (see
:ref:`asset_specifications` for more information).

Quick example 1.  Look up a template named ``foo.mustache`` within the
``templates`` directory of a Python package named ``mypackage``:

.. code-block:: python
   :linenos:

    @view_config(renderer="mypackage:templates/foo.mustache)
    def sample_view(request):
       return {'foo':1, 'bar':2}

Quick example 2.  Look up a template named ``foo.mustache`` within the
``templates`` directory of the "current" Python package (the package in which
this Python code is defined):

.. code-block:: python
   :linenos:

    @view_config(renderer="templates/foo.mustache)
    def sample_view(request):
       return {'foo':1, 'bar':2}

Quick example 3: manufacturing a response object using the result of
:func:`~pyramid.renderers.render` (a string) using a Mustache template:

.. code-block:: python
   :linenos:

   from pyramid.renderers import render
   from pyramid.response import Response

   def sample_view(request):
       result = render('mypackage:templates/foo.mustache',
                       {'foo':1, 'bar':2},
                       request=request)
       response = Response(result)
       response.content_type = 'text/plain'
       return response

Here's an example view configuration which uses a Mustache renderer
registered imperatively:

.. code-block:: python
   :linenos:

    # config is an instance of pyramid.config.Configurator

    config.add_view('myproject.views.sample_view',
                    renderer='myproject:templates/foo.mustache')


.. _mustache_templates:

Mustache Templates
------------------

The language definition documentation for Mustache templates is available from
`the Mustache manual <http://mustache.github.io/mustache.5.html>`_.

Given a :term:`Mustache` template named ``foo.mustache`` in a directory
in your application named ``templates``, you can render the template as
a :term:`renderer` like so:

.. code-block:: python
   :linenos:

   from pyramid.view import view_config

   @view_config(renderer='templates/foo.mustache')
   def my_view(request):
       return {'foo':1, 'bar':2}

When a Mustache renderer is used in a view configuration, the view must return
a :term:`Response` object or a Python *dictionary*.  If the view callable with
an associated template returns a Python dictionary, the named template will be
passed the dictionary as its keyword arguments, and the template renderer
implementation will return the resulting rendered template in a response to the
user.  If the view callable returns anything but a Response object or a
dictionary, an error will be raised.

Before passing keywords to the template, the keyword arguments derived from
the dictionary returned by the view are augmented.  The callable object --
whatever object was used to define the view -- will be automatically inserted
into the set of keyword arguments passed to the template as the ``view``
keyword.  If the view callable was a class, the ``view`` keyword will be an
instance of that class.  Also inserted into the keywords passed to the
template are ``renderer_name`` (the string used in the ``renderer`` attribute
of the directive), ``renderer_info`` (an object containing renderer-related
information), ``context`` (the context resource of the view used to render
the template), and ``request`` (the request passed to the view used to render
the template).  ``request`` is also available as ``req`` in Pyramid 1.3+.

.. index::
   single: Mustache template (sample)

A Sample Mustache Template
~~~~~~~~~~~~~~~~~~~~~~~~~~

Here's what a simple :term:`Mustache` template used under :app:`Pyramid` might
look like:

.. code-block:: xml
   :linenos:

    <!DOCTYPE html>
    <html>
    <head>
        <meta http-equiv="content-type" content="text/html; charset=utf-8" />
        <title>{{project}} Application</title>
    </head>
      <body>
         <h1>Welcome to <code>{{project}}</code>, an
	  application generated by the <a
	  href="http://docs.pylonsproject.org/projects/pyramid/current/"
         >pyramid</a> web
	  application framework.</h1>
      </body>
    </html>

The above template expects to find a ``project`` key in the set of keywords
passed in to it via :func:`~pyramid.renderers.render` or
:func:`~pyramid.renderers.render_to_response`.

Template Variables provided by Pyramid
--------------------------------------

Pyramid by default will provide a set of variables that are available within
your templates, please see :ref:`renderer_system_values` for more information
about those variables.

.. index::
   single: template renderer side effects


Changing the Content-Type of a Mustache-Renderered Response
------------------------------------------------------------

Here's an example of changing the content-type and status of the
response object returned by a Mustache-rendered Pyramid view:

.. code-block:: python
   :linenos:

   @view_config(renderer='foo.mustache')
   def sample_view(request):
       request.response.content_type = 'text/plain'
       response.status_int = 204
       return response

See :ref:`request_response_attr` for more information.


Unit Testing
------------

When you are running unit tests, you will be required to use
``config.include('pyramid_pystache')`` to add ``pyramid_pystache`` so that
its renderers are added to the config and can be used.::

    from pyramid import testing
    from pyramid.response import Response
    from pyramid.renderers import render

    # The view we want to test
    def some_view(request):
        return Response(
            render('mypkg:templates/home.mustache', {'var': 'testing'})
        )

    class TestViews(unittest.TestCase):
        def setUp(self):
            self.config = testing.setUp()
            self.config.include('pyramid_pystache')

        def tearDown(self):
            testing.tearDown()

        def test_some_view(self):
            from pyramid.testing import DummyRequest
            request = DummyRequest()
            response = some_view(request)
            # templates/home.mustache starts with the standard <html> tag
            self.assertTrue('<html' in response.body)


More Information
================

.. toctree::
 :maxdepth: 1

 glossary.rst
 api.rst

Reporting Bugs / Development Versions
=====================================

Visit http://github.com/darrenlucas/pyramid_pystache to download development or
tagged versions.

Visit http://github.com/darrenlucas/pyramid_pystache/issues to report bugs.

Indices and tables
------------------

* :ref:`glossary`
* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
