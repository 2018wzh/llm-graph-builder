# iCareer LLM-Graph-Builder对接文档
## 简历导入
### 使用LLM将简历转换成描述
> 使用LLM把简历的json转换成一段描述简历的文字，便于导入数据库。模型使用ecnu-max，prompt如下：
```
请用一段话总结以下JSON格式的简历文件。
要求：
1. **术语标准化** ：将JSON中的术语进行标准化，例如使用"Java"替代"Java语言"。
2. **主语统一** ：每一句的主语必须是简历的所属人（即简历的主人）。
3. **格式要求** ：语句长度尽可能短。
4. **内容完整性** ：严格遵照简历中的所有内容进行总结，不要丢失或忽略任何信息。
5. **无关信息** ：不要输出与简历内容无关的信息。
6. **唯一标识符**：输出中需要包含简历所属人的唯一标识符。
7. **换行要求**：在每一句话的句号后进行{换行}
JSON简历文件将在下面给出：
```
### 连接数据库
> 使用前需要连接数据库

#### POST /connect
#### Args
* `uri`= Neo4j uri, 
* `userName`= Neo4j db username, 
* `password`= Neo4j db password, 
* `database`= Neo4j database name
### 文件上传
> 将文件上传到后端，以便于下一步处理
#### POST /upload
#### Args
* `file`=The file to be uploaded, received in chunks,(Blob)
* `chunkNumber`=The current chunk number being uploaded,
* `totalChunks`=The total number of chunks the file is divided into (each chunk of 1Mb size),
* `originalname`=The original name of the file,
* `model`=The model associated with the file,
* `uri`=Neo4j uri, 
* `userName`= Neo4j db username, 
* `password`= Neo4j db password, 
* `database`= Neo4j database name
### 进行提取
> 从文件中提取关系并存入neo4j
#### POST /extract
#### Args
* `uri`=Neo4j uri, 
* `userName`= Neo4j db username, 
* `password`= Neo4j db password, 
* `database`= Neo4j database name
* `model`= LLM model,(ecnu_ecnu-max)
* `file_name` = File uploaded from device
* `source_type` = (local_file)
* `allowedNodes` = Node labels passed from settings panel. (Person,ContactInfomation,CareerObjective,WorkExperience,IntershipExperience,Skill,SelfAssessment,Award,EducationalBackground,Article,Subject,Year,ID)
* `allowedRelationship` = Relationship labels passed from settings panel (HAS_CONTACT_INFOMATION,HAS_CAREER_OBJECTIVE,HAS_WORK_EXPERIENCE,HAS_INTERNSHIP_EXPERIENCE,HAS_SKILL,HAS_SELF_ASSESSMENT,HAS_AWARD,AUTHORED_BY,BELONGS_TO,PUBLISHED_IN_YEAR,IDENTIFIED_BY)
* `retry_condition` = When retry extract use (delete_entities_and_start_from_beginning/start_from_beginning/start_from_last_processed_position).
### 失败时重试
> 提取失败时重试提取
#### POST /retry_processing
#### Args
* `uri`=Neo4j uri,
* `userName`= Neo4j db username,
* `password`= Neo4j db password,
* `database`= Neo4j database name,
* `file_name`= Name of the file which user want to Ready to Reprocess.
* `retry_condition` = One of the above 3 conditions which is selected for reprocessing.(delete_entities_and_start_from_beginning/start_from_beginning/start_from_last_processed_position)
### 后处理
> 提取完成后需要经过后处理才能进行查询
#### POST /post_processing
#### Args
* `uri`=Neo4j uri, 
* `userName`= Neo4j db username, 
* `password`= Neo4j db password, 
* `database`= Neo4j database name
* `tasks`= List of tasks to perform(["materialize_text_chunk_similarities","enable_hybrid_search_and_fulltext_search_in_bloom","materialize_entity_similarities","enable_communities"])
### 查看任务
> 查看正在进行的提取
#### POST /sources_list
#### Args
* `uri`=Neo4j uri, 
* `userName`= Neo4j db username, 
* `password`= Neo4j db password, 
* `database`= Neo4j database name 
### 更新进度
> 查看正在进行的提取的进度，可能用到了SSE，没跑通
#### GET /update_extract_status/{file_name}
#### Args
* `uri`=Neo4j uri, 
* `userName`= Neo4j db username, 
* `password`= Neo4j db password, 
* `database`= Neo4j database name
## 进行查询
### 聊天接口
> 通用聊天接口
#### POST /chat_bot
#### Args
* `uri`= neo4j uri
* `userName`= Neo4j database username
* `password`= Neo4j database password
* `model`= LLM model(ecnu_ecnu-reasoner/ecnu_ecnu-reasoner-lite)
* `question`= User query for the chatbot
* `session_id`= Session ID used to maintain the history of chats during the user's connection (UUID，由前端生成，用来区分不同用户的上下文)
* `mode` = chat mode to use (graph_vector_fulltext(经过测试)/vector/graph/graph_vector/fulltext/entity_vector/global_vector)
* `document_names` = the names of documents to be filtered works for vector mode and vector+Graph mode 
## 备注
> 示例的响应参见postman的导出文件