# Json Webhook to Microsoft Teams

This Azure Function handles incoming HTTP POST requests containing JSON security alerts, extracts key fields, and sends formatted alert messages to a Microsoft Teams channel via an incoming webhook.

## Features

- Extracts key information from the JSON payload (timestamp, IP addresses, rules violated).
- Formats the data in Markdown for easy reading in Microsoft Teams.
- Posts the message using a Microsoft Teams webhook URL.

## Step-by-Step Breakdown:

1. **Import Statements:**

    * The script imports several namespaces necessary for handling HTTP requests, JSON manipulation, and logging:

        * `System.IO`, `System.Net.Http`, `System.Text`, etc., provide access to file reading, HTTP communication, and string encoding.
        * `Microsoft.AspNetCore.Mvc`, `Microsoft.Azure.WebJobs`, `Microsoft.Extensions.Logging` support Azure Function definitions and logging.
        * `Newtonsoft.Json` is used for JSON serialization and deserialization.

1. **Function Declaration:**

   * The static class `JsonWebhookFunction` is defined, containing the Azure function logic.
   * The method `Run` is decorated with the `[FunctionName("JsonWebhookFunction")]` attribute, which names the function.

1. **HTTP Trigger Setup:**

   * The function uses an HTTP trigger (`HttpTrigger`) with a `POST` method and `Function` authorization level. This means the function listens for incoming HTTP POST requests.

1. **Logging Incoming Payload:**
   * The `ILogger log` object is used to log a message indicating that an incoming JSON payload has been received.
1. **Read and Deserialize JSON Payload:**
   * The incoming JSON body is read from the HTTP request using a `StreamReader` and stored in the `requestBody`.
   * The `requestBody` is deserialized into a dynamic object (`data`) using `JsonConvert.DeserializeObject`, allowing flexible access to the JSON properties.
1. **Extract Key Fields from JSON:**
   * Several key fields are extracted from the `data` object:
     * **Timestamp:** Extracts the timestamp and formats it as a `DateTime` string.
     * **Request ID, Message, Source/Destination IPs and Ports, Path:** These fields are extracted and default to `"N/A"` if missing from the JSON.
     * **Rules List:** Extracts the list of rules violated (if present), and converts it into a list of dynamic objects.
1. **Handle Rules List:**
   * If any rules are found in the JSON payload, each rule's `id`, `message`, and `severity` are formatted into a string. If no rules are present, the output defaults to `"No rules violated."`.
1. **Format Message for Microsoft Teams:**
   * A message is constructed in Markdown format to be sent to Microsoft Teams. The message contains:
     * A security alert with the extracted details (e.g., timestamp, request ID, source/destination IP and ports).
     * The list of violated rules (if any).
     * The full JSON payload is embedded as a code block.
1. **Send Message to Microsoft Teams Webhook:**
   * The script sends the formatted message to a pre-configured Microsoft Teams incoming webhook URL (`teamsWebhookUrl`).
   * A `HttpClient` instance is used to make a `POST` request with the formatted message as a JSON payload.
1. **Handle Response:**
    * If the `POST` request to Teams is successful (i.e., the webhook URL responds with a success status code), the function returns an `OkObjectResult` with a success message.
    * If the request fails, a `BadRequestObjectResult` is returned with the error status code.

## Usage

1. Set up your Microsoft Teams webhook URL.
1. Create and Deploy the Azure Function.
1. Trigger the function by sending a JSON payload using an HTTP POST request.

## Requirements

- Visual Studio Code
- .NET 6.0 (or higher)
- Azure Function Tools
- Microsoft Teams Incoming Webhook

## Steps to Create and Deploy the Azure Function

1. **Create a Local Azure Function Project** :
    1. Open  **Visual Studio Code** .
    1. Open the **Command Palette** (`Cmd+Shift+P`) and type `Azure Functions: Create New Project`.
    1. Select the folder where you want to create the project.
    1. Select the following settings:
        * **Language** : C#
        * **Template** : HTTP trigger
        * **Authorization Level** : Function (so it's secured with a function key)


1. **Replace the Template Code with Your Code** :

    * Once the project is created, you'll see a generated file for the HTTP trigger. Replace the default code inside the generated function file (e.g., `Function1.cs`) with your code from above (`JsonWebhookFunction.cs`).
    * Ensure that the file is renamed to something appropriate, like `JsonWebhookFunction.cs`.

1. **Install Dependencies** :

    * Your function uses **Newtonsoft.Json** for JSON serialization/deserialization, so you'll need to install this package:
        1. Open a terminal in VS Code (`Ctrl+` `or View > Terminal`).
        1. Run the following command to install  **Newtonsoft.Json** :
            ```dotnet add package Newtonsoft.Json```
            *  This will update your `.csproj` file with the necessary dependency.

1. **Test Locally** :

    * Before deploying, test your function locally.
    * In the terminal, run:
    <pre class="!overflow-visible"><div class="dark bg-gray-950 contain-inline-size rounded-md border-[0.5px] border-token-border-medium relative"><div class="flex items-center text-token-text-secondary bg-token-main-surface-secondary px-4 py-2 text-xs font-sans justify-between rounded-t-md h-9">bash</div><div class="sticky top-9 md:top-[5.75rem]"><div class="absolute bottom-0 right-2 flex h-9 items-center"><div class="flex items-center rounded bg-token-main-surface-secondary px-2 font-sans text-xs text-token-text-secondary"><span class="" data-state="closed"><button class="flex gap-1 items-center py-1"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="icon-sm"><path fill-rule="evenodd" clip-rule="evenodd" d="M7 5C7 3.34315 8.34315 2 10 2H19C20.6569 2 22 3.34315 22 5V14C22 15.6569 20.6569 17 19 17H17V19C17 20.6569 15.6569 22 14 22H5C3.34315 22 2 20.6569 2 19V10C2 8.34315 3.34315 7 5 7H7V5ZM9 7H14C15.6569 7 17 8.34315 17 10V15H19C19.5523 15 20 14.5523 20 14V5C20 4.44772 19.5523 4 19 4H10C9.44772 4 9 4.44772 9 5V7ZM5 9C4.44772 9 4 9.44772 4 10V19C4 19.5523 4.44772 20 5 20H14C14.5523 20 15 19.5523 15 19V10C15 9.44772 14.5523 9 14 9H5Z" fill="currentColor"></path></svg>Copy code</button></span></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="!whitespace-pre hljs language-bash">func start
    </code></div></div></pre>
    * This will start your Azure Function locally. It should output a URL like `http://localhost:7071/api/JsonWebhookFunction`. Use `curl` or a tool like Postman to test your function by sending a POST request with a JSON body.

1. **Deploy to Azure** :
    1. In the **Command Palette** (`Cmd+Shift+P`), type `Azure Functions: Deploy to Function App`.
    1. If you haven’t created a Function App in Azure yet, the deployment process will ask you to create one:
        * **Subscription** : Choose your Azure subscription.
        * **Resource Group** : You can create a new one or select an existing one.
        * **Name** : Provide a unique name for your function app.
        * **Plan Type** : Choose **Consumption Plan** (pay per execution) for simple use cases.
        * **Location** : Choose a nearby region.
    1. Once the app is created, VS Code will deploy your function code to Azure.
1. **Configure the Microsoft Teams Webhook URL** :

    * Before deploying, replace the placeholder Microsoft Teams webhook URL in the code (`teamsWebhookUrl`) with the actual one. You can find or generate the webhook URL from your Microsoft Teams channel by following these steps:
        1. Go to the desired **Teams** channel.
        1. Click on the **ellipsis (⋯)** next to the channel name.
        1. Choose  **Connectors** .
        1. Search for and configure  **Incoming Webhook** , then generate a URL.

1. **Monitor and Test on Azure** :

    * After deployment, you can test the live function by sending a POST request to the Azure URL. To find your function's URL:
        1. Go to the **Azure** tab in VS Code.
        1. Find your Function App, right-click, and select  **Copy Function URL** .
        1. Use this URL to send a POST request, similar to your local test.

## Script

**NOTE:** Replace `REPLACE_ME_WITH_MS_TEAMS_WEBHOOK_URL` with the actual Microsoft Teams URL you saved in a previous step

```
using System.IO;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Linq;

public static class JsonWebhookFunction
{
    [FunctionName("JsonWebhookFunction")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
        ILogger log)
    {
        log.LogInformation("Incoming JSON webhook payload.");

        // Read the incoming JSON payload
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);

        // Extract and format key fields from the JSON
        var timestamp = data?.record?["@timestamp"] != null ? DateTime.Parse(data.record["@timestamp"].ToString()).ToString("yyyy-MM-dd HH:mm:ss") : "N/A";
        var requestId = data?.record?.request_id ?? "N/A";
        var message = data?.record?.msg ?? "N/A";
        var sourceIp = data?.record?.source?.ip ?? "N/A";
        var sourcePort = data?.record?.source?.port_num ?? "N/A";
        var destinationIp = data?.record?.destination?.ip ?? "N/A";
        var destinationPort = data?.record?.destination?.port_num ?? "N/A";
        var path = data?.record?.path ?? "N/A";

        // Safely cast dynamic to list for rules
        List<dynamic> rulesList = data?.record?.rules != null ? data.record.rules.ToObject<List<dynamic>>() : new List<dynamic>();

        // Handle the list of rules
        var rules = rulesList.Any()
            ? string.Join("\n", rulesList.Select(rule =>
                $" - Rule {rule.id}: {rule.message} (Severity: {rule.severity})"))
            : "No rules violated.";

        // Format the message using Markdown for Microsoft Teams
        var formattedMessage = new
        {
            text = $"### **🚨 Security Alert: {message}**\n\n" +
                $"**Timestamp**: {timestamp}\n" +
                $"**Request ID**: {requestId}\n" +
                $"**Source IP/Port**: {sourceIp}:{sourcePort}\n" +
                $"**Destination IP/Port**: {destinationIp}:{destinationPort}\n" +
                $"**Rules Violated**: \n{rules}\n\n" +
                $"**Path**: {path}\n" +
                $"---\n\n" +
                $"**Full JSON Payload**: \n```json\n{JsonConvert.SerializeObject(data, Formatting.Indented)}\n```"
        };

        // Send the message to Microsoft Teams Incoming Webhook URL
        string teamsWebhookUrl = "REPLACE_ME_WITH_MS_TEAMS_WEBHOOK_URL";
        var client = new HttpClient();
        var jsonContent = new StringContent(JsonConvert.SerializeObject(formattedMessage), Encoding.UTF8, "application/json");

        HttpResponseMessage response = await client.PostAsync(teamsWebhookUrl, jsonContent);

        if (response.IsSuccessStatusCode)
        {
            return new OkObjectResult("Message posted to Teams");
        }
        else
        {
            return new BadRequestObjectResult($"Failed to post message to Teams. Status code: {response.StatusCode}");
        }
    }
}

```
