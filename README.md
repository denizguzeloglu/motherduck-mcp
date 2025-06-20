MotherDuck MCP Server - Detailed Installation Guide
This guide explains the step-by-step manual installation of the MotherDuck MCP Server on Windows.

📋 Table of Contents
Requirements

Manual Installation

Configuration Options

Claude Desktop Integration

Usage Examples

Troubleshooting

🔧 Requirements
System Requirements
Operating System: Windows 10/11 (Linux and macOS also supported)

Python: Version 3.10 or higher (tested with 3.12.7)

Git: To clone the repository

Claude Desktop: Used as the MCP client

Python Packages
duckdb==1.3.0

mcp>=1.9.4

python-dotenv

Other dependencies will be installed automatically

🚀 Manual Installation
Step 1: Clone the Repository
bash
Copy
Edit
# Create the project directory
mkdir D:\AOT
cd D:\AOT

# Clone the repository
git clone https://github.com/motherduckdb/mcp-server-motherduck.git

# Enter the project directory
cd mcp-server-motherduck
Step 2: Create Python Virtual Environment
bash
Copy
Edit
# Create virtual environment
python -m venv venv

# Activate (choose depending on your terminal):
# For PowerShell:
.\venv\Scripts\Activate.ps1

# For Command Prompt:
venv\Scripts\activate.bat

# For Git Bash:
source venv/Scripts/activate
Note: If you get an execution policy error in PowerShell:

powershell
Copy
Edit
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Step 3: Install Dependencies
bash
Copy
Edit
# Upgrade pip
python -m pip install --upgrade pip

# Install the project and dependencies
pip install -e .

# If python-dotenv is missing, install manually
pip install python-dotenv
Step 4: Test the Installation
bash
Copy
Edit
# View help menu
mcp-server-motherduck --help

# Test with an in-memory database
mcp-server-motherduck --db-path :memory:
Expected output:

pgsql
Copy
Edit
[motherduck] INFO - 🦆 MotherDuck MCP Server v0.6.0
[motherduck] INFO - Ready to execute SQL queries via DuckDB/MotherDuck
[motherduck] INFO - Database client initialized in `duckdb` mode
[motherduck] INFO - ✅ Successfully connected to duckdb database
Use Ctrl+C to stop the test.

⚙️ Configuration Options
1. In-Memory Database (Recommended for testing)
json
Copy
Edit
{
  "mcpServers": {
    "motherduck-local": {
      "command": "D:\\AOT\\mcp-server-motherduck\\venv\\Scripts\\mcp-server-motherduck.exe",
      "args": [
        "--db-path",
        ":memory:"
      ]
    }
  }
}
2. Local DuckDB File
json
Copy
Edit
{
  "mcpServers": {
    "motherduck-local": {
      "command": "D:\\AOT\\mcp-server-motherduck\\venv\\Scripts\\mcp-server-motherduck.exe",
      "args": [
        "--db-path",
        "D:\\mydata\\database.db"
      ]
    }
  }
}
3. MotherDuck Cloud
First, create an account on MotherDuck and obtain your token.

json
Copy
Edit
{
  "mcpServers": {
    "motherduck-cloud": {
      "command": "D:\\AOT\\mcp-server-motherduck\\venv\\Scripts\\mcp-server-motherduck.exe",
      "args": [
        "--db-path",
        "md:",
        "--motherduck-token",
        "YOUR_MOTHERDUCK_TOKEN_HERE"
      ]
    }
  }
}
4. Read-Only Mode (for security)
json
Copy
Edit
{
  "mcpServers": {
    "motherduck-readonly": {
      "command": "D:\\AOT\\mcp-server-motherduck\\venv\\Scripts\\mcp-server-motherduck.exe",
      "args": [
        "--db-path",
        "database.db",
        "--read-only"
      ]
    }
  }
}
🔌 Claude Desktop Integration
Locate the Config File
Windows: %APPDATA%\Claude\claude_desktop_config.json

Alternative: Claude Desktop → Settings → Developer → Edit Config

Integration Steps
Exit Claude Desktop

Add one of the configurations above into the config file

Save the file

Restart Claude Desktop

Open a new chat and see your server in the MCP menu

Multiple Server Configuration
json
Copy
Edit
{
  "mcpServers": {
    "motherduck-local": {
      "command": "D:\\AOT\\mcp-server-motherduck\\venv\\Scripts\\mcp-server-motherduck.exe",
      "args": ["--db-path", ":memory:"]
    },
    "motherduck-cloud": {
      "command": "D:\\AOT\\mcp-server-motherduck\\venv\\Scripts\\mcp-server-motherduck.exe",
      "args": [
        "--db-path", "md:",
        "--motherduck-token", "YOUR_TOKEN"
      ]
    },
    "motherduck-filedb": {
      "command": "D:\\AOT\\mcp-server-motherduck\\venv\\Scripts\\mcp-server-motherduck.exe",
      "args": ["--db-path", "D:\\databases\\mydata.db"]
    }
  }
}
💡 Usage Examples
Basic SQL Operations
sql
Copy
Edit
-- Create table
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    signup_date DATE
);

-- Insert data
INSERT INTO users VALUES 
    (1, 'Ahmet Yılmaz', 'ahmet@email.com', '2024-01-15'),
    (2, 'Ayşe Demir', 'ayse@email.com', '2024-02-20');

-- Query data
SELECT * FROM users WHERE signup_date > '2024-01-01';

-- Aggregation
SELECT COUNT(*) as total_users, 
       DATE_TRUNC('month', signup_date) as month
FROM users 
GROUP BY month;
Load CSV File
sql
Copy
Edit
-- Load from CSV
COPY sales FROM 'D:\data\sales.csv' (AUTO_DETECT TRUE);

-- Read Parquet file
SELECT * FROM read_parquet('D:\data\big_data.parquet');

-- Read from S3 (MotherDuck Cloud)
SELECT * FROM 's3://bucket-name/data/*.parquet';
Advanced Features
sql
Copy
Edit
-- Window functions
SELECT name, 
       sale_amount,
       ROW_NUMBER() OVER (ORDER BY sale_amount DESC) as rank
FROM sales;

-- Using CTE
WITH monthly_summary AS (
    SELECT DATE_TRUNC('month', date) as month,
           SUM(amount) as total
    FROM sales
    GROUP BY 1
)
SELECT * FROM monthly_summary WHERE total > 10000;
🔍 Troubleshooting
Problem: "No module named 'dotenv'"
Solution:

bash
Copy
Edit
pip uninstall python-dotenv -y
pip install --force-reinstall python-dotenv
Problem: "mcp-server-motherduck: command not found"
Solution:

bash
Copy
Edit
# Ensure venv is active
which python  # Should point to venv directory

# Alternative execution
python -m mcp_server_motherduck --db-path :memory:
Problem: "Server Disconnected" in Claude Desktop
Fixes:

Check for JSON syntax errors in the config file

Use double backslashes (\\) in file paths

Ensure the venv path is correct

Restart Claude Desktop completely

Check logs under Developer → MCP Logs

Problem: PowerShell Execution Policy
Solution:

powershell
Copy
Edit
# Allow for current user
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Or temporary bypass
powershell -ExecutionPolicy Bypass -File ".\venv\Scripts\Activate.ps1"
Problem: Invalid MotherDuck Token
Fixes:

Make sure your token is valid

Enclose token in quotes

Generate a new token from MotherDuck Dashboard

📚 Additional Resources
DuckDB Documentation

MotherDuck Documentation

MCP Protocol Specification

Project GitHub Page

🤝 Contributing
Fork the project

Create a feature branch (git checkout -b feature/new-feature)

Commit your changes (git commit -am 'Added new feature')

Push the branch (git push origin feature/new-feature)

Open a Pull Request

📄 License
This project is licensed under the MIT License. See the LICENSE file for details.