---
trigger: always_on
description: The application developed in this project is called `sqluv`, a cross-platform application built with Golang. `sqluv` is a terminal application with a Text User Interface (hereinafter referred to as TUI). Its core functionalities include **connecting to multiple RDBMSs** and **executing SQL queries**.
---

# **Project Overview**  

The application developed in this project is called `sqluv`, a cross-platform application built with Golang. `sqluv` is a terminal application with a Text User Interface (hereinafter referred to as TUI). Its core functionalities include **connecting to multiple RDBMSs** and **executing SQL queries**.  

In addition to RDBMS, it can also load CSV, TSV, and LTSV files from **local storage, HTTPS, or Amazon S3** into SQLite3 (in-memory) and execute SQL queries on them. It is not possible to execute RDBMS connections and file loading simultaneously. If no startup arguments are provided, the application will attempt to connect to an RDBMS. If startup arguments are present, it will attempt to load files instead.  

# **Project Directory Structure**  

- **config**: Manages environment variables, startup arguments, and configuration files (DB connection settings, color theme settings, SQL query history).  
- **di**: Manages Dependency Injection.  
- **doc**: Manages documentation.  
- **docker**: Manages files used by the Dockerfile.  
- **domain/model**: Manages domain objects.  
- **domain/repository**: Manages interfaces related to temporary storage and persistence.  
- **infrastructure/memory**: Implements temporary storage interfaces defined in `domain/repository`.  
- **infrastructure/mock**: Mocks interfaces defined in `domain/repository`.  
- **infrastructure/persistence**: Implements persistence-related interfaces defined in `domain/repository`.  
- **infrastructure**: Defines common functions and errors shared across the `infrastructure` directory.  
- **interactor/mock**: Mocks interfaces defined in `usecase`.  
- **interactor**: Implements interfaces defined in `usecase`.  
- **testdata**: Manages test data.  
- **tui**: Manages the TUI.  
- **usecase**: Manages the use case interfaces called from the TUI.  

# **Overview of the Text User Interface (TUI)**  

The TUI is implemented using the `github.com/rivo/tview` library and consists of the following components:  

- **TUI management structure** (`tui.TUI` struct)  
- **Component management structure** (`tui.home` struct)  
- **Sidebar displaying table and column lists** (`tui.sidebar` struct)  
- **Query text area for writing SQL queries** (`tui.queryTextArea` struct)  
- **Table view displaying SQL query execution results** (`tui.queryResultTable` struct)  
- **SQL query execution button** (`tui.executeButton` struct)  
- **SQL query history button** (`tui.historyButton` struct)  
- **View displaying statistical results of SQL query execution** (`tui.rowStatistics` struct)  
- **Dialog for displaying errors and information** (`tui.dialog` struct)  
- **Footer displaying key binding information** (`tui.footer` struct)  
- **Structure managing color themes** (`tui.Theme` struct)  

The above components make up the TUI that appears **after connecting to an RDBMS or loading a file**. Before connecting to an RDBMS, a **modal window** (`tui.connectionModal` struct) is displayed.

# Architecture of sqluv

The project adopts Clean Architecture. The TUI depends on the `usecase` package (interfaces). The interfaces in the `usecase` package are implemented in the `interactor` package. The `interactor` depends on the `domain/model` package and the `domain/repository` package (interfaces). The `domain/repository` package is implemented in the `infrastructure/memory` package and the `infrastructure/persistence` package. The `infrastructure/memory` and `infrastructure/persistence` packages depend on the `domain/model` package.

The `domain/model` package manages domain objects. Most structs are implemented as Value Objects, and initialization functions are provided.

# SQL Query History Specification

The `history.db` file exists in the directory that manages the configuration files for `sqluv` (e.g., `~/.config/sqluv`). Every time `sqluv` successfully executes an SQL query, it saves the query history to `history.db` using SQLite3. The code related to `sqluv`'s configuration files is located in `config/config_file.go`.

`sqluv` displays the SQL query history list screen when the [History] button on the home screen is pressed or when `Ctrl-h` is entered. It also supports fuzzy search for query history, which is implemented using `github.com/lithammer/fuzzysearch/fuzzy`. The implementation of fuzzy search can be found in `tui/tui.go`.

### Interface Specification  

Following Go conventions, interface names are formed by combining a verb with "er." Each interface contains only a single method. The interface name follows the pattern *Noun + Verb + "er"*, while the method name follows the pattern *Verb + Noun*. Once an interface name is determined, its method name is automatically derived. For example, the `FileGetter` interface contains the `GetFile()` method.  

### Dependency Injection Specification  


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nao1215/sqluv](https://github.com/nao1215/sqluv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
