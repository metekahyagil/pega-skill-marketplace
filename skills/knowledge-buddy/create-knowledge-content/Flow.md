I would like to modify the flow a little bit. I would like to make the communication more interactive with the enduser. Lets start our updates with the create-knowledge-content skill. I really like the AskUserQuestion tool. So I want to utilize this more.
You have everything you need in the SKILL.md and the references. As we implement each section please use these.
We must keep everything simple and to the point. NO UNNECESSARY information. We must considerate of the token usage,
Make sure that you adhere the DX API configuration. Below examples are from a specific instance to capture the calls.


We will keep the 00-connection.md as is.



FLOW-CREATE

1 - We create the case
To create new content we use
{"caseTypeID":"PegaFW-KB-Work-Article","content":{"pyAddCaseContextPage":{}},"processID":"pyStartCase"}
https://pega.44a201b254c54.pegaenablement.com/prweb/app/KnowledgeBuddy/api/application/v2/cases?viewType=page

2 - We fetch the collections.
:method
POST
:path
/prweb/app/KnowledgeBuddy/api/application/v2/data_views/D_IndexList
{"dataViewParameters":{},"paging":{"pageNumber":1,"pageSize":50},"query":{"select":[{"field":"pyID"},{"field":"CollectionName"}],"distinctResultsOnly":"true"}}

We use the AskUserQuestion tool to allow the user to select. If too many we should give the whole list instead of the AskUserQuestion tool.

3 - We refresh
:method
PATCH
:path
/prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-KB-WORK-ARTICLE%20KB-2020!CREATEFORM_DEFAULT/actions/Create/refresh

4- We fetch the content type for the selected Collection ID (DC-1 in this case)
method
POST
:path
/prweb/app/KnowledgeBuddy/api/application/v2/data_views/D_ContentAccessDataSourcesByCollection
{"dataViewParameters":{"Available":"Yes","CollectionID":"DC-1"}}

5- Add one or more access roles.
:method
POST
:path
/prweb/app/KnowledgeBuddy/api/application/v2/data_views/D_BuddyAccessRoleList

We use the AskUserQuestion tool to allow the user to select. If too many we should give the whole list instead of the AskUserQuestion tool.
User can select one or more.

6- We use the AskUserQuestion tool to Enable Advanced Settings.
    
    6.a    
    If user selects No:
    Then set the following:

    6.b
    If user selects Yes:
    We use the AskUserQuestion tool to allow the user to select. 
    6.b.1 - Chunking method (NONE or TITLE or SIZE or ABSTRACT) cannot pick more than 1. IF USER SELECTS NONE then Chunk size and Chunk overlap should not be collected.

    (Only when NONE Is not selected.)
    6.b.2 - Chunking size (Default 1000) 
    6.b.3 - Chunk overlap (Default 200)

:method
PATCH
:path
/prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-KB-WORK-ARTICLE%20KB-2020!CREATEFORM_DEFAULT/actions/Create?viewType=page

EXAMPLE

{"content":{"Collection":{"pyID":"DC-1"},"Datasource":{"pyID":"SRC-1"},"AdvancedSettings":false},"pageInstructions":[{"target":".ContentAccessConfigurations","content":{},"listIndex":1,"instruction":"INSERT"},{"content":{"AccessRoleName":"KnowledgeBuddy:Admin"},"target":".ContentAccessConfigurations","listIndex":1,"instruction":"UPDATE"},{"target":".ContentAccessConfigurations","content":{},"listIndex":2,"instruction":"INSERT"},{"content":{"AccessRoleName":"KnowledgeBuddy:Author"},"target":".ContentAccessConfigurations","listIndex":2,"instruction":"UPDATE"},{"target":".ContentAccessConfigurations","content":{},"listIndex":3,"instruction":"INSERT"},{"target":".ContentAccessConfigurations","listIndex":3,"instruction":"DELETE"}]}

OR 

{"content":{"Collection":{"pyID":"DC-1"},"Datasource":{"pyID":"SRC-2"},"AdvancedSettings":true,"IndexParams":{"ChunkingMethod":"SIZE","ChunkSize":1000,"ChunkOverlap":200}},"pageInstructions":[{"target":".ContentAccessConfigurations","content":{},"listIndex":1,"instruction":"INSERT"},{"content":{"AccessRoleName":"KnowledgeBuddy:Admin"},"target":".ContentAccessConfigurations","listIndex":1,"instruction":"UPDATE"},{"target":".ContentAccessConfigurations","content":{},"listIndex":2,"instruction":"INSERT"},{"content":{"AccessRoleName":"KnowledgeBuddy:BuddyManager"},"target":".ContentAccessConfigurations","listIndex":2,"instruction":"UPDATE"}]}


7. Refresh
:method
PATCH
:path
/prweb/app/KnowledgeBuddy/api/application/v2/cases/PEGAFW-KB-WORK-ARTICLE%20KB-2021/views/pyDetailsTabContent/refresh


FLOW-AUTHOR.

1- We use the AskUserQuestion tool to allow the user to select Content Format.
 Text vs File. 

2(a)- When Text is selected Use the AskUserQuestion tool to allow the user to enter
    1 - Title ()
    2 - Abstract
    3 - Number of Contents to ingest 1 or more.

method
PATCH
:path
/prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-KB-WORK-ARTICLE%20KB-2021!DRAFT_FLOW/actions/AuthorContent?viewType=form

{"content":{"ArticleType":"text","Title":"This is the title","Abstract":"This is the abstract"},"pageInstructions":[{"target":".Chunks","content":{},"listIndex":1,"instruction":"INSERT"},{"content":{"Content":"<p>This is first Content</p>"},"target":".Chunks","listIndex":1,"instruction":"UPDATE"},{"target":".Chunks","content":{},"listIndex":2,"instruction":"INSERT"},{"content":{"Content":"<p>This is second <strong>content</strong>.</p>"},"target":".Chunks","listIndex":2,"instruction":"UPDATE"}]}

{"content":{"ArticleType":"text","Title":"This is the title","Abstract":"This is the abstract"},"pageInstructions":[{"target":".Chunks","content":{},"listIndex":1,"instruction":"INSERT"},{"content":{"Content":"<p>This is first Content</p>"},"target":".Chunks","listIndex":1,"instruction":"UPDATE"},{"target":".Chunks","content":{},"listIndex":2,"instruction":"INSERT"},{"content":{"Content":"<p>This is second <strong>content</strong>.</p>"},"target":".Chunks","listIndex":2,"instruction":"UPDATE"}]}
https://pega.44a201b254c54.pegaenablement.com/prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-KB-WORK-ARTICLE%20KB-2021!DRAFT_FLOW/actions/AuthorContent/refresh


2(b)- When File is selected Use the AskUserQuestion tool to allow the user to enter
    a - Title ()
    b - Abstract
    c - Path to attachment.

PLEASE REFER TO OUR EXISTING FLOW for VALIDATION - YOU MUST REFRESH AFTER ATTACHING.

:method
POST
:path
/prweb/app/KnowledgeBuddy/api/application/v2/attachments/upload
appendUniqueIdToFileName
true
file
(binary)

Response: {"ID":"8f93ae83-78a8-44b5-a603-b7bd12e087c5"}

:method
PATCH
:path
/prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-KB-WORK-ARTICLE%20KB-2022!DRAFT_FLOW/actions/AuthorContent/refresh

{"content":{"ArticleType":"file","Title":"Title example","Abstract":"Abstract example"},"pageInstructions":[{"target":".ContentAttachment","content":{"ID":"8f93ae83-78a8-44b5-a603-b7bd12e087c5"},"instruction":"REPLACE"}]}

3 - Resolve

:method
PATCH
:path
/prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-KB-WORK-ARTICLE%20KB-2022!DRAFT_FLOW/actions/AuthorContent?viewType=form

query string viewType=form
{"content":{"ArticleType":"file","Title":"Title example","Abstract":"Abstract example"},"pageInstructions":[{"target":".ContentAttachment","content":{"ID":"8f93ae83-78a8-44b5-a603-b7bd12e087c5"},"instruction":"REPLACE"}]}

FLOW-INGEST

1 - Case summary & pyDetailsTabContent
display
:method
GET
:path
/prweb/app/KnowledgeBuddy/api/application/v2/cases/PEGAFW-KB-WORK-ARTICLE%20KB-2022/views/pyCaseSummary

 - :method
GET
:path
/prweb/app/KnowledgeBuddy/api/application/v2/cases/PEGAFW-KB-WORK-ARTICLE%20KB-2022/views/pyDetailsTabContent

Summarize the information.