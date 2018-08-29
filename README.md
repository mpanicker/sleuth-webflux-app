I modified the code to work with Brave(that is part of Spring Cloud 2). Original code did not compile

I also could not get the vagrant zipkin/ELK to run so i just started a zipkin docker image and changed the bootstrap 
to point to the server
Original code is at https://github.com/marcingrzejszczak/sleuth-webflux-app


= S1P Sleuth Demo

This repository contains a demo that demonstrates various less known
features of Spring Cloud Sleuth.

== How to use it

This demo repository is part of a bigger demo system. To get started
just initialize the submodules.

```bash
$ git submodules init
$ git submodules update
```

Now you can start the whole system (ELK, 4 demo apps, Zipkin) by running

```bash
$ pushd vagrant-elk-box
$ ./getReadyForConference.sh
$ popd vagrant-elk-box
```

Now, wait for some time for the Vagrant box to start, demo apps to start,
Zipkin to start and some requests get sent.

You're ready to run this application. Just start it and curl

```bash
$ curl --header "X-B3-Flags:1" localhost:8765/s1p
```

or using httpie

```bash
$ http localhost:8765/s1p X-B3-Flags:1
```

== What do we show in this app

=== Dalston

- Creating spans via annotations
* especially useful for interfaces (e.g. Spring Data)
- Baggage
* Sleuth specific (via baggage_ header prefix)
- Span adjusters
* list of custom adjusters of spans before reporting

=== Edgware

- RestTemplateBuilder support
* the built RestTemplate is tracing aware
- Zipkin load balanced/discovery using Eureka
* pass the service id in the URL
- Reactor Support
* tracing context passed through Flux
- Zipkin2 Support
* `starter-zipkin` now points to Zipkin2
- Deprecation of Stream server
* migrate to Zipkin2 with rabbit

=== Finchley

- WebFlux & WebClient support
* tracing context passed through the Flux
* WebClient is tracing aware
- Removal of Stream server
* you should use Zipkin 2 native messaging support
* migrate by having both the old Stream server and the new Zipkin 2 store
spans to the same storage
* start migrating apps from Sleuth Stream client to Zipkin 2
- Removal of OAuth2 support
* will be removed once it’s re-added in Security

== Demo Script

- Create a project from start.spring.io (Reactive Web, Zipkin, Lombok)
- Assuming that the `vagrant-elk-box` gets updated to Edgware for the
samples, need to update zipkin dependency to zipkin2 (assuming that
I don’t update initlizr) `[Edgware]`
- Write a `s1p` `Controller` returning `Mono<String>` `[Finchley]`
- Create a `MyService` class that will call `service1` via `WebClient` `[Finchley]`
- Add the `WebClient.Builder` bean - explain that `RestTemplate`
/ `RestTemplateBuilder` can be used `[Finchley / Edgware]`
- Set `spring.application.name` and `spring.sleuth.sampler.percentage=1.0`
in properties
- Run the app and see if things pop up in Zipkin
- Set baggage `baggage` -> `s1p` in controller and run again - show
how baggage gets picked in service1 `[Dalston]`
- Create 2 methods in `MyService`, one `@NewSpan(“surprise”)` the
other `@ContinueSpan(log = “very_important_method”), both with `@SpanTag`
method params. Methods should sleep and log some stuff out.  `[Dalston]`
- Run again, show new spans. Show the baggage logs, show the annotation logs
- Say that you don’t like the span names. You’d like the span that has
`http.path` to have a changed name. Create a `SpanAdjuster` Bean that
will check `http.path` tag to contain one with path `/s1p` and change
its name to `hacked!`. Rerun the application `[Dalston]`
- Change sampler percentage to `0.0` and show that spans are not sampled.
Now call `http localhost:8080/s1p X-B3-Flags:1` and show that the
debug mode works fine - the span is sampled `[Dalston]`
- Create a `Foo` class with `Uuid` field, change `@TagValue` to contain a
resolver on one of the methods. Create a bean
`TagValueResolver myCustomTagValueResolver` that will return
foo’s uuid or any object’s to string `[Dalston]`
- Show how you could find Zipkin from Service Discovery
`spring.zipkin.baseUrl: http://zipkinserver/` `[Edgware]`
- Add the `@EnableAsync` and `@Async` method on the `MyService`.
Add the `@SpanName` on the method and add some logging.
Until Edgware the method’s name was the span name - now it has changed `[Edgware]`
- Calling an external, unmanaged service. In `MyService` add
the `callPivotal` method. Create a span manually [Dalston]
Showing how to call a peer service. Create a `RestTemplateBuilder` bean `[Edgware]`,
that will point to `https://pivotal.io`. Build the `RestTemplate` in MyService
constructor. New method in `MyService` with `@NewSpan`. Add a tag `lc` => `Pivotal`.
Set `CS` before the call `try {} finally {}` over `RestTemplate` call. In `finally`
set `peer.service` to `pivotal`, `peer.ipv4` e.g.
`InetAddress.getByName("pivotal.io").getHostAddress() `,
`"peer.port", "80"`. Finally log event `CR`. Re-run and show the new pivotal
service `[Dalston]`