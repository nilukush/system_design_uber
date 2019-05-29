# system_design_uber
A system design for cab aggregator service, Uber

![System Design Uber Driver Partner](https://raw.githubusercontent.com/nilukush/system_design_uber/master/images/sd_uber_driver.jpg)

# Driver Partner Components
## Driver APP / M (FE)
Front end available to driver partner where he can activate his availability, after which geolocattion details will start getting pushed to a backend service, Driver Location Service. Geolocation details will be pushed at an interval of 5 secs and is configurable. This value can be optimized later.

## Driver Location Service (DLS)
Geolocation details are pushed into Redis cache and lets Driver Service know that driver has activated his availability.

## Driver Service (DS)
Driver Service stores and exposes details with respect to driver partners (mobile, email, vehicle reg no, vehicle type, etc.). It uses MySQL as it's datastore. For scaling, it makes use of indexes, partition, shard per requirement basis.

## Allocation Engine (AE)
Allocation Engine is responsible for matching a user with a driver partner when there is a request for a trip. Given user geolocation detail, it
* would identify it lies in which Region
    * Location information would be broken down into Country, State, City, Region

* Once it identifies that, then it would find out the nearest driver(s)
    * It will identify nearest drivers within a circle from source for a radius of say 3 km.
    * And pick all available driver partners
    
* It uses NoSQL databased to store region information.
    
* ####It calls Driver Recommendation Service to get recommended driver and mark this driver in use in DS.

## Driver Recommendation Service (DRS)
This service is responsible for considering various params on which driver selection can happen. These can range from
* Number of trips
* Rating
* User vehicle preference
* etc

It also invokes Pricing Service (PS) 

## Pricing Service (PS)
This service is responsible for computing the price based on
* Availability of driver partners
* Road distance
* Vehicle type
* Growth optimization
* etc

It has a Rule Engine, which Business can use to configure rules basis which pricing computataion may change.

## Notification Service (NS)
Once a user is matched with a driver partner, then a notification is sent to driver partner on his or her APP/ M. Notification service handles sending notifications to driver partners. This service is an event-based notificaiton service. It supports :
* Different types of notifications and their templates are stored here
    * SMS
    * Email
    * WhatsApp
    * App Notif
    * Browser Push
* SLA for different type of notifications
* Failed messages handling
* etc
* Exposing a panel to product to make changes from rule perspective for 

## Order Management Service (OMS)
Manages lifecycle of an order. Calls Order Payment Service to initiate payment. Different order states are Created, Pending, Failed, Completed.

For cash payments, it marks the transaction as authorized once order is created.

## Order Payment Service (OPS)
Integrates with PG aggregator for initiating transaction, doing transaction status check and marking transactions authorised, i.e, order is marked completed. Once payment is successful, it pushes this information on Kafka for prepaid payments. This information is consumed by Payout Engine (PE).

## Payout Engine (PE)
This is resposible for consuming all pre-paid payments made by user and then initiating payouts in Paytm accounts of driver partners.

## Driver Partner Identity Service (DPIS)
* This is responsible for authentication and signup of driver partners.
* Some use cases :
    * When driver partner signs up, a Paytm wallet is created for them, if it does not exist for that mobile of driver partner
    * If Paytm wallet exists, then link the Paytm wallet in Uber 

## Payment Gateway Aggregator (PGA)
A third party payment aggregator to integrate with different banks.

## User Components
## Rider Service (RS)
This service is going to provide user details to AE via IS.

## Identity Service
Responsible for user signup and authentication. It stores user identity details (email, mobnile, image, etc).