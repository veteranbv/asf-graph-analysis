# Apache Software Foundation Project Email Analysis

## Background and Purpose

The Apache Software Foundation (ASF) hosts a diverse array of open-source projects, each with its own vibrant community of contributors and users. These communities communicate through mailing lists, where discussions range from technical issues and feature requests to project governance and community events.

Analyzing the email data from ASF project mailing lists serves several purposes:

1. **Community Insights:** Understanding the dynamics of various project communities, including the most active contributors, the nature of their contributions, and how they interact with each other.

2. **Discussion Trends:** Identifying key topics of discussion across different projects, how they evolve over time, and their impact on the projects' development.

3. **Sentiment Analysis:** Gauging the overall sentiment within communities, which can be indicative of the projects' health and the communities' satisfaction with their progress.

4. **Issue Identification:** Detecting potential issues or areas of concern based on the frequency and nature of discussions around specific topics.

5. **Cross-Project Collaboration:** Exploring opportunities for collaboration and alignment between different ASF projects based on common themes or challenges.

6. **Future Planning:** Informing project maintainers and contributors about the communities' needs and interests, helping to guide the projects' future directions.

7. **Community Growth:** Identifying opportunities to engage new contributors and users, as well as potential committers and project management committee (PMC) members.

By analyzing the email data, we aim to gain valuable insights into the structure, behavior, and priorities of ASF project communities, ultimately contributing to the continued success and growth of these open-source projects.

## Source Data Model

The source data for this project is derived from the mailing lists of Apache Software Foundation projects. The data was extracted from the individual raw email messages, and enriched using the Mistral LLM. The data is structured as follows:

- **Email:** Each record represents an individual email message sent to a project's mailing list.
  - `project`: The name of the Apache project associated with the email (e.g., "nifi.apache.org").
  - `list`: The name of the mailing list to which the email was sent (e.g., "users").
  - `message-id`: A unique identifier for the email.
  - `in_reply_to`: The message-id of the email to which this email is a reply, if any.
  - `references`: A list of message-ids that this email references, indicating the email thread.
  - `header_datetime`: The date and time when the email was sent.
  - `sender_email`: The email address of the sender.
  - `sender_name`: The name of the sender.
  - `recipient_email`: The email address of the recipient (typically the mailing list address).
  - `subject`: The subject line of the email.
  - `content_SHA-256`: A SHA-256 hash of the email content & subject for integrity and uniqueness verification.
  - `sentiment_analysis`: LLM sentiment analysis performed on the email content (e.g., "positive", "negative", "neutral").
  - `category_classification`: LLM generated classification of the email content into categories (e.g., "Bug Reports", "Help Requests").
  - `summary`: LLM summary or brief description of the email content.
  - `key_topics`: LLM-derived array of key topics or keywords extracted from the email content.

This source data model provides the foundation for ingesting and analyzing the email communications within ASF project communities.

## Graph Data Model

The graph data model for this project is designed to capture the interactions and relationships within the ASF project communities based on their email communications. The model consists of the following entities (nodes) and relationships (edges).

### **Entities (Nodes):**

- **Project:** Apache Software Foundation project, such as Apache NiFi.
  - Attributes: `name` (e.g., "nifi.apache.org")
- **User:** Represents individuals who send or receive emails.
  - Attributes: `email`, `name`
- **Email:** Represents individual email messages.
  - Attributes: `message_id`, `subject`, `header_datetime`, `sentiment_analysis`, `category_classification`, `summary`, `content_SHA256`
- **Mailing List:** Represents the mailing lists to which emails are sent.
  - Attributes: `list_name`
- **Topic:** Represents key topics discussed in emails.
  - Attributes: `topic_name`

### **Relationships (Edges):**

- **SENT:** (User) -[:SENT]-> (Email) to represent that a user sent an email.
- **RECEIVED:** (Email) -[:RECEIVED]-> (User) to represent that a user received an email.
- **SENT_TO:** (Email) -[:SENT_TO]-> (Mailing List) to represent that an email was sent to a mailing list.
- **REPLIES_TO:** (Email) -[:REPLIES_TO]-> (Email) to represent that an email is a reply to another email, based on the `in_reply_to` field.
- **REFERENCES:** (Email) -[:REFERENCES]-> (Email) to represent the threading of emails based on the `references` field.
- **DISCUSSES:** (Email) -[:DISCUSSES]-> (Topic) to represent that an email discusses a particular topic.
- **BELONGS_TO:** (Mailing List) -[:BELONGS_TO]-> (Project) to represent that a mailing list belongs to a project.
- **ASSOCIATED_WITH:** (Email) -[:ASSOCIATED_WITH]-> (Project) to represent that an email is associated with a project.

The graph data model captures the interactions between users, emails, mailing lists, topics, and projects, providing a rich representation of the ASF project communities' communication patterns and relationships. This enables us to analyze the ASF project communities' interactions and discussions, providing insights into their structure, behavior, and priorities.

## **Detailed Potential Analyses:**

- **Communication Patterns:** Analyze the flow of communication between users, mailing lists, and topics to identify key discussion themes and patterns.
- **User Influence:** Identify influential users based on their participation in email threads, the sentiment of their messages, and their connections to various topics and mailing lists. Looking for users who are active, well-connected, and influential within the community.
- **Topic Evolution:** Track the evolution of topics over time to understand how discussions change and develop within the project.
- **Community Structure:** Explore the structure of the Apache NiFi community by analyzing the relationships between users, mailing lists, and topics.
- **Sentiment Analysis:** Analyze the sentiment of emails to gauge the overall mood and tone of discussions within the project.

### **Future Potential Analyses:**

- **Time-Based Analysis:** You can extend the model to include time-based analyses by leveraging the `header_datetime` attribute of emails to study patterns and trends over time.
- **Content Analysis:** The `summary` and `content_SHA256` attributes can be used for content-based analyses, such as identifying duplicate or similar emails and understanding the content of discussions.
- **Community Growth:** Analyze the growth of the community by tracking the participation of new users and contributors over time.
- **Cross-Project Collaboration:** Explore opportunities for collaboration and alignment between different ASF projects based on common themes or challenges.

## Ingest Queries

This section contains the Cypher queries and scripts used to ingest the email data into the graph database. Each query is accompanied by an explanation of its purpose and how it maps the data to the graph model. These queries are crucial for structuring the raw email data in a way that aligns with our graph data model.

```cypher
// Cypher query to ingest the email data into the graph database
LOAD CSV WITH HEADERS FROM 'file:///path/to/email_data.csv' AS row
MERGE (p:Project {name: row.project})
MERGE (ml:MailingList {list_name: row.list})
MERGE (e:Email {
  message_id: row.message_id,
  subject: row.subject,
  header_datetime: row.header_datetime,
  sentiment_analysis: row.sentiment_analysis,
  category_classification: row.category_classification,
  summary: row.summary,
  content_SHA256: row.content_SHA256
})
MERGE (u:User {email: row.sender_email, name: row.sender_name})
MERGE (u)-[:SENT]->(e)
MERGE (e)-[:SENT_TO]->(ml)
MERGE (ml)-[:BELONGS_TO]->(p)

WITH e, row
UNWIND split(row.key_topics, ',') AS topic
MERGE (t:Topic {topic_name: topic})
MERGE (e)-[:DISCUSSES]->(t)

WITH e, row
UNWIND split(row.references, ',') AS ref_id
MERGE (ref:Email {message_id: ref_id})
MERGE (e)-[:REFERENCES]->(ref)

WITH e, row
WHERE row.in_reply_to IS NOT NULL
MATCH (reply_to:Email {message_id: row.in_reply_to})
MERGE (e)-[:REPLIES_TO]->(reply_to)
```

## Data Analysis Queries

Here, we list the Cypher queries and scripts used for analyzing the ingested data. These queries are designed to extract insights from the graph database, such as identifying influential users, tracking the evolution of discussion topics, and analyzing communication patterns within the ASF project communities.

{{place-holder for the analysis queries}}

## Setup and Installation

This section provides instructions for setting up the project and installing any required dependencies. It includes steps for setting up Neo4j, importing the email data into the graph database, and preparing the environment for running the ingest and analysis queries.

{{place-holder for setup and installation instructions}}

## Usage

In this section, we explain how to use the project, including how to execute the ingest and analysis queries. It provides examples of commands or scripts to run, making it easy for users to get started with analyzing the ASF project email data.

{{place-holder for usage instructions}}

## License

This project is licensed under the Apache License 2.0. See the [LICENSE](LICENSE.txt) file for details.
