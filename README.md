
# REMReM Client Library

This library provides implementation of client side of [REMReM Generate](https://github.com/eiffel-community/eiffel-remrem-generate) and [REMReM Publish](https://github.com/eiffel-community/eiffel-remrem-publish) REST APIs. Its main purpose is to hide the underlying REST API calls and enable user to focus primarily on [Eiffel events](https://github.com/eiffel-community/eiffel). They can be handled in both forms:
* JSON objects defined by [schemas](https://github.com/eiffel-community/eiffel/tree/master/schemas) and
* Java objects defined by corresponding [classes](https://github.com/eiffel-community/eiffel-remrem-semantics/tree/master/src/main/java/com/ericsson/eiffel/semantics/events).

## Goals
* Simple setup,
* Single client for both Generate and Publish services,
* Easy error-handling,
* ...

# Requirements
The library is implemented as a pure Java library. It was delivered under JDK 1.8 and should work in all later releases. Also Maven 3.6 was used as a primary release.

## Download
Maven

    <dependency>
      <groupId>com.ericsson.eiffel</groupId>
      <artifactId>eiffel-remrem-client</artifactId>
      <version>0.1.1</version>
    </dependency>


# Usage

## Creating REMReM Client Instance
`RemremClientBuilder` is used to create a new instance of `RemremClient`.
The concept of builder will be described later, in section [Builder](builder).

    RemremClient client = RemremClient.builder()
        .setUrl("https://localhost:8443")  
        .setAuthentication("eiffel", "publicly-unknow")  
        .build();

## Basic operations
As mentioned earlier, REMReM Client Library supports operations provided by both
Generate and Publish services. The list of available operations is given by 
methods declared by [`RemremClient`](src/main/java/com/ericsson/eiffel/remrem/RemremClient.java)
interface. Performing an operation of a REMReM service is as simple as calling
a method.

For example to obtain list of available Eiffel event types method `eventTypes()`
is invoked. It returns a collection of event types in form of `String`s.

    Collection<String> types = client.eventTypes();

Similarly other methods can be invoked. As an example the following piece of 
code publishes an Eiffel event.

    String jsonEvent = "{
        \"meta\": {
        \"id\": \"1e5996dd-7ec0-4cbf-a3d6-d18bf2df915a\",
        \"type\": \"EiffelActivityCanceledEvent\",
        \"version\": \"3.0.0\",
       ...
    }";
    EventPublishResponse[] response = client.publish(jsonEvent);

Sometimes it's easier to work with Java objects instead of JSON strings. The
same operation can be performed providing corresponding instance of `Event` object.

    Event event = /* event initialization */;
    EventPublishResponse response = client.publish(event);


## `AutoCloseable`
`RemremClient` implements `AutoCloseable` interface and thanks to that it can 
utilize `try` section which ensures that `close()` method of the object is
automatically invoked when leaving the section.

    try (RemremClient client = RemremClient.builder()
            .setUrl("https://localhost:8443")  
            .setAuthentication("eiffel", "publicly-unknow")  
            .build()) {

        // Do something useful here
    }


## Error Handling
Any operation toward REMReM services can fail anytime due to a network issue, lack
of resources, unavailability of other services, etc. That was addressed by
the library design and hierarchy of exceptions was created to indicate that
something went wrong.

    Exception
        EiffelException
            RemremException
                RemremResponseException
                RemremPublishException

Most of `RemremClient` methods throw `RemremException`.
Note that `RemremException` may be used as a wrapper of other exceptions, e.g.
`IOException`.
The idea behind that is to simplify exceptions handling, because some methods
would declare several exceptions in `throws` clause and all the declared
exceptions would have to be handled.

    try (RemremClient client = RemremClient.builder()
            .setUrl("https://localhost:8443")  
            .setAuthentication("eiffel", "publicly-unknow")  
            .build()) {
        String template = client.template("EiffelActivityCanceledEvent");
        ...
    }
    catch (RemremException e) {
        log.error("Cannot generate template: " + e.getMessage());
    }

## Builder
Builder pattern is used to create instances of `RemremClient`.

### Default Builder
A default, build-in, builder is provided by method `RemremClient.builder()` and 
is implemented by class `RemremClientBuilder`.

It supports all the features and options provided by
[Eiffel REMReM Generate](https://github.com/eiffel-community/eiffel-remrem-generate)
and
[Eiffel REMReM Publish](https://github.com/eiffel-community/eiffel-remrem-publish) 
services.
The builder defines methods for the most commonly used features and options, e.g.
`setUrl()`, `setAuthentication`, etc.

    RemremClient.builder()
        .setUrl("https://localhost:8443")  
        .setAuthentication("eiffel", "publicly-unknow")
        .build();

However, not all the features have corresponding method defined.
Each feature can be set or get using method `setFeature()`, `getFeature()`,
respectively. For example, the above piece of code is equivalent to the code below.

    RemremClient.builder()
        .setFeature(FEATURE_URL, "https://localhost:8443")
        .setFeature(FEATURE_USERNAME, "eiffel")
        .setFeature(FEATURE_PASSWORD, "publicly-unkonw")
        .build();

All features are defined as constants---starting by 
`FEATURE_`---by class `RemremClientBuilder`. 

* `FEATURE_URL`                URL of REMReM service.
* `FEATURE_USERNAME`           Username (for authentication).
* `FEATURE_PASSWORD`           Password (for authentication).
* `FEATURE_MESSAGE_PROTOCOL`   Message protocol to request the REMReM services. 
                               Default value is `eiffelsemantics`.
* `FEATURE_MESSAGE_TYPE`       Event type to generate/publish.
* `FEATURE_INSECURE`           If the value is `true` the connection ignores
                               TLS handshake. This is useful to communicate with
                               a local service, where certificates are not installed.
* `FEATURE_RETRY_COUNT`        Number of retries.
* `FEATURE_USER_DOMAIN`        TODO: STRANGE... from which domain routingkey is prepared (Examples eiffel920, eiffel921,...)
* `FEATURE_TAG`                Reserved for future. Value `notag` is used.
* `FEATURE_ROUTING_KEY`        A message attribute used to route the messages to queue.
* `FEATURE_FAIL_IF_MULTIPLE_FOUND`  If set to `true` and multiple event IDs are found through any of the provided lookup definitions, then no event will be generated. Default value is `false`.
* `FEATURE_FAIL_IF_NONE_FOUND`      If set to `true` and no event ID is found through (at least one of) the provided lookup definitions, then no event will be generated. Default value is `false`.
* `FEATURE_LOOKUP_IN_EXTERNAL_ERS`  If set to `true` then REMReM will query external ERs and not just the locally used ER. The reason for the default value to be `false` is to decrease the load on external ERs. Here local ER means single ER which is using REMReM generate. External ER means multiple ER's which are configured in local ER. Default value is `false`.
* `FEATURE_LOOKUP_LIMIt`            The number of events returned, through any lookup definition given, is limited to this number. Default value is `1`.
* `FEATURE_OK_TO_LEAVE_OUT_INVALID_OPTIONAL_FIELDS`   If set to `true` it removes the optional event fields from the input event data that does not validate successfully.
* `parseData` TODO: ???

### Custom Builder
The library supports customization of builder and created `RemremClient`s.
Overloaded variant of `builder()` method supports that.

    RemremClientBuilder builder(String builderClassName, ClassLoader classLoader)

Utilization of the method is quite simple. The target code creating a new instance
of `RemremClient` differs only slightly from application of a default builder.

    String creatorClassName = "CustomRemremClientBuilder";
    ClassLoader classLoader = getClass().getClassLoader();

    RemremClient client = RemremClient.builder(creatorClassName, classLoader)
        .setUrl(URL)
        .setAuthentication(USER, PASSWORD)
        .build());

Existence of method `setFeature()` enables to pass custom features and values
to builder transparently.

    RemremClient client = RemremClient.builder(creatorClassName, classLoader)
        .setUrl(URL)
        .setAuthentication(USER, PASSWORD)
        .setFeature("custom_feature", "custom_value")
        .build());

Of course, implementation of `CustomRemremClientBuilder` isn't included as it is
out of the scope of the example above.



# License
The contents of this repository are licensed under the [Apache License 2.0](./LICENSE).

