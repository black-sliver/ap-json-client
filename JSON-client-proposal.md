# AP JSON Client definition

## Format

A json object with the following items

| key           | type     | description
| ------------- | -------- |------------
| formatVersion | `int`    | revision of this format specification, currently 0
| fileVersion   | `int`    | revision of the definition, 0 is draft, increment by 1 per release
| game          | `string` | game name as used in apbp
| world         | `string` | optional, world directory name as used in AP, for reference only
| platform      | `string` | defines possible bridges as well as integer byte order
| bridge        | `object` | declare supported versions of bridges (usb2snes/sni), see below
| identify      | `object` | declare memory to identify game, state, seed and slot, see below
| checks        | `object` | declare locations' collect flags, see below
| scouts        | `object` | declare locations' scout flags, see below
| items         | `array`  | declare items, see below
| send          | `object` | instructions to send events, see below

### bridge

Key is bridge name (i.e. `sni` or `qusb2snes`), value is minimum known good version as `[int,int,int]` or `null`.
If value is `null` then *no* version is known good. This should only be used if the game requires a specific feature,
not if the client simply does not support a bridge. That case should be handled by the client internally.

#### example
```jsonc
{
    "sni": [0,0,59] // error if sni<=0.0.58 is being used
}
```

### identify

identify is an object with `block name` -> `block` as defined below

known blocks are
* id (compare string, e.g. ROM header)
* version (compare int, e.g. basepatch version)
* seed
* slot
* ingame
* goal

#### block

| key     | type    | description
| ------- | ------- | -----------
| address | int     | start address of the block
| size    | int     | size of the block
| data    | varying | gameId, gameVersion: value; seed, slot: null; ingame, goal: data definition

#### data definition

read below, but a single data point `[offset, mask, [value...]]]`, not an array

### checks, scouts

checks and scouts are objects with `block name` -> `block` as defined below

#### block

| key     | type  | description
| ------- | ----- | -----------
| address | int   | start address of the block
| size    | int   | size of the block
| data    | array | data inside the block

#### data

data definition is an array of `[id, [offset, mask, [values...]]]`

A location `id` is collected/scouted `if (memory[offset] & mask) in values`.

Bytes/values are unsigned. If `mask < 2^8` compare 8bit, `mask < 2^16` 16bit, etc. Byte order depends on platform.
Special `values` of `[-1]` means any value `!= 0`.

#### example
```jsonc
{
    "alchemy": {
        "address": 0xf52570,
        "size": 5,
        "data": [
            [64000, [0, 1, 0x01, [0x01]]], // Acid Rain
            [64001, [0, 1, 0x02, [0x02]]]  // Atlas
            // ...
        ]
    },
    "bosses_1": {
        "address": 0xf5225f,
        "size": 2,
        "data": [
            [65050, [1, 1, 0x10, [0x10]]] // Thraxx
            // ...
        ]
    }
    // ...
}
```

Iterate over all blocks for items, reading `size` bytes from `address` and
on completion iterate over `data` checking each value.


### items

array of `[id, [ints...]]`

`ints...` depends on the game, for example it could be `amount, item_type, item_value`

### send

This needs more brainpower than available right now.

We need to be able to describe sending
* `item` including reading index, possibly supporting "last index" as well as "expected index"
* `scout`
* `chat`
* `death_link`

The writes have to use placeholders for
* `sender` (int, possibly ascii as well)
* `location` (int, possibly ascii as well)
* `index` (int, 0 for the first item)
* `next_index` (int, 1 for the first item)
* `item int 0...N` (as defined in items above)
* `message` (ascii, for chat)

Writes of strings have to define
* `max_length`

Writes of ints have to define
* `bits`


## Sample
```jsonc
{
    // TODO
}
