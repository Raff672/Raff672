import os
import discord
from discord.ext import commands
from discord.utils import get
from dotenv import load_dotenv

# Charger les variables d'environnement depuis un fichier .env
load_dotenv()
TOKEN = os.getenv('DISCORD_TOKEN')

# Initialisation du bot
intents = discord.Intents.default()
intents.members = True  # Nécessaire pour accéder aux membres du serveur
bot = commands.Bot(command_prefix="!", intents=intents)

# Quand le bot est prêt
@bot.event
async def on_ready():
    print(f'Bot connecté en tant que {bot.user}')

# Commande pour kick un membre
@bot.command()
@commands.has_permissions(kick_members=True)
async def kick(ctx, member: discord.Member, *, reason=None):
    await member.kick(reason=reason)
    await ctx.send(f'{member.mention} a été expulsé pour: {reason}')

# Commande pour ban un membre
@bot.command()
@commands.has_permissions(ban_members=True)
async def ban(ctx, member: discord.Member, *, reason=None):
    await member.ban(reason=reason)
    await ctx.send(f'{member.mention} a été banni pour: {reason}')

# Commande pour mute un membre
@bot.command()
@commands.has_permissions(manage_roles=True)
async def mute(ctx, member: discord.Member, *, reason=None):
    mute_role = get(ctx.guild.roles, name="Muted")
    if not mute_role:
        # Créer un rôle Muted s'il n'existe pas
        mute_role = await ctx.guild.create_role(name="Muted")
        for channel in ctx.guild.channels:
            await channel.set_permissions(mute_role, speak=False, send_messages=False)

    await member.add_roles(mute_role)
    await ctx.send(f'{member.mention} a été mute pour: {reason}')

# Commande pour unmute un membre
@bot.command()
@commands.has_permissions(manage_roles=True)
async def unmute(ctx, member: discord.Member):
    mute_role = get(ctx.guild.roles, name="Muted")
    await member.remove_roles(mute_role)
    await ctx.send(f'{member.mention} a été unmute.')

# Commande pour envoyer un DM
@bot.command()
async def dm(ctx, member: discord.Member, *, message):
    await member.send(message)
    await ctx.send(f'DM envoyé à {member.mention}')

# Commande pour ouvrir un ticket
@bot.command()
async def ticket(ctx):
    guild = ctx.guild
    category = get(guild.categories, name="Tickets")
    
    if not category:
        # Créer la catégorie si elle n'existe pas
        category = await guild.create_category("Tickets")
    
    channel = await guild.create_text_channel(f"ticket-{ctx.author.name}", category=category)
    await channel.set_permissions(ctx.author, read_messages=True, send_messages=True)
    
    await channel.send(f'{ctx.author.mention}, voici ton ticket. Un modérateur te répondra sous peu.')

# Gérer les erreurs de permission
@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.MissingPermissions):
        await ctx.send(f'{ctx.author.mention}, tu n\'as pas les permissions nécessaires pour utiliser cette commande.')

# Démarrage du bot
DISCORD_TOKEN=MTIwODA4ODc3MTYyMzUyMjM5NQ.GX-SRJ.o5PAreLW9IxR2jisIfQe9ZK2XFHTWGvhw3cJ0g
