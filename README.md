# spore-dob-0

DOB0 protocol aims to create a flexiable rendering process of the DNA bytes. It's the first implementation in DOB protocol family, so we name it with the number `ZERO`.

## Protocol detail
DOB0 protocol requires a parsing pattern that helps to define which part of DNA bytes indicates what trait name and which trait value to select from the traits pool.

DOB0 protocol requires DOB artist to pre-define a collection DNA traits pool, as the pattern, and each single or batch bytes in DNA will be recongnized an offset pointer that to indicate a specific trait item in the pool. The combination of all selected trait items is the final rendered DNA, for instance:

```javascript
// DNA bytes in Spore 
{
    contentType: "dob/0",
    content: {
        dna: "0xefc2866a311da5b6dfcdfc4e3c22d00d024a53217ebc33855eeab1068990ed9d"
    },
    // or content: "0xefc2866a311da5b6dfcdfc4e3c22d00d024a53217ebc33855eeab1068990ed9d",
    // or content: ["0xefc2866a311da5b6dfcdfc4e3c22d00d024a53217ebc33855eeab1068990ed9d"]
    content_id: "0x3b0e340b6c77d7b6e4f1fb2946d526ba65bfd196a27d9a7e5b6f06b82af5d07e"
}

// Pattern instance in Cluster
{
    name: "DOBs collection",
    description: {
        description: "Unicorn Collection",
        dobs: {
            decoder: {
                type: "code_hash"// or "type_id",
                hash: "0x4f441345deb88edb39228e46163a8f11ac7736376af8fe5e791e194038a3ec7b",
            },
            pattern: [
                [
                    "Face",
                    "string",
                    0,
                    1,
                    "options",
                    ["Laugh", "Smile", "Sad", "Angry"]
                ],
                [
                    "Age",
                    "number",
                    1,
                    1,
                    "range",
                    [0, 100]
                ],
                [
                    "BirthMonth",
                    "number",
                    2,
                    1,
                    "options",
                    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
                ],
                ...
            ]
        }
    }
}
```

`0xefc2866a311da5b6dfcdfc4e3c22d00d024a53217ebc33855eeab1068990ed9d` is the DNA bytes, which DOB0 decoder will parse one by one. `pattern: ...` is the pattern created by Cluster artist, which will be also parsed in DOB0 decoder in the meantime. In addition, the pattern is a JSON array, reference is here: https://docs.spore.pro/dob0/protocol#pattern-definition.

For real-world use case, this DOB0 decoder program is referenced by [decoder-template-rust](https://github.com/sporeprotocol/decoder-template-rust) and compiled into RISC-V binary. Then, we have two different methods to put it on-chain:
1. record the hash of binary on-chain, which refers to `code_hash`
2. deploy the binary into an on-chain CKB cell with `type_id` enabled, using its `type_script.args`

`type: "code_hash"` means the below `hash` is a CKB personalizied blake2b hash of DOB0 decoder RISC-V binary. To be contrast, `type: "type_id"` means the below `hash` is a `type_id` args value that points to an on-chain cell which keeps the DOB0 decoder RISC-V binary in ins `data` field.

## Diagram
![plot](./assets/DOB0.jpg)

## Run
Install `ckb-vm-runner`:
```sh
$ git clone https://github.com/nervosnetwork/ckb-vm
$ cargo install --path . --example ckb-vm-runner
```

For quick run:

```sh
$ cargo run-riscv -- ac7b88 "[[\"Name\",\"string\",0,1,\"options\",[\"Alice\",\"Bob\",\"Charlie\",\"David\",\"Ethan\",\"Florence\",\"Grace\",\"Helen\"]],[\"Age\",\"number\",1,1,\"range\",[0,100]],[\"Score\",\"number\",2,1,\"raw\"]]"

or

$ cargo build-riscv --release
$ ckb-vm-runner target/riscv64imac-unknown-none-elf/release/spore-dobs-decoder ac7b88 "[[\"Name\",\"string\",0,1,\"options\",[\"Alice\",\"Bob\",\"Charlie\",\"David\",\"Ethan\",\"Florence\",\"Grace\",\"Helen\"]],[\"Age\",\"number\",1,1,\"range\",[0,100]],[\"Score\",\"number\",2,1,\"raw\"]]"


"[{\"name\":\"Name\",\"traits\":[{\"String\":\"Ethan\"}]},{\"name\":\"Age\",\"traits\":[{\"Number\":23}]},{\"name\":\"Score\",\"traits\":[{\"Number\":136}]}]"
```

How to integrate:
1. install `ckb-vm-runner` into your back server natively
2. call `ckb-vm-runner` with the path of `spore-dob-0` binary, DNA and Pattern parameters in your server code (refer to above quick run)
3. parse the JSON traits result
