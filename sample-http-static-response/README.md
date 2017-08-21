Static Http response sample ![Logo][1]
==============

Usually incoming requests on server components are handled by a Citrus test case where the tester defines the received request message. Also the test
case provides a proper response message to the calling client. There may be times we do not want to handle request/response in the test case. Instead we
need to define a static response message that should always be sent from a server component in Citrus.

This sample shows how to setup a Citrus Http server component that automatically provides response messages to incoming requests.

Objectives
---------

We want to have a server component that provides a static response message to calling clients. Depending on the requested resource path the server
should provide different response messages. we can do this in Citrus with a little bit of Spring bean configuration:

```
<citrus-http:server id="todoListServer"
              port="8080"
              auto-start="true"
              timeout="10000"
              endpoint-adapter="dispatchingEndpointAdapter" />

<citrus:dispatching-endpoint-adapter id="dispatchingEndpointAdapter"
                                 mapping-key-extractor="mappingKeyExtractor"
                                 mapping-strategy="mappingStrategy"/>
```

The *todoListServer* is a normal server component in Citrus. The endpoint-adapter is different though and defines the way how to automatically respond to calling clients.
The server uses a **dispatchingEndpointAdapter**. This endpoint adapter implementation uses a mapping key extractor and a mapping strategy in order to map incoming requests to response generating
adapters.

```
<bean id="mappingKeyExtractor" class="com.consol.citrus.endpoint.adapter.mapping.HeaderMappingKeyExtractor">
    <property name="headerName" value="citrus_http_request_uri"/>
</bean>

<bean id="mappingStrategy" class="com.consol.citrus.endpoint.adapter.mapping.SimpleMappingStrategy">
    <property name="adapterMappings">
        <map>
            <entry key="/todo" value-ref="todoResponseAdapter"/>
            <entry key="/todolist" value-ref="todoListResponseAdapter"/>
        </map>
    </property>
</bean>
```

The mapping key extractor implementation evaluates the Http header **citrus_http_request_uri** which is an internal header representing the called request path. Depending on that request path value the
mapping strategy maps incoming requests to different response generating adapter implementations. Here in this example we define **/todo** and **/todolist** request paths with response
adapters.

```
<citrus:static-response-adapter id="todoResponseAdapter">
    <citrus:payload>
        {
          "id": "${todoId}",
          "title": "${todoName}",
          "description": "${todoDescription}"
        }
    </citrus:payload>
</citrus:static-response-adapter>

<citrus:static-response-adapter id="todoListResponseAdapter">
    <citrus:payload>
        [
          {
            "id": "${todoId}",
            "title": "${todoName}",
            "description": "${todoDescription}"
          }
        ]
    </citrus:payload>
</citrus:static-response-adapter>
```

The response adapters provide static response messages. In summary we have a small Http server component that automatically responds to incoming request messages
with static message payloads.

Run
---------

The sample application uses Maven as build tool. So you can compile, package and test the
sample with Maven.
 
     mvn clean verify -Dembedded
    
This executes the complete Maven build lifecycle.

During the build you will see Citrus performing some integration tests.

Citrus test
---------

Execute all Citrus tests by calling

     mvn verify

You can also pick a single test by calling

     mvn verify -Dit.test=<testname>

You should see Citrus performing several tests with lots of debugging output in your terminal. 
And of course green tests at the very end of the build.

Of course you can also start the Citrus tests from your favorite IDE.
Just start the Citrus test using the TestNG IDE integration in IntelliJ, Eclipse or Netbeans.

Further information
---------

For more information on Citrus see [www.citrusframework.org][2], including
a complete [reference manual][3].

 [1]: http://www.citrusframework.org/img/brand-logo.png "Citrus"
 [2]: http://www.citrusframework.org
 [3]: http://www.citrusframework.org/reference/html/
 [4]: http://www.citrusframework.org/reference/html/validation-json.html
