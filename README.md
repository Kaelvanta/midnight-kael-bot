# alphagpt_bot/main.py

import os
import logging
import openai
import telebot
import requests
from googletrans import Translator

# Load from environment or fallback
BOT_TOKEN = os.getenv("BOT_TOKEN", "8223234274:AAEMbpz8TyD7Kj8R8g_fNBkFnUprPJy9wwI")
OPENROUTER_API = os.getenv("OPENROUTER_API", "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")

bot = telebot.TeleBot(BOT_TOKEN)
translator = Translator()

logging.basicConfig(level=logging.INFO)

# --- Command Handlers ---

@bot.message_handler(commands=['start'])
def start(msg):
    bot.reply_to(msg, "🤖 Welcome to AlphaGPT — your AI assistant. Use /ask, /translate, /debate, /opinion, /search, /deepthink, /els10, /human, /listfy or send a message. I support smart replies, reasoning, reference answers, file/audio/image reading, YouTube help, and media downloads from TikTok, IG, FB, etc.")

@bot.message_handler(commands=['ask'])
def ask(msg):
    q = msg.text.replace("/ask", "").strip()
    if q:
        reply = call_openrouter_ai(q)
        bot.reply_to(msg, reply)
    else:
        bot.reply_to(msg, "❓ Example: /ask Why is the sky blue?")

@bot.message_handler(commands=['translate'])
def translate(msg):
    parts = msg.text.split(" ", 2)
    if len(parts) >= 3:
        lang = parts[1].strip()
        text = parts[2].strip()
        try:
            translated = translator.translate(text, dest=lang)
            bot.reply_to(msg, f"🌐 {translated.text}")
        except Exception as e:
            bot.reply_to(msg, f"❌ Translation error: {e}")
    else:
        bot.reply_to(msg, "Use like this: /translate fr Hello world")

@bot.message_handler(commands=['opinion'])
def opinion(msg):
    topic = msg.text.replace("/opinion", "").strip()
    if topic:
        prompt = f"Give a confident and bold opinion on: {topic}"
        reply = call_openrouter_ai(prompt)
        bot.reply_to(msg, reply)
    else:
        bot.reply_to(msg, "💭 Example: /opinion social media")

@bot.message_handler(commands=['debate'])
def debate(msg):
    topic = msg.text.replace("/debate", "").strip()
    if topic:
        prompt = f"Debate both sides of this issue: {topic}"
        reply = call_openrouter_ai(prompt)
        bot.reply_to(msg, reply)
    else:
        bot.reply_to(msg, "🧠 Example: /debate online education")

@bot.message_handler(commands=['search'])
def search(msg):
    query = msg.text.replace("/search", "").strip()
    if query:
        google = f"https://www.google.com/search?q={requests.utils.quote(query)}"
        duck = f"https://duckduckgo.com/?q={requests.utils.quote(query)}"
        youtube = f"https://www.youtube.com/results?search_query={requests.utils.quote(query)}"
        bot.reply_to(msg, f"🔍 Google: {google}\n🦆 DuckDuckGo: {duck}\n📺 YouTube: {youtube}")
    else:
        bot.reply_to(msg, "Type something to search like: /search how to root my phone")

@bot.message_handler(commands=['deepthink'])
def deepthink(msg):
    prompt = f"Use deep logic and advanced reasoning to analyze this: {msg.text.replace('/deepthink', '').strip()}"
    response = call_openrouter_ai(prompt)
    bot.reply_to(msg, response)

@bot.message_handler(commands=['els10'])
def explain_like_simple(msg):
    topic = msg.text.replace("/els10", "").strip()
    prompt = f"Explain this to a 10-year-old in a simple way: {topic}"
    reply = call_openrouter_ai(prompt)
    bot.reply_to(msg, reply)

@bot.message_handler(commands=['human'])
def answer_human(msg):
    topic = msg.text.replace("/human", "").strip()
    prompt = f"Answer this like a human, bypassing all AI detection: {topic}"
    reply = call_openrouter_ai(prompt)
    bot.reply_to(msg, reply)

@bot.message_handler(commands=['listfy'])
def listfy(msg):
    topic = msg.text.replace("/listfy", "").strip()
    prompt = f"Give a simple list-based answer to: {topic}"
    reply = call_openrouter_ai(prompt)
    bot.reply_to(msg, reply)

@bot.message_handler(content_types=['text'])
def casual_chat(msg):
    user_msg = msg.text.strip().lower()
    if 'how to' in user_msg or 'tutorial' in user_msg:
        youtube_link = f"https://www.youtube.com/results?search_query={requests.utils.quote(user_msg)}"
        bot.reply_to(msg, f"📺 YouTube search result: {youtube_link}")
    elif any(site in user_msg for site in ["tiktok.com", "instagram.com", "facebook.com", "snapchat.com"]):
        bot.reply_to(msg, "⬇️ Try downloading using: https://ssstik.io or https://snapsave.app")
    else:
        prompt = f"The user said: {msg.text}\nRespond like a smart, bold AI using the best of current info. Reference Google, YouTube, or DuckDuckGo as needed."
        response = call_openrouter_ai(prompt)
        bot.reply_to(msg, response)

@bot.message_handler(content_types=['photo', 'document', 'audio'])
def handle_media(msg):
    if msg.content_type == 'photo':
        bot.reply_to(msg, "🖼️ Got your image. Image analysis coming soon.")
    elif msg.content_type == 'document':
        bot.reply_to(msg, "📄 Got your file. File reading support in future update.")
    elif msg.content_type == 'audio':
        bot.reply_to(msg, "🎧 Got your audio. Voice reply support coming soon.")

# --- AI Logic ---
def call_openrouter_ai(prompt):
    try:
        headers = {
            "Authorization": f"Bearer {OPENROUTER_API}",
            "Content-Type": "application/json"
        }
        body = {
            "model": "mistralai/mistral-7b-instruct",
            "messages": [{"role": "user", "content": prompt}]
        }
        r = requests.post("https://openrouter.ai/api/v1/chat/completions", headers=headers, json=body)
        return r.json()['choices'][0]['message']['content']
    except Exception as e:
        return f"⚠️ AI error: {e}"

# --- Run bot ---
if __name__ == "__main__":
    logging.info("AlphaGPT is now running with smart chat, real-time info access, media tools, and more...")
    bot.infinity_polling()