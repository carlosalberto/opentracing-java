# OpenTracing-Java 0.30 compatibility layer.

The `opentracing-v030` artifact provides a 0.30 API compatibility layer which comprises:
1. Exposing all the the 0.30 packages under `io.opentracing.v_030` (`io.opentracin.v_030.propagation`, `io.opentracing.v_30.util`, etc).
2. A Shim layer to wrap 0.31 Tracer and expose it under the 0.30 API.

## Shim Layer

The basic shim layer is exposed through `TracerShim`, which wraps a `io.opentracing.Tracer` object and exposes it under the `io.opentracing.v_030.Tracer` interface:

```java
import io.opentracing.v_030.ActiveSpan;
import io.opentracing.v_030.Tracer;
import io.opentracing.v_030.shim.TracerShim;

Tracer tracer = new TracerShim(yourUpstreamTracer);
try (ActiveSpan span = tracer.buildSpan("parent").startActive()) {
}
```

`TracerShim` does not support `ActiveSpan.capture()` and hence there's no `ActiveSpan.Continuation` support. See details in how to support it below.

## Reference counting and Continuation support

For supporting `ActiveSpan.capture()` and `Continuation`s the `AutoFinishTracerShim` class must be used instead, in conjuction with using `io.opentracing.util.AutoFinishScopeManager` as `Tracer.scopeManager()` (which preserves the reference-count system used in 0.30).

```java
import io.opentracing.v_030.ActiveSpan;
import io.opentracing.v_030.Tracer;
import io.opentracing.v_030.shim.AutoFinishTracerShim;

Tracer tracer = new AutoFinishTracerShim(yourUpstreamTracer);
try (ActiveSpan span = tracer.buildSpan("parent").startActive()) {
    ActiveSpan.Continuation cont = span.capture();
    ...
}
```
