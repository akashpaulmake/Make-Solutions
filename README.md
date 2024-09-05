**Problem Statement: **

Imagine you’re using a service which has certain API limits, it could be by min, hour or overall limit. If exceeded either there will be additional invoice or it will rate limit the request. 

**Solution** - What if we monitor the API request in Make and delay it further so it doesn’t return any rate limit error 

**Overall Idea - **

Make in general helps you to connect any API endpoint using our no code tool, however if you’re using any specific endpoint more than their specified limit you will either have a rate limit error or you may need to pay more to use it.  \
 \
Generally there’s no straightforward solution available in Make, however a platform like Make we say it **“Never Say No”** it’s our true DNA that we’re able to achieve things and figure out different workarounds to achieve more things in Make. It’s truly relies with in the Design Pattern.

**How we’re solving this issue - **To achieve this, We’ve figured out a scenario design pattern. in addition to the existing scenario if adjusted it can manage the API limit based on the configuration. 

In order to implement this design pattern, we will need a monitoring scenario which will facilitate all the monitoring services and work as a cloud lock which will hold and release scenarios.

Then any scenario that is required to be monitored, should be adjusted accordingly to send the data in the beginning and the end to manage the state of the scenario running status. 

We’ve two models to monitor the scenario - 



1. Model one - Monitor the request by min and hold any request that exceeds limits within the execution min. 
2. Model two - Monitor the overall scenario running status and hold the API limit until the scenario completes its requests

While both offers different possibilities to monitor any execution, depending on the use case this can be selected in the request.  \
 \


**Diagram -  \
 \
 \
**
![alt_text](images/image1.jpg "image_tooltip")
** \
**

**How to Set this up - **We will explain it in step by step as there is multiple steps required to configure this.  \
 \
**Pre-setup -  \
 \
1. We’ve built a custom function to ease up the count of specific apps that needs to be monitored. [ Important - it should be performed before importing the scenario ] \
 \
**Create a Custom function and give it a name of **countof**

And paste this function \
 \
`function countof(arr, item) {`


```
   let count = 0;
   for (let i = 0; i < arr.length; i++) {
       if (arr[i] === item) {
           count++;
       }
   }
   return count;
}
```


**Step 1 - **Set a Global Variable at Organization or team level, here we will define the API limit based on the Application.


![alt_text](images/image2.png "image_tooltip")
** \
 \
**

Name it as per your convenience, we will be adjusting this parameter once we will setup the scenarios. 

**Step 2 - **Download the Monitoring Scenario blueprint and export it in Make.  \




* Configure the webhook in the first step \

* Map the organization variable which will set the API limit for the monitoring scenario ( At this moment in this version this monitoring design can only handle one app but it can expanded further ) \
 \

![alt_text](images/image3.png "image_tooltip")
 \

* Setup Make > Get a Scenario: Create a connection with scenario:read/write api token for this module to fetch the modules used in any scenario dynamically. \
 \

* Setup Datastore > Search, Create a record :  \
	- Create the data structure based on the model and save the data store.

*  select the model based on the choice and apply the data structure in the specific route. Check the below image for reference


![alt_text](images/image4.png "image_tooltip")


 \
**Step 3 - **In this step, we will implement the Cloud Lock in your production scenarios.  \
 \
 HTTP Module should be added at the beginning of the scenario with the below parameters to the monitoring scenario. 

Add a HTTP module ( if you’ve already downloaded the sample scenario, you can copy and paste it anywhere before the module which you want to monitor )

 \
 \
**Sample Request - **

{

  "app_name": "util",

  "scenario_id": "91615",

  "status": "block",

  "model": "1",

  "timestamp": "1725373841",

  "executionid": "9e6633388bc64303b245f6689afb0621"

} \


**Body Parameters: **


<table>
  <tr>
   <td><strong>key</strong>
   </td>
   <td><strong>Value</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td><strong>app_name </strong>
   </td>
   <td>Example - netsuite
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>scenario_id</strong>
   </td>
   <td>{{scenario_id}}
   </td>
   <td>Map the variable from mapping panel
   </td>
  </tr>
  <tr>
   <td><strong>status</strong>
   </td>
   <td>block
   </td>
   <td>Block - in the beginning  \
Unblock - at the end of the scenario
   </td>
  </tr>
  <tr>
   <td><strong>executionid</strong>
   </td>
   <td>{{executionId}}
   </td>
   <td>Map the variable from mapping panel
   </td>
  </tr>
  <tr>
   <td><strong>model</strong>
   </td>
   <td>1
   </td>
   <td>1 - select model 1
<p>
2 - select model 2
   </td>
  </tr>
  <tr>
   <td><strong>timestamp</strong>
   </td>
   <td>{{timestamp}}
   </td>
   <td>Map the variable from mapping panel
   </td>
  </tr>
</table>


**Response - **

{

  "app_name": "util",

  "current_limit": "5",

  "key_min": " util0309200",

  "status": "unblock",

  "scenario_id": "91615",

  "model": "1",

  "executionid": "9e6633388bc64303b245f6689afb0621"

}

**Sample of CURL request -  \
 \
**

**curl -X POST '{{webhookurl}}?app_name=netsuite&scenario_id=91503&status=block&timestamp=1725373350&model=1&exectionid=d61a793c6b044af2af2d9bac4bc1ec41' -H 'Accept-Encoding: gzip, br, deflate' -H 'User-Agent: Make/production'**


![alt_text](images/image5.png "image_tooltip")


**Step 4 - **Setup Cloud Release in this step

Add a HTTP > Make a Request module at the end of the scenario and configure rest of the parameters ( if you’ve already downloaded the sample scenario, you can copy and paste it anywhere before the module which you want to monitor ) \


**Body > Application/type- JSON**

** \
**{

  "app_name": "util",

  "current_limit": "5",

  "key_min": " util0309200",

  "status": "unblock",

  "scenario_id": "91615",

  "model": "1",

  "executionid": "9e6633388bc64303b245f6689afb0621"

}


<table>
  <tr>
   <td><strong>key</strong>
   </td>
   <td><strong>Value</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td><strong>app_name </strong>
   </td>
   <td>
   </td>
   <td>Map the variable from HTTP module output 
   </td>
  </tr>
  <tr>
   <td><strong>scenario_id</strong>
   </td>
   <td>{{scenario_id}}
   </td>
   <td>Map the variable from mapping panel
   </td>
  </tr>
  <tr>
   <td><strong>status</strong>
   </td>
   <td>unblock
   </td>
   <td>Block - in the beginning  \
Unblock - at the end of the scenario
   </td>
  </tr>
  <tr>
   <td><strong>key_min</strong>
   </td>
   <td>
   </td>
   <td>Map the variable from HTTP module output 
   </td>
  </tr>
  <tr>
   <td><strong>executionid</strong>
   </td>
   <td>{{executionId}}
   </td>
   <td>Map the variable from mapping panel
   </td>
  </tr>
  <tr>
   <td><strong>model</strong>
   </td>
   <td>1
   </td>
   <td>1 - select model 1
<p>
2 - select model 2
   </td>
  </tr>
  <tr>
   <td><strong>timestamp</strong>
   </td>
   <td>{{timestamp}}
   </td>
   <td>Map the variable from mapping panel
   </td>
  </tr>
</table>


 \



![alt_text](images/image6.png "image_tooltip")
