# build-fabric-ontology-demo
Build a Microsoft Fabric Ontology demo in under one hour!
* Fast Path - just grab the notebooks and install a demo
* Custom Path - create your own Ontology and demo data. This will require  editing the Goals.md file, and asking your Copilot to regenerate the notebook files for the Lakehouse tables and Ontology.

# FAST PATH - INSTALL EXISTING DEMO
Just want to install an Ontology demo for Healthcare, with demo data, and try out a data agent? 
1. Create a Fabric Workspace and a Lakehouse
2. Upload the notebooks from the notebooks folder into your workspace
3. Run the notebook 00_orchestrate_all to run notebooks 01-05 and create your Lakehouse tables
4. Review the 'Verification' table in the lakehouse to ensure your demo data was successfully generated.
5. Run the Create_Healthcare_Ontology notebook. 
    * Note: Creating the underlying graph db in Fabric can be slow: it took about 11 minutes on an F64 capacity before everything is ready.
6. Create a Fabric Data Agent and use your Ontology as a data source
7. Use data_agent_instructions.md as the AI instructions for your data agent
8. Try it out! See how the data_agent_prompts.md work with your agent.


# CUSTOM PATH - BRING YOUR OWN ONTOLOGY
## SETUP YOUR EDITING ENVIRONMENT
I built my custom demo using the following tools:
1. VS Code
2. GitHub Copilot, with a _good_ model (like ChatGPT 5.4)
3. [Skills for Microsoft Fabric](https://github.com/microsoft/skills-for-fabric)
4. [Microsoft Learn MCP Server](https://learn.microsoft.com/en-us/training/support/mcp)


## CREATE AN ONTOLOGY
* Begin with the [Ontology Playground](https://microsoft.github.io/Ontology-Playground/) to create your own Ontology from scratch, or modify an existing one.

![Ontology Playground](images/ontology-playground.png)
* Export your Ontology design as a .RDF file - It's the file icon in the upper-right corner of the Ontology playground
![Import Export Ontology](images/ImportExportOntology.png)
* Fun fact! You can ask M365 Copilot, or ChatGPT, etc. to create an Ontology.rdf file for you using an existing RDF file as a file. 
* For example, I prompted an LLM for the "Lord of the Rings" ontology using one of the Playground samples as an example. It worked!
![LOTR Ontology](images/LOTR_query.jpg)


## CREATE THE LAKEHOUSE TABLES AND DATA
At this point, you should have:
1. VS Code with your add-ins
2. The files from this repo
3. Your own ontology.rdf file that you want to build
4. A Fabric Workspace and Lakehouse where you can build this demo.

EDIT THE GOALS.MD FILE





## CREATE THE ONTOLOGY (and Graph database)




## CREATE SOME FUN QUERIES



## BUILD YOUR DATA AGENT

