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

Here's what could potentially be done with it:

- Write unit tests for all panels.

- Write unit tests for debugger stuff I stole from Werkzeug (there's a lot of
  it).

- Currently the ordering in which the panels are displayed is the ordering in
  which the panels are processed when a request comes in.  This is not ideal
  because at least one panel "hook" (``wrap_handler``) is effectively global
  for the entire request, and currently the display ordering controls whether
  the profile panel wraps the timing panel or whether the timing panel wraps
  the profile panel.  We should probably either decide to make it impossible
  for more than one panel to hook this hookpoint (and combine the timing and
  profiling panels into one), or come up with a separate ordering for
  invocation vs. display.

- Show response headers in the HTTP Headers panel.

- Blaise asked for review of his sqla panel.

