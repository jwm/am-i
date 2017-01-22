I got tired of having to enable `vacation(1)` manually when I go on
vacation. I already track when I'm not working using Google Calendar,
so I wrote this program to automatically check whether I'm on vacation
and invoke a subprogram (in my case, `vacation(1)`) when I am. Instead of
executing a subprogram, it can also use its exitstatus to signal whether
a given condition currently applies, based on Google Calendar events.

It's generic enough to be useful in a wide variety of cases. If you track
events in a Google Calendar and want to determine whether a type of event
applies right now, it should be easy to configure this program to work
for you.

It tries to be intelligent about weekends and workday hours. For example,
you can configure it (using the day_start/day_end/weekend items) so that
if it's Friday and you're on vacation (the 'vacation' condition applies),
it will use a return date of the following Monday.

It's not the prettiest thing I've ever written and the packaging isn't
Pythonic, but it works, and it's a decent example of using the Google
Calendar v3 API.


Installing
==========

- `pip install google-api-python-client`
- Edit the global configuration variables at the top of `am-i`.
- Follow Step 1 of the [Google Calendar API Python
  Quickstart](https://developers.google.com/google-apps/calendar/quickstart/python)
  to enable the Calendar API for your Google Account.
- `am-i init` to set up OAuth. If you're running `am-i` on a remote host,
  add `--noauth_local_webserver` to emit a URL you can copy and paste
  into a local web browser.


Useful invocations
==================

- Automatically detect when you're on vacation and invoke the `vacation(1)`
  utility to send an autoreply.

      am-i vacation jwm -- vacation -m @MSG_FILE@ jwm

- Determine whether you're on call. Use this as a conditional in your
  mail filteirng rules to determine, for example, whether to forward
  Nagios alerts to you via SMS only when you're on call.

      am-i oncall jwm


Message template variables
==========================

This program can expand certain variables in a message template.
- $RETURN_ON
  The date when the specified condition is no longer met. For example,
  when calendar events indicate you're no longer on vacation. The
  format of this date may be overridden on the command line.
- $RETURN_ON_ROMAN
  $RETURN_ON, expressed in Roman numerals. This variable is not subject
  to the specified date format.


Subprogram variables
====================

This program can pass variables as arguments to subprograms.
- @MSG_FILE@
  If this variable is present in the subprogram's arguments, variables in
  the specified message template will be expanded. When expansion is
  complete, the resulting message is written to a temporary file. This
  variable is replaced by the filename of the temporary file.
- @RETURN_ON@
  The date when the specified condition is no longer met. For example,
  when calendar events indicate you're no longer on vacation. The
  format of this date may be overridden on the command line.


Exit status
===========

- If a subprogram *is* specified: exits nonzero if the subprogram
  invocation failed or an error was encountered in determining
  whether the specified condition applies. Otherwise, exits
  successfully (exitstatus 0). This is useful for invoking this
  program from a mail filter (such as procmail or maildrop) to
  invoke `vacation(1)` when on vacation.
- If a subprogram is *not* specified: exits successfully (exit status 0)
  if the specified condition applies. Exits with status 1 if the condition
  does not apply. Exits with any other exit status if an error was
  encountered. This makes the program useful as a condition in shell
  scripts or mail filtering recipes.
