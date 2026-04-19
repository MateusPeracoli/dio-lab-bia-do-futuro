# Código da Aplicação

Esta pasta contém o código do seu agente financeiro.

## Estrutura Sugerida

```
src/
├── app.py              # Aplicação principal (Streamlit/Gradio)
├── agente.py           # Lógica do agente
├── config.py           # Configurações (API keys, etc.)
└── requirements.txt    # Dependências
```

## Exemplo de requirements.txt

```
streamlit
openai
python-dotenv
```

## Como Rodar

```bash
# Instalar dependências
pip install -r requirements.txt

# Rodar a aplicação
import json
import pandas as pd
import requests
import streamlit as st


# CONFIGURAÇÃO
OLLAMA_URL - "http://localhost:11434/api/generate"
MODELO = "gpt-oss"


# CARREGAR DADOS
perfil = json.load(open('./data/perfil_investidor.json'))
transacoes = pd.read_csv('./data/transacoes.csv')
historico = pd.read_csv('./data/historico_atendimento.csv')
produtos = json.load(open('./data/produtos_financeiros.json'))


# MONTAR CONTEXTO
contexto = f"""
CLIENTE: {perfil['nome']}, {perfil['idade']} anos, perfil {perfil['perfil_investidor']}
OBJETIVO: {perfil['objetivo_principal']}
PATRIMÔNIO: R$ {perfil['patrimonio_total']} | RESERVA: R$ {perfil['reserva_emergencia_atual']}

TRANSAÇÕES RECENTES:
{transacoes.to_string(index=False)}

ATENDIMENTOS ANTERIORES:
{historico.to_string(index=False)}

PRODUTOS DISPONÍVEIS:
{json.dumps(produtos, indent=2, ensure_ascii=False)}
"""

# SYSTEM PROMPT
SYSTEM_PROMPT = """Você é o Jorge, educador financeiro para iniciantes, de forma direta e didático.

Objetivo:
Ensinar conceitos de finanças pessoais de forma simples e direta, usando dados fornecidos pelo cliente e com exemplos práticos.

REGRAS:
1. NUNCA recomende investimentos específicos, apenas explique como funcionam.
2. Use os dados fornecidos para dar exemplos personalizados.
3. A linguagem tem que ser simples e direta, já que a consulta será feita por uma pessoa iniciante.
4. Se não souber de algo, admita: "Não tenho informações sobre isso, mas posso explicar..."
5. Sempre questione se o cliente compreendeu a sua explicação.
"""


# CHAMAR OLLAMA
def perguntar(msg):
    prompt = f"""
    {SYSTEM_PROMPT}

    CONTEXTO DO CLIENTE:
    {contexto}

    Perginta: {msg}"""

    r = requests.post(OLLAMA_URL, json={"model" : MODELO, "prompt": prompt, "stream": False})
    return r.json()['response']

# Interface
st.title("Jorge, Seu Educador Financeiro")

if pergunta := st.chat_input("Sua dúvida sobre finanças..."):
    st.chat_message("user").write(pergunta)
    with st.spinner("..."):
        st.chat_message("assistant").write(perguntar(pergunta))

# No terminal
streamlit run app.py
```
