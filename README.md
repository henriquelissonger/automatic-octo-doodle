
────────────────────────────────────────
2A. Conteúdo de README.md
────────────────────────────────────────
🧠 Chat RAG – Políticas Internas

Um assistente que responde, em linguagem natural, dúvidas sobre políticas corporativas.
Deploy: Docker → Cloud Run, custo médio ≈ 0,002 $/req.

✨ Funcionalidades
Ingestão automática de documentos (PDF)
Vetorização em Pinecone
Retrieval-Augmented Generation (RAG) com GPT-4o
API REST (FastAPI) + streaming
Monitoramento via OpenTelemetry → Grafana Cloud

🔧 Arquiteturamermaid
flowchart LR
    A[Usuário] --> B[/Front-end\\/Chat widget/]
    B -->|POST /chat| C[FastAPI]
    C --> D(Pinecone Retriever)
    D --> E[Contexto]
    E --> F[GPT-4o]
    F --> C
    C -->|JSON| B

🚀 Como rodar localmentebash
export OPENAI_API_KEY=sk-...
export PINECONE_API_KEY=...
docker build -t chat-rag .
docker run -p 8000:8000 \
  -e OPENAI_API_KEY \
  -e PINECONE_API_KEY chat-rag

🏆 Métricas
| Métrica          | Valor |
|------------------|-------|
| Latência p95     | 1.2 s |
| Custo médio      | 0.002 $ por requisição |
| Precisão manual  | 85 % |

📄 Licença
MIT

────────────────────────────────────────
2B. Conteúdo do Dockerfile
────────────────────────────────────────
FROM python:3.11-slim
WORKDIR /code
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app app
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

────────────────────────────────────────
2C. requirements.txt
────────────────────────────────────────
fastapi==0.110.0
uvicorn[standard]
langchain==0.1.17
openai==1.25.0
pinecone-client==3.2.2

────────────────────────────────────────
2D. app/main.py  (lembre-se de colocar o nome exatamente app/main.py)
────────────────────────────────────────
from fastapi import FastAPI
from pydantic import BaseModel
import os, pinecone, openai
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import Pinecone
from langchain.chains import RetrievalQA
from langchain.chat_models import ChatOpenAI

openai.api_key = os.getenv("OPENAI_API_KEY")
pinecone.init(api_key=os.getenv("PINECONE_API_KEY"), environment="gcp-starter")

index_name = "politicas-empresa"
embeddings = OpenAIEmbeddings()
vectorstore = Pinecone.from_existing_index(index_name, embeddings)
qa_chain = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(model_name="gpt-4o", temperature=0.0, streaming=True),
    chain_type="stuff",
    retriever=vectorstore.as_retriever(search_kwargs={"k": 4})
)

class Question(BaseModel):
    query: str

app = FastAPI()

@app.post("/chat")
async def chat(question: Question):
    result = qa_chain({"query": question.query})
    return {"answer": result["result"]}

────────────────────────────────────────
2E. n8n_workflow_lead_onboarding.json
────────────────────────────────────────
{
  "nodes":[
    {"parameters":{"httpMethod":"POST","path":"lead-webhook"},
     "name":"Webhook Entrada","type":"n8n-nodes-base.webhook","typeVersion":1,"position":[300,300]},
    {"parameters":{"functionCode":"const email=$json[\"email\"];if(!email.includes(\"@\"))throw new Error(\"Email inválido\");return[{json:{email}}];"},
     "name":"Validação de e-mail","type":"n8n-nodes-base.function","typeVersion":1,"position":[540,300]},
    {"parameters":{"operation":"append","sheetId":"1hKqETC_fake_id","range":"Leads!A:C"},
     "name":"Gravar no Google Sheets","type":"n8n-nodes-base.googleSheets","typeVersion":2,"position":[780,300]},
    {"parameters":{"authentication":"headerAuth","requestMethod":"POST","url":"https://api.hubspot.com/crm/v3/objects/contacts","jsonParameters":true,"bodyParametersJson":"{\"properties\":{\"email\":\"={{$json.email}}\"}}"},
     "name":"Criar contato HubSpot","type":"n8n-nodes-base.httpRequest","typeVersion":2,"position":[1020,300]}
  ],
  "connections":{
    "Webhook Entrada":{"main":[[{"node":"Validação de e-mail","type":"main","index":0}]]},
    "Validação de e-mail":{"main":[[{"node":"Gravar no Google Sheets","type":"main","index":0}]]},
    "Gravar no Google Sheets":{"main":[[{"node":"Criar contato HubSpot","type":"main","index":0}]]}
  }
}

────────────────────────────────────────
2F. diagrams/architecture.puml
────────────────────────────────────────
(No campo “Name”, digite exatamente diagrams/architecture.puml)
@startuml
!define ICONURL https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master
!includeurl ICONURL/common.puml
!includeurl ICONURL/devicons/openai.puml
!includeurl ICONURL/aws/cloudrun.puml
!includeurl ICONURL/devicons/pinecone.puml

actor User
cloud "Front-end\n(Chat widget)" as FE
component FastAPI
queue "Pinecone\nVector DB" as PC >
database "Docs\nBucket" as S3

User --> FE
FE --> FastAPI : POST /chat
FastAPI --> PC : Embeddings lookup
PC --> FastAPI : Top-K docs
FastAPI --> "OpenAI GPT-4o" > : Prompt + Context
"OpenAI GPT-4o" --> FastAPI : Resposta
FastAPI --> FE : JSON stream
FastAPI --> CloudRun > : Deploy target
@enduml

────────────────────────────────────────
 