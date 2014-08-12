lttng-dynamic-instrumentation
=============================

Notes and documentation
#Working
1. Instrument function entry
2. Retrive function parameter that are of type int, char and char*.
3. Instrument function exit

#Not working

1. Inserting Multiple dynamic tracepoints
2. Removing dynamic tracepoints
3. Disabling dynamic tracepoints
4. Activate tracepoint only if the session has started. (now crashes if tracepoint is triggered with to session started)
5. Probe instrumentation


#TODO

#How it is done?
##Overview
![Alt text](img/lttng-di.png "High level diagram")
##Structures
struct tracepoint
struct lttng_probes_desc;
struct lttng_event_di_fied;
