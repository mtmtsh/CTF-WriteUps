# Programming

## Missing Pieces 25.34% (321 / 1267)

渡されたソースファイル`miss.c`の標準出力部分が以下のように欠けていた

```c
// Print the result as a string
    printf("Resulting string: ");
    for (size_t i = 0; i < flag_length; ++i) {
        printf("%c");
    }
```

`print("%c")`の部分を、`print("%c", flag[i])`と書き換えることでFlagが出力

## Winning Factors 26.68 % (338 / 1267)

`147.182.245.126:33001`という文字列が与えられたので、`nc 147.182.245.126 33001`を実行

```
$ nc 147.182.245.126 33001
Calculate the factorial of 162.
Too slow!
```

2行目でランダムな数の階乗を計算する命令が出され、その3秒後には3行目の`Too slow!`が出力されてサーバとの通信が切れる

階乗を計算するプログラムを書いて、その出力を手動で入力する方式を試したが、3秒以内に(手動入力が)間に合わなかった

そこで、

1. リモートサーバへの接続出力の取得
2. 階乗計算
3. サーバへのデータ入力
4. その返答の取得

この一連の流れを自動的に行うPythonのプログラムを書いた

```python
import subprocess
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n - 1)

proc = subprocess.Popen(['nc', '147.182.245.126','33001'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)

output = proc.stdout.readline().decode('utf-8')

print(output)

n = int(output.split(' ')[4].replace('.', ''))

ans = factorial(n)

# ansとEnter を送信
proc.stdin.write(str(ans).encode('utf-8'))
proc.stdin.flush()

print("test:")

# 結果を取得
stdout, stderr = proc.communicate()
output = stdout.decode('utf-8')
print("output:",output)

proc.terminate()
```

これで3秒以内の入力が間に合い、Flagが得られた

## The Silver Swan 8.68 % (110 / 1267)

Flagが何らかの形でencodeされたファイル`the_silver_swan.enc`と、そのencodeとdecodeの方法が書かれたperlのプログラム`the_silver_swan.pl`が与えられた
`the_silver_swan.pl`にはところどころ`# TODO`のコメントアウトがされており、作りかけであることが分かる
これをひとつずつ指示に従って埋めていくと、Flagが得られた

perlのプログラムの文法に慣れていなかったので、今回はGitHub CopilotにPythonへのTranslateをしてもらってから問題に取り組んだ
以下が作成したプログラム

```python
import re

# Define rol function: rotate left
def rol(num, rotations):
    return ((num << rotations) & 0xFF) | (num >> (8 - rotations))

# Define ror function: rotate right
def ror(num, rotations):
    return (num >> rotations) | ((num << (8 - rotations)) & 0xFF)

# Poem text
poem = """
    The silver Swan, who, living, had no Note,
    when Death approached, unlocked her silent throat.
    Leaning her breast against the reedy shore,
    thus sang her first and last, and sang no more:
    "Farewell, all joys! O Death, come close mine eyes!
     More Geese than Swans now live, more Fools than Wise."
"""

# Split poem into tokens
tokens = re.split(r'\s+', poem.strip())

# Define search patterns
m = 'Sewell'
m = re.sub(r'^..', '', m)

myt1 = myt2 = myt3 = myt4 = ''

for token in tokens:
    # Remove punctuation
    token = re.sub(r'[^a-zA-Z]', '', token)

    # Select the Mighty Ones for AMHO
    if re.search(m, token):
        myt1 = token
    if re.search(r'..a.h', token):
        myt2 = token
    if re.search(myt1, token):
        myt3 = token
    if re.search(r'F.{4}', token):
        myt4 = token

# Construct the amho string
amho = f"{myt1}, {myt2}; {myt3}, {myt4}!"
amho_len = len(amho)

# Create dictionary for bits count
amho_bits = {}

# Loop through each character in the amho string
for char in amho:
    if char in amho_bits:
        continue

    amho_bits[char] = sum(1 for j in range(8) if ord(char) & (1 << j))

# Encryption/Decryption flag
encrypt = 0

if encrypt:
    # Read the binary file "the_silver_swan.txt"
    try:
        with open("the_silver_swan.txt", "rb") as fh:
            contents = b""
            key_idx = 0

            while True:
                line = fh.read(301)
                if not line:
                    break

                line2 = bytearray()
                for i in range(len(line)):
                    dec_char = line[i]

                    # Get how many bits to rotate
                    rotate_bits = amho_bits[amho[key_idx % len(amho)]]

                    # Rotate to the left
                    enc_char = rol(dec_char, rotate_bits)

                    line2.append(enc_char)
                    key_idx += 1

                contents += line2

            # Output encrypted content
            print(contents.decode('utf-8', errors='replace'))

    except Exception as e:
        print(f"Error: {e}")

else:  # Decrypting
    # Read the binary file "the_silver_swan.enc"
    try:
        with open("the_silver_swan.enc", "rb") as fh:
            contents = ""
            key_idx = 0

            while True:
                line = fh.read(301)
                if not line:
                    break

                line2 = ""
                for enc_char in line:
                    rotate_bits = amho_bits[amho[key_idx % len(amho)]]

                    # Rotate to the right
                    dec_char = ror(enc_char, rotate_bits)

                    line2 += chr(dec_char)
                    key_idx += 1

                contents += line2

            # Output decrypted content
            print(contents)

    except Exception as e:
        print(f"Error: {e}")
```

これを`python`で実行すると以下のように出力し、大量の不規則なテキストの中にFlagが埋め込まれていた

```
lclllloooodddddddddxxxxxxxdddddxxxxxxxxxxxxxxxxxxxxxdddxxxxxxxddoooddxdddxdddxxxxddxxxxxooodxxxxxxddxxxxdoxO0OOKXK00O0XKKKKKKKKXXKKXKK0OO0KKKkxOOOkdddxkOOkkxddkxxddxOKKKOkxkk0KKXXOxxxkxxxddxOOO0KKKKKKKKKKXKKKK0kxOKXKKKXKKKKK0OxoxKXKKK0klkXK0OkkkOOO0K0kdxxxxxxOKKKKKK0OOkkddxdod0KKKOdoodook0kddc:ccoxx
llloolllllllooooooodxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxdddxxxxxxxddddddxxxxx     you cannot say, flag{But I was told by others to do thus... Or that... Virtue was not convenient at the time.} This will not suffice. Remember that.    0XKKKKOdd0KOkxkO000KKK0kOK0KXXKKKKKK0OkxxxxkxxxdokKKOxdodkkxO0kdololldxx
lodxxddoolllllllooddddxxxxxxxxxxxxxxxxxxxxxxxxxxdddxxxxxxxdddddddxxxxxxddxxxxxddxxxxddoodxxxxxxdddxxxxxxld0K0O0KK0000KKK0OO0KK00000KOkkO00kkk00OkkOOOkkkkxodkkkkOkkkxkKKKKK000OOOkxkOkxk0K0OOKXXXXKKK0OO0000000000OOOO00OkxkOKKK0dd0KKKKKOxdkkkO00KKKKKKKKkx0K0KKKKKKKKKKKKKK0Oxlldddk0KOkdooxdok0Oxddolod
```
