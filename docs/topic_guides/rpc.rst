RPC
===

Overview
--------

Synchronous communication with lymph services is realised through RPC. 
RPC messages are sent via ØMQ. If a RPC call fails, it is the responsibility of 
the calling code to deal with it.


Registering methods as RPC callable
-----------------------------------

Any class inheriting from :class:`lymph.Interface` can receive RPC calls. By specifying the
``name`` argument when initializing the class, the lymph service will be reachable through its
interface name ``name``.

By default the service is registered under the name given when you configure
the service.

.. code::

    import lymph

    class EchoService(lymph.Interface):
        pass


.. code-block:: yaml

    interfaces:
        echo:
            class: project.interfaces:EchoService

will be reachable with the service name ``echo``. This is the name with which lymph knows that
the RPC messages should be sent to ``EchoService``.

In order to make a method in a lymph interface class RPC callable, it is sufficient to
add the :func:`@lymph.rpc()` (or :func:`@lymph.raw_rpc` for accessing the channel object) decorator in
front of it.

.. decorator:: rpc()

    Marks the decorated interface method as an RPC method.
    
    .. code::
    
        import lymph
    
        class Example(lymph.Interface):
            @lymph.raw_rpc()
            def do_ack(self, channel, message):
                """
                HERE SOME FANCE HELP TEXT
                """
                assert isinstance(channel, lymph.core.channels.ReplyChannel)
                assert isinstance(message, lymph.core.messages.Message)
                channel.ack()

           @lymph.rpc()
           def echo(self, message):
               return message

If a docstring is specified after the RPC method definition, it will be used as a description
of the service and will be returned by ``lymph inspect``.

Difference between lymph.rpc and lymph.raw_rpc
----------------------------------------------

``lymph.rpc``
^^^^^^^^^^^^^

The :func:`lymph.rpc` decorator is easier to understand compared to :func:`lymph.raw_rpc`
since the former work as any Python function where what ever the RPC function return will be sent
to the caller, as for exceptions there is two cases depending on the ``raises`` argument of
:func:`lymph.rpc` :

- If the exception raised inside the RPC function is an instance of a class that is part of the
  ``raises`` argument then the client will see a :exc:`RemoteError`.
- Else the result will be a **NACK**.


``lymph.raw_rpc``
^^^^^^^^^^^^^^^^^

When :func:`lymph.raw_rpc` is used the underlying method call has to have the following form:

.. code::

    def some_rpc_method(self, channel, **kwargs):
        …

The ``channel`` argument takes a :class:`lymph.ReplyChannel` object which takes care of the communication
from and to the RPC caller. From within the responding method, you communicate through the ``channel``
object with the calling party. The ``ReplyChannel`` object provides you with the following methods:

.. method:: reply(body)

    :param body: reply

    sends ``body`` as a reply back to the caller

    .. code-block:: python

        import lymph

        class EchoService(lymph.Interface):

            @lymph.raw_rpc()
            def echo(self, channel, text=None):
                channel.reply(text)


.. method:: ack(unless_reply_sent=False)

    :param unless_reply_sent: only send the acknowledgment if a reply has already been sent

    sends an acknowledgment to the caller.

.. method:: nack(unless_reply_sent=False)

    :param unless_reply_sent: only send the non-acknowledgment if a reply has already been sent

    sends a non-acknowledgment to the caller.

.. method:: error(body)

    :param body: error

    sends an error to the caller.


Sending RPC calls
-----------------

In order to send RPC calls from within lymph services, you need to pass the call through
the ``proxy`` class. You can obtain the system's proxy by calling the ``proxy`` method:

.. method:: proxy(address)

    returns a proxy object that can be used to conveniently send requests to
    another service.

    .. code-block:: python

        echo = self.proxy('echo')
        result = echo.upper(text='foo')
        assert result == 'FOO'

    This is equivalent to ``self.request('echo', 'echo.upper', text='foo')``.

The proxy object proxies any method that is called in the proxy class, into a corresponding
RPC call. It does not however make sure, that the RPC call actually exists. It will send the
call regardless of availability and timeout accordingly if no response is obtained.

Any value that is returned by the RPC call is also returned by the call to the corresponding
proxy method. In the example above, the service with the name ``echo`` provides the ``upper(text)``
endpoint. By calling the corresponding proxy method in the proxy object, the payload 
``text='foo'`` is sent to the endpoint and its result returned and saved in the ``result``
variable.

RPC calls are synchronous, i.e. program execution is halted until the RPC call returns an
answer or it times out. If you require asynchronous communication, please refer to 
:doc:`events`.


Deferred RPC calls
------------------

By default, RPC blocks until the response is received. A deferred RPC call mechanism
is available if you wish to consume the RPC response later, or simply ingore
it.

The call interface is similar to making a regular RPC call, with the addition of
adding .defer call after it.

In that case, the call will return a Future (the actual implementation is a 
gevent AsyncResult which will block only when it's .get method is called.

For instance:

    .. code-block:: python

        echo = self.proxy('echo')
        result_future = echo.upper.defer(text='foo')
        # do other stuff
        result = result_future.get()
        assert result == 'FOO'


