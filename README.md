

Um assistente que responde, em linguagem natural, dúvidas sobre políticas corporativas.  
Deploy: Docker → Cloud Run, custo médio ≈ 0,002 $/req.

## ✨ Funcionalidades
1. Ingestão automática de documentos (PDF)  
2. Vetorização em Pinecone  
3. Retrieval-Augmented Generation (RAG) com GPT-4o  
4. API REST (FastAPI) + streaming  
5. Monitoramento via OpenTelemetry → Grafana Cloud  

## 🔧 Arquitetura
```mermaid
flowchart LR
    A[Usuário] --> B[/Front-end\\/Chat widget/]
    B -->|POST /chat| C[FastAPI]
    C --> D(Pinecone Retriever)
    D --> E[Contexto]
    E --> F[GPT-4o]
    F --> C
    C -->|JSON| B

Métrica	Valor
Latência p95	1.2 s
Custo médio	0.002 $ por requisição
Precisão manual	85 %


