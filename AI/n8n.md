There are **over 1,900 integration nodes** in modern n8n, covering SaaS applications, databases, AI services, developer tools, cloud platforms, communication apps, and utilities. The exact number changes as new integrations are added. ([n8n][1])

Rather than memorizing thousands of nodes, learn the **core nodes** first. Once you understand these, you can use almost any integration node.

# 1. Trigger Nodes (Start a Workflow)

These nodes start the automation.

| Node                     | Purpose                                 |
| ------------------------ | --------------------------------------- |
| Manual Trigger           | Start a workflow manually for testing   |
| Schedule Trigger         | Run at specific times or intervals      |
| Webhook                  | Start from an HTTP request              |
| Error Trigger            | Run when another workflow fails         |
| Workflow Trigger         | Trigger from another workflow           |
| Execute Workflow Trigger | Receive data from another workflow      |
| Chat Trigger             | Start from a chat interface             |
| Local File Trigger       | Watch files/folders                     |
| RSS Feed Trigger         | Detect new RSS posts                    |
| SSE Trigger              | Listen to Server Sent Events            |
| n8n Trigger              | Trigger on n8n events                   |
| MCP Server Trigger       | Receive Model Context Protocol requests |
| Form Trigger             | Start when an n8n form is submitted     |

---

# 2. Logic Nodes

Control how data flows.

| Node              | Purpose                            |
| ----------------- | ---------------------------------- |
| IF                | Conditional branching              |
| Switch            | Multiple conditions                |
| Filter            | Keep matching items                |
| Remove Duplicates | Remove duplicate records           |
| Merge             | Combine two data streams           |
| Compare Datasets  | Compare two datasets               |
| Wait              | Pause execution                    |
| Stop and Error    | Stop workflow intentionally        |
| Loop Over Items   | Process items one at a time        |
| Split Out         | Split arrays into individual items |
| Limit             | Limit number of records            |
| Sort              | Sort records                       |
| Aggregate         | Group data                         |
| Summarize         | Calculate totals and statistics    |

---

# 3. Data Transformation Nodes

Modify data before sending it elsewhere.

| Node              | Purpose                         |
| ----------------- | ------------------------------- |
| Set (Edit Fields) | Add or modify fields            |
| Rename Keys       | Rename JSON properties          |
| Code              | JavaScript or Python processing |
| Date & Time       | Manipulate dates                |
| XML               | Convert XML ↔ JSON              |
| JSON              | Work with JSON                  |
| HTML              | Parse HTML                      |
| Markdown          | Convert Markdown                |
| CSV               | Read/write CSV                  |
| Compression       | Zip and unzip files             |
| Convert to File   | Create PDFs, CSVs, etc.         |
| Extract From File | Extract text or tables          |
| Crypto            | Encrypt, hash, sign             |
| JWT               | Create or verify JWTs           |
| TOTP              | Generate OTP codes              |

---

# 4. HTTP & API Nodes

Interact with REST APIs.

| Node               | Purpose                 |
| ------------------ | ----------------------- |
| HTTP Request       | Call any REST API       |
| GraphQL            | Execute GraphQL queries |
| Webhook Response   | Respond to a webhook    |
| Respond to Webhook | Return HTTP responses   |

---

# 5. AI Nodes

n8n includes a large set of AI-focused nodes organized into clusters. ([n8n Docs][2])

Examples include:

* AI Agent
* Chat Model
* OpenAI
* Anthropic
* Google Gemini
* Mistral
* Ollama
* Embeddings
* Vector Store
* Memory
* Retriever
* Document Loader
* Text Splitter
* Output Parser
* Tool
* Guardrails
* MCP Client

---

# 6. Database Nodes

| Node          | Purpose            |
| ------------- | ------------------ |
| PostgreSQL    | SQL operations     |
| MySQL         | SQL operations     |
| MariaDB       | SQL operations     |
| Microsoft SQL | SQL Server         |
| MongoDB       | NoSQL database     |
| Redis         | Cache and queues   |
| SQLite        | Local database     |
| Supabase      | PostgreSQL backend |
| Airtable      | Cloud database     |
| Baserow       | No-code database   |

---

# 7. File Nodes

| Node        | Purpose                 |
| ----------- | ----------------------- |
| Read Files  | Read local files        |
| Write Files | Save files              |
| FTP         | Transfer files          |
| SFTP        | Secure file transfer    |
| SSH         | Execute remote commands |

---

# 8. Email Nodes

| Node              | Purpose              |
| ----------------- | -------------------- |
| Gmail             | Send/read Gmail      |
| Microsoft Outlook | Outlook email        |
| IMAP Email        | Read inbox           |
| SMTP Email        | Send email           |
| Send Email        | Generic email sender |

---

# 9. Communication Nodes

Examples include:

* Slack
* Discord
* Microsoft Teams
* Telegram
* Twilio
* WhatsApp
* Signal
* Mattermost

---

# 10. Google Workspace Nodes

Examples include:

* Google Sheets
* Google Drive
* Google Docs
* Google Calendar
* Gmail
* Google Contacts
* Google Tasks
* Google BigQuery

---

# 11. Microsoft 365 Nodes

Examples include:

* Excel
* Outlook
* OneDrive
* SharePoint
* Teams
* Dynamics 365

---

# 12. Developer Nodes

Useful for software engineers.

* GitHub
* GitLab
* Bitbucket
* Docker
* Kubernetes
* Jenkins
* Jira
* Linear
* Postman
* PagerDuty

---

# 13. Cloud Nodes

Examples include:

* AWS
* Azure
* Google Cloud
* Cloudflare
* DigitalOcean
* Vercel
* Netlify

---

# 14. CRM Nodes

Examples include:

* Salesforce
* HubSpot
* Zoho CRM
* Pipedrive
* Monday.com
* Notion

---

# 15. Social Media Nodes

Examples include:

* LinkedIn
* Facebook
* Instagram
* X (Twitter)
* TikTok
* YouTube

---

# 16. E-commerce Nodes

Examples include:

* Shopify
* WooCommerce
* Stripe
* PayPal
* Square

---

# 17. Productivity Nodes

Examples include:

* Notion
* Trello
* Asana
* ClickUp
* Todoist
* Monday.com

---

# 18. Messaging Nodes

Examples include:

* Telegram
* Slack
* Discord
* Twilio SMS
* WhatsApp

---

# 19. Monitoring Nodes

Examples include:

* Datadog
* Grafana
* Prometheus
* Sentry

---

# 20. Utility Nodes

Examples include:

* LDAP
* DNS
* QR Code
* UUID
* Binary Data
* Environment Variables

---

## The 20 most important nodes to master first

If you can confidently use these, you'll be able to build most real-world automations:

1. Manual Trigger
2. Schedule Trigger
3. Webhook
4. HTTP Request
5. Set (Edit Fields)
6. Code
7. IF
8. Switch
9. Merge
10. Loop Over Items
11. Aggregate
12. Filter
13. Wait
14. Respond to Webhook
15. Execute Workflow
16. Google Sheets
17. Gmail
18. Slack
19. OpenAI / AI Agent
20. PostgreSQL

These nodes form the foundation for most production workflows and are the best place to begin. The complete node library is organized into core nodes, cluster nodes, and app integrations, with new integrations added regularly. ([n8n Docs][3])

For the training manual you're building, I recommend organizing the course around these 20 core nodes first, then expanding into API integrations, AI agents, databases, cloud services, and advanced automation patterns.

[1]: https://n8n.io/integrations?utm_source=chatgpt.com "Best apps & software integrations | n8n"
[2]: https://docs.n8n.io/integrations/builtin/cluster-nodes/?utm_source=chatgpt.com "Cluster nodes | n8n Docs"
[3]: https://main--n8n-docs.netlify.app/integrations/builtin/node-types/?utm_source=chatgpt.com "Node types | n8n Docs"
