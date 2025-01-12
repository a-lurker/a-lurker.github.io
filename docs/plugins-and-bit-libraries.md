# Plugins and bit libaries

Lua 5.1 used by openLuup and Vera does not incorporate bitwise operators. Noting that Lua 5.2 and above do have native bitwise operators. There is broad ranging discussion on bit libraries for Lua [here](http://lua-users.org/wiki/BitwiseOperators).

The lack of a bitwise library can be a problem, as some plugins make use of bit operations. Here are some solutions:

## nixio library
OpenWrt based devices and hence Vera devices, already have the [nixio library](https://openwrt.github.io/luci/api/modules/nixio.bit.html) installed.

The nixio library in a Vera Edge will be found in:
```text
/rom/usr/lib/lua/nixio.so
/usr/lib/lua/nixio.so
```
And you can see what it offers by executing this code in the test code box:
```lua
local bit = require("bit")
print(pretty(bit))
```
Resulting in:
```lua
{
    arshift = function: 0x1791028,
    band = function: 0x1790fc8,
    bits = 52,
    bnot = function: 0x1790f38,
    bor = function: 0x1790f18,
    bswap = function: 0x17911f0,
    bxor = function: 0x1791090,
    cast = function: 0x1791170,
    check = function: 0x1791130,
    div = function: 0x1791110,
    lshift = function: 0x1791068,
    max = 4.5035996273705e+15,
    rshift = function: 0x1790fa0,
    set = function: 0x1790f60,
    tobit = function: 0x17911b0,
    unset = function: 0x17910d0
}
```
## bit32 library
The Lua 5.2 bit library has been back ported to suit Lua 5.1 as "bit32". You can read about its [operators](https://www.lua.org/manual/5.2/manual.html#6.7) with [more info here](https://luarocks.org/modules/siffiejoe/bit32).
```bash
sudo luarocks install bit32
```
The operators are as follows:
```txt
bit32.arshift (x, disp)
bit32.band (···)
bit32.bnot (x)
bit32.bor (···)
bit32.btest (···)
bit32.bxor (···)
bit32.extract (n, field [, width])
bit32.replace (n, v, field [, width])
bit32.lrotate (x, disp)
bit32.lshift (x, disp)
bit32.rrotate (x, disp)
bit32.rshift (x, disp)
```
## BitOP library
Other Linux like devices, such as Raspberry Pis can also use [Lua BitOP](http://bitop.luajit.org/). Install it like so:
```bash
sudo apt install lua-bitop
```
## Lua code
As opposed to installing libraries like the above, you can use (or distribute) some Lua code in a file like [this example](https://github.com/kengonakajima/lua-msgpack/blob/master/luabit.lua). Place the file in the main luup engine directory:
```text
/etc/cmh-ludl/luabit.lua
```
This approach gives you the opportunity to easily remap the operator names. So in the above Lua code you could change the library interface to the following:
```lua
local bit = {
 -- bit operations
 bnot = bit_not,
 band = bit_and,
 bor  = bit_or,
 bxor = bit_xor,
 rshift = bit_rshift,
 lshift = bit_lshift,
 bxor2 = bit_xor2,
 blogic_rshift = bit_logic_rshift,

 -- utility func
 tobits = to_bits,
 tonumb = tbl_to_number,
}
```
