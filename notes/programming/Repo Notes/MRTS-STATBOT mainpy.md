Source: https://github.com/Arnav7501/MRTS-STATBOT/blob/main/main.py

```py
# This example requires the 'message_content' intent.
from discord.ext import tasks
import discord 
from discord import Embed #discord pretty embed feature
import requests
from bs4 import BeautifulSoup #HTML webscraper library
import patchnotes  #imports pathnotes.py ??
import re
import os
from dotenv import load_dotenv #Gets environment settings from a .env file

load_dotenv() #loads env settings 
APIKEY = os.getenv('APIKEY') #discord api key?
intents = discord.Intents.default()
intents.message_content = True
client = discord.Client(intents=intents)


def scrape_wiki(unitName):
    try:
        url = f"https://mrts.fandom.com/wiki/{unitName}"

    # Send a GET request to fetch the HTML content
        response = requests.get(url)

# Create a BeautifulSoup object to parse the HTML
        soup = BeautifulSoup(response.content, "html.parser")

# Extract the desired information


# Print the extracted information

        # finds the div with the class "mw-parser-output"
        td_tag = soup.find("div", class_="mw-parser-output")

# Extract the text content of the td tag
        # removes the leading and trailing elements
        content = td_tag.text.strip() if td_tag else None
        #formats the array idk :shrug:
        array = [line for line in content.split("\n") if line.strip()]

        images = soup.find_all('img') #finds all image tags
        image = images[1].get('src') #obtains image srcs from image tabs 
        array.append(image) #places image in available space idk 
        return array

    except Exception as e:
        return 1


async def statScraper(unitName, message):
    array = scrape_wiki(unitName)
    image = array[-1] #gets image from array, witch itself is form web scraping
    embed = Embed() #creates a discord embed
    embed.title = array[0] #makes the title of the embed 

    if array[13] == 'S-Range':
        embed.add_field(name=array[2], value=array[5])
        embed.add_field(name=array[3], value=array[6])
        embed.add_field(name=array[4], value=array[7])
        embed.add_field(name=array[8], value=array[10])
        embed.add_field(name=array[9], value=array[11])
        embed.add_field(name=array[12], value=array[15])
        embed.add_field(name=array[13], value=array[16])
        embed.add_field(name=array[14], value=array[17])
        embed.add_field(name=array[18], value=array[20])
        embed.add_field(name=array[19], value=array[21])
        embed.add_field(name=array[22], value=array[23])

    else:
        embed.add_field(name=array[2], value=array[5])
        embed.add_field(name=array[3], value=array[6])
        embed.add_field(name=array[4], value=array[7])
        embed.add_field(name=array[8], value=array[10])
        embed.add_field(name=array[9], value=array[11])
        embed.add_field(name=array[12], value=array[14])
        embed.add_field(name=array[13], value=array[15])
        embed.add_field(name=array[16], value=array[18])
        embed.add_field(name=array[17], value=array[19])
        embed.add_field(name=array[20], value=array[21])

    embed.set_image(url=image) #sets image of embed to the scraped image
    await message.channel.send(embed=embed) #sends the message to discord 

   # await message.channel.send(f"ERROR, {e}")

#not sure about this one, client login ???
#probably for bot management
@client.event
async def on_ready():
    print(f'We have logged in as {client.user}')
    check_patchnotes.start()

old = "https://pastebin.com/raw/xchHf3Gp"
old = patchnotes.TableToDict(old) #turns pastebin (raw file) to dictionary


@tasks.loop(hours=1)  # Set the interval to 1 hour
async def check_patchnotes():
    global old
    variable = patchnotes.PrintChanges(
        old, patchnotes.TableToDict("https://pastebin.com/raw/xchHf3Gp"))
    if len(variable) > 0:
        channel = client.get_channel(1109558632292556903)
        await channel.send(variable)
        old = patchnotes.TableToDict("https://pastebin.com/raw/xchHf3Gp")
        await channel.send("DMNKS has changed some stats <@763582841866682368> <@246824672367345674> <@621516858205405197>")


@client.event
async def on_message(message):
    if message.author == client.user: #anti-selfbot mechanism??
        return

    if message.content.startswith(';troop'): #command 
        content = message.content[6:]
        await statScraper(content, message) #lists out the stats of the unit using statScraper()

    if message.content.startswith(';rankdps'): #command
        singleTargetArray = ["Archer", "Swordman",
                             "Knight", "Longbower", "Crossbower", "Ballista"]
        splashArray = ["Mage", "Wizard", "Catapult"]
        singleTargetDictionary = {}
        splashTargetDictionary = {}

        #runs through all units in the two arrays defined above
        #single target unit ranking
        for i in range(len(singleTargetArray)):
            tempValue = scrape_wiki(singleTargetArray[i]) #scrapes the units
            singleTargetDictionary[tempValue[0]] = tempValue[21] #gets single target stats 
        sorted_dict = dict(
            sorted(singleTargetDictionary.items(), key=lambda x: x[1])) #sorts the units in singletargetdict
        print("SORTED_DICT", sorted_dict)
        embed = Embed()
        embed.title = "Single Target"

        for key in sorted_dict:
            embed.add_field(name=key, value=sorted_dict[key], inline=False)
        await message.channel.send(embed=embed)

        #splash damage unit ranking
        for i in range(len(splashArray)):
            tempValue = scrape_wiki(splashArray[i]) #scrapes units 
            splashTargetDictionary[tempValue[0]] = tempValue[23] #gets unit stats ?
        sorted_dict = dict(
            sorted(splashTargetDictionary.items(), key=lambda x: x[1])) #sorts unit dicts
        print("SORTED_DICT", sorted_dict)
        embed = Embed() 
        embed.title = "Splash Target"
        for key in sorted_dict:
            embed.add_field(name=key, value=sorted_dict[key], inline=False)
        await message.channel.send(embed=embed)

    if message.content.startswith(';patchnotes'): #patchnotes 
        if len(message.content) == 11:
            await message.channel.send(patchnotes.PrintChanges(patchnotes.TableToDict(old), patchnotes.TableToDict("https://pastebin.com/raw/xchHf3Gp"))) 
            #prints changes in pastebin? 
        else: 
            #broken pastebin state / failstate
            content = message.content[12:]
            pattern = r'^https?://(?:www\.)?pastebin\.com/raw/[a-zA-Z0-9]+$'
            if (re.match(pattern, content) is None):
                await message.channel.send("i get the feeling that's not a patebin link")
            else:
                await message.channel.send(patchnotes.PrintChanges(patchnotes.TableToDict(content), patchnotes.TableToDict("https://pastebin.com/raw/xchHf3Gp")))
                #dmnks UID? 
        if message.author.id == 263351384466784257:
            await message.channel.send("you know what would be cool? if you did this alread for us")


client.run(
    APIKEY)```