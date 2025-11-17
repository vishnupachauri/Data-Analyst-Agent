# Data-Analyst-Agent
This project builds an autonomous multi-agent system for conversational data analysis using Llama 3 within a Medallion Architecture. It validates user queries, generates SQL/Python insights, and delivers visualizations, enabling safe, accurate, and accessible analytics for all users.

---------------------------------------01_Environment_Setup ---------------------------------------
This script is designed to set up the environment for a data-analysis workflow using Spark, Databricks, Hugging Face, and Unity Catalog.
It performs three major tasks:

1. Initialize a Spark Session

The script starts a Spark application called “LLMAgentDataAnalysis”, which will be used to process and transform data.

2. Define All Important Paths and Configurations

It specifies:

The input dataset location (a CSV file in Databricks Volumes).

The Bronze, Silver, and Gold table output paths for the Medallion architecture.

The Databricks connection details, such as host, warehouse ID, and SQL endpoint.

The Unity Catalog catalog and schema where tables will be stored.

This helps the rest of the pipeline know where data is coming from and where processed data should be saved.

3. Retrieve API Tokens Securely

The code tries to fetch two secret tokens from Databricks Secrets:

Hugging Face API token

Databricks API token

If the secrets are found, they are loaded securely into the environment.

If not, the script falls back to hard-coded tokens (with a warning).
This ensures the notebook still runs—even though it's insecure for production.

4. Return All Configuration Values

At the end, the script uses:

dbutils.notebook.exit()


to return a JSON object containing all the important configurations and tokens.

This allows other notebooks or agents to use these values as input to continue the pipeline.

----------------------------------------------------------------------------------------------------


-----------------------02_Data_Pipeline_Medallion_Architecture ------------------------------

This notebook processes your retail dataset using the Medallion Architecture, which has three layers: Bronze, Silver, and Gold. Each layer improves the quality and structure of the data.

It uses the environment/settings provided by the previous notebook, 01_Environment_Setup.

1. Retrieve Environment Variables

The notebook first loads all the important configuration values passed from the previous setup notebook (paths, catalog, schema, API keys, etc.).
If the values cannot be retrieved, it sets fallback defaults so the notebook can still run.

Purpose:
Ensure every part of the pipeline uses the same consistent settings.

2. Bronze Layer – Raw Ingestion

The notebook:

Reads the raw CSV file from a Unity Catalog Volume.

Cleans column names (spaces, symbols, uppercase → standardized format).

Saves this unprocessed but standardized dataset as a Bronze Delta table.

Purpose:
Store the raw data in a stable, structured, and queryable format, but without modifying the actual values.

3. Silver Layer – Cleaned & Standardized Data

From the Bronze table, the notebook:

Converts the invoice date into a proper timestamp.

Calculates the total amount for each transaction.

Removes cancelled invoices (based on invoice IDs starting with “C”).

Renames columns to consistent names.

Fixes missing customer IDs by replacing null values with a placeholder.

It then saves this transformed dataset as a Silver Delta table.

Purpose:
Clean and standardize the data so it becomes reliable and ready for analytics.

4. Gold Layer – Business KPIs

From the Silver table, the notebook computes:

Monthly sales (year–month).

Total quantity sold.

Total sales amount.

Number of transactions.

It groups these metrics by month, country, product code, and description, and stores them in the Gold Delta table.

Purpose:
Produce a business-ready dataset containing key performance indicators that can be used for dashboards, reporting, or insights.

5. Completion Signal

At the end, the notebook sends a success message so that any workflow or orchestration tool knows the pipeline ran successfully.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------

------------------------------03_Multi_Agent_System_Definition-----------------------------------


This notebook defines and initializes a multi-agent system for data analysis using LLM (Llama 3) and Databricks SQL.
It mainly sets up two agents —
(1) an Input Validator Agent that filters valid user queries, and
(2) a Data Analyst Agent that performs Text-to-SQL and creates visualizations.
Finally, it returns all configurations as JSON for use in the orchestrator notebook.”

🧩 Block 1: Imports and Environment Setup

“In the first part, we import all required libraries — like Spark, LangChain, HuggingFaceHub, pandas, matplotlib, and Databricks SQL connector.
Then, we make sure that the Databricks SQL connector is installed and start a Spark session if not already running.
Logging is also configured so that we can track each process step and error clearly.”

🗣 Key point to mention:

“This part ensures that the entire environment is ready for LLM + SQL + visualization work.”

⚙️ Block 2: Retrieve Configuration Variables

“This block retrieves global variables that were passed from the previous notebook, such as API tokens, Databricks host, warehouse ID, catalog, and schema.
It uses dbutils.widgets to read a JSON string and then extracts and validates the required variables.”

🗣 If asked why validation is needed:

“We check that all critical parameters are not empty — because missing tokens or URLs would break our database connection.”

🧠 If run locally, fallback values are provided using environment variables for development only.

🤖 Block 3: Initialize the Large Language Model

“Here, we initialize the LLM — Meta-Llama-3-8B-Instruct — from Hugging Face Hub.
The temperature is kept low (0.1) for more factual, deterministic answers, and we limit the maximum tokens to 512 for efficiency.”

🗣 Key point:

“This is the main brain that both agents will use for understanding and reasoning.”

🧠 Block 4: Input Validator Agent

“This is the first agent. It uses a simple LLMChain with a prompt that checks if the user’s query is valid or invalid.
A query is valid only if it asks something related to sales, products, or transactions — otherwise, it’s rejected.”

🗣 Example to say:

“For example, if someone says ‘Show me monthly sales by country,’ it’s VALID.
But if they say ‘Hi, how are you?’ or something malicious, it’s INVALID.”

✅ Purpose: Protects the system from irrelevant or risky inputs.

🧱 Block 5: Databricks SQL Connection

“This block connects the system to the Databricks SQL warehouse.
We extract the hostname from the Databricks URL and then use SQLDatabase.from_databricks() to connect using catalog, schema, and tokens.”

🗣 Explain simply:

“This connection allows the agent to query real data stored in Databricks using SQL.”

📊 Block 6: Python Visualization Tool

“This is a custom LangChain tool that takes the SQL query output and creates a visualization.
It tries to read the data as CSV, and if that fails, it tries a space-separated format using regex.”

🗣 Explain logic:

“Once the data is parsed into a DataFrame, it decides which chart to make automatically:

Line chart for monthly sales

Bar chart for top countries or products

Scatter plot or generic bar chart for other cases.
It then encodes the chart as a Base64 image and returns it as HTML so it can be displayed directly.”

✅ Purpose: Converts raw query results into readable visual insights automatically.

🧰 Block 7: SQL Toolkit and Memory Setup

“Here, we combine the LLM with the SQLDatabase using LangChain’s SQLDatabaseToolkit.
We add our visualization tool to it, and also use ConversationBufferMemory — so the agent remembers context between user queries.”

🗣 Say:

“This memory helps in follow-up questions like ‘now show me only the top 5.’”

🧮 Block 8: Create the Data Analyst Agent

“Now we create the main Data Analyst Agent using create_sql_agent().
This agent uses the LLM, the SQL toolkit, and our custom visualization tool.
It can automatically convert a natural language question into an SQL query, execute it on Databricks, and visualize the result.”

🗣 In short:

“This is the heart of the system — it performs text-to-SQL and visual analysis autonomously.”

🧾 Block 9: Return JSON Configuration

“Finally, the notebook doesn’t return Python objects directly — because Databricks notebooks can’t pass complex objects using dbutils.notebook.run().
Instead, we prepare a JSON payload with all model and database configuration details.
Then we exit using dbutils.notebook.exit() so the next notebook can rebuild the agents.”

🗣 Summarize:

“This makes our system modular — setup and orchestration are separate, which is good for scalability.”

🧠 Final Summary to Say at End

“So overall, this notebook sets up two agents —
one for validation and another for analysis and visualization — both powered by Llama 3 and Databricks SQL.
It ensures clean environment setup, secure variable handling, intelligent visualization, and easy integration with the orchestrator notebook."

"""
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

------------------------------04_Main_Orchestrator_and_Testing----------------------------------------


This notebook is the main orchestrator for running the multi-agent data analysis system.
It re-initializes both agents from prior notebooks, exposes a single run function to process queries end-to-end, and includes test cases to demonstrate validation, SQL answering, and visualization.

🧩 Block 1: Re-Initialization (Configs → Agents)
“First, we call 01_Environment_Setup to fetch fresh config (tokens, hosts, warehouse info).
Then we call 03_Multi_Agent_System_Definition, which returns a JSON payload with all rebuild parameters.
Using this, we re-create:
• the LLM (Llama-3 via HuggingFace)
• the Input Validator Agent (VALID/INVALID gatekeeper)
• the Databricks SQL connection (catalog, schema, warehouse)
• the custom visualization tool
• the SQL toolkit + memory
• and the Data Analyst Agent (Text-to-SQL + tool-calling).”
🗣 Key point to mention:
“This notebook doesn’t redefine logic—it rebuilds the exact agents from configuration, ensuring reproducibility and clean separation of concerns.”
⚙️ Block 2: Orchestrator Function (run_multi_agent_system)
“This function accepts a user query and runs the full pipeline:


Sends the query to the Validator Agent.


If the result is VALID, invokes the Data Analyst Agent which executes SQL and (when appropriate) produces a visualization.


Uses Databricks display to show rich outputs.
If the query is INVALID, it returns a polite scope message.”


🗣 Why this design?:
“By centralizing validation and analysis in one call, the orchestrator delivers a consistent, controlled entry point for all queries.”
🧪 Block 3: Test Cases (Demo Suite)
“This section runs a variety of queries to showcase behavior:
• Valid numeric aggregation: ‘total sales for 2011’
• Valid categorical top-N: ‘top 5 most sold products’
• Country breakdown: ‘sales amount per country’
• Explicit visualization requests
• Monthly trend plotting
• Schema discovery (tables and schemas)
• Off-topic / greetings to show rejections by the validator.”
🗣 Key point:
“These tests prove the entire flow—validate → analyze → visualize/display—and demonstrate both happy paths and guardrails.”

🧠 If someone asks “what actually changes here vs the definition notebook?”
“03_* defines the agents and returns a JSON contract; 04_* rebuilds those agents from that contract and runs them with real queries.”
🗣 Example to say (for the function):
“If a user asks, ‘Show me the total sales amount for each country, and visualize it,’ the orchestrator verifies it’s a valid retail query, then the analyst agent generates SQL, fetches results from Databricks, and triggers the visualization tool to return a plot embedded as HTML.”

✅ Final Summary to Say at End
“Overall, this orchestrator notebook reconstructs the agents from configuration, exposes a clean run function for end-to-end processing, and demonstrates the system through comprehensive test cases. It ensures modular architecture (definition vs execution), repeatability, and a clear user pathway from query validation to SQL analysis and visualization.”


