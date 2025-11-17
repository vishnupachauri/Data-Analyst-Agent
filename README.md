📌 Data-Analyst-Agent — Project Overview

Data-Analyst-Agent is an autonomous, LLM-driven, multi-agent analytics system built on top of Llama 3, LangChain, Databricks SQL, and the Medallion Architecture (Bronze → Silver → Gold).
It enables users to ask natural-language questions about sales data and receive accurate SQL-driven answers, enriched visualizations, and safe, validated responses.

The system combines:

Intelligence Layer → LLM reasoning, query validation, multi-agent collaboration

Execution Layer → Text-to-SQL conversion, SQL querying, visualization

Data Foundation Layer → Clean data pipeline using Bronze/Silver/Gold Delta tables

This layered design ensures trustworthiness, modularity, and end-to-end automation.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------


📘 01_Environment_Setup — Environment Initialization

This script configures all essential resources required for the full AI data analysis system. It prepares Spark, Databricks, Hugging Face, the filesystem, and Unity Catalog paths.

It performs four major responsibilities:

1. Spark Initialization

Launches a Spark session called LLMAgentDataAnalysis.

Spark will be responsible for reading the raw CSV file, applying transformations, and writing Bronze/Silver/Gold Delta tables.

Why:
Spark provides distributed processing, schema inference, and Delta Lake integration — essential for building a reliable data foundation.

2. Configuration of Paths and Data Locations

The script defines:

Input dataset path (raw CSV stored in Databricks Volumes)

Output paths for:

Bronze table — raw structured data

Silver table — cleaned data

Gold table — aggregated business KPIs

Databricks SQL workspace details:

Host URL

SQL Warehouse ID

HTTP path

Unity Catalog catalog & schema used for table registration

Why:
A consistent configuration layer prevents hardcoding, simplifies orchestration, and ensures all downstream notebooks read from the same environment.

3. Secure Retrieval of API Tokens

Uses Databricks Secrets to load:

Hugging Face API key (model access)

Databricks personal access token (SQL warehouse authentication)

If secrets are unavailable, the system loads safe fallback values exclusively for development/testing.

Why:
Secure credential management is mandatory for production systems to prevent key exposure and unauthorized access.

4. Structured Output for Downstream Notebooks

Before exiting, the script bundles all configuration values into a JSON object and returns it using:

dbutils.notebook.exit()


This JSON serves as the contract between setup and execution notebooks.

Why:
Databricks notebooks cannot pass Python objects directly — JSON ensures interoperability and modularization.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------


📘 02_Data_Pipeline_Medallion_Architecture — Complete ETL Pipeline

This notebook implements the full Medallion Architecture — a layered, scalable design for reliable data transformation.
It turns raw CSV data into high-quality business KPIs.

1. Importing Global Settings

Receives JSON returned by the setup notebook

Extracts warehouse, dataset paths, catalog/schema, and token values

If values are missing, safely assigns fallback defaults

Benefit:
Ensures all ETL steps synchronized with consistent configuration.

2. Bronze Layer — Raw Ingestion & Standardization
Operations:

Load the raw CSV file from Databricks Volumes

Apply column-name normalization:

Spaces → underscores

Lowercasing

Removal of invalid characters

Register the Bronze table as a Delta table using Unity Catalog

Why:

Bronze stores unaltered data, but with standardized schema

Makes ingestion reproducible and queryable

Ensures all transformations are traceable

3. Silver Layer — Cleansed, Conformed, and Trusted
Transformations include:

Convert string timestamps into real timestamp type

Calculate total_amount = quantity * price

Remove cancelled transactions (invoice IDs beginning with “C”)

Rename columns for consistency (e.g., invoice → invoice_id)

Replace missing customer IDs with placeholder values

Cast columns to correct data types

Why:

Ensures data correctness, consistency, and readiness for analytics

Silver is your single source of truth for all KPI calculations

The resulting Silver table is stored as a Delta table with improved schema.

4. Gold Layer — Aggregated Business KPIs
KPIs generated:

Monthly sales period (YYYY-MM)

Total quantity sold

Total revenue

Number of transactions

Grouped by:

Month

Country

Stock code

Product description

Why:

Gold tables represent high-value analytics consumed by dashboards, BI tools, and LLM agents.

5. Finalization

Outputs a pipeline completion message for orchestration tools or workflow jobs.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------


📘 03_Multi_Agent_System_Definition — AI Agent Construction

This notebook constructs the two-agent system that powers conversational analytics.

🧩 Block 1: Imports & Environment Preparation

Loads all supporting frameworks:

LangChain (agent orchestration)

Hugging Face Hub (LLM loading)

Databricks SQL connector

Spark

Pandas, Matplotlib

Logging

Ensures SQL connector is installed and Spark is available.

Role:
Prepares the environment for LLM reasoning + SQL execution + visualization.

🧩 Block 2: Load Configurations

Extracts:

API tokens

SQL warehouse info

Catalog and schema

Model details

Includes validation checks to ensure all required parameters are non-empty.

Role:
Ensures the system has all credentials and paths needed to interact with Databricks and Hugging Face.

🧩 Block 3: Initialize Llama 3

Loads Llama 3 (8B Instruct) with:

Low temperature (0.1) for consistent, factual behavior

Token limit optimized for short analytical answers

Role:
Primary reasoning engine for both agents.

🧩 Block 4: Input Validator Agent

A lightweight LLMChain trained to label each user query as:

VALID → related to sales analysis

INVALID → off-topic, unsafe, or irrelevant

This prevents misuse and keeps the system domain-focused.

🧩 Block 5: Databricks SQL Connection

Creates an authenticated connection to the Databricks SQL Warehouse.

Role:
Allows the agent to run SQL queries on Bronze/Silver/Gold tables.

🧩 Block 6: Visualization Tool

A custom tool that:

Converts SQL query output into a DataFrame

Decides the best visualization type (line, bar, scatter)

Generates charts using matplotlib

Returns the chart as Base64-encoded HTML

Role:
Transforms raw data into visual insights, automatically.

🧩 Block 7: SQL Toolkit + Memory

Combines:

SQL database

SQL generator tools

Visualization tool

Conversation memory

Role:
Supports context-aware querying and follow-up questions.

🧩 Block 8: Data Analyst Agent

Uses LangChain’s create_sql_agent() to:

Convert natural-language questions to SQL

Execute SQL on Databricks

Generate plots

Produce analytical summaries

Role:
This is the main intelligence for answering the user’s analytical questions.

🧩 Block 9: JSON Configuration Output

Exports all rebuild parameters so the orchestrator notebook can reconstruct the agents.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
📘 04_Main_Orchestrator_and_Testing — Full System Execution

This notebook is the central execution hub.
It rebuilds the agents and exposes one unified function to run the entire system.

Block 1: Rebuild All Agents

Calls:

01_Environment_Setup → fetch environment variables

03_Multi_Agent_System_Definition → fetch agent configuration

Recreates:

LLM (Llama 3)

Validator Agent

Data Analyst Agent

SQL connection

Visualization tool

Toolkit + memory

Purpose:
Separates agent definition from agent execution, improving modularity and reproducibility.

Block 2: run_multi_agent_system() Function

Steps:

Validator Agent checks if query is valid

If valid:

Send to Data Analyst Agent

Convert to SQL

Execute on Databricks

Generate plot if needed

Return results

If invalid:

Return a safe fallback message

Purpose:
Provides a clean and controlled API for all user queries.

Block 3: Test Suite

Includes:

Sales aggregations

Top-selling products

Country comparisons

Monthly trend analysis

Schema inspection

Invalid/irrelevant queries

Purpose:
Demonstrates complete workflow correctness — validation → SQL → visualization.

Final Summary

The orchestrator ensures:

Clean reproduction of the multi-agent system

Consistent end-to-end query handling

A modular architecture ready for deployment

It is the user-facing entry point to the entire Data-Analyst-Agent platform.
