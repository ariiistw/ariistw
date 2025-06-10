import os
import requests
import sys
import time
import platform
import json
from datetime import datetime # <- Tambahkan import ini
from urllib.parse import quote
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
from telegram.constants import ChatAction

# Nama file untuk menyimpan konfigurasi (token dan nama bot)
CONFIG_FILE = "config_ai_bot.json"

# ANSI escape sequences for colors and styles (tetap sama)
class Colors:
    HEADER = '\033[95m'
    OKBLUE = '\033[94m'
    OKCYAN = '\033[96m'
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    BOLD = '\033[1m'
    UNDERLINE = '\033[4m'
    RESET = '\033[0m'
    # Warna baru untuk logging
    LOG_USER = '\033[93m'  # Kuning untuk pesan user
    LOG_BOT = '\033[96m'   # Cyan untuk balasan bot

def clear_console():
    os.system('cls' if platform.system() == 'Windows' else 'clear')

def slow_print(text, delay=0.03):
    for c in text:
        sys.stdout.write(c)
        sys.stdout.flush()
        time.sleep(delay)
    print()

# --- Fungsi-fungsi setup (get_bot_info, set_bot_description, dll) tetap sama ---
# (Saya singkat di sini agar tidak terlalu panjang, tidak ada perubahan di bagian ini)

def get_bot_info(token):
    url = f"https://api.telegram.org/bot{token}/getMe"
    try:
        response = requests.get(url, timeout=10)
        if response.status_code == 200:
            result = response.json()
            if result.get("ok"):
                return result.get("result")
        return None
    except requests.RequestException:
        return None

def set_bot_description(token, bot_name):
    description = f"🤖 Halo! Saya {bot_name}, asisten AI pribadi Anda. Kirimkan saya pesan apa saja, dan saya akan merespons dengan cerdas!"
    url = f"https://api.telegram.org/bot{token}/setMyDescription"
    data = {"description": description}
    try:
        response = requests.post(url, json=data, timeout=10)
        return response.json().get("ok", False)
    except requests.RequestException:
        return False

def prompt_token():
    return input(f"{Colors.BOLD}{Colors.OKCYAN}Masukkan API Bot Token Telegram Anda: {Colors.RESET}").strip()

def prompt_bot_name():
    return input(f"{Colors.BOLD}{Colors.OKCYAN}Masukkan Nama untuk Asisten AI Anda (contoh: Jarvis): {Colors.RESET}").strip()

def print_banner():
    banner = f"""
{Colors.OKGREEN}{Colors.BOLD}
   ___   _   _    _    ___ _____   _   ___ _   _ 
  / _ \\ / | / |  / \\  |_ _|_   _| / | / _ \\ | | |
 | | | || | | | / _ \\  | |  | |   | || | | | | | |
 | |_| || | | |/ ___ \\ | |  | |   | || |_| | |_| |
  \\___/ |_| |_/_/   \\_\\|___| |_|   |_| \\___/ \\___/ 
                                                  
{Colors.RESET}
    """
    print(banner)
    slow_print(f"{Colors.OKBLUE}Welcome to the Telegram AI Assistant Bot Setup!{Colors.RESET}")
    print("----------------------------------------------------")

def setup_bot():
    clear_console()
    print_banner()
    token = None
    bot_name = None

    if os.path.exists(CONFIG_FILE):
        print(f"{Colors.WARNING}Mendeteksi file konfigurasi ({CONFIG_FILE}), mencoba validasi...{Colors.RESET}")
        with open(CONFIG_FILE, "r") as f:
            config = json.load(f)
            token = config.get("token")
            bot_name = config.get("bot_name")

        if token and bot_name and get_bot_info(token):
            bot_info = get_bot_info(token)
            print(f"{Colors.OKGREEN}Token valid! Bot Anda: {Colors.BOLD}@{bot_info.get('username')}{Colors.RESET}")
            print(f"{Colors.OKGREEN}Nama Asisten AI: {Colors.BOLD}{bot_name}{Colors.RESET}")
            print(f"{Colors.OKBLUE}Setup selesai. Anda bisa memakai bot ini sekarang!{Colors.RESET}")
            return token, bot_name
        else:
            print(f"{Colors.FAIL}Konfigurasi tidak valid atau token sudah kadaluwarsa.{Colors.RESET}")
            print(f"{Colors.WARNING}Silakan masukkan detail baru.{Colors.RESET}")

    while True:
        token = prompt_token()
        print(f"{Colors.OKBLUE}Memvalidasi token... mohon tunggu.{Colors.RESET}")
        if get_bot_info(token):
            print(f"{Colors.OKGREEN}Token valid!{Colors.RESET}")
            break
        else:
            print(f"{Colors.FAIL}Token tidak valid. Silakan coba lagi.{Colors.RESET}")
    
    bot_name = prompt_bot_name()

    config_data = {"token": token, "bot_name": bot_name}
    with open(CONFIG_FILE, "w") as f:
        json.dump(config_data, f, indent=4)
    print(f"{Colors.OKGREEN}Konfigurasi berhasil disimpan di {CONFIG_FILE}{Colors.RESET}")

    print(f"{Colors.OKBLUE}Mengatur deskripsi bot...{Colors.RESET}")
    if set_bot_description(token, bot_name):
        print(f"{Colors.OKGREEN}Deskripsi bot berhasil diatur!{Colors.RESET}")
    else:
        print(f"{Colors.FAIL}Gagal mengatur deskripsi bot.{Colors.RESET}")

    return token, bot_name


# --- End of Setup Functions ---

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    bot_name = context.bot_data.get('bot_name', 'Asisten AI')
    await update.message.reply_text(f"Halo! Saya {bot_name}. Kirimkan saya pesan apa saja, dan saya akan membalasnya!")
    # Log /start command
    user = update.effective_user
    print(f"{Colors.LOG_USER}[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}] User {user.full_name} (@{user.username}) started the bot.{Colors.RESET}")


async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_message = update.message.text
    user = update.effective_user
    bot_name = context.bot_data.get('bot_name', 'jagoanuserbot')
    
    # =================================================================
    # BAGIAN BARU: Mencetak log pesan masuk ke console
    # =================================================================
    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    log_message = f"[{timestamp}] Pesan dari {user.full_name} (@{user.username}): \"{user_message}\""
    print(f"{Colors.LOG_USER}{log_message}{Colors.RESET}")
    # =================================================================

    await context.bot.send_chat_action(chat_id=update.effective_chat.id, action=ChatAction.TYPING)

    try:
        api_url = "https://api.agatz.xyz/api/gptlogic"
        logic_prompt = f"Generate humanized chatgpt text in Indonesian, you are an AI assistant named {bot_name}"
        params = {"logic": logic_prompt, "p": user_message}
        
        response = requests.get(api_url, params=params, timeout=30)
        response.raise_for_status()

        data = response.json()

        if data.get("status") == 200 and data.get("data", {}).get("result"):
            reply_text = data["data"]["result"]
            await update.message.reply_text(reply_text)
            
            # =================================================================
            # BAGIAN BARU: Mencetak log balasan bot ke console
            # =================================================================
            reply_timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            log_reply = f"[{reply_timestamp}] Balasan Bot '{bot_name}' kepada {user.full_name}: \"{reply_text[:80]}...\"" # Dibatasi 80 karakter agar rapi
            print(f"{Colors.LOG_BOT}{log_reply}{Colors.RESET}")
            # =================================================================

        else:
            error_message = data.get("message", "Tidak ada hasil yang valid.")
            await update.message.reply_text(f"❌ Maaf, terjadi kesalahan dari API: {error_message}")
            print(f"{Colors.FAIL}[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}] API Error: {error_message}{Colors.RESET}")

    except requests.exceptions.RequestException as e:
        await update.message.reply_text(f"❌ Maaf, saya tidak bisa menghubungi server AI saat ini. Coba lagi nanti.")
        print(f"{Colors.FAIL}[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}] Network Error: {e}{Colors.RESET}")
    except Exception as e:
        await update.message.reply_text(f"❌ Terjadi kesalahan yang tidak terduga.")
        print(f"{Colors.FAIL}[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}] Unexpected Error: {e}{Colors.RESET}")


def run_bot(token, bot_name):
    app = Application.builder().token(token).build()
    app.bot_data['bot_name'] = bot_name

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    
    print(f"\n{Colors.OKBLUE}{Colors.BOLD}Bot AI '{bot_name}' sedang berjalan... Log aktivitas akan muncul di bawah ini.{Colors.RESET}")
    app.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == "__main__":
    token, bot_name = setup_bot()
    if token and bot_name:
        run_bot(token, bot_name)
