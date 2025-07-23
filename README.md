# 🧠 Agentic AI Workshop – Customer Support Agent

## 🎯 Goal of the workshop
Learn how to build a powerful, intelligent customer support agent using **DataStax Langflow**. You’ll start by creating a simple chatbot with **IBM watsonx.ai** (IBM's generative AI platform), then enrich it with retrieval-augmented generation (RAG) by connecting it to your own FAQ knowledge base using **DataStax Astra DB**. Finally, you’ll add tools such as order lookups, product info access, calculators, and web tools to make your agent truly agentic—capable of reasoning, taking actions, and handling real-world customer support scenarios.

By the end of this workshop, you’ll have a fully functional AI agent that can:
- Answer FAQs (unstructured documents) using vector-based document search
- Retrieve live order and product details from structured data
- Combine tools (like calculators and lookups) to generate multi-step responses
- Adapt dynamically to user intent—just like a real support agent would

## 🛠️ Prerequisites
This workshop assumes you have access to:
1. [A Github account](https://github.com)

During the course, you'll gain access to the following by signing up for free:
1. [DataStax Astra DB](https://astra.datastax.com) (you can sign up through your **public** Github account and/or you will be provided with Token and AstraDB URL to access data during the workshop)
2. [IBM watsonx.ai](https://www.ibm.com/products/watsonx-ai) (you can sign up for a free trial and/or will be provided during the workshop)
3. [DataStax Langflow](https://langflow.org) access

Follow the below steps and note down the **Astra DB API Endpoint**, **Astra DB ApplicationToken**, **watsonx.ai Project ID**, **watsonx.ai API Key** and **watsonx.ai URL** as we'll need them later on.


### DataStax Astra DB

As part of the workshop, You are provided with pre created database in DataStax AstraDB NoSQL and Vectorstore (ex., vectorcollections) and collections (ex., companyfaq - vector collection, orders and products - non vector collections).

- Note down the **API Endpoint** which can be found in the *Document*.
- Note down the **Application Token** for later use

Optionally, You can also follow the steps in Appendix section to create your own

### IBM watsonx.ai

As part of the workshop, You are provided with pre created Watson.ai in IBM environment and you should note down the below keys

- Note down the **watsonx.ai URL** which can be found in the *Document*. (typically `https://us-south.ml.cloud.ibm.com/`)
- Note down the **Project ID** for later use
- Note down the **API key** for later use

Optionally, You can also follow the steps in Appendix section to have a free environment on your own

### Get access to Langflow
There are several way to gain access to Langflow. Pick the one that suits you best 😊:
- (easiest 🤩) Just follow the instructions and use Github Codespaces in this tutorial

Optionally, You can also follow the below steps

- [Langflow desktop](https://www.langflow.org/desktop) (currently only for Mac)
- [Managed Langflow on Astra](https://astra.datastax.com/langflow)

### ⚡️ Open this tutorial on Github Codespaces
To make life easier, we'll use the awesome Github Codespace functionality. Github offers you a completely integrated developer experience and resources to get started quickly. How?

1. Open the [langflow-watsonx-agentic-workshop](https://github.com/krishnannarayanaswamy/langflow-watsonx-agentic-workshop) repository
2. Click on `Use this template`->`Ceate new repository` as follows:

    ![codespace](./assets/create-new-repository.png)

3. Now select **your** github account and name the new repository (any name or just use `langflow-watsonx-agentic-workshop`). Ideally also set the description. Click `Create repository`

    ![codespace](./assets/repository-name.png)

4. Cool! You just created a copy in your own Gihub account! Now let's get started with coding. Click `Create codespace on main` as follows:

    ![create-codespace](./assets/create-codespace.png)

5. Configure the secrets as follows:

- Copy `.env.example` to `.env`
- Edit `.env` and provide the required variables:
    - `WATSONX_PROJECT_ID` (your watsonx.ai project ID)
    - `WATSONX_API_ENDPOINT` (your watsonx.ai endpoint, e.g. `https://us-south.ml.cloud.ibm.com/`)
    - `WATSONX_API_KEY` (your watsonx.ai API key)
    - `ASTRA_DB_API_ENDPOINT` and `ASTRA_DB_APPLICATION_TOKEN`

    Example `.env`:
    ```env
    WATSONX_PROJECT_ID=your-watsonx-project-id
    WATSONX_API_ENDPOINT=https://us-south.ml.cloud.ibm.com/
    WATSONX_API_KEY=your-watsonx-api-key
    ASTRA_DB_API_ENDPOINT=your-astra-endpoint
    ASTRA_DB_APPLICATION_TOKEN=your-astra-token
    ```

    ![codespace](./assets/codespaces.png)

6. Now we can run Langflow as follows in the terminal window.

    ```bash
    pip install uv
    uv sync
    uv run langflow run --env-file .env
    ```

    This starts Langflow and opens a port to your Codespace in the cloud. In case you lose track of the URL to Langflow, just click on `PORTS` in the terminal window.

    Monitor the terminal window and wait for the localhost. If Langflow started successfully it should look like below.

    ```bash
        ✓ Initializing Langflow...
        ✓ Checking Environment...
        ✓ Starting Core Services...
        ✓ Connecting Database...
        ✓ Loading Components...
        ✓ Adding Starter Projects...
        ✓ Launching Langflow...

        ╭─────────────────────────────────────────────────────────────────────────╮
        │                                                                         │
        │  Welcome to Langflow                                                    │
        │                                                                         │
        │  🌟 GitHub: Star for updates → https://github.com/langflow-ai/langflow  │
        │  💬 Discord: Join for support → https://discord.com/invite/EqksyE2EX9   │
        │                                                                         │
        │  We collect anonymous usage data to improve Langflow.                   │
        │  To opt out, set: DO_NOT_TRACK=true in your environment.                │
        │                                                                         │
        │  🟢 Open Langflow → http://localhost:7860                               │
        │                                                                         │
        ╰─────────────────────────────────────────────────────────────────────────╯

    ```

    ⚠️ Ensure you set the Port Visibility to Public, especially important for the MCP server part later on in this tutorial!

    ![public-port](./assets/public-port.png)

🎉 Congrats! You finished the set-up part of the workshop. Now for the fun part!

## 📦 Workshop follow-along

### 0. 🧱 Make sure environment properties are added as variables

#### Steps:
1. Open Langflow
2. Click `Profile` and then `+ Settings` on the top right corner
3. Collapse `Global Variables` on the `General` tab on the left side
4. Make sure you have the below global variables configured
    ```env
    WATSONX_PROJECT_ID=your-watsonx-project-id
    WATSONX_API_ENDPOINT=https://us-south.ml.cloud.ibm.com/
    WATSONX_API_KEY=your-watsonx-api-key
    ASTRA_DB_API_ENDPOINT=your-astra-endpoint
    ASTRA_DB_APPLICATION_TOKEN=your-astra-token
    ```
5. If they are not pre configured, click on `Add New` button key in the `Name` and `Value`
6. For credentials like `Token` and `API Key` make sure that `credential` is selected

Once configured they should look as below

![credentials](./assets/credential.png)

### 1. 🧱 Setup of a simple Langflow Chatbot
**Goal:** Create a chatbot with: input → model → output

#### Steps:
1. Open Langflow
2. Click `+ New Flow` / `+ Blank Flow`
3. Collapse `Inputs` and drag the `Chat Input` component to the canvas
4. Click in `Search` bar for componets and search for `IBM` and drag the `IBM watsonx.ai` `Model` component to the canvas. Connect `Input` to the `Chat Input`
    - Ensure that the `API Endpoint`, `Project ID` and `API Key` have been set correctly by slecting the small `Globe` button
    - Select `ibm/granite-3-3-8b-instruct` as the `model`  (or try another one and check out the differences)
5. Collapse `Outputs` and drag the `Chat Output` component to the canvas. Connect `Message` to the `Chat Output` component

![chatbot-flow](./assets/chatbot-flow.png)

👏 Amazing! You just built your first flow. Let's run it by clicking `▶️ Playground` and asking the question:

    What's the difference between AI and Machine Learning?

You'll see **watsonx.ai** answer your question nicely!

Alternatively, you can import the prebuilt flow into Langflow [./flows/basic_chat_flow.json](./flows/basic_chat_flow.json))

#### Steps:
1. Navigate to [basic chat flow](https://github.com/krishnannarayanaswamy/langflow-watsonx-agentic-workshop/blob/main/flows/basic_chat_flow.json) and Download the flow to your local desktop.
1. Click `+ Starter Project` on the top next to your flow name to go to Home Page or Project Page.
2. On the left pane. Click on `Upload flow` button.
3. Select the flow (json file) that you downloaded.
4. If a flow with the same name already exists, langlfow will create another one.

### 2. 🧭 Familiarize yourself with Langflow
**Goal:** Understand the main components of Langflow

#### Steps:
1. If still open, close the Playground popup
2. Select `Chat Input` and click `Controls`
3. You'll see a field name `Input Text`, type `What's the difference between AI and Machine Learning` in the text field. close the popup
4. Select `Chat Output`, the three dots `...` and click `Expand`
5. Do the same with `Chat Input`
6. Click the play button `▶️` on the `Chat Output` component and see the flow run

Notice the running time of the separate components.  
Click the magnifying glass `🔍` in the `watsonx.ai` component. This shows you a popup with the intermediate data that is passed to the next component. Very useful for debugging purposes!

![chatbot-run](./assets/chatbot-run.png)

### 3. 🧠 Agentic AI with Langflow
**Goal:** Create an AI Agent that has access to a URL tool and a Calculator tool

![basic-agentic-ai](./assets/basic-agentic-ai.png)

#### Steps:
1. Reproduce the above flow (or load it from [./flows/basic_agentic_ai.json](./flows/basic_agentic_ai.json))
2. When you add the `Agent` component, make sure you click on `Model Provider` property to change to `Custom`.
3.For watsonx.ai, ensure that the `API Endpoint`, `Project ID` and `API Key` have been set correctly. Also make sure the output is set to `Language Model`.
4. Ensure the model is set to a chat-capable model, such as `meta/meta-llama-3-3-70b-instruct`
5. When adding the `URL` and `Calculator` components to the canvas, select them and click `Tool mode`
6. Connect all the components

👏 Amazing! You just built your first AI Agent. Let's run it by clicking `▶️ Playground` and asking the question:

    What is 2x the value of a Euro in Dollars, using the latest most recent data

You'll get an answer stating a value of $2.34 or something (as of July 2025).

To see the magic behind, simply click the down arrow `🔽`. The Agent decided to use two tools:
1. The URL tool to fetch the current exchange rates
2. The Calculator tool to multiply the value by two

![tool-usage](./assets/tool-usage.png)

### 4. 📚 Retrieval-Augmented Generation (RAG) with Astra DB
**Goal:** Adding external knowledge by making use of Vector Search and RAG

#### Steps: Add Astra DB as a RAG tool in Langflow

Extend your existing Basic Agentic AI flow with the following:
1. Collapse `Vector Stores` and drag `Astra DB` to the canvas
2. Click the component and select `Tool mode`
3. Make sure the `Astra DB Application Token` is configured, then select your `vectorcollections` database and `companyfaq` collection
4. Click the config button behind `Actions` and update the three `Tool descriptions` by replacing
    - `Ingest and search documents in Astra DB` with
    - `Answer frequently asked questions (FAQs) about shipping, returns, placing orders, and more`
    - ![astra-rag-agent](./assets/rag-edit-tools.png)
    - Click `Close x`
    - ⚠️ This essential step ensures the Agent understands to use this specific tool to search for FAQs
4. Connect the `Astra DB` component to the `Agent` component
5. Search for `IBM Watson.ai Embeedings` component and drag it to the canvas
6. For watsonx.ai embeddings component, ensure that the `API Endpoint`, `Project ID` and `API Key` have been set.
7. Select the `ibm/granite-embedding-278m-multilingual` as the embedding model.
8. Connect the embedding model component to AstraDB `Embedding model` section

![astra-rag-agent](./assets/astra-rag-agent.png)  
For ease of use, this flow is also available here: [./flows/rag-agentic-ai.json](./flows/rag_agentic_ai.json).


#### 🚀 Now let's run the Agent in Langflow
Browse back to Langflow, click on `▶️ Playground` and click on `+` on the left side to start a new Chat. Then run the following question:

    How many hours do I need to wait for a domestic order?

 🎉 You'll see the Agent making use of the Astra DB knowledge base to find relevant content and then running the calculator to calculate the amount of hours to wait.

 ![langflow-vector-search](./assets/langflow-vector-search.png)

#### Steps 🛠️🔍: Add Order Lookup to the agent 
1. Return to your Langflow flow
2. Collapse `DataStax` or search for Astra DB Tool and drag `Astra DB Tool` to the canvas
3. Configure as follows:
    - **Tool Name:** `OrderLookup`  
    - **Tool Description:** `A tool used to look up an order based on its ID`   
    - **Collection Name:** `orders`  
    - Ensure `Astra DB Application Token` and `API endpoint` are configured
    - Click `Open Table`, click `+` to add a field and update the field name to `orderNumber`, then click `Save` (this allows the tool to use the orderNumber column for queries)
4. Connect the `Astra DB Tool` component to the `Agent` component while selecting `Tool` as output mode.

#### Steps 🛠️🔍: Add Products Lookup to the agent 
1. Collapse `DataStax` or search for Astra DB Tool and drag `Astra DB Tool` to the canvas
2. Configure as follows:
    - **Tool Name:** `ProductLookup`  
    - **Tool Description:** `A tool used to look up a product based on its ID`   
    - **Collection Name:** `products`  
    - Ensure `Astra DB Application Token` and `API endpoint` are configured
    - Click `Open Table`, click `+` to add a field and update the field name to `productId`, then click `Save` (this allows the tool to use the productId column for queries)
3. Connect the `Astra DB Tool` component to the `Agent` component while selecting `Tool` as output mode.

#### Steps 💬: Instruct the Agent
Let's provide our Agent a bit more information about what it's capable of doing and what guardrails to take into account. This enables more accuracy for our Customer Support Agent.

1. On the `Agent` component, click the square at `Agent Instructions`
2. Paste the following instruction:

```text
You are a skilled customer service agent supporting Customer Service Employees to answer questions from customers. Your primary responsibility is to use the available tools to accurately address user inquiries and provide detailed, helpful responses. You can:

- Look up order numbers to retrieve and share order details. Keep in mind that the date is the order date and that price is in USD.
- Access product information to provide relevant descriptions or specifications based on the retrieved product ids.
- Use the Astra DB knowledge base about Frequently Asked Questions on shipping, returns, placing orders, and more. Always use this tool to find relevant content!
- Use the Calculator tool to perform basic arithmetic. Only use the calculator tool!
- Use the URL tool to find known information on the internet or APIs to make the response more accurate.

Think step by step. If answering a question requires multiple tools, combine their outputs to deliver a comprehensive response.
Feel free to iterate a few times!
Example: For an inquiry about canceling an order, retrieve the order and product details, and also reference the FAQ for the cancellation policy.
Example: If there are questiona that require arithmetic, make sure to invoke the Calculator tool.
Example: If there is a need for external up-to-date information, make sure to invoke the URL tool.

Always aim to deliver clear, concise, and user-focused solutions to ensure the best possible experience.
```

![customer-support-agent](./assets/customer-support-agent.png)  
For ease of use, this flow is also available here: [./flows/customer-support-agent.json](./flows/customer_support_flow.json).

🥳 You did it! You now have an Agentic Flow with access to:
- A company FAQ knowledgebase
- Orders data
- Product data
- A calculator
- Capability to fetch information from the internet

Let's run some queries. For instance:

- What's the shipping status of order 1001?
- What was ordered with 1003?
- What date will order 1004 arrive?
- How can I cancel order 1001 and what is the shipping policy?
- What's the amount of order 1005 in Euros?
- The customer paid 110 euros for order 1001, how much should we return?

Observe how all the different tools are being used to answer the user's questions.

### 5. 📱 Create an external app that call the Langflow REST Endpoint
In this step we'll create a simple Python app that runs the Langflow flow.

#### Steps: Use the Langflow API endpoint in Python
1. Have a look at the app.py file. this is a frontent `Streamlit` application that will call the Langflow API to trigger our Customer support Agent we just created and tested in the `Playground`. 
2. In Langflow exit the Playground and click on `Share` in the right top corner and then click `API Access`
3. Click on `Python`
4. Copy the UUID of the flow (for ex., `b32e8180-f3c8-465f-91b4-3d67efb68774`) and save it
5. Open a new shell window where will will run this Streamlit applucation. Remmeber to keep the old window with langflow running.
6. export the flow id variable that `app.py` uses as below
```env
export LANGFLOW_FLOW_ID=<your flow id>
```
7. Go to Langflow and in the same API Access window in Langflow now click the `create and API key` link
5. In the new window, click `+ Add New`, type a description (e.g. Support Agent) and click `Generate API Key`
8. Make note of the generated API Key
    - ⚠️ This is the only time you'll see it, so make sure you save it somewhere handy!
9. Export the Langflow key that the `app.py` uses to run your flows
```env
export LANGFLOW_API_KEY=<your key generated above>
```
![langflow-python-api](./assets/langflow-python-api.png)

Let's run it!

```bash
pip install streamlit
pip install dotenv
streamlit run app.py
```

As a response you'll see a JSON structure that contains the actual answer and additonal metadata.  
The answer you're probably looking for is located inside the JSONPath `$.outputs[0].outputs[0].results.message.text`.

If you change line 31 to the following, you'll see the actual response: `print(response.json()['outputs'][0]['outputs'][0]['results']['message']['text'])`


![streamlit-front-end](./assets/streamlit-front-end.png)

### 7. Publish as an MCP server
MCP helps you build agents and complex workflows on top of LLMs. LLMs frequently need to integrate with data and tools, and MCP provides:
- A growing list of pre-built integrations that your LLM can directly plug into
- The flexibility to switch between LLM providers and vendors
- Best practices for securing your data within your infrastructure

Langflow easily enables you to publish your flows as an MCP server. Let see how!

#### Steps: Publish the flow as MCP server
1. Go back to your Langflow canvas
2. Click `Share` and `MCP Server`
3. Click `Edit Tools` and ensure the Tool name and Tool description describe something meaningful
    ![mcp-tool](./assets/mcp-tool.png)
3. Select the `JSON` tab and your preferred environment
4. Copy the configuration and store it somewhere for later use

#### Steps: Enable Claude desktop to call Langflow MCP
1. Download [Claude Desktop](https://claude.ai/download)
2. Create a (free) account and sign in
3. Click `Settings` and `Developer`
4. Click `Edit Config`, right-click `claude_desktop_config.json` and open it with your favorite IDE
5. Now paste the JSON contents from Langflow and save it
    - ⚠️ You may need to define the full path to `uvx`. On a linux based environment use `which uvx` to find out where it's installed. See the example below:

    ![claude-config](./assets/claude-config.png)

6. Restart Claude desktop to load the new config and allow Claude to read the tool availability

🥳 You did it! You now have access to the Langflow flow through a MCP client. Run a query like:

    What's the status of my order 1001

And you'll get something back like:

![claude-desktop](./assets/claude-desktop.png)

💡 There's more information about working with MCP in the [Claude developer docs](https://modelcontextprotocol.io/quickstart/user).

💡 In case you want to debug MCP servers or understand what's happening on the line, you can use [MCP Inspector](https://github.com/modelcontextprotocol/inspector). It's easy to run it through:

```sh
npx @modelcontextprotocol/inspector
```

##Appendix: Take home Playground

#### AstraDB: Set up an environment on your own

Make sure you have a vector-capable Astra database (get one for free at [astra.datastax.com](https://astra.datastax.com))
- Sign up or log in
- Click `Databases` and click `Create Database` 
- Select `Serverless (Vector)`, type a database name, i.e. `support_agent` and select `AWS` as Cloud Provider and `us-east-2` as Region
    - ⚠️ Please stick to these settings as that enables us to use the [Astra Vectorize](https://www.datastax.com/blog/simplifying-vector-embedding-generation-with-astra-vectorize) functionality
- Wait a few minutes for it to provision
- Note down the **API Endpoint** which can be found in the right pane underneath *Database details*.
- Click on `Generate Token` and give it a name, i.e. `support_agent-token` and click `Generate`. Now click on the copy button and paste the **Application Token** somewhere for later use

    ![astradb](./assets/astra-new-db.png)

#### IBM Watsonx.ai: Set up an environment on your own

- Go to [IBM watsonx.ai](https://www.ibm.com/products/watsonx-ai) and sign up for a free trial or log in.
- Once logged in, create a new project (or use an existing one).
- Navigate to the API Keys section (Administration → Access (IAM) → API keys) and create a new **API key**. Save this key securely.
- Note your **Project ID** (found in your watsonx.ai project details).
- You will also need the **watsonx.ai URL** (typically `https://us-south.ml.cloud.ibm.com/` or as shown in your project dashboard).
- For more details, see the [watsonx.ai documentation](https://dataplatform.cloud.ibm.com/docs/content/wsj/analyze-data/ml-authentication.html?context=wx).

    *(Optional: If you need a service instance, create a new instance of "watsonx.ai" in the IBM Cloud console and bind it to your project.)*

    *(You may also need to enable the appropriate LLM model in your project, such as `ibm/granite-3-3-8b-instruct` or similar.)*

#### Preparation: Creating a FAQ collection
In this step we'll create a new collection in your `support_agent` database in [Astra DB](https://astra.datastax.com) to store data as a knowledge base.

First we need to create a collection to store the data:
1. Browse to your `support_agent` database on [Astra DB](https://astra.datastax.com)
2. Click `Data Explorer` and click `Create Collection +`
3. Type your collection name, i.e. `company_faq` and enable `Vector-enabled collection`
4. Leave NVIDIA, NV-Embed-QA, 1024 and Cosine as it is
5. Click `Create Collection`

![astra-new-collection](./assets/astra-new-collection.png)

You just created a new empty collection to store knowledge base articles.

#### Steps: Add some articles to our knowledge base
Extend your flow with the following additional flow (scroll down a bit for a blank piece of canvas):
1. Collapse `Data` and drag `File` to the canvas
2. Click on `Upload a file` and upload [./data/Company_FAQ.pdf](./data/Company_FAQ.pdf) from this repository (you'll have to download it first)
3. Collapse `Processing` and drag `Split Text` to the canvas
4. Set the `Chunk size` to 500 (because NV-Embed-QA only allows 512 tokens at maximum) and `Chunk overlap` to 100 (so that every chunk has a bit of information from the previous one)
5. Collapse `Vector Stores` and drag `Astra DB` to the canvas
6. Make sure the `Astra DB Application Token` is configured, then select your `support_agent` database and `company_faq` collection
7. Click the play button `▶️` on the `Astra DB` component, see the flow run and observe the time consumed. You can also click the intermediate magnifying glasses `🔍` to debug the flow.

![astra-ingest](./assets/astra-ingest.png)

🙌 Congrats! You just loaded a PDF, converted it to plain text, chunked it and loaded it into a Vector enabled collection in Astra DB!

#### 👀 Check: Have a look at the data in Astra DB
In this step we'll have a look at the dataset in your `support_agent` database in [Astra DB](https://astra.datastax.com).

1. Browse to your `support_agent` database on [Astra DB](https://astra.datastax.com)
2. Click `Data Explorer` and click `company_faq`
3. Observe the data loaded into the collection on the right side of the screen
4. Toggle from `Table` to `JSON` view and collapse some of the rows to see what's inside

To see Vector Search in action, type the following in the text box `Search`:

    What are your shipping times

You'll see the chunk with shipping times show up as the first result. You just ran an Approximate Nearest Neighbor (ANN) search transparantly utilizing the Vectorize functionality in Astra DB that does the vectorization for you on demand.

![astra-vector-search](./assets/astra-vector-search.png)

### 5. 📈 Adding structured data
**Goal:** Every enterprise has structured data alongside unstructured data and often these are related to each other. This step enables your Agent to retrieve structured order and product details.

#### Preparation: Add Orders data
1. Browse to your `support_agent` database on [Astra DB](https://astra.datastax.com)
2. Click `Data Explorer` and click `Create Collection +`
3. Name the collection `orders`, disable the `Vector-enabled collection` switch and click `Create Collection`
4. Click `Load Data` and upload the file: [./data/sample_orders.csv](./sample_orders.csv)
5. Verify the data was loaded into Astra DB

#### Preparation: Add Products data
1. Click `Create Collection +`
2. Name the collection `products`, disable the `Vector-enabled collection` switch and click `Create Collection`
3. Click `Load Data` and upload the file: [./data/sample_products.csv](./sample_products.csv)
4. Verify the data was loaded into Astra DB
