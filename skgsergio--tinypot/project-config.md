---
trigger: always_on
description: You are an expert in SQL and Tinybird. Follow these instructions when working with .datasource and .pipe files:
---


You are an expert in SQL and Tinybird. Follow these instructions when working with .datasource and .pipe files:

<command_calling>
You have commands at your disposal to develop a tinybird project:
- tb build: to build the project locally and check it works.
- tb deployment create --wait --auto: to create a deployment and promote it automatically
- tb test run: to run existing tests
- tb --build endpoint url <pipe_name>: to get the url of an endpoint, token included.
- tb --build endpoint data <pipe_name>: to get the data of an endpoint. You can pass parameters to the endpoint like this: tb --build endpoint data <pipe_name> --param1 value1 --param2 value2
- tb --build token ls: to list all the tokens
There are other commands that you can use, but these are the most common ones. Run `tb -h` to see all the commands if needed.
When you need to work with resources or data in the Tinybird environment that you updated with the build command, add always the --build flag before the command. Example: tb --build datasource ls
When you need to work with resources or data in cloud, add always the --cloud flag before the command. Example: tb --cloud datasource ls
</command_calling>
<development_instructions>
- When asking to create a tinybird data project, if the needed folders are not already created, use the following structure:
├── connections
├── copies
├── datasources
├── endpoints
├── fixtures
├── materializations
├── pipes
└── tests
- The local development server will be available at http://localhost:7181. Even if some response uses another base url, use always http://localhost:7181.
- After every change in your .datasource, .pipe or .ndjson files, run `tb build` to build the project locally.
- When you need to ingest data locally in a datasource, create a .ndjson file with the same name of the datasource and the data you want and run `tb build` so the data is ingested.
- The format of the generated api endpoint urls is: http://localhost:7181/v0/pipe/<pipe_name>.json?token=<token>
- Before running the tests, remember to have the project built with `tb build` with the latest changes.
</development_instructions>
When asking for ingesting data, adding data or appending data do the following depending on the environment you want to work with:
<ingest_data_instructions>
- When building locally, create a .ndjson file with the data you want to ingest and do `tb build` to ingest the data in the build env.
- We call `cloud` the production environment.
- When appending data in cloud, use `tb --cloud datasource append <datasource_name> <file_name>`
- When you have a response that says “there are rows in quarantine”, do `tb --build|--cloud datasource data <datasource_name>_quarantine` to understand what is the problem.
</ingest_data_instructions>
<datasource_file_instructions>
Follow these instructions when creating or updating .datasource files:

<datasource_file_instructions>
    - Content cannot be empty.
    - The datasource names must be unique.
    - No indentation is allowed for property names: DESCRIPTION, SCHEMA, ENGINE, ENGINE_PARTITION_KEY, ENGINE_SORTING_KEY, etc.
    - Use MergeTree engine by default.
    - Use AggregatingMergeTree engine when the datasource is the target of a materialized pipe.
    - Use always json paths to define the schema. Example: `user_id` String `json:$.user_id`,
</datasource_file_instructions>

</datasource_file_instructions>

<pipe_file_instructions>
Follow these instructions when creating or updating .pipe files:

<pipe_file_instructions>
    - The pipe names must be unique.
    - Nodes do NOT use the same name as the Pipe they belong to. So if the pipe name is "my_pipe", the nodes must be named different like "my_pipe_node_1", "my_pipe_node_2", etc.
    - Nodes can't have the same exact name as the Pipe they belong to.
    - Avoid more than one node per pipe unless it is really necessary or requested by the user.
    - No indentation is allowed for property names: DESCRIPTION, NODE, SQL, TYPE, etc.
    - Allowed TYPE values are: endpoint, copy, materialized.
    - Add always the output node in the TYPE section or in the last node of the pipe.
</pipe_file_instructions>


<sql_instructions>
    - The SQL query must be a valid ClickHouse SQL query that mixes ClickHouse syntax and Tinybird templating syntax (Tornado templating language under the hood).
    - SQL queries with parameters must start with "%" character and a newline on top of every query to be able to use the parameters. Examples:
    <invalid_query_with_parameters_no_%_on_top>
    SELECT * FROM events WHERE session_id={{String(my_param, "default_value")}}
    </invalid_query_with_parameters_no_%_on_top>
    <valid_query_with_parameters_with_%_on_top>
    %
    SELECT * FROM events WHERE session_id={{String(my_param, "default_value")}}
    </valid_query_with_parameters_with_%_on_top>
    - The Parameter functions like this one {{String(my_param_name,default_value)}} can be one of the following: String, DateTime, Date, Float32, Float64, Int, Integer, UInt8, UInt16, UInt32, UInt64, UInt128, UInt256, Int8, Int16, Int32, Int64, Int128, Int256
    - Parameter names must be different from column names. Pass always the param name and a default value to the function.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skgsergio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
