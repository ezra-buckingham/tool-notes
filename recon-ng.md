# Recon-ng
## Description
A feature rich recon framework designed with the goal of providing a powerful environment to quickly and thoroughly conduct open source, web-based recon
* Great at host-validation and seeing what type of software is running
* _The base version has no modules installed by default_
* Very powerful because it will store data from commands and then be able to use that data on subsequent runs 
* Can use contact data to search for leaked credentials and then try to crack those leaked credentials
## Contexts
Global
* You will know this when it says `[recon-ng][default]`
Workspace
* You will know this when it says `[recon-ng][<workspace>]`
Module
* You will know this when it says `[recon-ng][<workspace>][module]`
## Common Commands
`help <optional command of which you can get more info`
### Workspaces
Workspaces allow you to organize your work and contain
* A database `db` (use help flag for more info)
`workspace create <name>` 
* So you can organize your work
#### Database
You can store collected data in a database and you can access this dats using the `recon-web` command
`db <command>`
`show <table name>`
#### Workspace Options (Rarely need to be adjusted)
`options list`
* Show all options set in the current workspace
`options set <option> <value>`
`options unset <option>`
### Marketplace
This is how you get new modules installed into your framework
`marketplace search <optional string>`
* This will give you a full list of the available modules
* If provided a string to search, will search with wildcards
`marketplace install <everything after last slash in path>`
* This will install the module and dependencies
### Modules
These are functions that will help you with recon
`run`
* Will run the module with the specified module and global options
`info`
* Will show information about the module
`back`
* Will exit out of the module
`modules <option> <module>`
`modules load <module>`
* Will load the module in and display in terminal
#### Module Options
`options list`
* Show all options set in the current module
`options set <option> <value>`
`options unset <option>`
## Example Executions
