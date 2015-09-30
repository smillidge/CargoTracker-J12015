Cargo Tracker and Micro-services
================================

This project takes the original [cargo tracker](https://cargotracker.java.net/) 
application which is, in the terminology of microservices a ***monolith***, and 
splits out the pathfinder service as a separate micro service. 

PathFinder Microservice
-----------------------

The Pathfinder micro service was originally embedded into the core Cargo tracker
application. In this project it is separated out as an individual RESTful service
with its own maven project *pathfinder*.

The Pathfinder application can then be run on Payara Micro using the command;

```shell
java -jar payara-micro.jar --deploy pathfinder-1.0-SNAPSHOT.war --autoBindHttp
```

This can also be run multiple times to "scale out" the micro service. 

Load balancing configuration snippets for Nginx and HAProxy are provided at the 
root of the project to show how you would configure these tools to integrate
the cargo tracker application and the pathfinder micro service into a functional
service.

CargoTracker Monolith
---------------------

The *cargo-monolith* maven project is the original Cargo Tracker application with
the path finder functionality extracted and removed. 

To ensure the application knows the URL of the path finder microservice the URL 
must be specified. There are two options for this;
* You must edit the ejb-jar.xml in the WEB-INF directory and set below to the correct URL
```xml
            <env-entry>
                <env-entry-name>graphTraversalUrl</env-entry-name>
                <env-entry-type>java.lang.String</env-entry-type>
                <env-entry-value>http://127.0.0.1/pathfinder/rest/graph-traversal/shortest-path</env-entry-value>
            </env-entry>
```

Alternatively the ExternalRoutingService EJB can also retrieve the URL directly from JNDI.
If you bind the URL to JNDI at /graphTraversalUrlJNDI

This can be done in GlassFish using the asadmin command
```shell
create-custom-resource --restype java.lang.String --factoryclass org.glassfish.resources.custom.factory.PrimitivesAndStringFactory --property value="http\:\/\/127.0.0.1\/pathfinder\/rest\/graph-traversal\/shortest-path" graphTraversalUrlJNDI
 ```

Once this has been configured the cargo-tracker.war file can be deployed to GlassFish

Running the Application
-----------------------

To run the application you must run GlassFish or Payara Server and deploy the 
configured cargo-tracker application as described above. You must also run one or
more Payara Micro instances. Payara Micro automatically binds to the next available
http port starting from 8080 when using the command line above.

Once you have the application deployed and Payara Micro running cargo tracker can
 be accessed via http://127.0.0.1/cargo-tracker/.
To see the Pathfinder micro service in action navigate to;
* Administration Interface
* Book
* run through the booking wizard.

In your Payara Micro log you will see a message like below when the service is called;
```shell
[2015-09-30T15:30:55.051+0100] [Payara 4.1] [INFO] [] [net.java.pathfinder.api.GraphTraversalService] [tid: _ThreadID=23 _ThreadName=http-listener(5)] [timeMillis: 1443623455051] [levelValue: 800] Path Finder Service called for USNYC to USDAL
```
If you are using the nginx snippet or haproxy snippet you should see load balancing between
the Payara Micro instances. Demonstrating how easy it is to scale out the microservice.

