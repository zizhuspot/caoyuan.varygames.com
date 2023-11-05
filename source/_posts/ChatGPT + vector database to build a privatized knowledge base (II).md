---
title: ChatGPT + vector database to build a privatized knowledge base (II)
date: 2023-11-05 00:03:00
categories: 
  - Technology
tags: 
  - Backend Technology Sharing
  - ChatGPT
  - good solution
  - service
  - development
  - catastrophic crash
  - network
  - Database
description: For ChatGPT + vector database how to put the knowledge base on the ground to realize, how to use the vector database to build a privatized knowledge base, and provides examples of MySQL table design and vector database table design. Explained the interaction flow of the knowledge base and the interface for external calls
cover: https://s2.loli.net/2023/11/05/IYKOjTgmr6WFvSw.webp
---
> [ChatGPT+Vector database to build a privatized knowledge base](https://juejin.cn/post/7227079326594859068 "https://juejin.cn/post/7227079326594859068") The meaning of vector databases has been introduced.
This time, let's go hands-on and take a look at the schema design and interaction flow first

# 1. Table Structure Design

## 1. MySQL Table Design

## 1. knowledge_base (Knowledge Base Summary Table)

```sql
CREATE TABLE `knowledge_base` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '知识库id',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_by` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '创建者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `update_by` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '更新者',
  `name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '知识库名称',
  `description` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '知识库描述',
  `vector_collection_name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '向量数据库的表名',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='知识库总表';
```

### 2、knowledge_file（Knowledge base document management）

```sql
CREATE TABLE `knowledge_file` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '文件id',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_by` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '创建者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `update_by` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '更新者',
  `knowledge_id` bigint NOT NULL COMMENT '知识库id',
  `file_name` varchar(65) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '文件名',
  `oss_id` bigint NOT NULL COMMENT 'ossId',
  `file_status` int NOT NULL DEFAULT '1' COMMENT '0向量处理中，1未激活，2已完成，3失败',
  `fail_reason` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '失败原因',
  `slice_type` int DEFAULT NULL COMMENT '切分类型：1分隔符，2字数',
  `slice_value` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '切分规则数据',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='知识库文件管理';
```

### 4、knowledge_file_slice_vector（Knowledge base document slicing steering volume data table）

```sql
CREATE TABLE `knowledge_file_slice_vector` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_by` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '创建者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `update_by` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '更新者',
  `knowledge_id` bigint DEFAULT NULL COMMENT '知识库id',
  `knowledge_file_id` bigint DEFAULT NULL COMMENT '知识库文件id',
  `slice_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci COMMENT '切片数据',
  `vector_id` bigint DEFAULT NULL COMMENT '向量数据id',
  PRIMARY KEY (`id`),
  KEY `idx_knowledge` (`knowledge_id`),
  KEY `idx_knpwledge_file` (`knowledge_file_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='知识库文件切片转向量数据表';
```

### 4、knowledge_usage_config （Knowledge base applications）

```sql
CREATE TABLE `knowledge_usage_config` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_by` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '创建者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `update_by` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '更新者',
  `app_name` varchar(30) DEFAULT NULL COMMENT '应用配置名称',
  `app_description` varchar(255) DEFAULT NULL COMMENT '应用配置描述',
  `app_icon` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '应用图标',
  `prompts_config` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci COMMENT 'prompts模板',
  `knowledge_id` bigint DEFAULT NULL COMMENT '知识库id',
  `top_k` int DEFAULT NULL COMMENT 'topK',
  `top_p` double DEFAULT NULL COMMENT 'topP',
  `temperature` varchar(5) DEFAULT NULL COMMENT '温度',
  `app_code` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'appCode',
  `app_secret` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'appSecret',
  PRIMARY KEY (`id`),
  KEY `idx_app` (`app_code`,`app_secret`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='知识库应用';
```

## 2. Vector database table design

**Note: A knowledge base corresponds to a vector data table **

Data Id, data title, data text, data vector eigenvalue

## 2. Interaction flow

![](https://s2.loli.net/2023/11/05/1jLlQEKopi6WmPn.webp)

online address：[www.processon.com/diagraming/...](https://link.juejin.cn?target=https%3A%2F%2Fwww.processon.com%2Fdiagraming%2F64bf380800357b03b718c4b3 "https://www.processon.com/diagraming/64bf380800357b03b718c4b3")

# 3. External calls

Refer to the interface entry parameter:

```json
{
    "textValue": "查询问题",
    "appCode": "应用appCode",
    "appSecret": "应用appSecret"
}
```

Refer to the interface out reference:

```json
{
    "code":200,
    "msg":"操作成功",
    "data":{
        "result":"返回的结果",
        "sourceVoList":[
            {
                "title":"来源标题",
                "text":"来源内容"
            }
        ]
    }
}
```
