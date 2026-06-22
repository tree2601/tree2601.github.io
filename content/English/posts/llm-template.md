---
date: '2026-06-22T00:00:00+08:00'
draft: false
title: 'LLM Call Templates'
categories: ["AI"]
tags: ["LLM", "Qwen", "Embedding", "Reranker", "MinerU"]
keywords: ["LLM", "Qwen3.6", "Qwen3 Embedding", "Qwen3 Reranker", "MinerU"]
---

### Overview

This note collects four copy-paste friendly call templates: `qwen3.6-27b`, `qwen3-embedding`, `qwen3-reranker`, and `MinerU`.

### Environment Variables

```bash
pip install openai requests

export API_KEY="your-api-key"
export LLM_API="http://your-llm-host/v1"
export LLM_MODEL="qwen3.6-27b"

export EMBEDDING_API="http://your-embedding-host/v1"
export EMBEDDING_MODEL="qwen3-embedding"

export RERANKER_API="http://your-reranker-host/score"
export RERANKER_MODEL="qwen3-reranker"

export MINERU_API_KEY="your-mineru-api-key"
```

### qwen3.6-27b

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("API_KEY"),
    base_url=os.getenv("LLM_API"),
)

response = client.chat.completions.create(
    model=os.getenv("LLM_MODEL", "qwen3.6-27b"),
    messages=[
        {"role": "system", "content": "You are an assistant that strictly returns JSON."},
        {"role": "user", "content": "Return a JSON object with the fields name and age."},
    ],
    temperature=0,
    response_format={"type": "json_object"},
    extra_body={"chat_template_kwargs": {"enable_thinking": False}},
)

print(response.choices[0].message.content)
```

### qwen3-embedding

```python
import os
from openai import OpenAI

EMBEDDING_INSTRUCTION = "Given a retrieval query, retrieve relevant titles that can answer the query."

client = OpenAI(
    api_key=os.getenv("API_KEY"),
    base_url=os.getenv("EMBEDDING_API"),
)

def format_query(query: str) -> str:
    return f"Instruct: {EMBEDDING_INSTRUCTION}\nQuery: {query.strip()}"

def embed(text: str, *, is_query: bool = False) -> list[float]:
    response = client.embeddings.create(
        model=os.getenv("EMBEDDING_MODEL", "qwen3-embedding"),
        input=format_query(text) if is_query else text.strip(),
    )
    return response.data[0].embedding

query_vector = embed("Can pregnant women use this?", is_query=True)
doc_vector = embed("Pregnant women must not use this product.")

print(len(query_vector), len(doc_vector))
```

### qwen3-reranker

```python
import os
import requests

PREFIX = (
    "<|im_start|>system\n"
    "Judge whether the Document meets the requirements based on the Query and the Instruct provided. "
    "Note that the answer can only be \"yes\" or \"no\".<|im_end|>\n"
    "<|im_start|>user\n"
)
SUFFIX = "<|im_end|>\n<|im_start|>assistant\n<think>\n\n</think>\n\n"
INSTRUCTION = "Find the most relevant documents for the user query."

def format_query(query: str) -> str:
    return f"{PREFIX}<Instruct>: {INSTRUCTION}\n<Query>: {query.strip()}"

def format_document(document: str) -> str:
    return f"<Document>: {document.strip()}{SUFFIX}"

payload = {
    "model": os.getenv("RERANKER_MODEL", "qwen3-reranker"),
    "query": format_query("Pregnant women must not use this product."),
    "documents": [format_document(doc) for doc in [
        "Pregnant women must not use this product.",
        "Pregnant women should remember to use this product.",
        "This product is prohibited for pregnant women.",
        "Pregnant women can use this product.",
    ]],
    "top_n": 4,
}

response = requests.post(
    os.getenv("RERANKER_API"),
    headers={
        "Content-Type": "application/json",
        "Authorization": f"Bearer {os.getenv('API_KEY')}",
    },
    json=payload,
    timeout=300,
)
response.raise_for_status()
print(response.json()["results"])
```

### MinerU

```python
import os
import time
import requests

TOKEN = os.getenv("MINERU_API_KEY")
FILE_PATH = "example.pdf"
BASE_URL = "https://mineru.net/api/v4"
HEADERS = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {TOKEN}",
}

data = {
    "files": [{"name": FILE_PATH, "data_id": "example-001"}],
    "model_version": "vlm",
}

res = requests.post(f"{BASE_URL}/file-urls/batch", headers=HEADERS, json=data, timeout=300)
res.raise_for_status()
result = res.json()
batch_id = result["data"]["batch_id"]
upload_url = result["data"]["file_urls"][0]

with open(FILE_PATH, "rb") as f:
    upload = requests.put(upload_url, data=f, timeout=300)
    upload.raise_for_status()

for _ in range(60):
    time.sleep(5)
    res = requests.get(f"{BASE_URL}/extract-results/batch/{batch_id}", headers=HEADERS, timeout=300)
    res.raise_for_status()
    item = res.json()["data"]["extract_result"][0]
    if item.get("state") == "done":
        print(item["full_zip_url"])
        break
else:
    raise TimeoutError(f"MinerU task is not done: {batch_id}")
```
