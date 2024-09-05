# API Rate Limit Monitoring with Make

## Problem Statement

When using a service with API limits (e.g., per minute, hour, or overall), exceeding these limits can result in additional charges or rate-limiting errors. To prevent these issues, it's essential to monitor API requests and delay them as needed to avoid rate limit errors.

## Solution

Make provides a way to connect any API endpoint using a no-code tool. However, if a specific endpoint is used more than the allowed limit, you might encounter rate limit errors or incur additional charges. While Make does not offer a straightforward solution for this, our platform's "Never Say No" approach inspires us to devise creative workarounds.

### How We're Solving This Issue

To address this, we've developed a scenario design pattern that you can implement to manage API limits based on the required configuration.

#### Design Overview

To implement this design pattern, a monitoring scenario is needed to oversee all services. This scenario acts as a cloud lock, holding and releasing requests as required.

Any scenario that needs monitoring should be adjusted to send data at both the start and end, managing the scenario's running status.

![Reference Image](images/api_Limiter.jpg)


### Monitoring Models

We offer two models for monitoring scenarios:

- **Model One**: Monitors requests by the minute and holds any requests that exceed the limits within that minute.
- **Model Two**: Monitors the overall scenario's running status and holds API limits until all requests are completed.

Both models provide different ways to monitor execution, allowing you to choose the best option based on your use case.

## Folder Structure

- **`functions`**: Contains the `countof` function script. This function is required for counting specific number of modules from a scenairo.
- **`datastructure`**: Contains data structures for Model 1 and Model 2. These structures define how data is stored and managed during monitoring.
- **`Scenarios`**: Contains the Cloudlock & Monitoring scenario blueprint to demonstrate the monitoring setup. This can be imported into Make for reference.

---

## Setup Instructions

### Pre-Setup

1. **Create a Custom Function**:
   - Before importing the scenario, use the custom function found in the [`functions`](custom_function/countof.js) folder. This function, `countof`, is required to count specific API calls.

### Step-by-Step Setup

#### Step 1: Set a Global Variable

- Define the API limit at the organization or team level. Name it according to your convenience. This parameter will be adjusted later when setting up the scenarios.

#### Step 2: Download and Configure the Monitoring Scenario Blueprint

1. **Import the Monitoring Scenario Blueprint** into Make, available in the [`Scenaiors`](akashpaulmake/app_monitoring_cloudlock/scenarios/app-monitoring-cloudlock.json) folder.
2. **Configure the Webhook**:
   - Set up a webhook in the first step.
   - Map the organization variable to set the API limit for monitoring (current version supports one app but can be expanded).
3. **Setup Make Scenario**:
   - Create a connection using `scenario:read/write` API token to dynamically fetch modules used in any scenario.
4. **Setup Datastore**:
   - Use the data structures for [Model 1](datastructure/model1.json) and [Model 2](datastructure/model2.json) provided in the `datastructure` folder. Choose the model based on the use case and apply the data structure accordingly.

![Reference Image](images/datastore_setup.png)

#### Step 3: Implement Cloud Lock in Production Scenarios

1. **Add HTTP Module at the Beginning**:
   - Add an HTTP module with the following parameters:
     ```json
     {
       "app_name": "util",
       "scenario_id": "91615",
       "status": "block",
       "model": "1",
       "timestamp": "1725373841",
       "executionid": "9e6633388bc64303b245f6689afb0621"
     }
     ```
   - *Sample Request Body Parameters:*

| Key         | Value                             | Description                           |
|-------------|-----------------------------------|---------------------------------------|
| app_name    | Example - netsuite                | The name of the application           |
| scenario_id | `{{scenario_id}}`                 | Variable from mapping panel           |
| status      | block                             | Block at the beginning, unblock at end|
| executionid | `{{executionId}}`                 | Variable from mapping panel           |
| model       | 1                                 | 1 for model 1, 2 for model 2          |
| timestamp   | `{{timestamp}}`                   | Variable from mapping panel           |

- **Sample CURL Request**:
   ```sh
   curl -X POST '{{webhookurl}}?app_name=netsuite&scenario_id=91503&status=block&timestamp=1725373350&model=1&executionid=d61a793c6b044af2af2d9bac4bc1ec41' -H 'Accept-Encoding: gzip, br, deflate' -H 'User-Agent: Make/production'
   ```

#### Step 4: Set Up Cloud Release

1. **Add an HTTP > Make a Request Module** at the end of the scenario:
   - Configure the parameters similarly to the Cloud Lock, but use the "unblock" status.
   - Sample Request:
     ```json
     {
       "app_name": "util",
       "current_limit": "5",
       "key_min": "util0309200",
       "status": "unblock",
       "scenario_id": "91615",
       "model": "1",
       "executionid": "9e6633388bc64303b245f6689afb0621"
     }
     ```

- *Sample Body Parameters:*

| Key         | Value                             | Description                           |
|-------------|-----------------------------------|---------------------------------------|
| app_name    | Map from HTTP module output        | The name of the application           |
| scenario_id | `{{scenario_id}}`                 | Variable from mapping panel           |
| status      | unblock                           | Block at the beginning, unblock at end|
| key_min     | Map from HTTP module output        | Key derived from previous response    |
| executionid | `{{executionId}}`                 | Variable from mapping panel           |
| model       | 1                                 | 1 for model 1, 2 for model 2          |
| timestamp   | `{{timestamp}}`                   | Variable from mapping panel           |

## Additional Notes

- Ensure to adjust parameters correctly based on the monitoring model selected.
- Test the setup thoroughly in a staging environment before applying it to production.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

### How to Use the Files

- **Functions**: The `countof` function in the `functions` folder should be added to your Make instance as a custom function.
- **Data Structures**: Use the JSON files in the `datastructure` folder to set up the correct data model in your datastore.
- **Sample Scenario**: The `Scenarios` folder contains a scenario blueprint that demonstrates how to set up the monitoring process. Import this scenario into Make and adjust as necessary.

Feel free to further refine or customize this README according to your project's needs! Make sure the file paths are correctly set up in your GitHub repository for the links to work properly.