# inform-message-1
Прошу обратить внимание на функцию `once_done`, там где комментарий это то, что выдает ошибку. Wit это то, с помощью чего оно должно рабоать. Полезные ссылки:

https://wit.ai/docs/ - обработка аудио

https://github.com/wit-ai/pywit - обработка аудио

https://guide.pycord.dev/voice/receiving/#starting-out - для записи файла аудио

https://github.com/Pycord-Development/pycord/blob/master/examples/audio_recording.py - для записи файла аудио

https://docs.pycord.dev/en/stable/api.html - документация к дискорд ботам на основе pycord

```from enum import Enum
import discord
from discord.ext import commands
import asyncio
import pathlib
import ffmpeg
import base64
import requests
from wit import Wit
client = Wit("my_wit_token")
token='my_token'
bot = commands.Bot(command_prefix = ["."], intents = discord.Intents.all(), help_command = None)
connections = {}

@bot.event
async def on_ready():
    print("Bot - Online..")

@bot.command()
async def start(ctx): 
    if ctx.message.author.voice:
        voice = ctx.author.voice
        channel = ctx.author.voice.channel
        voice1 = ctx.message.guild.voice_client
        if not voice1:
            vc = await voice.channel.connect()  
            print("Старт в канале '" + channel.name + "'")
            connections.update({ctx.guild.id: vc})
            vc.start_recording(
                discord.sinks.WaveSink(), 
                once_done,  
                ctx.channel 
            )
            print("Запись начата в канале '" + channel.name + "'")
        else:
            if channel.id != voice1.channel.id:
                name_from = voice1.channel.name
                if ctx.guild.id in connections:
                    vc = connections[ctx.guild.id]
                    vc.stop_recording() 
                    del connections[ctx.guild.id] 
                    print("Стоп в канале '" + name_from + "'")
                    print("Стоп записи в канале '" + name_from + "'")
                    await asyncio.sleep(2)
                    vc = await voice.channel.connect()  
                    print("Перемещение из канала '" + name_from + "' в канал '" + channel.name + "'")
                    connections.update({ctx.guild.id: vc})  
                    vc.start_recording(
                        discord.sinks.WaveSink(),  
                        once_done,  
                        ctx.channel 
                    )
                    print("Запись начата в канале '" + channel.name + "'")
                else:
                    print("Ошибка")
                #print("Начало записи в канале '" + channel.name + "'")
            else:
                print("Бот уже в этом канале - '" + channel.name + "'")
    else:
        print("Автора команды нет в голосовом канале")

async def once_done(sink: discord.sinks, channel: discord.TextChannel, *args):
    recorded_users = [
        f"<@{user_id}>"
        for user_id, audio in sink.audio_data.items()
    ]
    text = "Сними коментарий через 2 строчки вниз"
    await sink.vc.disconnect() 
    files = [discord.File(audio.file, f"{user_id}.{sink.encoding}") for user_id, audio in sink.audio_data.items()]
    #text = client.speech(files, {'Content-Type': 'audio/wav'})
    await channel.send(text, files=files)  

@bot.command()
async def stop(ctx):
    channel = ctx.author.voice.channel
    voice = ctx.message.guild.voice_client
    if voice:
        if ctx.author.voice:
            if channel.id != voice.channel.id:
                print("Бот находится в другом канале - '" + voice.channel.name + "'")
            else:
                if ctx.guild.id in connections:
                    vc = connections[ctx.guild.id]
                    vc.stop_recording()  
                    del connections[ctx.guild.id]  
                    print("Стоп в канале '" + channel.name + "'")
                    print("Стоп записи в канале '" + channel.name + "'")
                else:
                    print("Ошибка")
        else:
            print("Автор сообщения не находится в голосовом канале")
    else:
        print("Бот не играет")

@bot.command()
async def info(ctx):
    i = 0
    if ctx.message.guild.voice_client != None:
        for i in range(len(ctx.message.guild.voice_client.channel.members)):
            print(str(ctx.message.guild.voice_client.channel.members[i]) + " - " + str(ctx.message.guild.voice_client.channel.members[i].voice + "\n================================"))
            i += 1
    else:
        print("Бот не играет")

bot.run(token)
```



