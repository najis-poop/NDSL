# Najis's Discord Scripting Language (NDSL)

--------------------------------------------

NDSL is a scripting language for scripting bot. Designed for auto-moderation or tag command. Can also use for other purpose.

The language still under development but you can still use it. Please report bug that you found.

## Usage

Simple automod using NDSL

```py
import ndsl

tasks = {} # you may use a database for this 

@bot.group("automod")
async def automod(_: commands.Context):
    pass

@automod.command("create")
@commands.has_permissions(administrator=True)
async def create_task(ctx: commands.Context, name: str, *, code: str):
    c = ndsl.clean_code(code)
    
    env = ndsl.StaticEnv()
    env.putvar("message", ndsl.MessageSymbol, const=True) # so static checker won't throw unknown message variable error
    result = ndsl.parse_compile(c, env)

    if result.error:
        await ctx.reply(embed=result.create_error_embed())
        return
    
    tasks[name] = result.bytecode

    await ctx.reply(f"Succesfully created {name} task")

@automod.command("remove")
@commands.has_permissions(administrator=True)
async def remove_task(ctx: commands.Context, *, name: str):
    if name not in tasks:
        await ctx.reply(f"No {name} task found")
        return
    
    del tasks[name]
    await ctx.reply(f"Succesfully removed {name} task")

@bot.event
async def on_message(msg: discord.Message):
    await bot.process_commands(message)
    m_obj = ndsl.wrap(msg)

    env = ndsl.Env()
    env.put("message", m_obj, const=True)
    for task in tasks:
        ndsl.run(task, env) # this would create new thread so it can create asyncio loop without blocking current loop
        if m_obj.message is None:
            break # some task would delete the message
```

```ndsl
// example of task that will delete message with blacklisted word

// we don't want it to accidentally delete message by admin
if message.author.is_admin() {
    return; // yes, you can return outside function 
}

var banned_words = [
    "badword",
    "drowdab",
    "najis-poop-troll" // we don't want one member leak my password to all
];

for word in message.content.lower().split() {
    for bword in banned_words {
        if word.compare(bword) > 90 { // checking how close both words is
            message.reply("You cannot say that word!");
            message.delete();
            break all;
        }
    }
}

```
