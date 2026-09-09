# grug · [![](https://dcbadge.limes.pink/api/server/https://discord.com/invite/ufeJ6MBXJG)](https://discord.com/invite/ufeJ6MBXJG)

grug is a programming language with one job: make your mods immortal.

grug's vision is that a mod (also called a plugin or extension) written today will continue to run 100 years from now, regardless of how much around it changes. For example, a Minecraft mod written in grug aims to run in any version of the game, from Alpha 1.1.2_01 (2010) to the latest, eliminating [version chasing](https://www.youtube.com/watch?v=yfRq_a5wEEg) by requiring no manual changes to mod source code whatsoever.

https://github.com/user-attachments/assets/2a7949ae-643c-4274-9a06-12e528affe97

In the video above, four different Minecraft environments all hot reload the same grug file:
* Minecraft 1.20.6 with Forge
* Minecraft Beta 1.7.3 with Ornithe
* Minecraft Beta 1.7.3 with StationAPI
* Minecraft Alpha 1.1.2_01 with Ornithe

grug is still undergoing heavy evolution, so expect breaking changes as the language and its implementations mature.

## How?

`mod_api.json` declares the full mod↔host [API](https://en.wikipedia.org/wiki/API) surface (entities, classes, and functions), specifying which versions of the application each individual function is available in. By restricting mods to this declared API, grug frees moderators from security-oriented code reviews, letting them focus on content changes like images and audio. Since it follows a [well-defined schema](https://github.com/grug-lang/grug-tests/blob/main/mod_api_schema.json), the community can build websites that render it as browsable documentation. Players can override `mod_api.json` via [DLL injection](https://en.wikipedia.org/wiki/DLL_injection), so a closed-source host's modding API can keep expanding after its developers stop, via a community-maintained `mod_api.json` rather than per-mod additions. grug backends can be swapped on the fly, so players can pick whichever gives the best performance.

grug achieves this immortality with a minimal, strongly-typed language, a small [LALR(1) grammar](https://github.com/grug-lang/grug-tests/blob/main/grug_grammar.lark), and opt-in standard library features. Mods compile to a lossless, whitespaceless [JSON](https://en.wikipedia.org/wiki/JSON) [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree), which enables automatic upgrading, downgrading, and cross-language [transpilation](https://en.wikipedia.org/wiki/Source-to-source_compiler), making grug a universal modding language. From that AST, mods compile to [grug IR](https://github.com/grug-lang/grug-ir), which backends can transpile to other formats, such as [LLVM](https://en.wikipedia.org/wiki/LLVM) IR. By using [reference counting](https://en.wikipedia.org/wiki/Reference_counting) instead of [garbage collection](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)), grug prevents lag spikes and allows LLVM to optimize most memory allocations away. Host functions compile to grug IR ahead of time too, which eliminates [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) overhead and makes them inlinable. grug entities use the [actor model](https://en.wikipedia.org/wiki/Actor_model), communicating with each other only through host functions and optionally running across any number of threads.

## Simple Example

This is `mods/monsters/zombie-Actor.grug`:
```py
print("Brainzzzz...")

export tick() {
    # `me` is the current zombie running this tick() function.
    player: Actor = me.get_nearest_player()

    # Attack the player if it is close
    if me.distance(player) < 100 {
        player.add_health(-3)
    }
}
```

## Advanced Example

This is `mods/cheats/godmode-Entity.grug`. It shows virtually _every_ feature grug has, so if a feature isn't shown, like defining your own types, grug simply doesn't have it:
```py
# This is a member variable (persists across ticks),
# which means every entity gets its own copy of it.
i: number = 0

# All code in member scope runs when the entity is created.
while i < 3 {
    # This play_sound() host function is declared by mod_api.json.
    # r"" is a resource string, validating that `mods/cheats/sounds/activation.mp3` exists.
    # grug watches the mods directory so any file can be hot reloaded.
    # Resource strings deliberately can't refer to resources in other mods.
    play_sound(r"sounds/activation.mp3")

    i = i + 1
}

# Option is a generic class declared by mod_api.json.
# List and Dict are other generic classes many games declare.
# Option.new() is a static method.
opt_player: Option[Player] = Option.new()

# The host can call this exported function, and is declared by mod_api.json.
# Other grug files can't call this exported function directly.
export spawn_from_parent(parent: Entity) {
    # This .cast() is a method.
    # The game raises a runtime error if it wasn't spawned by a Player.
    player: Player = parent.cast()

    opt_player.set(player)
}

export tick() {
    # The game would raise a runtime error if spawn_from_parent()
    # weren't always called first by the game.
    player: Player = opt_player.unwrap()

    if player.health() < 10 {
        _heal_player(player)
    }
}

# The host can't call local functions.
# Local function names must start with an underscore.
local _heal_player(player: Player) {
    # Arguments can optionally document the parameter name for readability.
    player.set_health(amount=100)

    # pos() returns Pos.
    # e"" is an entity string, which validates that `sparkle-Entity.grug`
    # exists somewhere in `mods/vanilla/`.
    # spawn() declares it expects an Entity, but grug permits subtypes like Particle.
    # Removing the `vanilla:` prefix restricts the search to this grug file's mod.
    player.pos().spawn(e"vanilla:sparkle")
}
```

## Blog Post and YouTube Video

grug started out as a simple wishlist in [my first blog post](https://mynameistrez.github.io/2024/02/29/creating-the-perfect-modding-language.html). It evolved alongside [`grug.c`](https://github.com/grug-lang/grug/tree/legacy).

I turned the blog post into a presentation for work, and posted it to YouTube, where it got a lot more attention than I had anticipated: [Creating grug: the perfect modding language](https://www.youtube.com/watch?v=4oUToVXR2Vo)

grug has matured a lot after the blog post and YouTube video were published, but its essence has stayed the same.

## Contributing

The contributing guidelines are the same for all repositories in the grug-lang organization. Create an issue before you make any pull request, as that gives everyone the chance to discuss it.

If you are planning to work on an issue, leave a comment asking to be assigned to the issue. Only work on issues that have the `planned` label.

High-quality, concise LLM-generated PRs are **only** welcome for issues labeled `good first issue`. They do not need to be assigned to you first, since these issues are written in enough detail to make the PRs easy to review. PRs that do not address all review feedback will be closed.

grug has an issue board that can be filtered to list [good first issues](https://github.com/orgs/grug-lang/projects/1/views/1?sliceBy%5Bvalue%5D=good+first+issue) and [planned issues](https://github.com/orgs/grug-lang/projects/1/views/1?sliceBy%5Bvalue%5D=planned).

## Links

### Implementations

grug implementations consist of three core components: API bindings, a grug parser frontend, and a backend execution engine.

These grug implementations by the community pass all 500+ official [grug tests](https://github.com/grug-lang/grug-tests), and include 'Hello, world!' examples:
- [grug-for-python](https://github.com/grug-lang/grug-for-python)
- [grug-for-c](https://github.com/grug-lang/grug-for-c) (don't use this yet, as it's very new)
- [grug-rs](https://github.com/grug-lang/grug-rs)
- [grug-for-lua](https://github.com/grug-lang/grug-for-lua)

### Games

- [grug for Minecraft](https://github.com/grug-lang/grug-for-minecraft) (grug's flagship project)
- [grug for ComputerCraft](https://github.com/grug-lang/grug-for-computercraft)
- [grug for Cortex Command](https://github.com/grug-lang/grug-for-cortex-command)
- [grug Box2D and raylib game](https://github.com/grug-lang/grug-box2d-and-raylib-game) (will be rewritten)
- [grug terminal game: C/C++](https://github.com/grug-lang/grug-terminal-game-c-cpp) (will be rewritten)
- [grug terminal game: Java](https://github.com/grug-lang/grug-terminal-game-java) (will be rewritten)

### Other

- [grug-vscode](https://github.com/grug-lang/grug-vscode)
- [grug-bench](https://github.com/grug-lang/grug-bench)
- [grug-ir](https://github.com/grug-lang/grug-ir)
- [grug-discord-bot](https://github.com/grug-lang/grug-discord-bot)
