# Eiffel-REMRem-Library

Eiffel REMReM library is implemented to interact with REMReM service
# About this repository
The contents of this repository are licensed under the [Apache License 2.0](./LICENSE).

# Introduction

Eiffel REMReM library is used to send the request and get the response from the configured REMReM service. with the help of this library we can interact with the REMReM service.

# Instalation

This project does not need any installation. It is used as external library that will be included as a dependency to other projects.

This project is build by maven, by executing following commands:  

    mvn clean install

# Compatibility

* Maven 3.6.1 and greater versions
* JDK 1.8 and greater versions

For supporting latest features, Eiffel REMReM Library should use the latest version of [Eiffel REMReM Semantics](https://github.com/eiffel-community/eiffel-remrem-semantics).

# Configuration

Before running the java library we have to configure the working REMReM-service instances and credentials in TestOperations class. If we want to run the multiple REMReM clients test cases we have to add differant working REMReM-ports or we can give the same port to pass the test cases.

Below are the Parameters:  

    public static final String URL_BASE = "remrem-host-address:";  
    public static final String URL = URL_BASE + "remrem-port";  
    public static final String URL1 = URL;  
    public static final String URL2 = URL_BASE + "remrem-port1";  
    public static final String URL3 = URL_BASE + "remrem-port2";  
    public static final String USER = "username";  
    public static final String PASSWORD = "password";

# Usage

If we want to send any request to REMReM-service first we have to build REMReM-client like this.

    RemremClient client = RemremClient.builder()  
            .setUrl(URL)  
            .setAuthentication(USER, PASSWORD)  
            .setInsecure(INSECURE)  
            .build();

We can set the additional features if we want, like...

    RemremClient client = RemremClient.builder()  
            .setUrl(URL)  
            .setAuthentication(USER, PASSWORD)  
            .setInsecure(INSECURE)  
            .setFeature(RemremClientBuilder.FEATURE_FAIL_IF_MULTIPLE_FOUND, "true")  
            .setFeature(RemremClientBuilder.FEATURE_FAIL_IF_NONE_FOUND, "true")  
            .setFeature(RemremClientBuilder.FEATURE_ROUTING_KEY, "#")  
            .build();

Once after building the client we can send the request to REMReM-service.

Examples:  

    client.publish(event);  
    client.generate(event, eventType);


# Sample Code with Explanation

    private RemremClient buildRemremClient() throws RemremException {
        return RemremClient.builder()  
            .setUrl("http://localhost:8080")  
            .setAuthentication("dummy", "dummy")  
            .setInsecure(true)  
            .build();  
    }

    public void testPublishJsonString() throws RemremException, IOException {   
        try (RemremClient client = buildRemremClient()) {  
            String jsonEvent = readFileAsString("EiffelActivityTriggeredEvent");   
            EventPublishResponse[] response = client.publish(jsonEvent);  
            assertEquals("One response expected", 1, response.length);  
        }  
    }  

The buildRemremClient() will build and return the RemremClient Object with provided features(URL, USER and PASSWORD). After creation will call the particular endpoint related method.

The readFileAsString() method will read the event data from particular file(file name = EiffelActivityTriggeredEvent).

