# Orchestrion x Chi sampling test app

## Build

You will need [orchestrion](https://github.com/DataDog/orchestrion) to build this app.
The `orchestrion.tool.go` was kept to a minimal instrumentation with only `chi/v5` and the Datadog
tracer added at compile time.

Feel free to add more dependencies and/or fallback to all dependencies  using `orchestrion pin`.

```shell
orchestrion go build -o chi-sampling .
```

## Tracer configuration

### Sampling rules

In order to verify that the traces are dropped we need to force sampling to `0`.
We can do it at runtime via `DD_TRACE_SAMPLING_RULES` [documentation](https://docs.datadoghq.com/tracing/trace_pipeline/ingestion_mechanisms/?tab=go#in-tracing-libraries-user-defined-rules)

```shell
$ export DD_TRACE_SAMPLING_RULES='[{"resource": "GET /ready", "sample_rate": 0}, {"resource": "GET /health", "sample_rate": 0}]'
```

Those rules will force traces with the resource name `GET /ready` and `GET /health` to a 0 sampling rate.
The traces will be sent to the agent and dropped there. `GET /welcome` will be using the default agent sampling
rate (1) and kept.

### Global configuration

Make sure you have all the needed variables by the tracer to work properly.

```shell
$ export DD_API_KEY={YOUR_API_KEY}
$ export DD_ENV=chi.local
$ export DD_SERVICE=chi.orchestrion
$ export DD_TRACE_AGENT_PORT=XXXX // If the agent is running on a non-default port
```

## Run

After exporting the sampling rules env variable run the app with `DD_TRACE_DEBUG=true` in order to see if a trace is
sampled.

```shell
$ DD_TRACE_DEBUG=true ./chi-sampling
```

### cURL

#### /welcome (kept)

```shell
$ curl localhost:8181/welcome
```

Will produce the following trace logs with `_sampling_priority_v1:1`, trace is kept (`PriorityAutoKeep`)

```text
2025/06/27 11:21:18 Datadog Tracer v2.0.1 DEBUG: Started Span: dd.service=chi.orchestrion dd.env=chi.local dd.trace_id="685e628e000000000fd0c5ed899761e9" dd.span_id="1139628329731056105" dd.parent_id="0", Operation: http.request, Resource: http.request, Tags: map[component:go-chi/chi.v5 env:local.dev http.host:localhost:8181 http.method:GET http.url:http://localhost:8181/welcome http.useragent:curl/8.7.1 language:go runtime-id:89afed86-b1cc-4259-8d8a-4589ace7944b span.kind:server], map[_dd.agent_psr:1 _dd.profiling.enabled:0 _dd.top_level:1 _sampling_priority_v1:1]
2025/06/27 11:21:18 "GET http://localhost:8181/welcome HTTP/1.1" from [::1]:53546 - 200 7B in 2.833µs
2025/06/27 11:21:18 Datadog Tracer v2.0.1 DEBUG: Finished Span: dd.service=chi.orchestrion dd.env=chi.local dd.trace_id="685e628e000000000fd0c5ed899761e9" dd.span_id="1139628329731056105" dd.parent_id="0", Operation: http.request, Resource: GET /welcome, Tags: map[component:go-chi/chi.v5 env:chi.local http.host:localhost:8181 http.method:GET http.route:/welcome http.status_code:200 http.url:http://localhost:8181/welcome http.useragent:curl/8.7.1 language:go runtime-id:89afed86-b1cc-4259-8d8a-4589ace7944b span.kind:server], map[_dd.agent_psr:1 _dd.profiling.enabled:0 _dd.top_level:1 _dd.trace_span_attribute_schema:0 _sampling_priority_v1:1 process_id:92963]
```

#### /ready (kept)

```shell
$ curl localhost:8181/ready
```

Will produce the following trace logs with `_sampling_priority_v1:-1`, trace is kept (`PriorityUserReject`)

```text
2025/06/27 11:32:46 Datadog Tracer v2.0.1 DEBUG: Started Span: dd.service=chi.orchestrion dd.env=chi.local dd.trace_id="685e653e00000000473653a7fb5842aa" dd.span_id="5131380806376768170" dd.parent_id="0", Operation: http.request, Resource: http.request, Tags: map[component:go-chi/chi.v5 env:chi.local http.host:localhost:8181 http.method:GET http.url:http://localhost:8181/ready http.useragent:curl/8.7.1 language:go runtime-id:89afed86-b1cc-4259-8d8a-4589ace7944b span.kind:server], map[_dd.agent_psr:1 _dd.profiling.enabled:0 _dd.top_level:1 _sampling_priority_v1:1]
2025/06/27 11:32:46 "GET http://localhost:8181/ready HTTP/1.1" from [::1]:54258 - 200 5B in 3.75µs
2025/06/27 11:32:46 Datadog Tracer v2.0.1 DEBUG: Finished Span: dd.service=chi.orchestrion dd.env=chi.local dd.trace_id="685e653e00000000473653a7fb5842aa" dd.span_id="5131380806376768170" dd.parent_id="0", Operation: http.request, Resource: GET /ready, Tags: map[component:go-chi/chi.v5 env:chi.local http.host:localhost:8181 http.method:GET http.route:/ready http.status_code:200 http.url:http://localhost:8181/ready http.useragent:curl/8.7.1 language:go runtime-id:89afed86-b1cc-4259-8d8a-4589ace7944b span.kind:server], map[_dd.profiling.enabled:0 _dd.rule_psr:0 _dd.top_level:1 _dd.trace_span_attribute_schema:0 _sampling_priority_v1:-1 process_id:92963]
```

#### /health (kept)

```shell
$ curl localhost:8181/health
```

Will produce the following trace logs with `_sampling_priority_v1:-1`, trace is kept (`PriorityUserReject`)

```text
2025/06/27 11:33:50 Datadog Tracer v2.0.1 DEBUG: Started Span: dd.service=chi.orchestrion dd.env=chi.local dd.trace_id="685e657e000000000e38555e9ca5b6e8" dd.span_id="1024662780070180584" dd.parent_id="0", Operation: http.request, Resource: http.request, Tags: map[component:go-chi/chi.v5 env:chi.local http.host:localhost:8181 http.method:GET http.url:http://localhost:8181/health http.useragent:curl/8.7.1 language:go runtime-id:89afed86-b1cc-4259-8d8a-4589ace7944b span.kind:server], map[_dd.agent_psr:1 _dd.profiling.enabled:0 _dd.top_level:1 _sampling_priority_v1:1]
2025/06/27 11:33:50 "GET http://localhost:8181/health HTTP/1.1" from [::1]:54324 - 200 7B in 3.75µs
2025/06/27 11:33:50 Datadog Tracer v2.0.1 DEBUG: Finished Span: dd.service=chi.orchestrion dd.env=chi.local dd.trace_id="685e657e000000000e38555e9ca5b6e8" dd.span_id="1024662780070180584" dd.parent_id="0", Operation: http.request, Resource: GET /health, Tags: map[component:go-chi/chi.v5 env:chi.local http.host:localhost:8181 http.method:GET http.route:/health http.status_code:200 http.url:http://localhost:8181/health http.useragent:curl/8.7.1 language:go runtime-id:89afed86-b1cc-4259-8d8a-4589ace7944b span.kind:server], map[_dd.profiling.enabled:0 _dd.rule_psr:0 _dd.top_level:1 _dd.trace_span_attribute_schema:0 _sampling_priority_v1:-1 process_id:92963]
```
