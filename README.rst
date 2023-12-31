Deform
======

.. image:: https://github.com/Pylons/deform/workflows/Build%20and%20test/badge.svg?branch=main
    :target: https://github.com/Pylons/deform/actions?query=workflow%3A%22Build+and+test%22+branch%3Amain
    :alt: Build Status

.. image:: https://img.shields.io/pypi/v/deform
    :target: https://pypi.org/project/deform/
    :alt: Current Release

.. image:: https://readthedocs.org/projects/deform/badge/?version=main
    :target: https://docs.pylonsproject.org/projects/deform/en/main/
    :alt: Main Documentation Status

.. image:: https://readthedocs.org/projects/deform/badge/?version=latest
    :target: https://docs.pylonsproject.org/projects/deform/en/latest/
    :alt: Latest Documentation Status

.. image:: https://img.shields.io/pypi/pyversions/deform
    :alt: Python Support

.. contents:: :local:


Introduction
------------

Deform is a Python form library for generating HTML forms on the server side.
`Date and time picking widgets <https://deformdemo.pylonsproject.org/datetimeinput/>`_,
`rich text editors <https://deformdemo.pylonsproject.org/richtext/>`_, `forms with
dynamically added and removed items
<https://deformdemo.pylonsproject.org/sequence_of_mappings/>`_ and a few other `complex
use cases <https://deformdemo.pylonsproject.org/>`_ are supported out of the box.

Deform integrates with the `Pyramid web framework <https://trypyramid.com/>`_
and several other web frameworks. Deform comes with `Chameleon templates
<https://chameleon.readthedocs.io/en/latest/>`_ and `Bootstrap 5
<https://getbootstrap.com/docs/5.3/>`_ styling. Under the hood, `Colander schemas
<https://github.com/Pylons/colander>`_ are used for serialization and
validation. The `Peppercorn <https://github.com/Pylons/peppercorn>`_ library
maps HTTP form submissions to nested structure.

Although Deform uses Chameleon templates internally, you can embed rendered
Deform forms into any template language.

Use cases
---------

Deform is ideal for complex server-side generated forms. Potential use cases
include:

* Complex data entry forms

* Administrative interfaces

* Python based websites with high amount of data manipulation forms

* Websites where additional front end framework is not needed

Installation
------------

Install using `pip and Python package installation best practices <https://packaging.python.org/tutorials/installing-packages/>`_::

    pip install deform

Example
-------

`See all widget examples <https://deformdemo.pylonsproject.org>`_. Below is a sample
form loop using the `Pyramid <https://trypyramid.com/>`_ web framework.

.. image:: https://github.com/Pylons/deform/raw/main/docs/example.png
    :width: 400px

Example code:

.. code-block:: python

    """Self-contained Deform demo example."""
    from __future__ import print_function

    from pyramid.config import Configurator
    from pyramid.session import UnencryptedCookieSessionFactoryConfig
    from pyramid.httpexceptions import HTTPFound

    import colander
    import deform


    class ExampleSchema(deform.schema.CSRFSchema):

        name = colander.SchemaNode(
            colander.String(),
            title="Name")

        age = colander.SchemaNode(
            colander.Int(),
            default=18,
            title="Age",
            description="Your age in years")


    def mini_example(request):
        """Sample Deform form with validation."""

        schema = ExampleSchema().bind(request=request)

        # Create a styled button with some extra Bootstrap 3 CSS classes
        process_btn = deform.form.Button(name='process', title="Process")
        form = deform.form.Form(schema, buttons=(process_btn,))

        # User submitted this form
        if request.method == "POST":
            if 'process' in request.POST:

                try:
                    appstruct = form.validate(request.POST.items())

                    # Save form data from appstruct
                    print("Your name:", appstruct["name"])
                    print("Your age:", appstruct["age"])

                    # Thank user and take him/her to the next page
                    request.session.flash('Thank you for the submission.')

                    # Redirect to the page shows after succesful form submission
                    return HTTPFound("/")

                except deform.exception.ValidationFailure as e:
                    # Render a form version where errors are visible next to the fields,
                    # and the submitted values are posted back
                    rendered_form = e.render()
        else:
            # Render a form with initial default values
            rendered_form = form.render()

        return {
            # This is just rendered HTML in a string
            # and can be embedded in any template language
            "rendered_form": rendered_form,
        }


    def main(global_config, **settings):
        """pserve entry point"""
        session_factory = UnencryptedCookieSessionFactoryConfig('seekrit!')
        config = Configurator(settings=settings, session_factory=session_factory)
        config.include('pyramid_chameleon')
        deform.renderer.configure_zpt_renderer()
        config.add_static_view('static_deform', 'deform:static')
        config.add_route('mini_example', path='/')
        config.add_view(mini_example, route_name="mini_example", renderer="templates/mini.pt")
        return config.make_wsgi_app()

This example is in `deformdemo repository <https://github.com/Pylons/deformdemo/>`_. Run the example with pserve::

     pserve mini.ini --reload

Status
------

This library is actively developed and maintained. Deform 2.x branch has been used in production on several sites since 2014. Automatic test suite has 100% Python code coverage and 500+ tests.

Projects using Deform
---------------------

* `Websauna <https://websauna.org/>`_

* `Kotti <http://kotti.pylonsproject.org/>`_

* `Substance D <http://www.substanced.net/>`_

Community and links
-------------------

* `Widget examples <https://deformdemo.pylonsproject.org>`_

* `PyPI <https://pypi.org/project/deform/>`_

* `Issue tracker <https://github.com/Pylons/deform/issues>`_

* `Widget examples repo <https://github.com/Pylons/deformdemo/>`_

* `Documentation <https://docs.pylonsproject.org/projects/deform/en/latest/>`_

* `Support <https://pylonsproject.org/community-support.html>`_
