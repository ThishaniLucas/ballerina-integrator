# Service Composition

This guide walks you through the process of implementing a service composition using Ballerina language. A service composition is an aggregate of services collectively composed to automate a particular task or business process. 

## <a name="what-you-build"></a>  What you’ll build
To understanding how you can build a service composition using Ballerina, let's consider a real-world use case of a Travel agency that arranges complete tours for users. A tour package includes Airline ticket reservation, Hotel room reservation and Car rental. Therefore, the Travel agency service requires communicating with other necessary back-ends. The below diagram illustrates this use case clearly.

![alt text](/images/service_composition.png)

## <a name="pre-req"></a> Prerequisites
 
- JDK 1.8 or later
- [Ballerina Distribution](https://ballerinalang.org/docs/quick-tour/quick-tour/#install-ballerina)
- A Text Editor or an IDE 

Optional Requirements
- Ballerina IDE plugins (IntelliJ IDEA, VSCode, Atom)

## <a name="developing-service"></a> Developing the service

### <a name="before-begin"></a> Before you begin
##### Understand the package structure
Ballerina is a complete programming language that can have any custom project structure as you wish. Although language allows you to have any package structure, we'll stick with the following package structure for this project.

```
service-composition
├── TravelAgency
│   ├── AirlineReservation
│   │   ├── airline_reservation_service.bal
│   │   └── airline_reservation_service_test.bal
│   ├── CarRental
│   │   ├── car_rental_service.bal
│   │   └── car_rental_service_test.bal
│   ├── HotelReservation
│   │   ├── hotel_reservation_service.bal
│   │   └── hotel_reservation_service_test.bal
│   ├── travel_agency_service.bal
│   └── travel_agency_service_test.bal
└── README.md

```

Package `AirlineReservation` contains the service that provides online flight ticket reservations.

Package `CarRental` contains the service that provides online car rentals.

Package `HotelReservation` contains the service that provides online hotel room reservations.

`travel_agency_service.bal` file provides travel agency service, which consumes the other three services and arranges a complete tour for the requested user.


### <a name="Implementation"></a> Implementation

Let's look at the implementation of the Travel agency service, which acts as the composition initiator.

To arrange a complete tour travel agency service requires communicating with three other services Airline reservation, Hotel reservation, and Car rental respectively. All these services accept POST requests with appropriate JSON payloads and respond with JSON payloads. Request and response payloads are similar for all three backend services.

Sample request payload,

```bash
{"Name":"Bob", "ArrivalDate":"12-03-2018", "DepartureDate":"13-04-2018", "Preference":<service_dependent_preference>};
```

Sample response payload,

```bash
{"Status":"Success"}
```

When a client initiates a request to arrange a tour, the Travel agency service first needs to communicate with the Airline reservation service to book a flight ticket. 
Refer [airline_reservation_service.bal](https://github.com/ballerina-guides/service-composition/blob/master/TravelAgency/AirlineReservation/airline_reservation_service.bal) to check the implementation of Airline reservation service.


Once the airline ticket reservation is successful, Travel agency service needs to communicate with the Hotel reservation service to reserve hotel rooms. 
Refer [hotel_reservation_service.bal](https://github.com/ballerina-guides/service-composition/blob/master/TravelAgency/HotelReservation/hotel_reservation_service.bal) to check the implementation of Hotel reservation service.


Finally, Travel agency service needs to connect with the Car rental service to arrange internal transports.
Refer [car_rental_service.bal](https://github.com/ballerina-guides/service-composition/blob/master/TravelAgency/CarRental/car_rental_service.bal) to check the implementation of Car rental service.


If all successful, the Travel agency service will confirm and arrange the complete tour for the user. Skeleton of `travel_agency_service.bal` file attached below. Inline comments added for better understanding.


##### travel_agency_service.bal

```ballerina
package TravelAgency;

import ballerina.net.http;

// Travel agency service to arrange a complete tour for a user
@http:configuration {basePath:"/travel", port:9090}
service<http> travelAgencyService {

    // Endpoint to communicate with Airline reservation service
    endpoint<http:HttpClient> airlineReservationEP {
        create http:HttpClient("http://localhost:9091/airline", {});
    }

    // Endpoint to communicate with Hotel reservation service
    endpoint<http:HttpClient> hotelReservationEP {
        create http:HttpClient("http://localhost:9092/hotel", {});
    }

    // Endpoint to communicate with Car rental service
    endpoint<http:HttpClient> carRentalEP {
        create http:HttpClient("http://localhost:9093/car", {});
    }

    // Resource to arrange a tour
    @http:resourceConfig {methods:["POST"], consumes:["application/json"], produces:["application/json"]}
    resource arrangeTour (http:Connection connection, http:InRequest inRequest) {
        http:OutResponse outResponse = {};

        // Json payload format for an http out request
        json outReqPayload = {"Name":"", "ArrivalDate":"", "DepartureDate":"", "Preference":""};

        // Try parsing the JSON payload from the user request

        // If payload parsing fails, send a "Bad Request" message as the response
        

        // Reserve airline ticket for the user by calling Airline reservation service

        // Reserve hotel room for the user by calling Hotel reservation service

        // Renting car for the user by calling Car rental service
        
        
        // If all three services response positive status, send a successful message to the user
        outResponse.setJsonPayload({"Message":"Congratulations! Your journey is ready!!"});
        _ = connection.respond(outResponse);
    }
}

```

Let's now look at the code segment that is responsible for communicating with Airline reservation service. 

```ballerina
// Reserve airline ticket for the user by calling Airline reservation service
http:OutRequest outReqAirline = {};
http:InResponse inResAirline = {};

// construct the payload
json outReqPayloadAirline = outReqPayload;
outReqPayloadAirline.Preference = airlinePreference;
outReqAirline.setJsonPayload(outReqPayloadAirline);

// Send a post request to airlineReservationService with appropriate payload and get response
inResAirline, _ = airlineReservationEP.post("/reserve", outReqAirline);

// Get the reservation status
string airlineReservationStatus = inResAirline.getJsonPayload().Status.toString();

// If reservation status is negative, send a failure response to user
if (airlineReservationStatus.equalsIgnoreCase("Failed")) {
    outResponse.setJsonPayload({"Message":"Failed to reserve airline! " +
                                          "Provide a valid 'Preference' for 'Airline' and try again"});
    _ = connection.respond(outResponse);
    return;
}
```

The above code shows how the Travel agency service initiates a request to Airline reservation service to book a flight ticket. `airlineReservationEP` is the endpoint we defined through which our Ballerina service communicates with the external Airline reservation service.


Let's now look at the code segment that is responsible for communicating with Hotel reservation service. 

```ballerina
// Reserve hotel room for the user by calling Hotel reservation service
http:OutRequest outReqHotel = {};
http:InResponse inResHotel = {};
// construct the payload
json outReqPayloadHotel = outReqPayload;
outReqPayloadHotel.Preference = hotelPreference;
outReqHotel.setJsonPayload(outReqPayloadHotel);

// Send a post request to hotelReservationService with appropriate payload and get response
inResHotel, _ = hotelReservationEP.post("/reserve", outReqHotel);

// Get the reservation status
string hotelReservationStatus = inResHotel.getJsonPayload().Status.toString();
// If reservation status is negative, send a failure response to user
if (hotelReservationStatus.equalsIgnoreCase("Failed")) {
    outResponse.setJsonPayload({"Message":"Failed to reserve hotel! " +
                                     "Provide a valid 'Preference' for 'Accommodation' and try again"});
    _ = connection.respond(outResponse);
    return;
}
```
Travel agency service communicates with the Hotel reservation service to book a room for the client as shown above. The endpoint defined for this external service call is `hotelReservationEP`.


Finally, let's look at the code segment that is responsible for communicating with Car rental service. 

```ballerina
// Renting car for the user by calling Car rental service
http:OutRequest outReqCar = {};
http:InResponse inResCar = {};
// construct the payload
json outReqPayloadCar = outReqPayload;
outReqPayloadCar.Preference = carPreference;
outReqCar.setJsonPayload(outReqPayloadCar);

// Send a post request to carRentalService with appropriate payload and get response
inResCar, _ = carRentalEP.post("/rent", outReqCar);

// Get the rental status
string carRentalStatus = inResCar.getJsonPayload().Status.toString();
// If rental status is negative, send a failure response to user
if (carRentalStatus.equalsIgnoreCase("Failed")) {
    outResponse.setJsonPayload({"Message":"Failed to rent car! " +
                                          "Provide a valid 'Preference' for 'Car' and try again"});
    _ = connection.respond(outResponse);
    return;
}

```

As shown above, Travel agency service rents a car for the requested user by calling the Car rental service. `carRentalEP` is the endpoint defined to communicate with the external Car rental service.

## <a name="testing"></a> Testing 

### <a name="try-it"></a> Try it out

1. Start all 4 http services by entering the following commands in separate terminals. This will start the `Airline Reservation`, `Hotel Reservation`, `Car Rental` and `Travel Agency` services in ports 9091, 9092, 9093 and 9090 respectively.

   ```bash
    <SAMPLE_ROOT_DIRECTORY>$ ballerina run TravelAgency/AirlineReservation/
   ```
   ```bash
    <SAMPLE_ROOT_DIRECTORY>$ ballerina run TravelAgency/HotelReservation/
   ```
   ```bash
    <SAMPLE_ROOT_DIRECTORY>$ ballerina run TravelAgency/CarRental/
   ```
   ```bash
    <SAMPLE_ROOT_DIRECTORY>$ ballerina run TravelAgency/
   ```
   
2. Invoke the `travelAgencyService` by sending a POST request to arrange a tour,

   ```bash
    curl -v -X POST -d \
    '{"Name":"Bob", "ArrivalDate":"12-03-2018", "DepartureDate":"13-04-2018",
     "Preference":{"Airline":"Business", "Accommodation":"Air Conditioned", "Car":"Air Conditioned"}}' \
     "http://localhost:9090/travel/arrangeTour" -H "Content-Type:application/json"
    ```

    The `travelAgencyService` should respond something similar,
    
    ```bash
     < HTTP/1.1 200 OK
    {"Message":"Congratulations! Your journey is ready!!"}
    ``` 
   
   
### <a name="unit-testing"></a> Writing unit tests 

In ballerina, the unit test cases should be in the same package and the naming convention should be as follows,
* Test files should contain _test.bal suffix.
* Test functions should contain test prefix.
  * e.g.: testTravelAgencyService()

This guide contains unit test cases for each service implemented above. 

Test files are in the same packages in which the service files are located.

To run the unit tests, go to the sample root directory and run the following command
   ```bash
   <SAMPLE_ROOT_DIRECTORY>$ ballerina test TravelAgency/
   ```

To check the implementations of these test files, refer [airline_reservation_service_test.bal](https://github.com/ballerina-guides/service-composition/blob/master/TravelAgency/AirlineReservation/airline_reservation_service_test.bal), [hotel_reservation_service_test.bal](https://github.com/ballerina-guides/service-composition/blob/master/TravelAgency/HotelReservation/hotel_reservation_service_test.bal), [car_rental_service_test.bal](https://github.com/ballerina-guides/service-composition/blob/master/TravelAgency/CarRental/car_rental_service_test.bal) and [travel_agency_service_test.bal](https://github.com/ballerina-guides/service-composition/blob/master/TravelAgency/travel_agency_service_test.bal).


## <a name="deploying-the-scenario"></a> Deployment

Once you are done with the development, you can deploy the services using any of the methods that we listed below. 

### <a name="deploying-on-locally"></a> Deploying locally
You can deploy the RESTful services that you developed above, in your local environment. You can create the Ballerina executable archives (.balx) first and then run them in your local environment as follows,

Building 
   ```bash
    <SAMPLE_ROOT_DIRECTORY>$ ballerina build TravelAgency/<Package_Name>
   ```

Running
   ```bash
    <SAMPLE_ROOT_DIRECTORY>$ ballerina run <Exec_Archive_File_Name>
   ```

### <a name="deploying-on-docker"></a> Deploying on Docker

You can use the Ballerina executable archives (.balx) that we created above and create docker images for the services using the following commands, 

```bash
<SAMPLE_ROOT_DIRECTORY>$ ballerina docker <Exec_Archive_File_Name>  
```

Once you have created the docker images, you can run them using docker run as follows, 

```bash
docker run -p <host_port>:<service_port> --name <container_instance_name> -d <image_name>:<tag_name>
```

For example,

To run the Travel agency service,

```bash
docker run -p <host_port>:9090 --name ballerina_TravelAgency -d TravelAgency:latest
```

### <a name="deploying-on-k8s"></a> Deploying on Kubernetes
(Work in progress) 


## <a name="observability"></a> Observability 

### <a name="logging"></a> Logging
(Work in progress) 

### <a name="metrics"></a> Metrics
(Work in progress) 


### <a name="tracing"></a> Tracing 
(Work in progress) 
