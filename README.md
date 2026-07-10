# grug · [![](https://dcbadge.limes.pink/api/server/https://discord.com/invite/ufeJ6MBXJG)](https://discord.com/invite/ufeJ6MBXJG)

grug is a modding language with one job: make your mods immortal. grug's vision is that a mod written today will continue to run 100 years from now, regardless of how much around it changes. A Minecraft mod written in grug aims to run in any version of the game, from Beta 1.7.3 (2011) to the latest, with no manual changes to its source code whatsoever.

grug achieves this with a minimal, strongly-typed language, a small [LALR(1) grammar](https://github.com/grug-lang/grug-tests/blob/main/grug_grammar.lark), and no standard library. Mods compile to a lossless, whitespaceless JSON AST, which enables automatic upgrading, downgrading, and cross-language transpilation. From that AST, mods compile to [grug IR](https://github.com/grug-lang/grug-ir). Host functions compile to grug IR ahead of time too, which eliminates FFI overhead and makes them inlinable.

`mod_api.json` declares the full mod-host API surface (entities, classes, and functions), and players can override it. grug backends are hot swappable, so games can pick whichever gives the best performance. Hot reloading of code and resources speeds up iteration during development.

grug is still undergoing heavy evolution, so expect breaking changes as the language and its implementations mature.

<video src="https://github.com/user-attachments/assets/31959bcf-e933-4080-bbb6-3c76fe8bfa39" width="100%" autoplay controls loop muted></video>

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

This is `mods/cheats/godmode-Entity.grug`:
```py
# This is a member variable (persists across frames),
# which means every entity gets its own copy of it.
i: number = 0

# All code in member scope runs when the entity is created.
while i < 3 {
    # This play_sound() host function is declared by mod_api.json.
    # r"" is a resource string, which lets grug periodically check
    # that `mods/cheats/sounds/activation.mp3` exists and hot reload it.
    # Resource strings deliberately can't refer to resources in other mods.
    play_sound(r"sounds/activation.mp3")

    i = i + 1
}

# Optional is a generic class declared by mod_api.json.
# List and Dict are other generic classes many games declare.
opt_player: Optional[Player] = optional()

# The host can call this exported function, and is declared by mod_api.json.
# Other grug files can't call this exported function directly.
export spawn_from_parent(parent: Entity) {
    # This .cast() is a method.
    # The game raises a runtime error if it wasn't spawned by a Player.
    player: Player = parent.cast("Player")

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
    player.set_health(100)

    # pos() returns Pos.
    # e"" is an entity string, which lets grug periodically check
    # that `sparkle-Entity.grug` exists somewhere in mods/vanilla/.
    # spawn() declares it expects an Entity, but grug permits subtypes like Particle.
    # Removing the `vanilla:` prefix would restrict the search to this mod only.
    player.pos().spawn(e"vanilla:sparkle")
}
```

## Blog Post and YouTube Video

grug started out as a simple wishlist in [my first blog post](https://mynameistrez.github.io/2024/02/29/creating-the-perfect-modding-language.html). It evolved alongside [`grug.c`](https://github.com/grug-lang/grug/tree/legacy).

I turned the blog post into a presentation for work, and posted it to YouTube, where it got a lot more attention than I had anticipated: [Creating grug: the perfect modding language](https://www.youtube.com/watch?v=4oUToVXR2Vo)

grug has matured a lot after the blog post and YouTube video were published, but its essence has stayed the same.

## Contributing

Create an issue before you make any pull request, as that gives everyone the chance to discuss it.

If you are planning to work on an issue, leave a comment asking to be assigned to the issue. Only work on issues that have the `planned` label.

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

- [grug for Minecraft](https://github.com/grug-lang/grug-for-minecraft) (will be rewritten)
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
