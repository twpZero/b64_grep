# b64_grep

Attackers base64-encode their commands so your `grep` and SIEM searches find nothing. Decoding everything first is slow, and often impossible on terabytes of logs. Well, you don't need to decode: b64_grep gives you the regex that matches the **encoded** string directly.

```bash
$ python3 b64_grep_gen.py -W Invoke-Mimikatz
SQBuAHYAbwBrAGUALQBNAGkAbQBpAGsAYQB0A|kAbgB2AG8AawBlAC0ATQBpAG0AaQBrAGEAdAB6A|JAG4AdgBvAGsAZQAtAE0AaQBtAGkAawBhAHQAe
```

Paste that into grep, Splunk, KQL, an EDR console, anything that takes a regex.

<img width="616" alt="Simple use" src="https://github.com/twpZero/b64_grep/assets/49241247/c9fd540f-c18a-4626-93f7-786574b580a6">

## Install and run

No dependencies, Python 3 only:

```bash
git clone https://github.com/twpZero/b64_grep.git && python3 b64_grep/b64_grep_gen.py password
```

## Try it on sample data

`samples/4688_sample.csv` holds six process-creation events (Windows event 4688) with two PowerShell `-EncodedCommand` lines. The data is illustrative, built for this demo, and the payload is defanged (`example.invalid`).

```bash
$ grep -cE "$(python3 b64_grep_gen.py -W DownloadString)" samples/4688_sample.csv
1
$ grep -cE "$(python3 b64_grep_gen.py -W Get-Service)" samples/4688_sample.csv
1
$ grep -c "DownloadString" samples/4688_sample.csv     # plain grep is blind
0
```

## Usage

```bash
python3 b64_grep_gen.py password                  # UTF-8
python3 b64_grep_gen.py -W Invoke-Mimikatz        # PowerShell -EncodedCommand (UTF-16LE)
python3 b64_grep_gen.py -e latin-1 password       # any Python codec
cat iocs.txt | python3 b64_grep_gen.py -W         # one regex per input line
```

| Option | Meaning |
|---|---|
| `text` | Text to encode. Read from stdin, one line at a time, when omitted. |
| `encoding` / `-e ENCODING` | Any Python codec name (default `utf8`). |
| `-W` | Windows / PowerShell encoding. Same as `-e utf_16_le`. |

Double encoding:

<img width="1712" alt="Double encoding" src="https://github.com/twpZero/b64_grep/assets/49241247/41a1b03e-4e04-4d83-9d05-b6686d6d7cf5">

### As a library

```python
from b64_grep_gen import gen_b64_regex, alternative_b64encode

alternative_b64encode("password")            # ['cGFzc3dvc', 'Bhc3N3b', 'wYXNzd29yZ']
gen_b64_regex("Invoke-Mimikatz", "utf_16_le")
```

## Why not YARA's `base64` modifier?

YARA has `base64` and `base64wide`, and they're great where YARA runs: files and memory. Your logs live in a SIEM, your command lines in an EDR console, your proxy logs in a text file. YARA doesn't run there. A regex does.

## How it works

Base64 turns 3 input bytes into 4 output characters. So the encoding of a string depends on where it starts in the data: at byte offset 0, 1 or 2 of a 3-byte group.

```
abc     -> YWJj
zabc    -> emFiYw==      "abc" starts at offset 1
zzabc   -> enphYmM=      "abc" starts at offset 2
```

b64_grep encodes your string at all 3 offsets, then drops the characters at both ends that also depend on the surrounding bytes. What's left is stable whatever comes before or after, so one of the 3 values always appears in the encoded data. The shift is done in bytes, so multi-byte encodings like UTF-16 work too.

Resources:

* [Searching for content in base-64 strings](https://www.leeholmes.com/searching-for-content-in-base-64-strings/), Lee Holmes
* [Base64](https://en.wikipedia.org/wiki/Base64), Wikipedia
* [YARA base64 strings](https://yara.readthedocs.io/en/stable/writingrules.html#base64-strings)

## When NOT to use this

* **The string is short.** Under 4 or 5 characters, the trimmed values are 1 to 3 characters long and match almost everything (`foo` gives `Z|Zvb|mb`).
* **You don't know the encoding.** UTF-8 and UTF-16LE give completely different values. When in doubt, search for both.
* **The data is also compressed or encrypted.** Many droppers gzip before they base64. b64_grep only sees through one layer of base64.
* **You can decode cheaply.** If your pipeline already decodes base64 fields (some EDRs do), search the decoded field instead.

## Roadmap

* Ready-to-paste output for Sigma, Splunk SPL and KQL.

## Licence

```C
/*
 * ----------------------------------------------------------------------------
 * "THE BEER-WARE LICENSE" (Revision 42):
 * <TWP_Zero> wrote this file.  As long as you retain this notice you
 * can do whatever you want with this stuff. If we meet some day, and you think
 * this stuff is worth it, you can buy me a beer in return.
 * ----------------------------------------------------------------------------
 */
```

Encoded is not hidden.
