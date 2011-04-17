Puppet Python Module with Virtualenv, Pip and Gunicorn support
==============================================================

Module for configuring Python with virtualenvs and installation
of packages inside them with pip in addition to serving
Python WSGI applications including Django through Gunicorn.

This module support installing packages specified in a
`requirements.txt` file and update said packages when the file
changes. This way you only have to define your requirements in
one place: in the VCS for your application code.

Tested on Debian GNU/Linux 6.0 Squeeze. Patches for other
operating systems welcome.


TODO
----

* Proper Gunicorn init script with status and restart.
* Monitoring of Gunicorn process.
* Installtion of Gunicorn and setproctitle into the virtualenv.
* Uninstallation of packages no longer provided in the
  requirements file.


Installation
------------

Clone this repo to a virtualenv directory under your Puppet
modules directory:

    git clone git://github.com/uggedal/puppet-module-python.git python

If you don't have a Puppet Master you can create a manifest file
based on the notes below and run Puppet in stand-alone mode
providing the module directory you cloned this repo to:

    puppet apply --modulepath=modules test_python.pp


Usage
-----

To install Python with development dependencies simply import the
module:

    import python::dev

You can install a specific version of Python by importing the
module with this special syntax:

    class { "python::dev": version => "2.5" }

Note that classes in Puppet are singletons and not more than one
can be created even if you provide different paramters to them.
This means that the `python` class can only be used to install one
version. If you need more coexising versions you could create a new
class based on the current one prefixed with the actual version.


### Virtualenv

To install and configure virtualenv, import the module:

    import python::venv

Setting up a virtualenv is done with the `python::venv::isolate`
resource:

    python::venv::isolate { "/usr/local/venv/mediaqueri.es" }

Note that you'll need to define a global search path for the `exec`
resource to make the `python::venv::isolate` resource function
properly. This should ideally be placed in `manifests/site.pp`:

    Exec {
      path => "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    }

If you have several version of Python installed you can specifiy
which interpreter you'd like the virtualenv to contain:

    python::venv::isolate { "/usr/local/venv/mediaqueri.es":
      version => "2.5",
    }

You can also provide an owner and group which will be the owner
of the virtualenv files:

    python::venv::isolate { "/usr/local/venv/mediaqueri.es":
      owner => "www-mgr",
      group => "www-mgr",
    }

If you point to a [pip requirements file][requirements.txt] Puppet will
install the specified packages and upgrade them when the file changes:

    python::venv::isolate { "/usr/local/venv/mediaqueri.es":
      requirements => "/var/www/mediaqueri.es/requirements.txt",
    }


### Gunicorn

To use Gunicorn for serving WSGI applications you'll first have to include
its class which manages a directory for storing pid files and unix sockets
for all your Gunicorn instances:

    include python::gunicorn

If you don't want all your Gunicorn instances running as root you should
specify an unprivileged user when including the Gunicorn class:

    class { "python::gunicorn": owner => "www-mgr", group => "www-mgr" }

Serving a WSGI application is done by using the following definition type
and specifying a virtualenv, the source of your application code and
the WSGI application module:

    python::gunicorn::instance { "blog":
      venv => "/usr/local/venv/blog",
      src => "/usr/local/src/blog",
      wsgi_module => "blog:app",
    }

A Django application does not need a WSGI application module argument:

    python::gunicorn::instance { "cms":
      venv => "/usr/local/venv/cms",
      src => "/usr/local/src/cms",
      django => true,
    }


[requirements.txt]: http://www.pip-installer.org/en/latest/requirement-format.html