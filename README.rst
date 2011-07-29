Breckenridge sprint 7/28 - 7/31 2011
====================================

Attendees
---------

- Casey Duncan
- Whit Morriss
- Michael Merickel
- Niall O'Higgins
- Eric Bieschke
- Chris McDonough

Topics
------

``pyramid_debugtoolbar``
~~~~~~~~~~~~~~~~~~~~~~~~

``pyramid_debugtoolbar`` is a Pyramid add on that provides a pop-out toolbar
that floats over an application while you're developing.  Its github repo is
at https://github.com/Pylons/pyramid_debugtoolbar .  The readme there is
correct as of right now.

It currently only works with the Pyramid trunk
(https://github.com/Pylons/pyramid).

It's pretty far along at this point.  Here's what could potentially be done
with it:

- Write unit tests for all panels.

- Write unit tests for exception handling stuff I stole from Werkzeug
  (there's a lot of it).

- Selenium tests.

- Currently the ordering in which the panels are displayed is the ordering in
  which the panels are processed when a request comes in.  This is not ideal
  because at least one panel "hook" (``wrap_handler``) is effectively global
  for the entire request.  This means, for example, that the display ordering
  controls whether the profile panel wraps the timing panel or whether the
  timing panel wraps the profile panel.  We should probably either decide to
  make it impossible for more than one panel to hook this hookpoint (and
  combine the timing and profiling panels into one), or come up with a
  separate ordering for invocation vs. display.

- Show response headers in the HTTP Headers panel.

- Blaise asked for review of his sqla panel.

- Currently a circref between DebugToolbar and request.  Fix or decide its
  unnecessary.

- Provide a ``debugtoolbar.exc_eval`` knob which controls the exception
  handler; display the exception but disallow eval when false.

- Fix minor formatting error when the exception console is used (literal html
  in feedback header).

- Documentation for the toolbar.

- Prevent exception console from being used except from certain IP addresses.

- Replace usage of weberror on trunk with usage of pyramid_debugtoolbar.

- View source from profiler.

Registration System
~~~~~~~~~~~~~~~~~~~

Provide some developer tools that make it easier to write registration forms
and logic.  Michael and I have talked about this.  Here's some scratchpad
pseudocode::

   # lolreg package
   # -------------------------------------------------------------
   from pyramid.interfaces import IAuthenticationPolicy
   from zope.interface import implements

   class LolRegAuthenticationPolicy(object):
       implements(IAuthenticationPolicy)
       def __init__(self, backend):
           self.backend = backend

       def authenticated_userid(self, request):
           # use self.backend to figure out who the guy is and if he exists
           pass

       # ..  other IAuthenticationPolicy methods ...
       
   def includeme(config):
       settings = config.settings
       backend_factory = settings.get('lolreg.backend_factory',
                                      'lolreg.sqla.SQLARegistrationBackend')
       backend_factory = config.maybe_dotted(backend_factory)
       backend = backend_factory(settings)
       config.add_route('lolreg.register', '/register', factory=backend)
       config.add_route('lolreg.activate', '/activate', factory=backend)
       config.set_authentication_policy(LolRegAuthenticationPolicy(backend))

   # lolreg.sqla package

   from sqlalchemy import Column
   from sqlalchemy import Integer
   from sqlalchemy import Unicode

   from sqlalchemy.ext.declarative import declarative_base

   from sqlalchemy.orm import scoped_session
   from sqlalchemy.orm import sessionmaker

   from zope.sqlalchemy import ZopeTransactionExtension
   from zope.interface import implements

   from lolreg.interfaces import IRegistrationBackend

   DBSession = scoped_session(sessionmaker(extension=ZopeTransactionExtension()))
   Base = declarative_base()

   from sqlalchemy import engine_from_config

   class User(Base):
       __tablename__ = 'users'
       id = Column(Integer, primary_key=True)
       name = Column(Unicode(255), unique=True)
       # etc

   class Group(Base):
       __tablename__ = 'groups'
       id = Column(Integer, primary_key=True)
       name = Column(Unicode(255), unique=True)
       # etc

   class SQLARegistrationBackend(object):
       implements(IRegistrationBackend)
       def __init__(self, settings):
           engine = engine_from_config(settings, 'lol.sqlalchemy.')
           DBSession.configure(bind=engine)
           Base.metadata.bind = engine
           Base.metadata.create_all(engine)
           
       def add_user(self, **kw):
           session = DBSession()
           user = User(**kw)
           session.add(user)

       def add_group(self, whatever):
           # whatever
           pass

       def activate(self, token):
           # whatever
           pass

   # lolreg.views package
   # -------------------------------------------------------------

   from pyramid.view import view_config

   @view_config(route_name='lolreg.register')
   def register_form(request):
       pass

   @view_config(route_name='lolreg.activate')
   def activate(request):
       request.context.activate(request.POST['token'])

   # user app
   # -------------------------------------------------------------

   from pyramid.config import Configurator
   from pyramid.view import view_config

   @view_config(route_name='lolreg.register')
   def my_register_form(request):
       # self-posting form
       pass

   @view_config(route_name='lolreg.activate')
   def my_activate(request):
       pass

   if __name__ == '__main__':
       config = Configurator()
       config.include('lolreg', route_prefix='/registration')

       # accept default views
       config.scan('lolreg.views')

       # or use your own views

       # config.scan('__main__')

       # or use default views then customize some

       # config.scan('lolreg.views')
       # config.commit()
       # config.add_view(my_register_form, route_name='lolreg.register')

MongoDB ACL/Collection Stuff
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Niall wants to make scaffolding to do ACL+collection stuff easier when used
with MongoDB.

Route-Prefixed Includes
~~~~~~~~~~~~~~~~~~~~~~~

Merge and document https://github.com/Pylons/pyramid/pull/222

Route Groups
~~~~~~~~~~~~

Michael has some code which implements "route groups".  We could try to give
that code a roll and see what Michael wants to do with it.

This also likely implies a change to ``pyramid_handlers``.

Random Pyramid Tasks
~~~~~~~~~~~~~~~~~~~~

The Pyramid TODO.txt at
https://github.com/Pylons/pyramid/blob/master/TODO.txt contains (under
"Should-Have") the list of features that will probably make it into 1.1.1.
Any of these is fair game.
