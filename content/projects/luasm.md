+++
date = "2026-03-09T13:38:18+01:00"
draft = false
title = "LuASM"
subtitle = "A library to parse and execute custom ASM."
license = "MIT"
version = "v0.1.0"

description = "LuASM is a simple library to parse and execute custom ASM written in Lua."

tags = ["lua", "assembly"]
+++


LuASM is a light‑weight Lua library that lets you define, parse and later execute a custom assembly‑like language.
And the library is designed to be deliberately minimal:

- No external dependencies – pure Lua 5.1+.
- Pluggable instruction set – you decide which mnemonics exist and how their operands are interpreted.

This means that you can design a machine for code like this:
```asm
mov rax, 1
print rax
```
and if executed, print the value `1`.

[You can find LuASM here](https://github.com/QuickWrite/luasm).

## What LuASM is and what it is not
With LuASM you can create your own custom assembly languages. But this does not mean that this is an assembler or a virtual machine.
This is an interpreter of custom assembly-like languages.

This means that LuASM **does not** assemble the content into bytecore nor does it execute bytecode.
It just parses the textfiles and executes them instruction by instruction.

### Why would this be useful?
This is mainly useful when you do not need complicated toolchains that actually assemble the code and then execute them.
These usecases mainly arise when you want to create toy languages in educational systems or video games. 
However this could also prove useful to just test out some assemblers without having the other overhead.

## Usage
To use the library, the library could either be installed usind luarocks or just by copying the `luasm.lua` file into the project.

After that, the library can easily be used to create custom ASMs:
```lua
local LuASM = require("luasm")

-- 1. Define the instruction set
local instructions = {
    LuASM.instruction("mov", {"imm", "reg"}, {}),
    LuASM.instruction("mov", {"reg", "reg"}, {}),
    LuASM.instruction("add", {"reg", "reg"}, {}),
    LuASM.instruction("jmp", {"label"}, {}),
}

-- 2. Create a runner (use default settings)
local asm = LuASM:new(instructions, {})

-- 3. Tokenize a source string
local src = [[
start:  mov 10 r0
        add  r0 r1
        jmp  start
]]
local tokenizer = asm:string_tokenizer(src)

-- 4. Parse
local result = asm:parse(tokenizer)

for i, instr in ipairs(result.instructions) do
    print(instr.op)   -- currently just the instruction name
end
```

which should result in something like this:
```text
mov
add
jmp
```

### Examples
There are some examples in the project in the [/examples](https://github.com/QuickWrite/luasm/tree/main/examples)-folder.
If you are interested, just take a look. :D
