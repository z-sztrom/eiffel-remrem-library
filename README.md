
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
A default, build-in, builder is provided by method `RemremClient.builder()`.

    RemremClient.builder()

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

Of course, implementation of `CustomRemremClientBuilder` isn't included as it is
out of the scope of the example above.


# Features and Options
Available functions and options should be described here:
* FEATURE_URL:                REMReM service url
* FEATURE_USERNAME:           username (for authentication)               
* FEATURE_PASSWORD:           password (for authentication)
* FEATURE_MESSAGE_PROTOCOL:   message protocol to request the REMReM services (Default value is, mp = eiffelsemantics)
* FEATURE_MESSAGE_TYPE:       eventType to generate/publish the event
* FEATURE_INSECURE:           if the value is False the connection will become insecure and it will ignore the certificate trust issues
* FEATURE_RETRY_COUNT:        no of retries
* FEATURE_USER_DOMAIN:        from which domain routingkey is prepared (Examples: eiffel920, eiffel921,...)
* FEATURE_TAG:                we are using always notag(Reserved for future purpose)
* FEATURE_ROUTING_KEY:        it is a message attribute, used to route the messages to queue
* FEATURE_RECIPIENT:          To add the REMReM recipient (we can add the multiple number of recipients, to support the library for multiple REMReM instances)


| Options                         | Default Value | Description                                                                                                                                                                                                                                                                                                                                                                                                                                |
|---------------------------------|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| FEATURE_FAIL_IF_MULTIPLE_FOUND:  | false         | If value is set to true and multiple event ids are found through any of the provided lookup definitions, then no event will be generated. |
| FEATURE_FAIL_IF_NONE_FOUND:      | false         | If value is set to true and no event id is found through (at least one of) the provided lookup definitions, then no event will be generated. |
| FEATURE_LOOKUP_IN_EXTERNAL_ERS:             | false          | If value is set to true then REMReM will query external ERs and not just the locally used ER. The reason for the default value to be False is to decrease the load on external ERs. Here local ER means Single ER which is using REMReM generate.  External ER means multiple ER's which are configured in Local ER.|
| FEATURE_LOOKUP_LIMIt:            | 1             | The number of events returned, through any lookup definition given, is limited to this number. |
| FEATURE_OK_TO_LEAVE_OUT_INVALID_OPTIONAL_FIELDS   | false      | If value is set to true it will remove the optional event feilds from the input event data that does not validated successfully.
| parseData     | false            |                                                                          |

# License
The contents of this repository are licensed under the [Apache License 2.0](./LICENSE).

