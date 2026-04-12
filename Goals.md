# GOAL
* Create a Microsoft Fabric Ontology, and the demonstration data and lakehouse necessary to show sample queries. 

# RESOURCES
* You are running inside VS Code on a local PC. 
* Microsoft Fabric skills are installed locally. 
* You have a Microsoft Learn MCP server for access to documentation

# CONSTRAINTS
* Always start with Microsoft Fabric skills and Microsoft learn documentation.
* If you are unsure of a step, stop and ask the user for clarification.


# STEP 1 - CREATE TABLES
* Create Microsoft Fabric notebooks to create each table described in the ontology.rdf document, and populate the table with demo data. 
* There should be no 'orphan' entries, i.e. every patient has appointments, every provider issues diagnoses and prescriptions, etc.
* All of the joins should work. 

## Provider Table
* Create the table as described in the .rdf file, but ensure the first column is "name"
* Create 20 providers.

## Patient Table
* Create the table as described in the .rdf filebut ensure the first column is "name"
* Create 50 patients

## Appointment Table
* Create the table as described in the .rdf file.
* Create 100 appointments, all of them in the past 3 months, all of them with a status of completed
* The appointments should be semi-randomly distributed, there will be several patients that have 5 appointments

## Diagnosis Table
* Create the table as described in the .rdf file.
* Create 20 diagnoses

## Prescription Table
* Create the table as described in the .rdf file.
* Create 30 prescriptions that are used to treat the 20 diagnoses. 


# STEP 2 - CREATE ONTOLOGY IN MICROSOFT FABRIC GUI
* Based on your knowledge of the ontology.rdf file and the step-by-step instructions for creating a Microsoft Fabric ontology directly from OneLake, please create a markdown file that is a step-by-step, click-by-click guide to build the ontology inside of Microsoft Fabric. 
* The guide should be written at the high school level and assume no knowledge of Microsoft Fabric, other than the ability to view the UI. 
* Here is the tutorial documentation to help you: https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-1-create-ontology?pivots=onelake

# STEP 2.5 - CREATE ONTOLOGY USING SEMPY
* Create the Ontology in my workspace by authoring a notebook "Create_Healthcare_Ontology" that uses Semantic Link Labs to call the REST API 'Create Ontology.'
* Use the local "sempy_notebook" skills file to learn how to create Fabric notebooks
* Use this documentation to call the REST API: https://learn.microsoft.com/en-us/rest/api/fabric/ontology/items/create-ontology?tabs=HTTP


# STEP 3 - CREATE GRAPH QUERIES
* As you know, the Fabric Ontologies feature will also create a Graph database. 
* You will generate a Query.md markdown file that contains 15 sample queries in Graph query language that can be copied and pasted into the Microsoft Fabric graph query editor. 
    * 5 of the queries will illustrate single-hop queries, i.e. provider to diagnosis
    * 10 of the queries will be multi-hop queries that showcase the power of ontologies and graphs.
    * Each query should have a brief description of what the query is doing or illustrating. Use comments.
    * There is no need to add "LIMIT 1000" because we are using this local dataset. 
## Resources
* Tutorial to query Fabric Graph database: https://learn.microsoft.com/en-us/fabric/graph/tutorial-query-code-editor
* GQL Quick reference guide: https://learn.microsoft.com/en-us/fabric/graph/gql-reference-abridged
* GQL Language guide for graph in Microsoft Fabric: https://learn.microsoft.com/en-us/fabric/graph/gql-language-guide


# STEP 4 - CREATE AI INSTRUCTIONS FOR FABRIC DATA AGENT
* You will create a markdown document called "data_agent_instructions.md" that provides the data agent with guidance about how to translate business users natural-language requests into responsive GQL queries. 
* Your document must not exceed 10,000 characters. 
* Your document should include one or two sample queries to enable one-shot learning. 
## Resources
* Provide data agent with instructions: https://learn.microsoft.com/en-us/fabric/data-science/how-to-create-data-agent#provide-instructions
* Best practices for configuring data agent: https://learn.microsoft.com/en-us/fabric/data-science/data-agent-configuration-best-practices



