# Named Tracers 

_Creating tracers using a factory mechanism and naming those tracers in accordance with the library that provides the instrumentation for a traced component._

## Suggested reading

* [Proposal: Tracer Components](https://github.com/open-telemetry/opentelemetry-specification/issues/10)
* [Global Instance discussions](https://github.com/open-telemetry/opentelemetry-specification/labels/global%20instance)

## Motivation

Instead of having a global tracer singleton, every instrumentation library needs to create its own tracer, which also has a proper name that identifies the library / logical component that uses this tracer. If there are two different instrumentation libraries for the same technology (e.g. MongoDb), these instrumentation libraries should have distinct names.

This proposal originates from an `opentelemetry-specification` proposal on [components](https://github.com/open-telemetry/opentelemetry-specification/issues/10), but the term "components" was quickly found to being too generic and possibly misleading, since it is already used frequently in differing contexts and connotations.

The purpose of _Named Tracers_ is the ability to allow for a more fine-grained control of the amount of tracing data being generated by a library or application. Associating a name (e.g. _"io.opentelemetry.contrib.mongodb"_) with a tracer allows for a logical coupling of this tracer and a certain technology or library (like _MongoDB_ in this example).

By using *Named Tracers* a library can enable / disable tracing of a specific library, which might be necessary in case:
* A library causes too much runtime or memory overhead.
* A library has issues or bugs (e.g. does not call `span.end()` in every case).
* "Double-Tracing" becomes a problem. This can be the case if a vendor (e.g. Dynatrace) might already have built-in auto-instrumentation for a certain library. In this case the vendor might want to decide (automatically or configuration-based) which of the implementations should be enabled or disabled.


## Explanation

From a user perspective, working with *Named Tracers* and *TracerFactories* is very similar to how logging frameworks like [log4j](https://www.slf4j.org/apidocs/org/slf4j/LoggerFactory.html) work. In analogy to requesting Logger objects through LoggerFactories, a tracing library would create specific 'Tracer' objects through a 'TracerFactory'.

```java
Tracer tracer = OpenTelemetry.Tracing.getTracerFactory().getTracer("io.opentelemetry.contrib.mongodb");
```

In a way, the `TracerFactory` replaces the global `Tracer` singleton object as a ubiquitous point to request a tracer instance.


## Internal details

By providing a TracerFactory and *Named Tracers*, a vendor or OpenTelemetry implementation gains more flexibility in providing tracers and can decide whether or not to produce spans for a specific traced component and even change this behavior at runtime.

In the simplest case, an OpenTelemetry implementation can return a single instance for a requested tracer regardless of the name specified. This could be the case for implementations that do no want/need to enable or disable a tracer.

Alternatively, an implementation can provide different tracers per specified tracer name, thus being able to associate this tracer with the component being traced. This allows for the possibility to enable / disable a tracer based on a custom configuration.
* Automatically set the `component` ("the component being traced") on every span being produced.

## Trade-offs and mitigations

## Prior art and alternatives

Alternatively, instead of having a `TracerFactory`, existing (global) tracers could return and additional indirection objects (called e.g. `TraceComponent`), which would be able to produce spans for specifically named traced components.

```java
  TraceComponent traceComponent = OpenTelemetry.Tracing.getTracer().componentBuilder("io.opentelemetry.contrib.mongodb");
  Span span = traceComponent.spanBuilder("someMethod").startSpan();
```

Overall, this would not change a lot since the levels of indirection until producing an actual span are the same.


## Open questions

## Future possibilities

By adapting this proposal, current implementations that do not honour the specified tracer name and provide a single global tracer, do not required much change. However they could change that behavior in future versions and provide more specific tracer implementations then. On the other side, if the mechanism of *Named Tracer* is not a part of the initial specification, such scenarios will be prevented and hard to retrofit in future version, should they be deemed necessary then.