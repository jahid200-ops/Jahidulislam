import telebot import time from indicators import check_trade_signal from utils import fetch_data, get_status_msg import os

API_KEY = os.getenv("TELEGRAM_API_KEY", "8136437526:AAFSB--egH92xenrd-h__xUxzd6meKP5mUk") CHAT_ID = int(os.getenv("TELEGRAM_CHAT_ID", 7872392779))

bot = telebot.TeleBot(API_KEY)

running = False market = "EURUSD" timeframe = "1m" wins = 0 losses = 0 send_no_signal = True

@bot.message_handler(commands=['start']) def start_bot(message): global running running = True bot.send_message(message.chat.id, "✅ Bot started.")

@bot.message_handler(commands=['stop']) def stop_bot(message): global running running = False bot.send_message(message.chat.id, "⛔ Bot stopped.")

@bot.message_handler(commands=['market']) def set_market(message): global market msg = message.text.split() if len(msg) > 1 and msg[1] in ['EURUSD', 'EURGBP', 'EURJPY']: market = msg[1] bot.send_message(message.chat.id, f"✅ Market set to {market}") else: bot.send_message(message.chat.id, "❌ Usage: /market EURUSD")

@bot.message_handler(commands=['timeframe']) def set_timeframe(message): global timeframe msg = message.text.split() if len(msg) > 1 and msg[1] in ['1m', '5m']: timeframe = msg[1] bot.send_message(message.chat.id, f"✅ Timeframe set to {timeframe}") else: bot.send_message(message.chat.id, "❌ Usage: /timeframe 1m")

@bot.message_handler(commands=['status']) def status(message): bot.send_message(message.chat.id, get_status_msg(running, market, timeframe, wins, losses))

@bot.message_handler(commands=['win']) def win(message): global wins wins += 1 bot.send_message(message.chat.id, f"👍 Win recorded! ({wins}W/{losses}L)")

@bot.message_handler(commands=['loss']) def loss(message): global losses losses += 1 bot.send_message(message.chat.id, f"👎 Loss recorded! ({wins}W/{losses}L)")

@bot.message_handler(commands=['nosignal']) def toggle_no_signal(message): global send_no_signal msg = message.text.split() if len(msg) > 1 and msg[1] in ['on', 'off']: send_no_signal = msg[1] == 'on' status = "enabled" if send_no_signal else "disabled" bot.send_message(message.chat.id, f"ℹ️ No-signal message {status}.") else: bot.send_message(message.chat.id, "❌ Usage: /nosignal on OR /nosignal off")

def run_bot(): global running, send_no_signal while True: if running: candles = fetch_data(market, timeframe) signal = check_trade_signal(candles) if signal: bot.send_message(CHAT_ID, f"📊 Signal: {signal}\n📈 Market: {market}\n🕒 Timeframe: {timeframe}") elif send_no_signal: bot.send_message(CHAT_ID, f"❌ No signal at the moment\n📈 Market: {market}\n🕒 Timeframe: {timeframe}") time.sleep(60)

import threading threading.Thread(target=run_bot).start() bot.polling()


