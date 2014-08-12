lttng-dynamic-instrumentation
=============================

Notes and documentation
#Working
1. Instrument function entry
2. Retrieve function parameter type
3. Retrieve function parameter name
4. Retrieve function parameter value
5. Instrument function exit

#Not working

1. Inserting Multiple dynamic tracepoints
2. Removing dynamic tracepoints
3. Disabling dynamic tracepoints
4. Probe instrumentation

#Known bugs
1. Trigerring tracepoint if the session has not started.

#TODO

#How it is done?
##Overview
![Alt text](img/lttng-di.png "High level diagram")
##Structures
struct tracepoint
struct lttng_probes_desc;
struct lttng_event_di_fied;
