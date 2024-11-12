from pyrogram import Client, filters
import requests

# Bot tokeningiz va API ID/Hash uchun sozlamalar
bot = Client("my_bot", bot_token="7869475442:AAFoNzuCNpUpSnJaBDkNzeE-TgYiCP-4Pok")

@bot.on_message(filters.command("start"))
async def start(bot, message):
    await message.reply("Salom! Men sizga TikTok, Instagram va YouTube'dan video yuklab olishda yordam bera olaman. Foydalanishdan oldin @music_hall_carrozzeria kanaliga obuna bo'lishingiz kerak.")

@bot.on_message(filters.text & filters.private)
async def download_video(bot, message):
    # Obuna tekshiruvi
    try:
        user = await bot.get_chat_member("music_hall_carrozzeria", message.from_user.id)
        if user.status not in ["member", "administrator", "creator"]:
            await message.reply("Botdan foydalanish uchun @music_hall_carrozzeria kanaliga obuna bo'ling.")
            return
    except:
        await message.reply("Obuna tekshiruvida xatolik yuz berdi. Iltimos, bot administratoriga murojaat qiling.")
        return

    url = message.text
    video_url = None

    if "tiktok.com" in url:
        # TikTok uchun yuklab olish
        api_url = f"https://tikmate.online/api/download?url={url}"
        response = requests.get(api_url)
        video_url = response.json().get("download_addr")

    elif "instagram.com" in url:
        # Instagram uchun yuklab olish
        api_url = f"https://igram.io/api/v1?url={url}"
        response = requests.get(api_url)
        video_url = response.json().get("video_url")

    elif "youtube.com" in url or "youtu.be" in url:
        # YouTube uchun yuklab olish
        from pytube import YouTube
        yt = YouTube(url)
        stream = yt.streams.get_highest_resolution()
        video_url = stream.url

    else:
        await message.reply("Kechirasiz, bu URL ni qo'llab-quvvatlamayman.")
        return

    # Video faylini Telegramga yuborish
    if video_url:
        await bot.send_video(message.chat.id, video=video_url)
    else:
        await message.reply("Video yuklab olishda xatolik yuz berdi.")

bot.run() 
