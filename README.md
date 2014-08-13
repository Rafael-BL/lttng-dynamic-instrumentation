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

##Dyninst
Dyninst is a powerful library that can retreive information on, modify address space, and infect code in a running process. This prototype uses dyninst to copy data structures in the tracee address space and to add calls to functions for the tracepoint registration and event recording.

website: http://www.dyninst.org/

git: http://git.dyninst.org/ 

##Structures
N.B. On the following explanation I will use a tracepoint named prov:name and has one integer(32 bits) field and char(8 bits) field.

In order to register a tracepoint at least four structures must be constructed. 
The tracepoint structure describes its name and signature. Only those two fields are used in this implementation of dynamic instrumentation. The name and signature fields which are respectively prov:name and name in our example.

<pre>
struct tracepoint {
	const char *name;
	int state;
	struct tracepoint_probe *probes;
	int *tracepoint_provider_ref;
	const char *signature;
	char padding[TRACEPOINT_PADDING];
};
</pre>

The following structure describes the tracepoints related to a provider. The "provider" field in this case would be set to "prov". The nr_events is set to the number of events for this provider. The field event_desc points to an array of nr_events event descriptions.

<pre>
struct lttng_probe_desc {
	const char *provider;
	const struct lttng_event_desc **event_desc;
	unsigned int nr_events;
	struct cds_list_head head;		/* chain registered probes */
	struct cds_list_head lazy_init_head;
	int lazy;				/* lazy registration */
	uint32_t major;
	uint32_t minor;
	enum lttng_probe_type type;
	char padding[LTTNG_UST_PROBE_DESC_PADDING];
};
</pre>

This structure represents a tracepoint. At the moment, we are interested in its name, its number of field and to its array of field descriptions.
<pre>
struct lttng_event_desc {
	const char *name;
	void (*probe_callback)(void);
	const struct lttng_event_ctx *ctx;	/* context */
	const struct lttng_event_field *fields;	/* event payload */
	unsigned int nr_fields;
	const int **loglevel;
	const char *signature;	/* Argument types/names received */
	union {
		struct {
			const char **model_emf_uri;
		} ext;
		char padding[LTTNG_UST_EVENT_DESC_PADDING];
	} u;
};
</pre>
This structure describe a field. In this implementation, the name and the type are useful. In our exmaple, we would have a field of type 32 bits and 8 bits integer.
<pre>
struct lttng_event_di_field {
	const char *name;
	struct lttng_type type;
	unsigned int nowrite;	/* do not write into trace */
	char padding[LTTNG_UST_EVENT_FIELD_PADDING];
};
</pre>

##Tracepoint registration
###Pseudo-code
<pre> liblttng-ust.so
void addIntLen(){
	event_len += sizeof(int);
}

void addCharLen(){
	event_len += sizeof(char);
}

void InitCtx(){
	//We need event lenght to reserve
	//space in the context ringbuffer
	if(isTpRegistered){
		init(ctx, event_len);
		IsCtxReady = true;
	}
}

void writeIntField(int value){
	if(!isCtxReady)
		return;
	write(ctx, &value);	
}
void writeCharField(char value){
	if(!isCtxReady)
		return;
	write(ctx, &value);	
}

void commitEvent(){
	if(!isCtxReady)
		return;
	commit(ctx);
}

registerTP(){}

registerProbe(){}

completeRegistration(){
	isTpRegistered = true;
}

lttng_ust_fake_function{
	registerTP();
	registerProbe();
	completeRegistration();
}

</pre>

The following pseudocode makes it look like function calls are added inside the target function. I doubt that it's how Dyninst really does it but it's simpler for my explanations.

<pre>
bool isTpRegistered = false;
bool isCtxReady = false;
int event_len = 0;
Context ctx;

void interestingFunction(int a, char b){
	addIntLen();
	addCharLen();
	InitCtx();
	writeIntField(a);
	writeCharField(b);
	commitEvent();
	//... original function
}
</pre>

##Overview
![Alt text](img/lttng-di.png "High level diagram")



