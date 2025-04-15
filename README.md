Below is the complete README.md in proper Markdown format. You can copy the content into a file named README.md and adjust any paths or details as needed.

⸻



# SQLite MCP Server

This repository contains an MCP (Model Context Protocol) server written in Python that connects to a SQLite database containing startup funding data. The server exposes table schemas as resources, provides a read-only SQL query tool, and offers prompt templates for common data analysis tasks. It is designed to work with MCP clients and language models (LLMs) and communicates via the STDIO protocol.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Setup and Installation](#setup-and-installation)
  - [Creating the Sample SQLite Database](#creating-the-sample-sqlite-database)
  - [Creating a Virtual Environment](#creating-a-virtual-environment)
  - [Running the MCP Server](#running-the-mcp-server)
  - [Installing into Claude Desktop](#installing-into-claude-desktop)
- [Usage](#usage)
- [Testing](#testing)
- [Logging](#logging)
- [License](#license)

## Overview

The MCP server uses the [MCP Python SDK](https://pypi.org/project/mcp/) (with CLI extras) to implement a server that:

- **Connects to a SQLite database.** For example, a database with startup funding information.
- **Exposes table schemas as MCP resources.**
- **Provides a tool for executing read-only SQL queries.**
- **Offers prompt templates** that help language models generate data analysis insights.
- **Communicates via the STDIO protocol** by reading JSON-RPC messages from standard input (stdin) and writing responses to standard output (stdout).

## Features

- **Resources:**  
  - `schema://sqlite/{table}`: Returns the SQL schema (CREATE TABLE statement) for a specific table.
  - `schema://sqlite/all`: Returns a JSON-formatted mapping of all table schemas.
  
- **Tools:**  
  - `sql_query`: A tool for executing read-only SQL queries. Only queries starting with `SELECT` are allowed.
  
- **Prompts:**  
  - `analyze_table_prompt`: Generates an analysis prompt for a specific table.
  - `describe_query_prompt`: Generates a prompt to explain what a given SQL query does.
  
- **STDIO Protocol:**  
  This MCP server uses the STDIO transport. It reads from stdin and writes responses to stdout, making it easy to run as a subprocess or integrate with tools such as the MCP Inspector.
  
- **Logging:**  
  Extensive logging (using Python’s logging module) is provided to trace server activity, debug errors, and monitor interactions. By default, logs are sent to stderr.

## Setup and Installation

### Creating the Sample SQLite Database

Create a database (e.g., `startups.db`) that contains startup funding data. Save the following script as `create_db.py` and run it to generate the database:

```python
import sqlite3

conn = sqlite3.connect("startups.db")
cursor = conn.cursor()

# Create a table for startup funding information.
cursor.execute('''
CREATE TABLE IF NOT EXISTS startups (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    startup_name TEXT NOT NULL,
    description TEXT,
    website TEXT,
    funding_amount REAL,
    funding_date TEXT,
    investors TEXT
)
''')

# Insert sample records.
startups = [
    ("AlphaTech", "Innovative AI startup", "https://alphatech.io", 5000000, "2023-05-15", "Investor A, Investor B"),
    ("BetaSoft", "Enterprise SaaS solution", "https://betasoft.com", 12000000, "2023-06-20", "Investor C"),
    ("Gamma Innovations", "Cutting-edge biotech research", "https://gammainnovations.org", 7500000, "2023-07-10", "Investor D, Investor E, Investor F"),
    ("Delta Ventures", "Fintech disrupting traditional banking", "https://deltaventures.net", 20000000, "2023-08-25", "Investor G"),
    ("Epsilon Dynamics", "Sustainability through green energy", "https://epsilondynamics.com", 10000000, "2023-09-05", "Investor H, Investor I"),
]

cursor.executemany('''
    INSERT INTO startups (startup_name, description, website, funding_amount, funding_date, investors)
    VALUES (?, ?, ?, ?, ?, ?)
''', startups)

conn.commit()
conn.close()

Run the script using:

python create_db.py

Creating a Virtual Environment

It is recommended to use a virtual environment for dependency isolation:
	1.	Create a Virtual Environment:

python -m venv venv


	2.	Activate the Virtual Environment:
	•	On macOS/Linux:

source venv/bin/activate


	•	On Windows:

venv\Scripts\activate


	3.	Install Dependencies:

pip install "mcp[cli]"



Running the MCP Server
	1.	Save the Server Code:
Save your MCP server code (see sqlite_mcp_server.py) in a file called sqlite_mcp_server.py.
	2.	Run the Server:
Execute the following command to run the MCP server (which communicates over the STDIO protocol):

python sqlite_mcp_server.py

This starts the MCP server as a standalone process that listens on stdin and writes to stdout. Log messages will be printed to your terminal’s stderr.

	3.	Optional – Running with UV:
If you use uv, you can run the server in development mode:

uv run sqlite_mcp_server.py



Installing into Claude Desktop

An installation script (install_to_claude.py) is provided to register the MCP server with Claude Desktop.
	1.	Save the Installation Script:
Save the installation script as install_to_claude.py.
	2.	Run the Installation Script:

python install_to_claude.py

This will update Claude Desktop’s configuration (typically located at:
	•	macOS: ~/Library/Application Support/Claude/claude_desktop_config.json
	•	Windows: %APPDATA%\Claude\claude_desktop_config.json)
The script adds an entry like:

{
    "mcpServers": {
        "sqlite_mcp_server": {
            "command": "python",
            "args": ["-u", "/absolute/path/to/sqlite_mcp_server.py"]
        }
    }
}


	3.	Restart Claude Desktop:
After updating the configuration, restart Claude Desktop so it launches your MCP server automatically.

Usage

Once the MCP server is running (directly via the command line or via Claude Desktop), MCP clients (or an LLM orchestrator) can use it to:
	•	Access Resources:
Fetch the schema for all tables via schema://sqlite/all or for an individual table, for example, schema://sqlite/test.
	•	Invoke Tools:
Execute the sql_query tool to run read-only SELECT queries (e.g., to list startups or filter by funding amount).
	•	Use Prompts:
Prompt templates such as analyze_table_prompt and describe_query_prompt help generate natural language instructions that the LLM uses to decide which tool to call.

For example, if a user asks:
“List startups with funding over $10 million”
the LLM might decide to invoke the sql_query tool with:

SELECT * FROM startups WHERE funding_amount > 10000000;

##Testing

A comprehensive client test script (sqlite_mcp_client_tests.py) is provided to demonstrate use-case tests that interact with the MCP server. The tests include:
	•	Listing resources.
	•	Retrieving specific table schemas.
	•	Executing valid and invalid SQL queries.
	•	Invoking prompt templates.

Run the test script using:

python sqlite_mcp_client_tests.py sqlite_mcp_server.py

This command starts the server as a subprocess over STDIO and runs all use-case tests, printing results and log information to the terminal.

##Logging

The server (and installation script) uses Python’s built-in logging module. By default:
	•	Logging messages are output to stderr (displayed in your terminal).
	•	To log to a file instead, modify the logging.basicConfig call in the scripts:

logging.basicConfig(
    filename='mcp_server.log',
    level=logging.DEBUG,
    format="[%(asctime)s] %(levelname)s - %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)



##License

This project is licensed under the MIT License. See the LICENSE file for details.

---

### Additional Notes

- Adjust any absolute paths in the installation script and README as necessary.
- Ensure that your SQLite database (`startups.db`) is correctly created before running the MCP server.
- The MCP server works on the STDIO protocol, making it easy to run as a subprocess or integrate with development tools like MCP Inspector or Claude Desktop.

This README provides clear instructions on how to set up your virtual environment, run your MCP server, and install it into Claude Desktop, as well as outlining how to test and use the server. If you need further modifications or additional sections, please let me know!