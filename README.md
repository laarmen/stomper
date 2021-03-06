=======
Stomper
=======

.. contents::

:Author:
    Oisin Mulvihill

Contributors:
    Micheal Twomey, Ricky Iacovou <iacovou at gmail dot com>,
    Arfrever Frehtes <arfrever dot fta at gmail dot com>,
    Niki Pore <niki pore at gmail dot com>



Introduction
------------

This is a python client implementation of the STOMP protocol.

The client is attempting to be transport layer neutral. This module provides
functions to create and parse STOMP messages in a programmatic fashion. The
messages can be easily generated and parsed, however its up to the user to do
the sending and receiving. The STOMP protocol specification can be found here:

- `Stomp Protocol <http://stomp.codehaus.org/Protocol/>`_

I've looked at the stomp client by Jason R. Briggs. I've based some of the
'function to message' generation on how his client does it. The client can
be found at the follow address however it isn't a dependency.

- `stompy <http://www.briggs.net.nz/log/projects/stomppy>`_

In testing this library I run against ActiveMQ project. The server runs
in java, however its fairly standalone and easy to set up. The projects
page is here:

- `ActiveMQ <http://activemq.apache.org/>`_


Source Code
-----------

The code can be accessed via subversion via google project hosting. Further
details can be found here:

- `Stomper http://code.google.com/p/stomper/`_


Examples
--------

Basic Usage
~~~~~~~~~~~

To see some basic code usage example see "*example/stomper_usage.py*". The unit test
"*tests/teststomper.py*" illustrates how to use all aspects of the code.


Receive/Sender
~~~~~~~~~~~~~~

The example "*receiver.py*" and "*sender.py*" show how messages and generated and then
transmitted using the twisted framework. Other frameworks could be used instead. The
examples also demonstrate the state machine I used to determine a response to received
messages.

I've also included "*stompbuffer-rx.py*"  and "*stompbuffer-tx.py*" as examples of using
the new stompbuffer module contributed by Ricky Iacovou.


Version History
---------------

0.2.5
~~~~~

Add the contributed fix for issue #14 by Niki Pore. The issue was reported by
Roger Hoover. This removes the extra line ending which can cause problems.


0.2.4
~~~~~

OM: A minor release fixing the problem whereby uuid would be installed on python2.5+. It
is not needed after python2.4 as it comes with python. Arfrever Frehtes Taifersar Arahesis
contributed the fix for this.


0.2.3
~~~~~

OM: I've fixed  issue #9  with the example code. All messages are sent and received correctly.


0.2.2
~~~~~

- Applied patch from esteve.fernandez to resolve "Issue 4: First Message not received" in the
example code (http://code.google.com/p/stomper/issues/detail?id=4&can=1).

- I've (Oisin) updated the examples to use twisted's line receiver and got it to "detect"
complete stomp messages. The old example would not work if a large amount of data was streamed.
In this case dataReceived would be called with all the chunks of a message. This means that it
would not be correct for it to attempt to unpack and react until the whole message has been
received. Using twisted's line receiver looking for the \x00 works like a charm for this.


This release integrates the bug fixes and the optional stompbuffer contributed by Ricky
Iacovou:

- Removed the trailing '\n\n' inserted by Frame.pack(). I believe that adding this is
incorrect, for the following reasons:

http://stomp.codehaus.org/Protocol gives the example:

CONNECT
login: <username>
passcode:<passcode>

^@

and comments, "the body is empty in this case". This gives the impression that the body
is *exactly* defined as "the bytes, if any, between the '\n\n' at the end of the header
and the null byte".

This works for both binary and ASCII payloads: if I want to send a string without a
newline, I should be able to, in which case the body should look like:

this is a string without a newline^@

... and the receiver should deal with this.

This impression is reinforced by the fact that ActiveMQ will complain if you supply a
content-length header with any other byte count than that described above.

I am also unsure about the newline after the null byte as nothing in the protocol says
that there should be a newline after the null byte. Much of the code in StompBuffer
actively expects it to be there, but I suspect that *relying* on a frame ending '\x00\n'
may well limit compatibility. It's not an issue with Stomper-to-Stomper communication,
of course, as the sender puts it, the receiver accepts it, and ActiveMQ happily sends
it along.

- StompBuffer has had a few fixes; most notably, a fix that prevents a content-length "header"
in the *body* from being picked up and used (!). The biggest change is a new method,
syncBuffer(), which allows a corrupted buffer to recover from the corruption. Note that
I've never actually *seen* the buffer corruption when using Twisted, but the thought
occurred to me that a single corrupt buffer could hang the entire message handling process.

- Fixed the typo "NO_REPONSE_NEEDED". I've changed it to NO_RESPONSE_NEEDED, but kept the
old variable for backwards compatibility;

- I've also modified the string format in send() to include the '\n\n' between the header
and the body, which I think is missing (it currently has only one '\n').

- Added CONNECTED to VALID_COMMANDS so syncBuffer() does not decide these messages are bogus.

- Added new unit test file teststompbuffer which covers the new functionality.

