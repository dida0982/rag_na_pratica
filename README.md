# RAG na prática — Assistente de FAQ com ChromaDB + OpenAI

Implementação mínima e didática de **RAG (Retrieval-Augmented Generation)** em Python: um assistente que responde dúvidas de clientes de um restaurante fictício (**Restaurante Sabores**) usando **somente** as informações de um FAQ, sem inventar respostas.

> *Minimal RAG pipeline in Python: text chunking with overlap, vector storage in ChromaDB and grounded answers with the OpenAI API.*

---

## Como funciona

```
texto.txt ──► chunking (1000 chars, overlap 200) ──► ChromaDB (persistente em ./db)
                                                          │
pergunta do usuário ──► busca semântica (top-2 trechos) ◄─┘
                                │
                                ▼
                 prompt de sistema + contexto recuperado ──► LLM (OpenAI) ──► resposta
```

1. **Ingestão (`parte_1.py`)**
   - Lê o FAQ em `texto.txt`.
   - Divide o texto em pedaços de 1000 caracteres com **sobreposição de 200**, para não perder contexto entre um trecho e outro.
   - Grava os pedaços em uma coleção do **ChromaDB** persistida em disco (`./db`), que gera os embeddings automaticamente.

2. **Consulta (`parte_2.py`)**
   - Recebe a pergunta do usuário no terminal.
   - Recupera os **2 trechos mais relevantes** por similaridade semântica.
   - Envia ao modelo um prompt de sistema com regras claras:
     - responder **apenas com base no contexto**;
     - se a informação não estiver no contexto, responder *"Desculpe mas não consigo ajudar."*;
     - encerrar com a assinatura do restaurante.

Essa restrição é o ponto central do RAG: reduzir alucinações e manter as respostas ancoradas em uma fonte confiável.

---

## Tecnologias

- **Python 3**
- **ChromaDB** — banco vetorial com persistência local
- **OpenAI API** — geração da resposta (`gpt-3.5-turbo`)

---

## Como executar

```bash
# 1. Ambiente virtual
python -m venv venv
# Windows (PowerShell)
./venv/Scripts/Activate.ps1
# Linux / macOS
source venv/bin/activate

# 2. Dependências
pip install chromadb openai

# 3. Configure sua chave da OpenAI em parte_2.py (variável api_key)

# 4. Indexe o FAQ
python parte_1.py

# 5. Faça perguntas
python parte_2.py
```

Exemplo de uso (ilustrativo):

```
Como posso lhe ajudar? Quanto custa o jantar durante a semana?
O jantar durante a semana (segunda a sexta, exceto feriados) custa R$ 149,90.
Foi um prazer lhe atender, não deixe de provar nossos sabores.
```

---

## Estrutura

```
.
├── texto.txt           # Base de conhecimento (FAQ do restaurante)
├── parte_1.py          # Chunking + indexação no ChromaDB
├── parte_2.py          # Recuperação + geração da resposta
├── codigos_usados.txt  # Passo a passo dos comandos usados
└── db/                 # Banco vetorial persistido
```

---

## Próximos passos

- Ler a chave da API de variável de ambiente (`OPENAI_API_KEY`) em vez do código
- Usar `upsert` na indexação para poder reindexar sem conflito de IDs
- Enviar o contexto recuperado como mensagem de usuário delimitada e citar a fonte de cada trecho
- Avaliar a qualidade das respostas com um conjunto de perguntas de teste
- Trocar o chunking por caracteres por divisão por sentenças/parágrafos

---

Feito por [Guilherme Barros](https://github.com/dida0982).
