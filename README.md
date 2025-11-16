# AI-call-agent

**🤖 AI Call Agent Workflow (n8n)**

This repository contains an n8n workflow that automates AI-powered outbound calls to customers using the Vapi.ai API
and Google Sheets as both the data source and tracker. The workflow fetches customer and question data, calls each customer via an AI voice assistant, and updates their call status automatically, all in a continuous loop.


**🚀 Overview**

**Workflow name**: AI Call Agent
**Automation platform:** n8n.io

**Primary integrations:**

- Google Sheets — stores customer data and call questions

- Vapi.ai — powers AI-driven phone calls

- HTTP Request — make calls via API

- Schedule Trigger — runs the workflow automatically (cron)

**🧭 Workflow Summary**

**Schedule Trigger** - Starts the workflow automatically on a recurring schedule (e.g., hourly or daily).

**Get Business Data** - Pulls customer information from a Google Sheet (Customer data tab), including:

- Business ID

- Business Name

- First Name

- Phone number

- Status

**NB:** this can vary based on your data points or customer data you want to pass to the voice assistant. 


**Loop Over Items** - Iterates through each customer record in batches to prevent rate limits.

**Set Business Data Fields** - Formats and prepares customer fields (First Name, Business Name, Phone Number, Business ID) for merging.

**Get Questions** - Fetches a list of predefined questions from another tab (Questions tab) in the same Google Sheet.

**Convert Question Arrays to String** - Joins all retrieved questions into a single comma-separated string to pass to the AI call API.

**Merge Data** - Combines customer details with the question string, creating a unified payload for each call.

**Make a Call (HTTP Request)** - Sends a POST request to https://api.vapi.ai/call with the following payload:


_{
  "assistantId": "YOUR_ASSISTANT_ID",
  "name": "NAME_OF_YOUR_VAPI_ASSISTANT",
  "phoneNumberId": "YOUR_PHONENUMBER_ID",
  "customer": {
    "number": "{{ $json['Phone Number'] }}"
  },
  "assistantOverrides": {
    "variableValues": {
      "first_name": "{{ $json['First Name'] }}",
      "business_name": "{{ $json['Business Name'] }}",
      "questions": "{{ $json.QuestionsString }}",
      "business_id": "{{ $json['Business ID'] }}"
    }
  }
}_



**Update Business Data** - After initiating each call, the workflow updates the customer’s Status column to "Calling" in the Google Sheet using their business_id as the unique key.

**Wait Node** - Waits 2 minutes before moving to the next call batch, helping manage concurrency and avoid hitting API limits.

**Repeat** - The loop continues until all customers in the sheet have been processed.



**🧠 Requirements**

To successfully run this workflow, you’ll need:

- An n8n account (self-hosted or cloud)

- Google Sheets OAuth2 credentials connected in n8n (you can use your preferred data source e.g Redash, Supabase etc).

- A Vapi.ai account with:

  - API key

  - Assistant ID

  - Phone number ID

**Required Google Sheets Setup**

1. Customer Data Sheet (tab: Customer data)
Contain:
- Business ID
- Business Name
- First Name
- Phone number
- Status
**NB:** Can contain other data points based on the scope requirement and preference

2. Questions Sheet (tab: Questions)
Contain a column titled Questions with the list of questions to be asked during the call.


**🔑 Environment Variables**

- **VAPI_API_KEY** - Your Vapi.ai API key for authentication
- **ASSISTANT_ID** - ID of your Vapi.ai AI Assistant
- **PHONENUMBER_ID** - ID of your registered Vapi.ai phone number

⚠️ Replace all placeholders (YOUR_VAPI_API_KEY, YOUR_ASSISTANT_ID, etc.) in the HTTP Request node before running.


**🛠 Setup Instructions**

1. Clone this repository

_git clone https://github.com/<your-username>/ai-call-agent-n8n.git_

2. Import the workflow

   - Open n8n
   - Navigate to Workflows → Import from File
   - Upload the _**ai-call-agent.json**_

**Add credentials**

- Connect Google Sheets via OAuth2
- Add your Vapi.ai API Key inside the HTTP Request node

**Configure your sheet**

- Update the document IDs and tab names in the Google Sheets nodes with your own sheet

**Activate and test**

- Run the workflow manually first

- Once validated, enable Schedule Trigger for automation


**💡 Example Use Case**

This workflow was designed for scenarios where businesses need to automate phone-based outreach, such as:

- Customer feedback collection
- User research and interviews for product teams
- Real-time customer engagement at milestones or drop-offs
- Business data verification
- Survey campaigns
- Reminder or follow-up calls



**📊 Future Improvements**

- Notify via Slack or email after each call batch
- Integration with CRM tools
