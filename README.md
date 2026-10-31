# 📘 The Ultimate `discord.py` Encyclopedia & Cheat Sheet

> **A complete, beginner‑friendly reference for the latest stable version of `discord.py` (v2.7.x)**  
> *Covering the entire public API, every event, embed style, button style, status type, and feature — word by word, in plain English.*

---

## 📑 Table of Contents

1.  [Introduction & Installation](#1-introduction--installation)
2.  [Core Concepts – Intents, Events, Client vs Bot](#2-core-concepts--intents-events-client-vs-bot)
3.  [The Client & Bot Objects](#3-the-client--bot-objects)
4.  [Data Models – Guild, Member, Channel, Message](#4-data-models--guild-member-channel-message)
5.  [Embeds – Every Field, Style, and Limit](#5-embeds--every-field-style-and-limit)
6.  [Buttons, Selects, Views & Modals (UI Components)](#6-buttons-selects-views--modals-ui-components)
7.  [Application Commands (Slash Commands)](#7-application-commands-slash-commands)
8.  [Events – The Complete Reference](#8-events--the-complete-reference)
9.  [Presence & Activity – Every Status Type](#9-presence--activity--every-status-type)
10. [Permissions, Roles & Audit Logs](#10-permissions-roles--audit-logs)
11. [Webhooks & Voice](#11-webhooks--voice)
12. [Utility Functions & Error Handling](#12-utility-functions--error-handling)
13. [Cogs & Extensions](#13-cogs--extensions)
14. [Quick‑Reference Cheat Sheet](#14-quick-reference-cheat-sheet)

---

## 1. Introduction & Installation

`discord.py` is a **modern, async‑ready API wrapper** for Discord written in Python.  
The **current stable version is 2.7.1** – the old v1.x line is deprecated and should not be used for new projects.

### 📦 Installation

```bash
# Standard installation
pip install discord.py

# With voice support
pip install discord.py[voice]
```

> **Requirement:** Python **3.8 or higher**.

### 🚀 Minimal Bot (Commands Extension)

```python
import discord
from discord.ext import commands
import os

intents = discord.Intents.default()
intents.message_content = True          # privileged – enable in Developer Portal
intents.members = True                  # privileged – enable in Developer Portal

bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    print(f"{bot.user} has connected to Discord!")

@bot.command()
async def ping(ctx):
    await ctx.send("Pong!")

bot.run(os.getenv("DISCORD_TOKEN"))
```

> **Golden rule:** Always pass `intents` explicitly. `discord.Client()` without intents is invalid in v2.0+.

---

## 2. Core Concepts – Intents, Events, Client vs Bot

### 2.1 Intents

**Intents** tell Discord which gateway events your bot wants to receive.  
Since v2.0, **intents are mandatory**.

| Category | Intent | Notes |
|----------|--------|-------|
| **Standard** | `guilds`, `messages`, `reactions`, `typing`, `voice_states`, etc. | No portal action needed |
| **Privileged** | `message_content`, `members`, `presences` | **Must be enabled in the Discord Developer Portal** |

```python
# Enable all standard intents + privileged ones you need
intents = discord.Intents.default()
intents.message_content = True
intents.members = True
intents.presences = True

# Or, for development only:
# intents = discord.Intents.all()
```

**Common intent flags:** `guilds`, `members`, `moderation`, `emojis`, `integrations`, `webhooks`, `invites`, `voice_states`, `presences`, `messages`, `guild_messages`, `dm_messages`, `reactions`, `typing`, `message_content`, `automod`.

---

### 2.2 Client vs Bot

| | `discord.Client` | `commands.Bot` |
|---|---|---|
| **Purpose** | Raw gateway connection – events only | Adds **prefix commands**, **help command**, **cogs** |
| **When to use** | Simple event‑only bots, slash‑command‑only bots | Traditional prefix‑command bots, hybrid bots |
| **Inherits from** | `discord.Client` | `discord.Client` |

```python
# Client – events only
client = discord.Client(intents=intents)

@client.event
async def on_message(message):
    if message.author == client.user:
        return
    if message.content == "hello":
        await message.channel.send("Hi!")

# Bot – prefix commands
bot = commands.Bot(command_prefix="!", intents=intents)
```

> **Tip:** You can use `app_commands` (slash commands) with either `Client` or `Bot`.

---

### 2.3 Registering Events – 3 Ways

1. **`@client.event`** – simplest, one handler per event.
2. **Subclassing** – override methods in a `Client` subclass.
3. **`@client.listen()`** – multiple handlers for the same event; `once=True` fires only once.

```python
# Way 3: listen()
@client.listen()
async def on_message(message):
    print("Listener 1:", message.content)

@client.listen()
async def on_message(message):
    print("Listener 2:", message.content)   # both run
```

> **All event handlers must be `async def` coroutines**. If an exception is raised, `on_error` is called automatically.

---

## 3. The Client & Bot Objects

### 3.1 `discord.Client` – Key Methods & Properties

| Property / Method | Type | Description |
|---|---|---|
| `client.user` | `ClientUser` | The connected bot user |
| `client.application_id` | `int` | Bot’s application ID |
| `await client.login(token)` | coroutine | Log in (rarely used directly – `run()` is preferred) |
| `await client.connect()` | coroutine | Connect to Discord (advanced) |
| `await client.close()` | coroutine | Gracefully close the connection |
| `await client.change_presence(...)` | coroutine | Change status / activity |
| `await client.fetch_user(id)` | coroutine | Fetch a user by ID |
| `await client.fetch_guild(id)` | coroutine | Fetch a guild by ID |
| `await client.fetch_channel(id)` | coroutine | Fetch a channel by ID |
| `client.get_guild(id)` | `Guild | None` | Get a cached guild |
| `client.get_channel(id)` | `Channel | None` | Get a cached channel |
| `client.get_user(id)` | `User | None` | Get a cached user |
| `client.guilds` | `list[Guild]` | All guilds the bot is in |
| `client.latency` | `float` | WebSocket latency in seconds |

---

### 3.2 `commands.Bot` – Additional Methods

| Method / Property | Description |
|---|---|
| `@bot.command()` | Define a prefix command |
| `@bot.hybrid_command()` | Define a command usable as both prefix & slash |
| `bot.add_cog(cog)` | Register a cog |
| `bot.load_extension("cogs.module")` | Load an extension |
| `bot.tree` | `app_commands.CommandTree` – for slash‑command syncing |
| `bot.command_prefix` | The configured prefix |
| `bot.get_command(name)` | Retrieve a command object |
| `bot.walk_commands()` | Iterate over all registered commands |

---

## 4. Data Models – Guild, Member, Channel, Message

All Discord objects share a **snowflake ID** (64‑bit integer) and a `created_at` property.

### 4.1 `discord.Guild` (Server)

| Attribute | Type | Description |
|---|---|---|
| `guild.id` | `int` | Snowflake ID |
| `guild.name` | `str` | Server name |
| `guild.owner_id` | `int` | Owner’s user ID |
| `guild.member_count` | `int` | Number of members |
| `guild.channels` | `list[GuildChannel]` | All channels |
| `guild.text_channels` | `list[TextChannel]` | Text channels only |
| `guild.voice_channels` | `list[VoiceChannel]` | Voice channels only |
| `guild.categories` | `list[CategoryChannel]` | Category channels |
| `guild.roles` | `list[Role]` | All roles |
| `guild.emojis` | `list[Emoji]` | Custom emojis |
| `guild.me` | `Member` | The bot’s own member object |
| `guild.icon` | `Asset | None` | Server icon |
| `guild.created_at` | `datetime` | Creation timestamp |

---

### 4.2 `discord.Member` (User in a Guild)

| Attribute | Type | Description |
|---|---|---|
| `member.id` | `int` | User ID |
| `member.name` | `str` | Username (without discriminator) |
| `member.display_name` | `str` | Nickname or username |
| `member.mention` | `str` | Mention string (`<@id>`) |
| `member.guild` | `Guild` | The guild this member belongs to |
| `member.roles` | `list[Role]` | Roles (excluding `@everyone`) |
| `member.top_role` | `Role` | Highest role |
| `member.status` | `Status` | `online`, `idle`, `dnd`, `offline` |
| `member.activity` | `Activity | None` | Current activity |
| `member.joined_at` | `datetime` | When they joined the guild |
| `member.premium_since` | `datetime | None` | Nitro boosting since |
| `await member.kick(reason=...)` | coroutine | Kick |
| `await member.ban(reason=...)` | coroutine | Ban |
| `await member.add_roles(role)` | coroutine | Add a role |
| `await member.remove_roles(role)` | coroutine | Remove a role |
| `await member.send(content)` | coroutine | DM the member |

---

### 4.3 Channels – Types & Key Classes

`discord.ChannelType` enumerates all channel kinds:

| `ChannelType` | Class | Description |
|---|---|---|
| `text` | `TextChannel` | Standard text channel |
| `voice` | `VoiceChannel` | Voice channel |
| `category` | `CategoryChannel` | Category grouping |
| `news` | `TextChannel` (news) | Announcement channel |
| `stage_voice` | `StageChannel` | Stage channel |
| `private` | `DMChannel` | Direct message |
| `group` | `GroupChannel` | Group DM |
| `public_thread` / `private_thread` / `news_thread` | `Thread` | Threads |

**Common `TextChannel` methods:**

| Method | Description |
|---|---|
| `await channel.send(content, embed=..., view=...)` | Send a message |
| `await channel.fetch_message(id)` | Fetch a specific message |
| `channel.history(limit=...)` | Async iterator over messages |
| `await channel.purge(limit=...)` | Bulk delete messages |
| `await channel.create_thread(...)` | Create a thread |
| `await channel.set_permissions(member, ...)` | Overwrite permissions |

---

### 4.4 `discord.Message`

| Attribute | Type | Description |
|---|---|---|
| `message.id` | `int` | Message ID |
| `message.content` | `str` | Raw text content |
| `message.author` | `User | Member` | Who sent it |
| `message.channel` | `Messageable` | Where it was sent |
| `message.guild` | `Guild | None` | Guild (None in DMs) |
| `message.attachments` | `list[Attachment]` | Files |
| `message.embeds` | `list[Embed]` | Embeds |
| `message.created_at` | `datetime` | Timestamp |
| `message.edited_at` | `datetime | None` | Last edit |
| `message.mentions` | `list[User | Member]` | Mentioned users |
| `message.reactions` | `list[Reaction]` | Reactions |
| `await message.edit(content=...)` | coroutine | Edit the message |
| `await message.delete()` | coroutine | Delete the message |
| `await message.reply(content)` | coroutine | Reply to the message |
| `await message.add_reaction(emoji)` | coroutine | Add a reaction |

---

## 5. Embeds – Every Field, Style, and Limit

An **Embed** is a rich, structured message block. The `discord.Embed` class gives you a builder‑style interface over the raw JSON payload.

### 5.1 Embed Structure & Limits

| Attribute | Limit | Set via |
|---|---|---|
| `title` | **256 chars** | `Embed(title=...)` |
| `description` | **4096 chars** | `Embed(description=...)` |
| `url` | – | `Embed(url=...)` |
| `colour` / `color` | – | `Embed(colour=...)` |
| `timestamp` | – | `Embed(timestamp=...)` |
| **`fields`** | **max 25 fields** | `add_field()` |
| └ `field.name` | **256 chars** | `add_field(name=...)` |
| └ `field.value` | **1024 chars** | `add_field(value=...)` |
| **`footer.text`** | **2048 chars** | `set_footer(text=...)` |
| **`author.name`** | **256 chars** | `set_author(name=...)` |
| **Total embed** | **6000 chars** (sum of title, description, field names, field values, footer, author name) | – |

> *Maximum of **25 fields** per embed**. Total character limit is **6000**.*

---

### 5.2 Every Embed Method

| Method | Sets | Parameters |
|---|---|---|
| `add_field(name, value, inline=True)` | Appends a field | `name ≤ 256`, `value ≤ 1024` |
| `insert_field_at(index, name, value, inline=True)` | Inserts a field at position | Index must be valid |
| `set_field_at(index, name, value, inline=True)` | Replaces a field | Raises `IndexError` if invalid |
| `remove_field(index)` | Deletes a field | Silently ignores invalid index |
| `clear_fields()` | Removes all fields | – |
| `set_footer(text, icon_url=None)` | Footer block | `text ≤ 2048` |
| `remove_footer()` | Clears footer | – |
| `set_image(url)` | Main image (large) | Pass `None` to remove |
| `set_thumbnail(url)` | Thumbnail (small, top‑right) | Pass `None` to remove |
| `set_author(name, url=None, icon_url=None)` | Author block | `name ≤ 256` |
| `remove_author()` | Clears author | – |

> All setter methods **return `self`**, enabling fluent chaining.

---

### 5.3 Colour Styles

`discord.Colour` (alias `discord.Color`) provides predefined constants and factory methods:

```python
# Predefined
discord.Colour.default()       # 0x000000
discord.Colour.teal()          # 0x1ABC9C
discord.Colour.dark_teal()     # 0x11806A
discord.Colour.green()         # 0x2ECC71
discord.Colour.dark_green()    # 0x1F8B4C
discord.Colour.blue()          # 0x3498DB
discord.Colour.dark_blue()     # 0x206694
discord.Colour.purple()        # 0x9B59B6
discord.Colour.dark_purple()   # 0x71368A
discord.Colour.magenta()       # 0xE91E63
discord.Colour.dark_magenta()  # 0xAD1457
discord.Colour.gold()          # 0xF1C40F
discord.Colour.dark_gold()     # 0xC27C0E
discord.Colour.orange()        # 0xE67E22
discord.Colour.dark_orange()   # 0xA84300
discord.Colour.red()           # 0xE74C3C
discord.Colour.dark_red()      # 0x992D22
discord.Colour.lighter_grey()  # 0x95A5A6
discord.Colour.dark_grey()     # 0x607D8B
discord.Colour.light_grey()    # 0x979C9F
discord.Colour.dark_theme()    # 0x36393F
discord.Colour.blurple()       # 0x7289DA
discord.Colour.greyple()       # 0x99AAB5
discord.Colour.random()        # Random colour

# Custom
discord.Colour(0xFF5733)       # From hex integer
discord.Colour.from_rgb(255, 87, 51)
discord.Colour.from_hsv(0.05, 0.8, 1.0)
discord.Colour.from_str("#FF5733")
```

---

### 5.4 Complete Embed Example

```python
embed = discord.Embed(
    title="🚀 Server Rules",
    description="Please read and follow these rules.",
    colour=discord.Colour.green(),
    timestamp=discord.utils.utcnow()
)
embed.set_author(name="Moderation Team", icon_url="https://example.com/mod.png")
embed.set_thumbnail(url="https://example.com/logo.png")
embed.add_field(name="1. Be respectful", value="No harassment or hate speech.", inline=False)
embed.add_field(name="2. No spam", value="Keep the chat clean.", inline=False)
embed.set_image(url="https://example.com/banner.png")
embed.set_footer(text="Last updated", icon_url="https://example.com/footer.png")

await channel.send(embed=embed)
```

---

## 6. Buttons, Selects, Views & Modals (UI Components)

Interactive UI in `discord.py` is built with **Views** (`discord.ui.View`), which contain **Items** such as **Buttons** and **Select Menus**. **Modals** are pop‑up forms that contain **Text Inputs**.

### 6.1 Button Styles – `discord.ButtonStyle`

| Style | Value | Colour | Use Case |
|---|---|---|---|
| `ButtonStyle.primary` | 1 | Blurple (blue) | Main actions |
| `ButtonStyle.secondary` | 2 | Grey | Secondary actions |
| `ButtonStyle.success` | 3 | Green | Confirm / positive |
| `ButtonStyle.danger` | 4 | Red | Destructive / negative |
| `ButtonStyle.link` | 5 | Grey (with link icon) | Opens a URL |

> `primary` / `blurple` = 1, `secondary` / `grey` = 2, `success` / `green` = 3, `danger` / `red` = 4, `link` / `hyperlink` = 5.

**Button parameters:**

| Parameter | Type | Description |
|---|---|---|
| `style` | `ButtonStyle` | Visual style |
| `label` | `str` | Text on the button |
| `emoji` | `PartialEmoji` / `str` | Emoji prefix |
| `custom_id` | `str` | Unique ID for callbacks |
| `url` | `str` | Required for `link` style |
| `disabled` | `bool` | Grey‑out the button |
| `row` | `int` (0–4) | Row position in the View |

```python
class MyView(discord.ui.View):
    @discord.ui.button(label="Click Me", style=discord.ButtonStyle.primary, emoji="👋")
    async def click_callback(self, interaction: discord.Interaction, button: discord.ui.Button):
        await interaction.response.send_message("You clicked!", ephemeral=True)

    @discord.ui.button(label="Visit Website", style=discord.ButtonStyle.link, url="https://example.com")
    async def link_callback(self, interaction: discord.Interaction, button: discord.ui.Button):
        pass   # link buttons don't fire callbacks
```

---

### 6.2 Select Menus – `discord.ui.Select`

| Property | Limit / Notes |
|---|---|
| **Max options** | **25** |
| `placeholder` | Text shown when nothing is selected |
| `min_values` | Default `1` |
| `max_values` | Default `1` |
| `disabled` | Boolean |

```python
class DropdownView(discord.ui.View):
    @discord.ui.select(
        placeholder="Choose a role...",
        options=[
            discord.SelectOption(label="Red", value="red", emoji="🔴"),
            discord.SelectOption(label="Green", value="green", emoji="🟢"),
            discord.SelectOption(label="Blue", value="blue", emoji="🔵"),
        ]
    )
    async def select_callback(self, interaction: discord.Interaction, select: discord.ui.Select):
        await interaction.response.send_message(f"You chose: {select.values[0]}")
```

---

### 6.3 Views – Container & Lifecycle

A **View** holds up to **5 rows** of components. It manages timeouts and interaction routing.

```python
class ConfirmView(discord.ui.View):
    def __init__(self):
        super().__init__(timeout=60)

    @discord.ui.button(label="Yes", style=discord.ButtonStyle.success)
    async def yes(self, interaction: discord.Interaction, button: discord.ui.Button):
        await interaction.response.send_message("Confirmed!", ephemeral=True)
        self.stop()

    @discord.ui.button(label="No", style=discord.ButtonStyle.danger)
    async def no(self, interaction: discord.Interaction, button: discord.ui.Button):
        await interaction.response.send_message("Cancelled.", ephemeral=True)
        self.stop()
```

**Key `View` parameters:** `timeout` (seconds, `None` = no timeout), `disable_on_timeout`.  
**Key methods:** `add_item()`, `remove_item()`, `stop()`, `is_finished()`.

---

### 6.4 Modals – Pop‑up Forms

Modals inherit from `discord.ui.Modal` and contain **TextInput** fields.

```python
class FeedbackModal(discord.ui.Modal, title="Feedback"):
    name = discord.ui.TextInput(label="Your Name", placeholder="John Doe")
    feedback = discord.ui.TextInput(
        label="Feedback",
        style=discord.TextStyle.paragraph,
        max_length=500,
        required=True
    )

    async def on_submit(self, interaction: discord.Interaction):
        await interaction.response.send_message(
            f"Thanks {self.name.value}! Feedback received.", ephemeral=True
        )

# Show the modal
await interaction.response.send_modal(FeedbackModal())
```

**TextInput styles:** `short` (default, single‑line), `paragraph` (multi‑line).

---

## 7. Application Commands (Slash Commands)

Use the `discord.app_commands` module to define **slash commands**, **user commands**, and **message commands**.

### 7.1 Slash Command Basics

```python
from discord import app_commands

@bot.tree.command(name="greet", description="Greet a user")
@app_commands.describe(user="The user to greet")
async def greet(interaction: discord.Interaction, user: discord.Member):
    await interaction.response.send_message(f"Hello, {user.mention}!")
```

### 7.2 Syncing the Command Tree

```python
@bot.event
async def on_ready():
    await bot.tree.sync()          # global sync (may take up to 1 hour)
    # or, for a specific guild (instant):
    # await bot.tree.sync(guild=discord.Object(id=YOUR_GUILD_ID))
```

### 7.3 Command Types & Checks

| Decorator | Type |
|---|---|
| `@app_commands.command()` | Slash command |
| `@app_commands.context_menu()` | User / Message command |
| `@app_commands.describe(**params)` | Describe parameters |
| `@app_commands.checks.has_permissions(...)` | Permission check |
| `@app_commands.checks.bot_has_permissions(...)` | Bot permission check |
| `@app_commands.checks.cooldown(...)` | Cooldown |

### 7.4 Autocomplete

```python
@bot.tree.command()
@app_commands.autocomplete(fruit=fruit_autocomplete)
async def pick(interaction: discord.Interaction, fruit: str):
    ...

async def fruit_autocomplete(interaction: discord.Interaction, current: str):
    fruits = ["apple", "banana", "cherry"]
    return [
        app_commands.Choice(name=f, value=f)
        for f in fruits if current.lower() in f.lower()
    ][:25]   # max 25 choices
```

---

## 8. Events – The Complete Reference

All events are listened to by `Client` and must be `async` coroutines.  
Below is a categorised list of the **most commonly used events**. (The full list contains 100+ events – see the official Event Reference for every single one.)

### 8.1 Application Commands

| Event | Fired when… |
|---|---|
| `on_application_command(context)` | A slash / app command is received |
| `on_application_command_completion(context)` | After checks pass and command completes |
| `on_application_command_error(context, exception)` | An app command raises an error |
| `on_unknown_application_command(interaction)` | An unknown app command is invoked |

### 8.2 Messages

| Event | Fired when… |
|---|---|
| `on_message(message)` | A message is sent |
| `on_message_edit(before, after)` | A message is edited |
| `on_message_delete(message)` | A message is deleted |
| `on_bulk_message_delete(messages)` | Messages are bulk‑deleted |
| `on_message_reaction_add(reaction, user)` | A reaction is added |
| `on_message_reaction_remove(reaction, user)` | A reaction is removed |
| `on_message_reaction_clear(message, reactions)` | All reactions are cleared |
| `on_typing(channel, user, when)` | Someone starts typing |

### 8.3 Guilds & Members

| Event | Fired when… |
|---|---|
| `on_guild_join(guild)` | The bot joins a guild |
| `on_guild_remove(guild)` | The bot leaves a guild |
| `on_guild_update(before, after)` | Guild settings change |
| `on_member_join(member)` | A member joins |
| `on_member_remove(member)` | A member leaves |
| `on_member_update(before, after)` | Nickname / roles / status change |
| `on_member_ban(guild, user)` | A user is banned |
| `on_member_unban(guild, user)` | A user is unbanned |
| `on_user_update(before, after)` | Username / avatar change |

### 8.4 Channels & Threads

| Event | Fired when… |
|---|---|
| `on_guild_channel_create(channel)` | A channel is created |
| `on_guild_channel_delete(channel)` | A channel is deleted |
| `on_guild_channel_update(before, after)` | A channel is updated |
| `on_thread_create(thread)` | A thread is created |
| `on_thread_delete(thread)` | A thread is deleted |
| `on_thread_update(before, after)` | A thread is updated |

### 8.5 Voice

| Event | Fired when… |
|---|---|
| `on_voice_state_update(member, before, after)` | A member joins / leaves / moves voice |
| `on_voice_channel_join(channel)`* | Bot joins a voice channel |
| `on_voice_channel_leave(channel)`* | Bot leaves a voice channel |

### 8.6 Lifecycle & Raw

| Event | Fired when… |
|---|---|
| `on_ready()` | Client is fully ready |
| `on_connect()` | Client connects to the gateway |
| `on_disconnect()` | Client disconnects |
| `on_error(event, *args, **kwargs)` | Any event handler raises |
| `on_socket_raw_receive(msg)` | Raw WebSocket message received |
| `on_socket_raw_send(payload)` | Raw WebSocket message sent |

> **Tip:** For a full, exhaustive list of **all 100+ events**, open the official Event Reference.

---

## 9. Presence & Activity – Every Status Type

### 9.1 Status Types – `discord.Status`

| Status | Meaning |
|---|---|
| `discord.Status.online` | Online (green dot) |
| `discord.Status.idle` | Idle / AFK (yellow moon) |
| `discord.Status.dnd` | Do Not Disturb (red minus) |
| `discord.Status.offline` | Offline or invisible (grey ring) |
| `discord.Status.invisible` | Invisible (appears offline) |

> `invisible` is **not a distinct gateway status** – it is sent as `offline`. Use it only when changing your own presence.

### 9.2 Activity Types – `discord.ActivityType`

| Type | Display Format |
|---|---|
| `ActivityType.playing` | “Playing {name}” |
| `ActivityType.streaming` | “Streaming {name}” (requires a Twitch/YouTube URL) |
| `ActivityType.listening` | “Listening to {name}” |
| `ActivityType.watching` | “Watching {name}” |
| `ActivityType.competing` | “Competing in {name}” |
| `ActivityType.custom` | Custom status (bots **cannot** use this) |

> Bots can use **Playing, Streaming, Listening, Watching, and Competing** – **not Custom**.

### 9.3 Changing Presence – `change_presence()`

```python
await bot.change_presence(
    status=discord.Status.idle,
    activity=discord.Activity(
        type=discord.ActivityType.watching,
        name="over the server"
    )
)

# Streaming example
await bot.change_presence(
    activity=discord.Streaming(
        name="Live coding",
        url="https://twitch.tv/yourchannel"
    )
)
```

> Use `change_presence()` **after** the bot is ready (e.g., inside `on_ready`).

---

## 10. Permissions, Roles & Audit Logs

### 10.1 `discord.Permissions`

`Permissions` is a **flag** object – you can read, set, and combine permission bits.

```python
# Check
if member.guild_permissions.administrator:
    ...

if channel.permissions_for(member).send_messages:
    ...

# Build
perms = discord.Permissions(
    send_messages=True,
    manage_messages=True,
    read_message_history=True
)

# Common permission flags
perms.administrator          # 1 << 3  – overrides all
perms.kick_members           # 1 << 1
perms.ban_members            # 1 << 2
perms.manage_channels        # 1 << 4
perms.manage_guild           # 1 << 5
perms.manage_roles           # 1 << 28
perms.manage_messages        # 1 << 13
perms.send_messages          # 1 << 11
perms.read_messages          # 1 << 10
perms.embed_links            # 1 << 14
perms.attach_files           # 1 << 15
perms.mention_everyone       # 1 << 17
perms.use_application_commands # 1 << 31
```

> For the **full permission table** (integer values and descriptions), see the official Discord permissions documentation.

### 10.2 Roles

```python
# Create a role
role = await guild.create_role(name="Moderator", colour=discord.Colour.blue())

# Assign / remove
await member.add_roles(role)
await member.remove_roles(role)

# Role properties
role.name, role.colour, role.position, role.permissions, role.mention
```

### 10.3 Audit Logs

`guild.audit_logs()` returns an async iterator of `AuditLogEntry` objects.

| Attribute | Description |
|---|---|
| `entry.action` | `AuditLogAction` enum (e.g., `member_kick`, `message_delete`) |
| `entry.user` | Who performed the action |
| `entry.target` | The affected object |
| `entry.reason` | Reason (if provided) |
| `entry.changes` | `AuditLogDiff` – before/after values |

```python
async for entry in guild.audit_logs(limit=10, action=discord.AuditLogAction.member_kick):
    print(f"{entry.user} kicked {entry.target}")
```

> The full `AuditLogAction` enum covers **80+ actions** – see the official source for every member.

---

## 11. Webhooks & Voice

### 11.1 Webhooks

Webhooks post messages **without a bot user**. Use `discord.Webhook` or `discord.SyncWebhook`.

```python
# Async
webhook = await discord.Webhook.from_url("WEBHOOK_URL", session=session)
await webhook.send("Hello from a webhook!", username="Custom Name", avatar_url="...")

# Sync (simplest)
from discord import SyncWebhook
webhook = SyncWebhook.from_url("WEBHOOK_URL")
webhook.send("Hello from a webhook!")
```

**Common `Webhook.send()` parameters:** `content`, `username`, `avatar_url`, `embed`, `file`, `view`, `wait`.

---

### 11.2 Voice

Install with `pip install discord.py[voice]`. Connect with `VoiceChannel.connect()`, which returns a `VoiceClient`.

```python
voice_client = await channel.connect()

# Play audio (requires an FFmpeg source)
voice_client.play(discord.FFmpegPCMAudio("song.mp3"))

# Disconnect
await voice_client.disconnect()
```

> **Dependency:** Voice playback requires **FFmpeg** installed on your system.

---

## 12. Utility Functions & Error Handling

### 12.1 `discord.utils`

| Function | Description |
|---|---|
| `utils.get(iterable, **attrs)` | First item matching attributes |
| `utils.find(predicate, seq)` | First item matching a predicate |
| `utils.get_or_create` | Get or create a model |
| `utils.snowflake_time(id)` | Convert snowflake → UTC `datetime` |
| `utils.time_snowflake(dt)` | Convert `datetime` → snowflake |
| `utils.utcnow()` | Current UTC‑aware `datetime` |
| `utils.format_dt(dt, style="R")` | Format a timestamp for Discord |
| `utils.escape_markdown(text)` | Escape Discord markdown |
| `utils.remove_markdown(text)` | Strip markdown |
| `utils.oauth_url(client_id, permissions)` | Generate bot invite URL |
| `utils.as_chunks(seq, size)` | Split into chunks |

> `format_dt` styles: `t` (short time), `T` (long time), `d` (short date), `D` (long date), `f` (short date+time), `F` (long date+time), `R` (relative).

### 12.2 Exception Hierarchy

```
DiscordException
├── ClientException
├── GatewayNotFound
├── HTTPException
│   ├── Forbidden        (403)
│   ├── NotFound         (404)
│   ├── DiscordServerError (5xx)
│   └── RateLimited      (429)
├── InvalidData
├── LoginFailure
└── ConnectionClosed
```

> App‑command‑specific errors live in `discord.app_commands.errors` (e.g., `CommandOnCooldown`, `CheckFailure`).

**Global error handler example:**

```python
@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.CommandOnCooldown):
        await ctx.send(f"Try again in {error.retry_after:.1f}s.")
    elif isinstance(error, commands.MissingPermissions):
        await ctx.send("You lack permissions.")
    else:
        raise error
```

---

## 13. Cogs & Extensions

**Cogs** organise commands and listeners into modular classes. Each extension file contains **one Cog** and a `setup()` function.

### 13.1 Cog Structure

```python
# cogs/greetings.py
import discord
from discord.ext import commands

class Greetings(commands.Cog):
    def __init__(self, bot: commands.Bot):
        self.bot = bot

    @commands.command()
    async def hello(self, ctx: commands.Context):
        await ctx.send(f"Hello, {ctx.author.mention}!")

    @commands.Cog.listener()
    async def on_member_join(self, member: discord.Member):
        channel = member.guild.system_channel
        if channel:
            await channel.send(f"Welcome, {member.mention}!")

async def setup(bot: commands.Bot):
    await bot.add_cog(Greetings(bot))
```

### 13.2 Loading Cogs

```python
# In your main file
await bot.load_extension("cogs.greetings")   # no .py extension
```

> **Important:** Inside a Cog, **never use `@bot.event`** – use `@commands.Cog.listener()` instead.

---

## 14. Quick‑Reference Cheat Sheet

### 🧩 Imports

```python
import discord
from discord.ext import commands
from discord import app_commands
from discord import ui
```

### 🛠️ Bot Setup

```python
intents = discord.Intents.default()
intents.message_content = True
intents.members = True
bot = commands.Bot(command_prefix="!", intents=intents)
```

### 📨 Sending Messages

```python
await channel.send("text")
await channel.send(embed=embed)
await channel.send("text", view=my_view)
await ctx.reply("text", mention_author=True)
await interaction.response.send_message("text", ephemeral=True)
```

### 🎨 Embed Quick‑Build

```python
embed = discord.Embed(title="Title", description="Desc", colour=discord.Colour.blue())
embed.add_field(name="Field", value="Value", inline=False)
embed.set_footer(text="Footer")
embed.set_thumbnail(url="...")
```

### 🔘 Button Quick‑Build

```python
class V(ui.View):
    @ui.button(label="OK", style=discord.ButtonStyle.success)
    async def ok(self, itx: discord.Interaction, btn: ui.Button):
        await itx.response.send_message("Done!", ephemeral=True)
```

### 🗑️ Deleting & Editing

```python
await message.delete()
await message.edit(content="new")
await message.add_reaction("✅")
```

### 📋 Permissions Check

```python
if ctx.author.guild_permissions.manage_messages:
    ...
```

### 🕒 Timestamp Formatting

```python
discord.utils.format_dt(datetime.now(), style="R")   # "in 2 minutes"
```

### 📜 All Statuses & Activities

| Status | Activity |
|---|---|
| `online` | `playing` |
| `idle` | `streaming` |
| `dnd` | `listening` |
| `offline` / `invisible` | `watching` / `competing` |

### 🧾 Event Quick‑List

```python
on_ready, on_message, on_message_edit, on_message_delete,
on_member_join, on_member_remove, on_member_update,
on_guild_join, on_guild_remove, on_reaction_add,
on_voice_state_update, on_application_command,
on_application_command_error, on_error
```

---

## 📚 Official Resources

| Resource | Link |
|---|---|
| **Official Documentation** | [https://discordpy.readthedocs.io](https://discordpy.readthedocs.io) |
| **API Reference (all classes)** | [https://discordpy.readthedocs.io/en/stable/api.html](https://discordpy.readthedocs.io/en/stable/api.html) |
| **Event Reference (all events)** | [https://discordpy.readthedocs.io/en/stable/api.html#event-reference](https://discordpy.readthedocs.io/en/stable/api.html#event-reference) |
| **GitHub Repository** | [https://github.com/Rapptz/discord.py](https://github.com/Rapptz/discord.py) |
| **Discord.py v2.0 Migration Guide** | [https://discordpy.readthedocs.io/en/stable/migrating.html](https://discordpy.readthedocs.io/en/stable/migrating.html) |

---
