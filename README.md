# grug · [![](https://dcbadge.limes.pink/api/server/https://discord.com/invite/ufeJ6MBXJG)](https://discord.gg/https://discord.com/invite/ufeJ6MBXJG)

grug is a minimal, strongly-typed modding language designed for long-term compatibility and "immortal" mods across platforms, from modern systems down to the NES. It features a [small LALR(1) grammar](https://github.com/grug-lang/grug-tests/blob/main/grug_grammar.lark), no standard library, and a lossless, whitespaceless JSON AST to enable automatic upgrading, downgrading, and cross-language transpilation. With hot reloading and full user override of backends and APIs, grug prioritizes portability, control, and permanence.

<video src="https://github.com/user-attachments/assets/31959bcf-e933-4080-bbb6-3c76fe8bfa39" width="100%" autoplay controls loop muted></video>

See the YouTube video [Creating grug: the perfect modding language](https://www.youtube.com/watch?v=4oUToVXR2Vo) for an introduction to the grug modding language, or read [its blog post](https://mynameistrez.github.io/2024/02/29/creating-the-perfect-modding-language.html):

[<img src="https://img.youtube.com/vi/4oUToVXR2Vo/maxresdefault.jpg" height="200">](https://youtu.be/4oUToVXR2Vo)

## Example

Here is a contrived `fib-Calculator.grug` that a program might have:
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
    fib_list: List[number] = List()

    # We will be memoizing (caching) the results
    # to speed up computation.
    memo: Dict[number, number] = Dict()

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

grug has an issue board that can be filtered to list [good first issues](https://github.com/orgs/grug-lang/projects/1/views/1?sliceBy%5Bvalue%5D=good+first+issue) or [planned issues](<https://github.com/orgs/grug-lang/projects/1/views/1?sliceBy%5Bvalue%5D=planned>).

## Links

### Implementations

grug implementations consist of three core components: API bindings, a grug parser frontend, and a backend execution engine.

These grug implementations by the community pass all 500+ official [grug tests](https://github.com/grug-lang/grug-tests), and include 'Hello, world!' examples:
- [grug-for-python](https://github.com/grug-lang/grug-for-python)
- [grug-for-c](https://github.com/grug-lang/grug-for-c) (don't use this yet, as it's very new)
- [grug-rs](https://github.com/grug-lang/grug-rs)

### Games

- [grug mod loader for Minecraft](https://github.com/grug-lang/grug-mod-loader-for-minecraft)
- [grug Box2D and raylib game](https://github.com/grug-lang/grug-box2d-and-raylib-game)
- [grug terminal game: C/C++](https://github.com/grug-lang/grug-terminal-game-c-cpp)
- [grug terminal game: Java](https://github.com/grug-lang/grug-terminal-game-java)

### Other

- [grug-vscode](https://github.com/grug-lang/grug-vscode)
- [grug-bench](https://github.com/grug-lang/grug-bench)
- [grug-discord-bot](https://github.com/grug-lang/grug-discord-bot)
