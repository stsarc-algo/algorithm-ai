# stsarc-ai

Microservices-Development
Service Registry(Eureka), Admin Server, Zipkin Server, Config Server, RestAPI's, Kafka, Redis Cache, DB

=============================================================== Service Registry (Eureka Server)
=> Service Registry is used to maintain all apis information like name, status, url and health at once place. => It is also called as Service Discovery. => We can use Eureka Server as service registry. => It will provide user interface to get apis info. Note: Eureka server provided by "Spring cloud Netflix" Library.

What we call it as Service Discovery? => It will discover the service URL based on the service name.
Steps to develop Service Registry Application (Eureka Server)
Create SpringBoot application with below dependency

Eureka Server (spring-cloud-starter-netflix-eureka-server)
devtools
Configure @EnableEurekaServer annotation in boot start class

Configure below properties in application.yml file

Once application started we can access Eureka dashboard using below url

url: localhost://8761

application.yml
spring: application: name: 01_Service_Registry server: port: 8761 eureka: client: register-with-eureka: false

Note: By default this project acting as both eureka server and Eureka client. So If we don't want to act this project as eureka client then we have to use below property in application.yml file.

eureka: client: register-with-eureka: false

Note-1: If "Service-Registry" project port is 8761 then clients can discover service-registry and will register automatically with service-registry.

Note-2 : If service-registry project running on any other port number then we have to register clients with service-registry manually.

Once application started we can access Eureka Dashboard using below URL

 URL : http://localhost:8761/
========================================================= What is the Admin-Server
=> It is used to monitor and manage all the API's at one place. Ex: 1) Health Checks 2) Config Props 3) Url Mappings 4) Beans Loaded 5) Change log level 6) thread dumps 7) heam dumps. etc...

=> It provides beautiful user interface to access all apis actuator endpoints at one place.

Steps to develop Spring Admin-Server
Create Boot application with "admin-server" dependency (select it while creating the project)

Configure @EnableAdminServer annotation at start class

Change Port Number (Optional)

spring: application: name: 02_Admin_Server server: port: 1111

Run the boot application

Access application URL in browser (We can see Admin Server UI)

 URL : http://localhost:1111/
============================================================ What is the Zipkin Server
=> It is used for distributed tracing of our requests Ex: 1) How much time taking to process an request 2) Which microservice is taking more time to process 3) How many services is involved in one request processing

=> It provides beautiful user interface to access apis execution details.

Steps to work with Zipkin Server
Download Zipin Jar file

 URL : https://zipkin.io/pages/quickstart.html
Run zipkin jar file

 $ java -jar <jar-name>		
Zipkin Server Runs on Port Number 9411

Access zipkin server dashboard

 URL : http://localhost:9411/	
============================================================ Steps to develop Actual Microservice(WELCOME Rest API)
Create Spring Boot application with below dependencies

Add below dependencies

eureka-discovery-client (To act this application as client to Eureka server)

A REST based service for locating services for the purpose of load balancing and failover of middle-tier servers.
admin-client (To act this application as Admin Client to Admin Server)

Required for your application to register with a Codecentric's Spring Boot Admin Server instance.
zipkin (To act this application as Zipkin Client to Zipkin Server)

Enable and expose span and trace IDs to Zipkin.
Spring web

devtools

actuator (If we add this actuator then admin server will provide the UI for the Monitoring the app)

Configure @EnableDiscoveryClient annotation at boot start class.

Create RestController with required method

@RestController public class WelcomeRestController {

@GetMapping("/welcome")
public String getWelcomeMsg() {
	String msg = "Welcome To Sagar java arena..!!";
	return msg;
}
}

Configure below properties in application.yml file
Note: If our Eureka server is running on the another port instead of (8761) then we need to register our Eureka client(This service) manually. To register this service manually we need to add below properties in our application.yml file.

Manual registration with Eureka Server
eureka: client: service-url: defaultZone: http://localhost:8761/eureka/

If Eureka server is running on 8761 then no need to register this microservice manually.

Our Microservice also register with the admin server. For register this service with admin server then we need to add below properties in application.yml file.
spring: boot: admin: client: url: http://localhost:1111/

spring: application: name: 04_Welcome_Service boot: admin: client: url: http://localhost:1111/ server: port: 8081 eureka: client: service-url: defaultZone: http://localhost:8761/eureka/ management: endpoints: web: exposure: include: '*'
Run the application and check in Eureka Dashboard (It should display in eureka dashboard)

Check Admin Server Dashboard (It should display) (we can access application details from here)

Ex: Beans, loggers, heap dump, thred dump, metrics, mappings etc...

Send Request to REST API method

Check Zipkin Server UI and click on Run Query button (it will display trace-id with details)

======================================================================== Steps to develop Actual Microservice GREET Rest API
Create Spring Boot application with below dependencies

 - eureka-discovery-client
 - admin-client	
 - zipkin

 - starter-web
 - devtools
 - actuator

 - openfeign
Configure @EnableDiscoveryClient annotation at boot start class.

Create RestController with required method

@RestController public class GreetRestController {

@GetMapping("/greet")
public String getGreetMsg() {
	String msg = "Good Morning";
	return msg;
}
}

Configure below properties in application.yml file
spring: application: name: 05_Greet_Service boot: admin: client: url: http://localhost:1111/ server: port: 9091 eureka: client: service-url: defaultZone: http://localhost:8761/eureka/ management: endpoints: web: exposure: include: '*'
Run the application and check in Eureka Dashboard (It should display in eureka dashboard)

Check Admin Server Dashboard (It should display) (we can access application details from here)

Ex: Beans, loggers, heap dump, thred dump, metrics, mappings etc...

Send Request to REST API method

Check Zipkin Server UI and click on Run Query button (it will display trace-id with details)

============================== Interservice communication
=> Add @EnableFeignClients annotation in GREET-API boot start class

=> Create FeignClient interface like below

@FeignClient(name = "WELCOME-API") public interface WelcomeApiClient {

@GetMapping("/welcome")
public String invokeWelcomeMsg();
}

=> Inject feign client into GreetRestController like below

@RestController public class GreetRestController {

@Autowired
private WelcomeApiClient welcomeClient;

@GetMapping("/greet")
public String getGreetMsg() {
	
	String welcomeMsg = welcomeClient.invokeWelcomeMsg();
	
	String greetMsg = "Good Morning, ";
	
	return greetMsg.concat(welcomeMsg);
}
}

=> Run the applications and access greet-api method

(It should give combined response)
================== Load Balancing
=> If we run our application in one server then burden will be increased on that server.

				1) Single Server should handle all the load

				2) Burden on server

				3) Response delay

				4) Server can crash

				5) Single Point of failure
=> To overcome above problems we will run our application in multiple servers so that we can distribute the requests to multiple servers.

=> Load Balancer is used to distribute requests to multiple servers.

=> We have below advantages with load balancer.

			1) Less burden on server
			2) Quick Responses to clients
			3) No Single point of failure		
=============================== Load Balancing For Welcome API
Remove server port number from welcome api yml file

Make changes in rest controller to send port number in response.

@RestController public class WelcomeRestController {

@Autowired
private Environment env;

@GetMapping("/welcome")
public String getWelcomeMsg() {
	String port = env.getProperty("server.port");
	String msg = "Welcome To SagarSoft..!! (" + port + ")";
	return msg;
}
}

Right click => Run as => run configuration => select welcome-api => VM Arguments => -Dserver.port=8081 and apply and run it. (-D) is system argument that we are passing to the JVM

Right click => Run as => run configuration => select welcome-api => VM Arguments => -Dserver.port=8082 and apply and run it.

Right click => Run as => run configuration => select welcome-api => VM Arguments => -Dserver.port=8083 and apply and run it.

Note: With this our api will run in 3 servers with 3 diff port numbers.

Check Eureka Dashboard and observer 3 instances available for welcome-service or not.

Start Greet-Service and send request to Greet-Service and check Interservice communication.

======================================== Working with Spring Cloud API Gateway
Create Spring boot application with below dependencies

 -> eureka-client
 -> Ractive-Gateway (Under the Spring cloud Routing) 
    Provides a simple, yet effective way to route to APIs in reactive applications. Provides cross-cutting concerns to those APIs such as security, monitoring/metrics, and resiliency.

 -> devtools
Configure @EnableDiscoveryClient annotation at boot start class

Configure API Gateway Routings in application.yml file like below

spring: application: name: API-Gateway cloud: gateway: routes: - id: api-1 uri: lb://WELCOME-SERVICE predicates: - Path=/welcome #If any request comes from the /welcome then send the request to the welcome service api - id: api-2 uri: lb://GREET-SERVICE #If any request comes from the /Greet then send the request to the greet service api predicates: - Path=/greet server: port: 3333

Break downs for the above application.yml file congigurations
Spring Cloud Gateway, which is part of the Spring Cloud ecosystem. The purpose of this configuration is to set up an API Gateway that routes requests to other microservices based on the request path.

Here’s a breakdown:
spring: application: name: API-Gateway

spring.application.name: Names this service as API-Gateway. This is useful for service discovery and logging.

Gateway Configuration
cloud: gateway: routes:

This section defines routing rules for the Spring Cloud Gateway. Each route contains:

An id (a unique name),

A uri (where to send the request),

A list of predicates (conditions under which the route is matched).

Route 1: api-1
  - id: api-1
    uri: lb://WELCOME-SERVICE
    predicates: 
      - Path=/welcome
id: api-1: This is the route ID.

uri: lb://WELCOME-SERVICE:

The prefix lb:// tells Spring Cloud to use load balancing via a service discovery tool (like Eureka).

Requests matching the /welcome path are routed to the WELCOME-SERVICE instance(s).

predicates: - Path=/welcome:

This means: "If the incoming request's path is exactly /welcome, forward it to WELCOME-SERVICE."

Similarly, this route maps /greet requests to the GREET-SERVICE microservice.
Create Filter to validate incoming request

 if request contains below header then it is valid request so process it.

 	Secret=sagarsoft@123

 if above header is not present then it is invalid request, don't process it.					
Note: We should write filter validation logic in API Gateway service. Bcoz it will become the central security. (We not write security code for every service separately)

@Component public class MyFilter implements GlobalFilter {

@Override
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

	System.out.println(" filter() - executed..... ");

	// validate request
	
	ServerHttpRequest request = exchange.getRequest();
	HttpHeaders headers = request.getHeaders();
	Set<String> keySet = headers.keySet();
	
	if(!keySet.contains("Secret")) {
		throw new RuntimeException("Invalid Request");
	}
	
	List<String> list = headers.get("Secret");
	if(!list.get(0).equals("sagarsoft@123")) {
		throw new RuntimeException("Invalid Request");
	}
	
	return chain.filter(exchange);
}
}

=============================================================================================================== What is Cloud Config Server
=> We are configuring our application config properties in application.properties or application.yml file.

Ex: DB Props, SMTP props, Kafka Props, App Messages etc...
=> application.properties or application.yml file will be packaged along with our application (it will be part of our app jar file).

=> If we want to make any changes to properties then we have to re-package our application and we have to re-deploy our application.

Note: If any changes required in config properties then We have to repeat the complete project build & deployment which is time consuming process.

=> To avoid this problem, we have to seperate our project source code and project config properties files.

=> To externalize config properties from the application we can use Spring Cloud Config Server.

=> Cloud Config Server is part of Spring Cloud Library.

Note: Application config properties files we will maintain in git hub repo and config server will load them and will give to our application based on our application-name.

		App-name : flights ====> flights.yml

		App-name : hotels ====> hotels.yml

		App-name : trains ====> trains.yml
=> Our microservices will get config properties from Config server and config server will load them from git hub repo.

================================ Developing Config Server App
Create Git Repository and keep ymls files required for microservices

 	Note: We should keep file name as application name

 	app name : greet  then file name : greet.yml

 	app name : welcome then file name : welcome.yml
Git Repo : https://github.com/mcasagar/configuration_properties
Create Spring Starter application with below dependencies

 a) Config Server (Under the Spring Cloud Config) 
 b) devtools
Write @EnableConfigServer annotation at boot start class

Configure below properties in application.yml file

spring: application: name: 07_Cloud_Config_Server cloud: config: server: git: uri: https://github.com/mcasagar/configuration_properties server: port: 9093

Run Config Server application
====================================================================== Config Client Development
Create Spring Boot application with below dependencies

 		a) web-starter
 		b) config-client
 		c) dev-tools
 		d) actuators
Create Rest Controller with Required methods

@RestController public class MsgRestController {

@Value("${msg}")
private String msg;

@GetMapping("/")
public String getMsg() {
	return msg;
}
}

Configure ConfigServer url in application.yml file like below
spring: application: name: welcome config: import: optional:configserver:http://localhost:9093

Run the application and test it.
============================================== How to reload config properties dynamically
=> We need to make below 3 changes to reload latest config props

-> configure @RefreshScope annotation at Rest controller class

-> Enable actuator endpoint 'refresh' in application.yml

-> Send post request to actuator endpoint 'refresh' from postman

-> After refresh, test your config client application
