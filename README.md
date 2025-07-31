# midnight-kael-bot
midnight_kael_bot/main.py

import os import logging import openai import telebot from googletrans import Translator

Load from environment or fallback

BOT_TOKEN = os.getenv("BOT_TOKEN", "8223234274:AAEMbpz8TyD7Kj8R8g_fNBkFnUprPJy9wwI") OPENROUTER_API = os.getenv("OPENROUTER_API", "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")

bot = telebot.TeleBot(BOT_TOKEN) translator = Translator()

logging.basicConfig(level=logging.INFO)

--- Command Handlers ---

@bot.message_handler(commands=['start']) def start(msg): bot.reply_to(msg, "🤖 Midnight Kael Bot is online! Use /ask, /translate, /debate, /opinion, or just chat with me.")

@bot.message_handler(commands=['ask']) def ask(msg): q = msg.text.replace("/ask", "").strip() if q: reply = call_openrouter_ai(q) bot.reply_to(msg, reply) else: bot.reply_to(msg, "❓ Send a question like: /ask Why is the sky blue?")

@bot.message_handler(commands=['translate']) def translate(msg): parts = msg.text.split(" ", 2) if len(parts) >= 3: lang = parts[1].strip() text = parts[2].strip() try: translated = translator.translate(text, dest=lang) bot.reply_to(msg, f"🌍 {translated.text}") except Exception as e: bot.reply_to(msg, f"❌ Translation error: {e}") else: bot.reply_to(msg, "Use like this: /translate es Hello world")

@bot.message_handler(commands=['opinion']) def opinion(msg): topic = msg.text.replace("/opinion", "").strip() if topic: prompt = f"What's your opinion on {topic}?" reply = call_openrouter_ai(prompt) bot.reply_to(msg, reply) else: bot.reply_to(msg, "🗣️ Send a topic like: /opinion AI in education")

@bot.message_handler(commands=['debate']) def debate(msg): topic = msg.text.replace("/debate", "").strip() if topic: prompt = f"Debate both sides of this: {topic}" reply = call_openrouter_ai(prompt) bot.reply_to(msg, reply) else: bot.reply_to(msg, "⚖️ Use like this: /debate Social media is good or bad")

@bot.message_handler(content_types=['text']) def casual_chat(msg): response = call_openrouter_ai(msg.text) bot.reply_to(msg, response)

--- AI Logic ---

def call_openrouter_ai(prompt): try: import requests headers = { "Authorization": f"Bearer {OPENROUTER_API}", "Content-Type": "application/json" } body = { "model": "mistralai/mistral-7b-instruct", "messages": [{"role": "user", "content": prompt}] } r = requests.post("https://openrouter.ai/api/v1/chat/completions", headers=headers, json=body) return r.json()['choices'][0]['message']['content'] except Exception as e: return f"⚠️ AI error: {e}"

--- Run bot ---

if name == "main": logging.info("Midnight Kael Bot is running...") bot.infinity_polling()

