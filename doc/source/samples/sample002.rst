.. BACpypes tutorial lesson 1

Sample 2 - Who-Is/I-Am Counter
==============================

This sample application builds on the first sample by overriding the default 
processing for Who-Is and I-Am requests, counting them, then continuing on
with the regular processing.

The description of this sample will be about the parts that are different from
sample 1.

Counters
--------

Python has a excellent *defaultdict* datatype from the *collections* module
that is perfect for this application.  It is very easy to use::

    >>> from collections import defaultdict
    >>> x = defaultdict(int)

The essential idea is that you can treat some key as having a default value
if it doesn't exist, so rather than doing this::

    >>> x['a'] = x.get('a', 0) + 1

You can do this::

    >>> x['a'] += 1

Processing Service Requests
---------------------------

When an instance of the :class:`app.Application` receives a request it attempts
to look up a function based on the message.  So when a WhoIsRequest APDU is
received, there should be a do_WhoIsRequest function.

The beginning is going to be standard boiler plate function header::

    def do_WhoIsRequest(self, apdu):
        """Respond to a Who-Is request."""
        if _debug: SampleApplication._debug("do_WhoIsRequest %r", apdu)

The middle is going to process the data in the request::

        # build a key from the source and parameters
        key = (str(apdu.pduSource),
            apdu.deviceInstanceRangeLowLimit,
            apdu.deviceInstanceRangeHighLimit,
            )

        # count the times this has been received
        whoIsCounter[key] += 1

And the end of the function is going to call back to the standard application
processing::

        # pass back to the default implementation
        BIPSimpleApplication.do_WhoIsRequest(self, apdu)

The do_IAmRequest function is similer::

    def do_IAmRequest(self, apdu):
        """Given an I-Am request, cache it."""
        if _debug: SampleApplication._debug("do_IAmRequest %r", apdu)

It uses a diferent key, but counts them the same::

        # build a key from the source, just use the instance number
        key = (str(apdu.pduSource),
            apdu.iAmDeviceIdentifier[1],
            )

        # count the times this has been received
        iAmCounter[key] += 1

And has an identical call to the base class::

        # pass back to the default implementation
        BIPSimpleApplication.do_IAmRequest(self, apdu)

Printing Results
----------------

By building the key out of elements in a useful order, it is simple enough
to sort the dictionary items and print them out, and being able to unpack
the key in the for loop is a nice feature of Python::

    print "----- Who Is -----"
    for (src, lowlim, hilim), count in sorted(whoIsCounter.items()):
        print "%-20s %8s %8s %4d" % (src, lowlim, hilim, count)
    print

Pairing up the requests and responses can be a useful excersize, but in most
cases the I-Am response from a device will be a unicast message directly back
to the requestor, so relying on broadcast traffic to analyze device and 
address binding is not as useful as it used to be.
