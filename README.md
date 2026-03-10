# ChatFramework

เฟรมเวิร์ก Chat AI ระดับองค์กรที่พัฒนาด้วย .NET Core 10 รองรับทั้ง Text Chat และ Voice Chat พร้อมความสามารถ Retrieval-Augmented Generation (RAG) และแนวทางการเชื่อมต่อระบบองค์กรแบบครบวงจร

## 1. ภาพรวมโครงการ (Project Overview)

ChatFramework เป็นแพลตฟอร์มแบบ backend-first สำหรับสร้างแชตบอตองค์กร โดยรองรับความสามารถหลักดังนี้

- รับและประมวลผลข้อความผู้ใช้ผ่าน API
- รองรับ Text Chat และ Voice Chat ใน workflow เดียวกัน
- เชื่อมต่อ LLM ได้ทั้ง Azure OpenAI และ OpenAI
- แปลงเสียงเป็นข้อความ (Speech-to-Text)
- แปลงข้อความตอบกลับเป็นเสียง (Text-to-Speech)
- ทำงานแบบ RAG ร่วมกับ Knowledge Base ภายในองค์กร
- รองรับการใช้งานระดับองค์กร เช่น observability, security, tenancy และ policy control

เป้าหมายหลักของโครงการ

- สร้างมาตรฐานกลางด้าน AI Chat สำหรับผลิตภัณฑ์ภายใน
- ลดเวลาในการพัฒนาจากต้นแบบสู่ production
- ออกแบบสถาปัตยกรรมให้ขยายต่อได้หลายโดเมนธุรกิจ

## 2. ภาพรวมสถาปัตยกรรม (Architecture Overview)

ChatFramework แนะนำให้ใช้ Architecture Trick สำหรับโปรเจกต์ .NET Core API โดยผสานแนวคิดต่อไปนี้เข้าด้วยกัน

- Clean Architecture: แยก Domain, Application, Infrastructure, API ชัดเจน
- Vertical Slice Architecture: แยกตาม Feature เพื่อให้แก้ไขและ deploy ได้รวดเร็ว
- Ports and Adapters: ใช้ interface เป็นสัญญาระหว่าง core logic และ external services
- CQRS แบบเบา (lightweight): แยก flow คำสั่ง (Command) และการอ่านข้อมูล (Query) ตามความเหมาะสม

หลักการสำคัญ (Architecture Trick)

- Dependency Rule: โค้ดชั้นในต้องไม่อ้างอิงโค้ดชั้นนอก
- Thin Controller: controller รับ request แล้วส่งต่อ use case ทันที
- Feature-First Structure: จัดโค้ดตาม business capability ไม่ใช่ตามชนิดไฟล์
- Cross-Cutting Pipeline: รวม validation, logging, tracing, authorization ไว้ใน pipeline เดียว
- Replaceable Adapters: เปลี่ยนผู้ให้บริการ LLM/Speech/Vector Store ได้โดยไม่กระทบ business logic

ภาพรวมเส้นทางคำขอ (request flow)

1. Client ส่งข้อมูลแบบข้อความหรือเสียง
2. API Endpoint/Controller รับคำขอและส่งเข้า Feature Handler
3. Application Layer ทำ orchestration และเลือก direct LLM หรือ RAG
4. RAG Module (ถ้าเปิดใช้งาน) ดึงบริบทจาก Knowledge Base
5. LLM Adapter สร้าง prompt และเรียก Azure OpenAI / OpenAI
6. Pipeline ทำ validation, policy check, logging และ telemetry
7. ส่งผลลัพธ์กลับเป็นข้อความ และแปลงเป็นเสียงเพิ่มเติมได้ตามต้องการ

โครงสร้างเชิงตรรกะ

- Presentation Layer: HTTP APIs, DTOs, authentication middleware
- Application Layer: Chat orchestration, use cases, workflow engine
- Domain Layer: Business rules, conversation context, policy logic
- Infrastructure Layer: LLM providers, vector database, storage, telemetry, speech services

## 3. เทคโนโลยีที่ใช้ (Technology Stack)

- Runtime: .NET Core 10
- Framework: ASP.NET Core Web API
- AI Providers: Azure OpenAI, OpenAI
- Speech Services:
  - Speech-to-Text: Azure Speech Service หรือผู้ให้บริการเทียบเท่า
  - Text-to-Speech: Azure Speech Service หรือผู้ให้บริการเทียบเท่า
- Data และ Storage:
  - Relational DB: SQL Server หรือ PostgreSQL
  - Cache: Redis
  - Vector Store: Azure AI Search / PostgreSQL pgvector / Pinecone (เลือกตามสภาพแวดล้อม)
- Observability:
  - OpenTelemetry
  - Application Insights / Prometheus + Grafana
- Security:
  - OAuth 2.0 / OpenID Connect
  - JWT Bearer tokens
  - RBAC และ policy-based authorization

## 4. องค์ประกอบของระบบ (System Components)

- API Layer
  - Chat Controller (Text Chat)
  - Voice Controller (Voice Chat)
  - Health และ diagnostics endpoints
- Chat Orchestrator
  - จัดการวงจรชีวิตของบทสนทนา
  - ตัดสินใจเลือกระหว่าง direct LLM หรือ RAG workflow
- LLM Connector
  - abstraction สำหรับเชื่อมต่อ Azure OpenAI / OpenAI
  - กลไก retry, timeout และ fallback
- Speech Module
  - STT adapter สำหรับเสียงขาเข้า
  - TTS adapter สำหรับเสียงขาออก
- RAG Module
  - ingestion และ chunking ของเอกสาร
  - สร้าง embeddings
  - retrieval จาก vector store และประกอบบริบท
- Knowledge Base Layer
  - ตัวเชื่อมต่อแหล่งข้อมูล (ไฟล์, intranet docs, DB)
  - จัดการ metadata tagging และ versioning
- Enterprise Module
  - tenant isolation
  - audit trail
  - rate limit และ quota control
- Cross-Cutting Components
  - Validation behavior
  - Exception handling middleware
  - Correlation ID + distributed tracing
  - Authorization policy enforcement

## 5. การติดตั้ง (Installation)

### สิ่งที่ต้องมี (Prerequisites)

- .NET SDK 10.x
- Git
- Credential สำหรับผู้ให้บริการ LLM และ Speech
- ทางเลือก: Docker (สำหรับ local dependencies)

### ขั้นตอนติดตั้ง

1. Clone repository

```bash
git clone <repository-url>
cd my-web-app-backend
```

2. Restore dependencies

```bash
dotnet restore
```

3. สร้างไฟล์ environment

```bash
cp .env.example .env
```

PowerShell (Windows)

```powershell
Copy-Item .env.example .env
```

4. Build โปรเจกต์

```bash
dotnet build
```

## 6. การตั้งค่า (Environment Variables)

กำหนดตัวแปรแวดล้อมต่อไปนี้ก่อนรันระบบ

| Variable                     | คำอธิบาย                                   | ตัวอย่าง                                           |
| ---------------------------- | ------------------------------------------ | -------------------------------------------------- |
| ASPNETCORE_ENVIRONMENT       | สภาพแวดล้อมการทำงาน                        | Development                                        |
| CHATFRAMEWORK_PORT           | พอร์ตของ API                               | 8080                                               |
| AUTH_AUTHORITY               | URL ของ OAuth/OIDC authority               | https://login.microsoftonline.com/<tenant-id>/v2.0 |
| AUTH_AUDIENCE                | API audience/client id                     | api://chatframework                                |
| OPENAI_PROVIDER              | ผู้ให้บริการ LLM (AzureOpenAI หรือ OpenAI) | AzureOpenAI                                        |
| AZURE_OPENAI_ENDPOINT        | Azure OpenAI endpoint                      | https://my-openai.openai.azure.com                 |
| AZURE_OPENAI_API_KEY         | Azure OpenAI API key                       | <secret>                                           |
| AZURE_OPENAI_CHAT_DEPLOYMENT | ชื่อ deployment บน Azure OpenAI            | gpt-4o-mini                                        |
| OPENAI_API_KEY               | OpenAI API key                             | <secret>                                           |
| OPENAI_MODEL                 | ชื่อโมเดล OpenAI                           | gpt-4.1-mini                                       |
| SPEECH_PROVIDER              | ผู้ให้บริการ Speech                        | AzureSpeech                                        |
| AZURE_SPEECH_KEY             | Speech service key                         | <secret>                                           |
| AZURE_SPEECH_REGION          | Speech service region                      | southeastasia                                      |
| VECTOR_STORE_PROVIDER        | ผู้ให้บริการ Vector DB                     | PgVector                                           |
| VECTOR_STORE_CONNECTION      | Vector DB connection string                | Host=localhost;Port=5432;...                       |
| DATABASE_CONNECTION          | Primary database connection                | Server=...;Database=...;                           |
| REDIS_CONNECTION             | Redis connection string                    | localhost:6379                                     |
| OTEL_EXPORTER_OTLP_ENDPOINT  | OpenTelemetry collector endpoint           | http://localhost:4317                              |

ข้อควรระวังด้านความปลอดภัย

- ห้าม commit secret ลง source control
- ใช้ secret manager ใน production เช่น Azure Key Vault, AWS Secrets Manager, HashiCorp Vault

## 7. การรันโปรเจกต์ (Running the Project)

รันในโหมดพัฒนา

```bash
dotnet run --project src/ChatFramework.Api
```

รันพร้อมกำหนด URL ชัดเจน

```bash
dotnet run --project src/ChatFramework.Api --urls "http://localhost:8080"
```

ตรวจสอบสถานะบริการ

```bash
curl http://localhost:8080/health
```

## 8. ตัวอย่าง API (API Example)

### ตัวอย่างคำขอ Text Chat

```http
POST /api/chat/message
Content-Type: application/json
Authorization: Bearer <token>

{
  "conversationId": "conv-001",
  "userId": "u-1001",
  "message": "สรุปนโยบายการลาของบริษัท",
  "enableRag": true,
  "locale": "th-TH"
}
```

### ตัวอย่างผลลัพธ์ Text Chat

```json
{
  "conversationId": "conv-001",
  "responseText": "นโยบายการลาของบริษัทประกอบด้วย...",
  "citations": [
    {
      "source": "HR-Policy-2026.pdf",
      "section": "Annual Leave"
    }
  ],
  "latencyMs": 842
}
```

### ตัวอย่างคำขอ Voice Chat

```http
POST /api/chat/voice
Content-Type: multipart/form-data
Authorization: Bearer <token>

form-data:
- audioFile: <binary>
- conversationId: conv-001
- enableRag: true
```

## 9. ลำดับการทำงาน Voice Chat (Voice Chat Flow)

1. Client อัปโหลดเสียงเข้ามาที่ Voice API
2. STT module แปลงเสียงเป็นข้อความ
3. Orchestrator เสริมบริบทคำขอและผล RAG (ถ้าเปิดใช้งาน)
4. LLM สร้างคำตอบเป็นข้อความ
5. TTS module แปลงข้อความเป็นเสียง
6. API ส่งกลับทั้งข้อความและ audio stream URL หรือ payload

ข้อพิจารณาสำหรับองค์กร

- กำหนดข้อจำกัดรูปแบบไฟล์เสียงและขนาดไฟล์
- บันทึกค่าความมั่นใจของผลถอดเสียงเพื่อติดตามคุณภาพ
- มี fallback เป็น text-only หากบริการเสียงขัดข้อง

## 10. เวิร์กโฟลว์ RAG (RAG Workflow)

1. Ingestion
   - รวบรวมเอกสารจากแหล่งข้อมูลที่เชื่อถือได้ในองค์กร
   - ทำ normalization และแบ่งเอกสารเป็น semantic chunks
2. Indexing
   - สร้าง embeddings ให้แต่ละ chunk
   - เก็บ embeddings และ metadata ใน vector store
3. Retrieval
   - แปลงคำถามผู้ใช้เป็น embedding
   - ดึงข้อมูล top-k พร้อมเงื่อนไขกรอง (tenant, department, tags)
4. Generation
   - สร้าง prompt สุดท้ายจากบริบทที่ดึงมาและ policy instruction
   - ส่งต่อให้ LLM provider
5. Grounded Response
   - ส่งคำตอบพร้อม citation และตัวชี้วัดความมั่นใจ

แนวปฏิบัติที่แนะนำ

- ใช้ metadata filtering เพื่อป้องกันข้อมูลข้าม tenant
- ปรับค่าขนาด chunk และ overlap ได้ตาม use case
- เพิ่ม evaluation pipeline เพื่อตรวจคุณภาพ retrieval และลด hallucination

## 11. โครงสร้างโฟลเดอร์ (Folder Structure)

โครงสร้างแบบย่อสำหรับเริ่มจากฝั่ง API อย่างเดียวก่อน

```text
my-web-app-backend/
├─ src/
│  └─ ChatFramework.Api/
│     ├─ Endpoints/
│     │  ├─ Chat/
│     │  └─ Voice/
│     ├─ Models/
│     ├─ Services/
│     ├─ Middleware/
│     ├─ Extensions/
│     ├─ appsettings.json
│     ├─ appsettings.Development.json
│     └─ Program.cs
├─ tests/
│  └─ ChatFramework.Api.Tests/
└─ README.md
```

โฟลเดอร์หลักที่ควรรู้ตอนเริ่ม

- Endpoints: เส้นทาง API แยกตามโดเมนงาน เช่น Chat และ Voice
- Models: Request/Response models และ DTO
- Services: business logic เบื้องต้นของ API
- Middleware: logging, error handling, auth
- Extensions: จุดรวมการ register services และ config

เมื่อระบบโตขึ้น ค่อยแยก Service ออกเป็น Application, Domain และ Infrastructure ตามแนว Clean Architecture ได้ภายหลัง โดยไม่ต้องรื้อ API ทั้งหมดใหม่

## 12. แผนพัฒนาในอนาคต (Future Improvements)

- ทำ Multi-LLM routing ตามนโยบาย latency/cost/quality
- รองรับการตอบแบบ real-time streaming (SSE / WebSocket)
- เพิ่ม guardrails ขั้นสูง เช่น PII redaction, jailbreak detection, policy engine
- จัดการ prompt และเวอร์ชันโมเดลแบบอัตโนมัติ
- สร้าง analytics dashboard สำหรับ usage, quality, และ cost
- เพิ่ม human handoff และ workflow สำหรับ escalation
- เพิ่มชุดทดสอบ offline evaluation สำหรับ RAG และ grounded response
- ปรับปรุงเสียงหลายภาษาให้เหมาะกับศัพท์เฉพาะของแต่ละโดเมน

---

สำหรับการใช้งานจริงในระดับองค์กร ควรผนวก ChatFramework เข้ากับมาตรฐานความปลอดภัยข้อมูล, data governance และกระบวนการ model risk management ขององค์กรอย่างเป็นทางการ
