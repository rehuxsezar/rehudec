import telebot
import marshal, base64, zlib

# Replace with your bot token
BOT_TOKEN = "7502592198:AAHGB7TxO367eU30hxgm2edQu_CTIMPH67M"
bot = telebot.TeleBot(BOT_TOKEN)

# Start command
@bot.message_handler(commands=["start"])
def start_command(message):
    bot.reply_to(message, "Welcome to Encryption Bot!\nSend me a Python file (.py) to encrypt.")

# Handle file uploads
@bot.message_handler(content_types=["document"])
def handle_document(message):
    try:
        # Get file info
        file_id = message.document.file_id
        file_info = bot.get_file(file_id)
        downloaded_file = bot.download_file(file_info.file_path)
        file_name = message.document.file_name

        # Check if it's a Python file
        if not file_name.endswith(".py"):
            bot.reply_to(message, "Please send a .py file!")
            return

        # Encrypt with marshal, base64, and zlib
        encrypted_code = marshal.dumps(compile(downloaded_file.decode(), '<string>', 'exec'))
        compressed_code = zlib.compress(encrypted_code)
        final_code = base64.b64encode(compressed_code).decode()

        encrypted_script = f"import marshal, base64, zlib\nexec(marshal.loads(zlib.decompress(base64.b64decode('{final_code}'))))"

        # Send the encrypted file back
        with open("encrypted_" + file_name, "w") as encrypted_file:
            encrypted_file.write(encrypted_script)

        with open("encrypted_" + file_name, "rb") as encrypted_file:
            bot.send_document(message.chat.id, encrypted_file)

        bot.reply_to(message, "✅ Encryption successful!")

    except Exception as e:
        bot.reply_to(message, f"❌ Encryption failed: {e}")

# Error handler
@bot.message_handler(func=lambda message: True)
def catch_all(message):
    bot.reply_to(message, "I don't understand that command. Send me a Python file instead!")

print("Bot is running...")
bot.polling()
