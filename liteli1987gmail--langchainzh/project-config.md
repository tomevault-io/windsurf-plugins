---
trigger: always_on
description: LangChain有一个SQL代理，它提供了与SQL数据库交互的更灵活的方式，比链更好。使用SQL代理的主要优势是：
---

# 代理商

LangChain有一个SQL代理，它提供了与SQL数据库交互的更灵活的方式，比链更好。使用SQL代理的主要优势是：

- 它可以根据数据库的模式和数据库的内容（例如描述特定表）来回答问题。
- 它可以通过运行生成的查询、捕获回溯并正确地重新生成它来恢复错误。
- 它可以根据需要多次查询数据库以回答用户的问题。
- 它将只从相关表中检索模式，从而节省令牌。

要初始化代理，我们将使用[create_sql_agent](https://api.python.langchain.com/en/latest/agent_toolkits/langchain_community.agent_toolkits.sql.base.create_sql_agent.html)构造函数。该代理使用`SQLDatabaseToolkit`，其中包含以下工具：

* 创建和执行查询
* 检查查询语法
* 检索表描述
* ...等等

## 设置

首先，获取所需的软件包并设置环境变量：


```python
%pip install --upgrade --quiet  langchain langchain-community langchain-openai
```

在本指南中，我们默认使用OpenAI模型，但您可以根据自己的选择替换它们的模型提供商。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()

# 若要使用LangSmith，请取消下面的注释。不是必需的。
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

下面的示例将使用SQLite连接和Chinook数据库。按照以下安装步骤在与此笔记本相同的目录中创建`Chinook.db`：

- 将[此文件](https://raw.githubusercontent.com/lerocha/chinook-database/master/ChinookDatabase/DataSources/Chinook_Sqlite.sql)保存为`Chinook_Sqlite.sql`
- 运行`sqlite3 Chinook.db`
- 运行`.read Chinook_Sqlite.sql`
- 测试`SELECT * FROM Artist LIMIT 10;`

现在，`Chinhook.db`在我们的目录中，我们可以使用基于SQLAlchemy的`SQLDatabase`类与它进行接口交互：


```python
from langchain_community.utilities import SQLDatabase

db = SQLDatabase.from_uri("sqlite:///Chinook.db")
print(db.dialect)
print(db.get_usable_table_names())
db.run("SELECT * FROM Artist LIMIT 10;")
```

    sqlite
    ['Album', 'Artist', 'Customer', 'Employee', 'Genre', 'Invoice', 'InvoiceLine', 'MediaType', 'Playlist', 'PlaylistTrack', 'Track']
    




    "[(1, 'AC/DC'), (2, 'Accept'), (3, 'Aerosmith'), (4, 'Alanis Morissette'), (5, 'Alice In Chains'), (6, 'Antônio Carlos Jobim'), (7, 'Apocalyptica'), (8, 'Audioslave'), (9, 'BackBeat'), (10, 'Billy Cobham')]"



## 代理

我们将使用OpenAI聊天模型和一个`"openai-tools"`代理，它将使用OpenAI的函数调用API来驱动代理的工具选择和调用。

正如我们可以看到的那样，代理首先会选择哪些表是相关的，然后将这些表的模式和一些示例行添加到提示中。


```python
from langchain_community.agent_toolkits import create_sql_agent
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
agent_executor = create_sql_agent(llm, db=db, agent_type="openai-tools", verbose=True)
```


```python
agent_executor.invoke(
    "列出每个国家的总销售额。哪个国家的客户花费最多？"
)
```

    
    
    [1m> 进入新的AgentExecutor链...[0m
    [32;1m[1;3m
    调用: ‘sql_db_list_tables’，参数: ‘{}’
    
    
    [0m[38;5;200m[1;3mAlbum，Artist，Customer，Employee，Genre，Invoice，InvoiceLine，MediaType，Playlist，PlaylistTrack，Track[0m[32;1m[1;3m
    调用: ‘sql_db_schema’，参数: ‘Invoice，Customer’
    
    
    [0m[33;1m[1;3m
    CREATE TABLE “Customer” (
      “CustomerId” INTEGER NOT NULL, 
      “FirstName” NVARCHAR(40) NOT NULL, 
      “LastName” NVARCHAR(20) NOT NULL, 
      “Company” NVARCHAR(80), 
      “Address” NVARCHAR(70), 
      “City” NVARCHAR(40), 
      “State” NVARCHAR(40), 
      “Country” NVARCHAR(40), 
      “PostalCode” NVARCHAR(10), 
      “Phone” NVARCHAR(24), 
      “Fax” NVARCHAR(24), 
      “Email” NVARCHAR(60) NOT NULL, 
      “SupportRepId” INTEGER, 
      PRIMARY KEY (“CustomerId”), 
      FOREIGN KEY(“SupportRepId”) REFERENCES “Employee” (“EmployeeId”)
    )
    
    /*
    Customer表中的3行：
    CustomerId	FirstName	LastName	Company	Address	City	State	Country	PostalCode	Phone	Fax	Email	SupportRepId
    1	LuÃ­s	GonÃ§alves	Embraer - Empresa Brasileira de AeronÃ¡utica S.A.	Av. Brigadeiro Faria Lima, 2170	SÃ£o JosÃ© dos Campos	SP	Brazil	12227-000	+55 (12) 3923-5555	+55 (12) 3923-5566	luisg@embraer.com.br	3
    2	Leonie	KÃ¶hler	None	Theodor-Heuss-StraÃŸe 34	Stuttgart	None	Germany	70174	+49 0711 2842222	None	leonekohler@surfeu.de	5
    3	FranÃ§ois	Tremblay	None	1498 rue BÃ©langer	MontrÃ©al	QC	Canada	H2G 1A7	+1 (514) 721-4711	None	ftremblay@gmail.com	3
    */
    
    
    CREATE TABLE “Invoice” (
      “InvoiceId” INTEGER NOT NULL, 
      “CustomerId” INTEGER NOT NULL, 
      “InvoiceDate” DATETIME NOT NULL, 
      “BillingAddress” NVARCHAR(70), 
      “BillingCity” NVARCHAR(40), 
      “BillingState” NVARCHAR(40), 
      “BillingCountry” NVARCHAR(40), 
      “BillingPostalCode” NVARCHAR(10), 
      “Total” NUMERIC(10, 2) NOT NULL, 
      PRIMARY KEY (“InvoiceId”), 
      FOREIGN KEY(“CustomerId”) REFERENCES “Customer” (“CustomerId”)
    )
    
    /*
    Invoice表中的3行：
    InvoiceId	CustomerId	InvoiceDate	BillingAddress	BillingCity	BillingState	BillingCountry	BillingPostalCode	Total
    1	2	2009-01-01 00:00:00	Theodor-Heuss-StraÃŸe 34	Stuttgart	None	Germany	70174	1.98
    2	4	2009-01-02 00:00:00	UllevÃ¥lsveien 14	Oslo	None	Norway	0171	3.96
    3	8	2009-01-03 00:00:00	GrÃ©trystraat 63	Brussels	None	Belgium	1000	5.94
    */[0m[32;1m[1;3m
    调用: ‘sql_db_query’，参数: ‘SELECT c.Country, SUM(i.Total) AS TotalSales FROM Invoice i JOIN Customer c ON i.CustomerId = c.CustomerId GROUP BY c.Country ORDER BY TotalSales DESC LIMIT 10;’
    响应: 要列出每个国家的总销售额，我可以查询“Invoice”和“Customer”表。我将在“CustomerId”列上加入这些表，并根据“BillingCountry”列对结果进行分组。然后，我将计算“Total”列的总和以获得每个国家的总销售额。最后，我将按总销售额的降序对结果进行排序。
    
    这是SQL查询：
    
    ```sql
    SELECT c.Country, SUM(i.Total) AS TotalSales
    FROM Invoice i
    JOIN Customer c ON i.CustomerId = c.CustomerId
    GROUP BY c.Country
    ORDER BY TotalSales DESC
    LIMIT 10;
    ```
    
    现在，我将执行此查询以获取每个国家的总销售额。
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liteli1987gmail/langchainzh](https://github.com/liteli1987gmail/langchainzh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
