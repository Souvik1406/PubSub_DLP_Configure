# PubSub_DLP_Configure

To integrate Google Cloud DLP logs with **Azure Sentinel** using **Google Pub/Sub**, you'll need to stream the logs from Google Cloud Logging into Azure Sentinel via **Azure Logic Apps** or **Azure Functions**. This setup allows you to capture logs from Google Pub/Sub and forward them to Azure Sentinel for analysis and correlation.

Here’s a step-by-step guide on how to achieve this integration:

---

### **1. Set Up Pub/Sub in Google Cloud**

First, create a **Pub/Sub Topic** and **Subscription** to collect DLP logs as described in the previous steps:

1.1 **Create a Pub/Sub Topic** (e.g., `dlp-logs-topic`).

1.2 **Create a Subscription** (e.g., `dlp-logs-subscription`) for the topic.

1.3 **Create a Log Export Sink** in **Google Cloud Logging** to send DLP logs to the Pub/Sub topic using a filter:

```plaintext
resource.type="audited_resource"
protoPayload.serviceName="dlp.googleapis.com"
```

Once the logs are flowing into Pub/Sub, proceed to integrate with Azure Sentinel.

---

### **2. Set Up Azure Sentinel and Azure Logic App**

#### 2.1 **Create an Azure Sentinel Workspace**

1. Go to the [Azure Portal](https://portal.azure.com/).
2. Navigate to **Azure Sentinel** and create a new Sentinel workspace if you don’t have one.
   - In **Create a new workspace**, select the resource group and location.
   - Choose a **Log Analytics workspace** for Azure Sentinel to store logs.
   - Click **Review + Create** to deploy the workspace.

---

### **3. Create a Logic App to Pull Logs from Pub/Sub**

You’ll need an **Azure Logic App** to pull the logs from Google Pub/Sub and send them to Azure Sentinel.

#### 3.1 **Create an Azure Logic App**
1. In the Azure Portal, search for **Logic Apps**.
2. Click **Create Logic App**.
3. Select your subscription, resource group, and provide a name for the logic app (e.g., `pull-gcp-dlp-logs`).
4. Set the region, and click **Create**.

#### 3.2 **Design the Logic App Workflow**
   After creation, open the logic app designer to build the workflow that fetches logs from Pub/Sub.

1. **Trigger**: Use the **Google Pub/Sub** connector (You may need to authenticate using Google Service Account).
   - **Trigger**: Configure the logic app to pull logs from the `dlp-logs-subscription`.
   
2. **Parse the Logs**: Depending on your log format, add a **Parse JSON** action to extract fields from the DLP logs.

3. **Send Logs to Azure Sentinel**:
   - Use **HTTP** or **Log Analytics Data Collector API** to send the parsed logs to the Azure Sentinel workspace.
   - The HTTP action can send the logs as a POST request to the **Log Analytics Workspace API**.

   Example URL:
   ```plaintext
   https://<workspace-id>.ods.opinsights.azure.com/api/logs?api-version=2016-04-01
   ```

   Headers:
   ```plaintext
   Content-Type: application/json
   Authorization: Bearer <Workspace API Key>
   ```
   
   Body:
   Send the parsed logs as JSON.

---

### **4. Configure the Data in Azure Sentinel**

#### 4.1 **Log Analytics Workspace Setup**
   - Go to the **Log Analytics Workspace** associated with Azure Sentinel.
   - Make sure the **Custom Logs** data source is enabled to receive logs from the Logic App.
   
#### 4.2 **Create Custom Log Rules**
   - Once logs start flowing in, use **Azure Sentinel’s custom log rules** to set up alerts, detections, and incidents based on specific patterns in the DLP logs.

---

### **5. Monitor and Analyze DLP Logs in Azure Sentinel**

1. Navigate to **Azure Sentinel > Logs**.
2. Use the **KQL (Kusto Query Language)** to query the incoming DLP logs from Google Cloud.
3. Create custom dashboards, alerts, and incidents in Azure Sentinel based on the DLP data.

---

### **Optional: Use Azure Functions (Alternative to Logic Apps)**

If you prefer to use an **Azure Function** instead of Logic Apps, you can:

- Deploy a Python or Node.js Azure Function that subscribes to the Google Pub/Sub topic, fetches logs, and forwards them to Azure Sentinel.
- Use the **Log Analytics API** to send the logs to Azure Sentinel from the Azure Function.

---

This approach ensures that your DLP logs from Google Cloud are seamlessly integrated with Azure Sentinel for real-time monitoring, security alerts, and advanced log analytics. Let me know if you need more specific details or assistance with any of these steps!
