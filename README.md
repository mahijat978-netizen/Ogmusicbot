# Ogmusicbot
Music bot codes
import asyncio from pyrogram import Client, filters from pyrogram.types import Message from pytgcalls import PyTgCalls, idle from pytgcalls.types.input_stream import InputAudioStream, InputStream import yt_dlp

====== CONFIG ======

API_ID = 1234567          # your api id API_HASH = "your_api_hash" BOT_TOKEN = "your_bot_token" SESSION = "your_session_string"  # string session for userbot (for VC join)

====== CLIENTS ======

bot = Client("music_bot", api_id=API_ID, api_hash=API_HASH, bot_token=BOT_TOKEN) user = Client(SESSION, api_id=API_ID, api_hash=API_HASH) call_py = PyTgCalls(user)

AFK system

afk_users = {}

Music queue

queues = {}

====== MUSIC UTILS ======

def yt_audio(url): ydl_opts = { 'format': 'bestaudio/best', 'quiet': True, 'nocheckcertificate': True, } with yt_dlp.YoutubeDL(ydl_opts) as ydl: info = ydl.extract_info(url, download=False) return info['url'], info.get("title", "Unknown")

====== AFK SYSTEM ======

@bot.on_message(filters.command("afk") & filters.group) async def afk_set(_, m: Message): afk_users[m.from_user.id] = "AFK" await m.reply(f"{m.from_user.first_name} is now AFK ✨")

@bot.on_message(filters.command("back") & filters.group) async def afk_back(_, m: Message): if m.from_user.id in afk_users: afk_users.pop(m.from_user.id) await m.reply(f"{m.from_user.first_name} is back ✅")

@bot.on_message(filters.text & filters.group) async def afk_check(_, m: Message): if m.reply_to_message and m.reply_to_message.from_user: uid = m.reply_to_message.from_user.id if uid in afk_users: await m.reply(f"{m.reply_to_message.from_user.first_name} is currently AFK 🚫")

===== MUSIC COMMANDS =====

@bot.on_message(filters.command("play") & filters.group) async def play_music(_, m: Message): if len(m.command) < 2: return await m.reply("Usage: /play <YouTube link>")

link = m.command[1]
audio_url, title = yt_audio(link)
chat_id = m.chat.id

if chat_id not in queues:
    queues[chat_id] = []

queues[chat_id].append((audio_url, title))

if len(queues[chat_id]) == 1:
    await call_py.join_group_call(
        chat_id,
        InputStream(InputAudioStream(audio_url)),
        stream_type="local_stream",
    )
    await m.reply(f"▶️ Playing **{title}**")
else:
    await m.reply(f"⏳ Added to queue: **{title}**")

@bot.on_message(filters.command("skip") & filters.group) async def skip_music(_, m: Message): chat_id = m.chat.id if chat_id in queues and len(queues[chat_id]) > 1: queues[chat_id].pop(0) next_url, title = queues[chat_id][0] await call_py.change_stream( chat_id, InputStream(InputAudioStream(next_url)) ) await m.reply(f"⏭️ Skipped! Now playing: {title}") else: await call_py.leave_group_call(chat_id) queues.pop(chat_id, None) await m.reply("❌ No more songs in queue.")

@bot.on_message(filters.command("stop") & filters.group) async def stop_music(_, m: Message): chat_id = m.chat.id await call_py.leave_group_call(chat_id) queues.pop(chat_id, None) await m.reply("⏹️ Stopped music")

===== ADMIN COMMANDS =====

@bot.on_message(filters.command("ban") & filters.group) async def ban_user(_, m: Message): if not m.reply_to_message: return await m.reply("Reply to a user to ban") await m.chat.ban_member(m.reply_to_message.from_user.id) await m.reply(f"🚫 Banned {m.reply_to_message.from_user.first_name}")

@bot.on_message(filters.command("mute") & filters.group) async def mute_user(_, m: Message): if not m.reply_to_message: return await m.reply("Reply to a user to mute") await m.chat.restrict_member(m.reply_to_message.from_user.id, permissions=[]) await m.reply(f"🔇 Muted {m.reply_to_message.from_user.first_name}")

@bot.on_message(filters.command("unmute") & filters.group) async def unmute_user(_, m: Message): if not m.reply_to_message: return await m.reply("Reply to a user to unmute") await m.chat.unban_member(m.reply_to_message.from_user.id) await m.reply(f"🔊 Unmuted {m.reply_to_message.from_user.first_name}")

@bot.on_message(filters.command("purge") & filters.group) async def purge(_, m: Message): if not m.reply_to_message: return await m.reply("Reply to a message to start purging") chat_id = m.chat.id msg_id = m.reply_to_message.id del_msg = [i for i in range(msg_id, m.id + 1)] await bot.delete_messages(chat_id, del_msg) await m.reply("🗑️ Purge complete")

===== UTILS =====

@bot.on_message(filters.command("ping")) async def ping(_, m: Message): await m.reply("🏓 Pong!")

@bot.on_message(filters.command("alive")) async def alive(_, m: Message): await m.reply("✅ Bot is alive and running!")

@bot.on_message(filters.command("help")) async def help_menu(_, m: Message): help_text = """ 🎶 Advanced Telegram Music Bot

🎵 Music: /play <yt link> - Play music /skip - Skip song /stop - Stop music

👤 AFK: /afk - Set AFK /back - Remove AFK

⚒️ Admin: /ban, /mute, /unmute, /purge

⚡ Utils: /ping, /alive, /help """ await m.reply(help_text)

===== STARTUP =====

async def main(): await bot.start() await user.start() await call_py.start() print("Bot is running...") await idle() await bot.stop() await user.stop()

if name == "main": asyncio.run(main())

