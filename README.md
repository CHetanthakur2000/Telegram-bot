import os
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
import yt_dlp

# ----------------- YouTube Downloader -----------------
def download_youtube(url):
    ydl_opts = {
        "outtmpl": "downloads/%(title)s.%(ext)s",
        "format": "best"
    }
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(url, download=True)
        return ydl.prepare_filename(info)   # Local file path return karega

# ----------------- Instagram Downloader -----------------
def download_instagram(url):
    # ⚠️ Instagram ke liye bhi yt-dlp kaam karta hai
    ydl_opts = {
        "outtmpl": "downloads/%(title)s.%(ext)s",
        "format": "best"
    }
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(url, download=True)
        return ydl.prepare_filename(info)

# ----------------- Handle Messages -----------------
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    url = update.message.text
    try:
        if "instagram.com" in url:
            file_path = download_instagram(url)
            await update.message.reply_video(video=open(file_path, "rb"))
            os.remove(file_path)  # optional: delete after sending

        elif "youtube.com" in url or "youtu.be" in url:
            file_path = download_youtube(url)
            await update.message.reply_video(video=open(file_path, "rb"))
            os.remove(file_path)  # optional: delete after sending

        else:
            await update.message.reply_text("❌ Please send a valid Instagram or YouTube link.")

    except Exception as e:
        await update.message.reply_text(f"⚠️ Error: {e}")

# ----------------- Start Command -----------------
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("👋 Send me any Instagram or YouTube link, I'll download it for you!")

# ----------------- Main -----------------
def main():
    app = Application.builder().token("YOUR_BOT_TOKEN").build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    app.run_polling()

if __name__ == "__main__":
    if not os.path.exists("downloads"):
        os.makedirs("downloads")
    main()
