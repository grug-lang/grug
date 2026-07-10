# grug · [![](https://dcbadge.limes.pink/api/server/https://discord.com/invite/ufeJ6MBXJG)](https://discord.com/invite/ufeJ6MBXJG)

grug is a minimal, strongly-typed modding language built for digital preservation, portability, and performance. It produces "immortal" mods that run across platforms from modern systems down to the NES, using a [small LALR(1) grammar](https://github.com/grug-lang/grug-tests/blob/main/grug_grammar.lark), no standard library, and a lossless, whitespaceless JSON AST for automatic upgrading, downgrading, and cross-language transpilation. Mods compile to [grug IR](https://github.com/grug-lang/grug-ir), which eliminates FFI overhead by compiling host functions to grug IR ahead of time, making them inlinable. A community-overridable `mod_api.json` patches missing functionality, swappable backends increase performance, and hot reloading speeds up iteration.

<video src="https://github.com/user-attachments/assets/31959bcf-e933-4080-bbb6-3c76fe8bfa39" width="100%" autoplay controls loop muted></video>

## Blog Post and YouTube Video

grug started out as a simple wishlist in [my first blog post](https://mynameistrez.github.io/2024/02/29/creating-the-perfect-modding-language.html). It evolved alongside [`grug.c`](https://github.com/grug-lang/grug/tree/legacy).

I turned the blog post into a presentation for work, and posted it to YouTube, where it got a lot more attention than I had anticipated: [Creating grug: the perfect modding language](https://www.youtube.com/watch?v=4oUToVXR2Vo)

grug has matured a lot after the blog post and YouTube video were published, but its essence has stayed the same.

## Simple Example

A game might have this `mods/monsters/zombie-Actor.grug`:
```py
print("Brainzzzz...")

i: number = 0
while i < 3 {
    # r"" is a resource string, which lets grug periodically check
    # that `mods/monsters/sounds/brainz.mp3` exists.
    # Resource strings deliberately can't refer to resources from other mods.
    play_sound(r"sounds/brainz.mp3")
    i = i + 1
}

export tick() {
    # `me` refers to this Actor instance of zombie-Actor.grug.
    player: Actor = me.get_nearest_player()

    # Attack the player if it is close
    if me.distance(player) < 100 {
        player.add_health(-3)

        # pos() returns Pos.
        # e"" is an entity string, which lets grug periodically check
        # that `blood_particle-Entity.grug` exists somewhere in mods/vanilla/.
        me.pos().spawn(e"vanilla:blood_particle")
    }
}
```

## Advanced example

A program might have this contrived `fib-Calculator.grug`:
```py
# This is a member variable, which means
# every entity gets its own copy of it.
count: number = 100

# The host can call this exported function.
# This function is declared by mod_api.json.
export run() {
    # We calculate the first 100 values
    # of the fibonacci sequence, and store them
    # in a local List called `fib_numbers`.
    # List is a generic class declared by mod_api.json.
    fib_numbers: List[number] = _fib_list(count)

    # Prints [0, 1, 1, 2, 3, 5, ...].
    # This host function is declared by mod_api.json.
    print_list(fib_numbers)
}

# The host can't call local functions.
# Local function names must start with an underscore.
local _fib_list(n: number) List[number] {
    fib_list: List[number] = list()

    # We will be memoizing (caching) the results
    # to speed up computation.
    memo: Dict[number, number] = dict()

    i: number = 0
    while i < n {
        # This .append() is a method.
        fib_list.append(_fib(i, memo))
        i = i + 1
    }

    return fib_list
}

local _fib(n: number, memo: Dict[number, number]) number {
    # If we already calculated it,
    # we don't have to recalculate it.
    if memo.has_key(n) {
        return memo.get(n)
    }

    result: number = n
    if n > 1 {
        # The fibonacci sequence is recursive:
        # fib(n) = fib(n-1) + fib(n-2)
        result = _fib(n - 1, memo) + _fib(n - 2, memo)
    }

    # Memoize the result, so that we
    # won't have to recalculate it next time.
    memo.set(n, result)
    return result
}
```

## Contributing

Create an issue before you make any pull request, as that gives everyone the chance to discuss it.

If you are planning to work on an issue, leave a comment asking to be assigned to the issue. Only work on issues that have the `planned` label.

grug has an issue board that can be filtered to list [good first issues](https://github.com/orgs/grug-lang/projects/1/views/1?sliceBy%5Bvalue%5D=good+first+issue) and [planned issues](<https://github.com/orgs/grug-lang/projects/1/views/1?sliceBy%5Bvalue%5D=planned>).

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
