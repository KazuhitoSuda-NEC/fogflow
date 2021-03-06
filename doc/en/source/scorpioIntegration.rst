*****************************************
Integrate FogFlow with Other NGSI-LD Broker
*****************************************


This tutorial introduces how FogFlow could be utilized as an advanced data analytics framework to enable on-demand data analytics
on top of the raw data captured in the NGSI-LD brokers, such as Scorpio, Orion-LD, and Stellio. 
The following diagram shows a simple example of how to do this in details, mainly including
three aspects with 8 steps

* how to fetch some raw data from an NGSI-LD broker into the FogFlow system (**Step 1-3**)
* how to use the serverless function in FogFlow to do customized data analytics (**Step 4**)
* how to push the generate analytics results back to the NGSI-LD broker for further sharing (**Step 5-8**)
 

.. figure:: figures/fogflow-ngsild-broker.png


Before looking into the detailed steps, please set up a FogFlow system and 
an NGSI-LD broker according to the following information. 

First, please refer  `FogFlow on a Single Machine`_ to set up a FogFlow system on a single host machine 

.. _`FogFlow on a Single Machine`: https://fogflow.readthedocs.io/en/latest/onepage.html

In terms of the NGSI-LD broker, we have different choices: Scorpio, Orion-LD, Stellio. 
Here we take Orion-LD as a concrete example to show the detailed steps. 
Please refer to the following steps to set up a Orion-LD broker on the same host machine. 
The integration with the other brokers (e.g., Scorpio, Stellio) will require some small changes to the port number 
in the requests and also the configuration files. 

.. code-block:: console

	# fetch the docker-compose file 
	wget https://raw.githubusercontent.com/smartfog/fogflow/development/test/orion-ld/docker-compose.yml
	
	# start the orion-ld broker
	docker-compose pull
	docker-Compose up -d 

Before you start the following steps, please check if your Orion-LD broker and FogFlow system is running properly. 

# check if the orion-ld broker is running

.. code-block:: console

	curl localhost:1026/ngsi-ld/ex/v1/version

# check if the FogFlow system is running properly
	
	open the FogFlow dashboard from your browser



How to Fetch data from Orion-LD to FogFlow 
================================================================


Step 1: issue a subscription to Orion-LD 


.. code-block:: console    

	curl -iX POST \
		  'http://localhost:1026/ngsi-ld/v1/subscriptions' \
		  -H 'Content-Type: application/json' \
		  -H 'Accept: application/ld+json' \
		  -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"; type="application/ld+json"' \
		  -d ' {
             	"type": "Subscription",
             	"entities": [{
                   "type": "Vehicle"
             	}],
             	"notification": {
                   "format": "keyValues",
                   "endpoint": {
                       "uri": "http://192.168.0.59:8070/ngsi-ld/v1/notifyContext/",
                       "accept": "application/ld+json"
             	    }
            	}
 			}'


Step 2: send an entity update to Orion-LD 


.. code-block:: console    

	
	curl --location --request POST 'http://localhost:1026/ngsi-ld/v1/entityOperations/upsert?options=update' \
	--header 'Content-Type: application/json' \
	--header 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"; type="application/ld+json"' \
	--data-raw '[
	{
	   "id": "urn:ngsi-ld:Vehicle:A106",
	   "type": "Vehicle",
	   "brandName": {
	                  "type": "Property",
	                  "value": "Mercedes"
	    },
	    "isParked": {
	                  "type": "Relationship",
	                  "object": "urn:ngsi-ld:OffStreetParking:Downtown1",
	                  "providedBy": {
	                                  "type": "Relationship",
	                                  "object": "urn:ngsi-ld:Person:Bob"
	                   }
	     },
	     "speed": {
	                "type": "Property",
	                "value": 120
	      },
	     "location": {
	                    "type": "GeoProperty",
	                    "value": {
	                              "type": "Point",
	                              "coordinates": [-8.5, 41.2]
	                    }
	     }
	}
	]'



Step 3: check if FogFlow receives the subscribed entity 


please prepare the CURL command to query the "Vehicle" entities from  FogFlow thinBroker. 


.. code-block:: console    

	curl -iX GET \
		  'http://localhost:8070/ngsi-ld/v1/entities?type=Vehicle' \
		  -H 'Content-Type: application/ld+json' \
		  -H 'Accept: application/ld+json' \
		  -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"; type="application/ld+json"' 



How to Program and Apply a Data Analytics Function 
================================================================

Step 4: apply fogfunction1 to do some customized data analytics


please change the code at "/application/operator/alert" to do some simple analysis, 
for example, generate an alert message when the speed of vehile is greater than some threshold. 


How to Push the Generated Result back to the NGSI-LD broker 
=========================================================================

please register fogfunction2 by default, including its operator, docker image, and fog function. 
Put them into the initialization list of the designer. 


Step 5: send a update message to trigger fogfunction2


.. code-block:: console    

	#please write the curl message to trigger fogfunction2


Step 6: check if fogfunction2 is created


explain where the users can check if the fogfunction2 is triggered. 


Step 7: check if Orion-LD has received the forwarded results


.. code-block:: console    

	curl -iX GET \
		  'http://localhost:8070/ngsi-ld/v1/entities?type=Alert' \
		  -H 'Content-Type: application/ld+json' \
		  -H 'Accept: application/ld+json' \
		  -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"; type="application/ld+json"' 

