# chat-bot-using-llm-models

import streamlit as st
from dotenv import load_dotenv
from langchain_mistralai import ChatMistralAI
from langchain_core.messages import (
    HumanMessage,
    AIMessage,
    SystemMessage,
)
import time

# ------------------------
# CONFIG
# ------------------------

load_dotenv()

st.set_page_config(
    page_title="Mood Based AI Chatbot",
    page_icon="😊",
    layout="centered"
)

# ------------------------
# CSS
# ------------------------

st.markdown("""
<style>

.stApp{
    background:#050b16;
}

.block-container{
    max-width:900px;
    padding-top:2rem;
}

.title{
    text-align:center;
    font-size:60px;
    font-weight:800;
    color:white;
    margin-bottom:10px;
}

.subtitle{
    text-align:center;
    color:#8b95a7;
    font-size:20px;
    margin-bottom:30px;
}

hr{
    border-color:#1f2937;
}

.stButton button{
    background:#161f2f;
    color:white;
    border-radius:12px;
    border:1px solid #2f3a4f;
    height:50px;
    width:180px;
    font-size:18px;
}

.stButton button:hover{
    border:1px solid #4f46e5;
}

div[data-testid="stRadio"] > div{
    flex-direction:row;
    gap:40px;
}

[data-testid="stChatMessage"]{
    background:#111827;
    border-radius:15px;
    padding:10px;
}

</style>
""", unsafe_allow_html=True)

# ------------------------
# HEADER
# ------------------------

st.markdown(
    '<div class="title">😊 Mood Based AI Chatbot</div>',
    unsafe_allow_html=True
)

st.markdown(
    '<div class="subtitle">Choose AI personality and start chatting</div>',
    unsafe_allow_html=True
)

# ------------------------
# MODE SELECTOR
# ------------------------

st.write("### Choose your AI Mode:")

mode = st.radio(
    "",
    ["😡 Angry", "😂 Funny", "🥺 Sad"],
    horizontal=True
)

st.divider()

# ------------------------
# RESET BUTTON
# ------------------------

if st.button("🔄 Reset Chat"):
    st.session_state.messages = []
    st.rerun()

# ------------------------
# SYSTEM PROMPTS
# ------------------------

if mode == "😡 Angry":
    system_prompt = (
        "You are an angry AI assistant. "
        "Reply in an irritated but not abusive way."
    )

elif mode == "😂 Funny":
    system_prompt = (
        "You are a funny AI assistant. "
        "Reply humorously and add jokes."
    )

else:
    system_prompt = (
        "You are a sad AI assistant. "
        "Reply in a sad emotional style."
    )

# ------------------------
# MISTRAL MODEL
# ------------------------

model = ChatMistralAI(
    model="mistral-small-latest",
    temperature=0.9
)

# ------------------------
# CHAT MEMORY
# ------------------------

if "messages" not in st.session_state:

    st.session_state.messages = [
        SystemMessage(content=system_prompt)
    ]

# Update personality if changed

if (
    st.session_state.messages
    and isinstance(
        st.session_state.messages[0],
        SystemMessage
    )
):
    st.session_state.messages[0] = (
        SystemMessage(content=system_prompt)
    )

# ------------------------
# DISPLAY HISTORY
# ------------------------

for msg in st.session_state.messages:

    if isinstance(msg, HumanMessage):

        with st.chat_message("user"):
            st.markdown(msg.content)

    elif isinstance(msg, AIMessage):

        with st.chat_message("assistant"):
            st.markdown(msg.content)

# ------------------------
# INPUT
# ------------------------

prompt = st.chat_input(
    "Say something..."
)

# ------------------------
# RESPONSE
# ------------------------

if prompt:

    st.session_state.messages.append(
        HumanMessage(content=prompt)
    )

    with st.chat_message("user"):
        st.markdown(prompt)

    response = model.invoke(
        st.session_state.messages
    )

    st.session_state.messages.append(
        AIMessage(content=response.content)
    )

    with st.chat_message("assistant"):

        placeholder = st.empty()

        text = ""

        for word in response.content.split():

            text += word + " "

            placeholder.markdown(text)

            time.sleep(0.02)
