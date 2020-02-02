# dictword-test
This tool gets the prefix used for word lookup in the dictionary directly from libnickel. It works without any patches or conflicts by using a lot of LD_PRELOAD and dynamic linker hacks.

Japanese (Kanji) is not supported (it uses a different function, and is only called on the Japanese dictionaries anyways).

This was last tested on 4.19.14123, but it should continue to work for the forseeable future.

Theoretically, it should be possible to run this on a non-kobo ARM device (i.e. in QEMU), but this hasn't been tested. The biggest possible issue with doing this would be the reads to `/dev/ntx_io` and `/sys/class/power_supply/mc13892_bat/status`, but it should work fine without those (and it can be patched out if needed).

**To build:**

```sh
make NILUJE=/toolchain/arm-nickel-linux-gnueabihf DOCKER=podman
```

If you are using docker, set DOCKER=docker. If you are building directly on the host, leave it out entirely and set NILUJE accordingly. If you are using a custom toolchain/sysroot (which isn't one of NiLuJe's), set CROSS_PREFIX instead and set the Qt flags appropriately if not using pkg-config. In addition, a Go toolchain is required for dwt-wrap if building directly on the host.

**To run (on a Kobo):**

```sh
chmod +x dictword-test.so && LD_PRELOAD=./dictword-test.so ./dictword-test.so
```

This can be run over SSH, telnet, or anything else.

**Example output:**

```
[root@(none) geek1011]# LD_PRELOAD=./dictword-test.so ./dictword-test.so "test"
Storing arguments
Initializing QCoreApplication
Loading libnickel
Loading DictionaryParser::htmlForWord
Calling DictionaryParser::htmlForWord
Intercepting getHtml
/mnt/onboard/.kobo/dict/dicthtml.zip/te.html
```

```
[root@(none) geek1011]# LD_PRELOAD=./dictword-test.so ./dictword-test.so "é"
Storing arguments
Initializing QCoreApplication
Loading libnickel
Loading DictionaryParser::htmlForWord
Calling DictionaryParser::htmlForWord
Intercepting getHtml
/mnt/onboard/.kobo/dict/dicthtml.zip/éa.html
```

**To run dwt-wrap (on a Kobo):**

```sh
chmod +x dwt-wrap dictword-test.so && ./dwt-wrap "word1" "word2" "word3"
```

This can be run over SSH, telnet, or anything else. In addition, you can specify a single "-" to read words from stdin.

**Example output for dwt-wrap:**

```
[root@(none) geek1011]# ./dwt-wrap "test" "é" "éfgh" "1234" "-asd"
"é": "éa",
"test": "te",
"1234": "11",
"éfgh": "éf",
"-asd": "11",
```

```
[root@(none) geek1011]# echo -e "test\nasd\n" | ./dwt-wrap -
"asd": "as",
"test": "te",
```