# WiFi QR Code Encoding/Decoding Algorithm

## Overview

The Aqara G3 hub uses a custom encoding scheme for WiFi credentials in QR codes. This encoding protects the SSID and password from casual observation while remaining simple enough to decode efficiently on embedded devices.

## How the Algorithm Works

The encoding algorithm transforms each character based on its ASCII value using different rules for different character ranges. The core principle is a simple mathematical transformation that makes the encoded text appear scrambled while being easily reversible.

### Standard Character Encoding (ASCII 36-126)

For most printable ASCII characters from `$` (36) to `~` (126), the algorithm uses a simple subtraction formula. Each character is encoded by subtracting its ASCII value from 162.

For example, let's encode the letter `G` which has ASCII value 71:
```
Encoded value = 162 - 71 = 91
Character 91 is '[' in ASCII
Therefore, 'G' encodes to '['
```

To decode, we simply reverse the operation:
```
Original value = 162 - 91 = 71
Character 71 is 'G' in ASCII
Therefore, '[' decodes to 'G'
```

### Special Characters (ASCII 32-35)

Characters in the range 32-35, which are space, exclamation mark, double quote, and hash symbol, receive special treatment. These characters are encoded by doubling them.

For example, encoding a space character (ASCII 32):
```
' ' encodes to '  ' (two spaces)
'!' encodes to '!!'
'"' encodes to '""'
'#' encodes to '##'
```

During decoding, when the decoder encounters two identical characters in this range, it outputs just one.

### Extended Character Ranges

For characters outside the standard printable ASCII range, the algorithm uses marker characters followed by an encoded value. The marker indicates which decoding formula to apply.

Characters with ASCII values 127-128 use the `#` marker:
```
Character value 127: Encoded as '#' + chr(165 - 127)
Character value 128: Encoded as '#' + chr(165 - 128)
```

Characters with ASCII values 129-208 use the `!` marker with a more complex formula involving bitwise operations.

Characters above 208 use the `"` marker, also with bitwise operations.

## Complete Example: Encoding "GL-S200-d0b"

Let's walk through encoding the SSID "GL-S200-d0b" step by step.

Starting with character `G`:
```
ASCII value of 'G' = 71
Since 71 is between 36 and 126, we use: 162 - 71 = 91
Character 91 = '['
```

Next character `L`:
```
ASCII value of 'L' = 76
162 - 76 = 86
Character 86 = 'V'
```

The hyphen `-`:
```
ASCII value of '-' = 45
162 - 45 = 117
Character 117 = 'u'
```

Character `S`:
```
ASCII value of 'S' = 83
162 - 83 = 79
Character 79 = 'O'
```

Character `2`:
```
ASCII value of '2' = 50
162 - 50 = 112
Character 112 = 'p'
```

Character `0`:
```
ASCII value of '0' = 48
162 - 48 = 114
Character 114 = 'r'
```

Second `0`:
```
ASCII value of '0' = 48
162 - 48 = 114
Character 114 = 'r'
```

Another hyphen `-`:
```
ASCII value of '-' = 45
162 - 45 = 117
Character 117 = 'u'
```

Character `d`:
```
ASCII value of 'd' = 100
162 - 100 = 62
Character 62 = '>'
```

Character `0`:
```
ASCII value of '0' = 48
162 - 48 = 114
Character 114 = 'r'
```

Character `b`:
```
ASCII value of 'b' = 98
162 - 98 = 64
Character 64 = '@'
```

Combining all encoded characters: `[VuOprru>r@`

## Decoding Example: "[VuOprru>r@" back to "GL-S200-d0b"

To decode, we reverse the process for each character.

Starting with `[`:
```
ASCII value of '[' = 91
Since 91 is between 36 and 126, we use: 162 - 91 = 71
Character 71 = 'G'
```

Character `V`:
```
ASCII value of 'V' = 86
162 - 86 = 76
Character 76 = 'L'
```

And continuing this process for each character gives us back the original SSID "GL-S200-d0b".

## Implementation in Code

The encoding function iterates through each character of the input string, determines which range it falls into, and applies the appropriate transformation. The decoding function recognizes the patterns in the encoded string and reverses the transformations.

This algorithm provides a balance between obscurity and simplicity, making it suitable for embedded systems where complex cryptographic operations might be too resource-intensive. However, it should be noted that this is merely obfuscation, not encryption, and provides no real security against anyone who understands the algorithm.

## QR Code Structure

The complete QR code data follows this format:
```
b=bindkey&d=domain&x=encoded_ssid&y=encoded_password&l=language
```

Where the SSID and password are encoded using the algorithm described above, while other parameters remain in plaintext. This allows the system to process network configuration data from QR codes while providing basic protection for sensitive credentials.
