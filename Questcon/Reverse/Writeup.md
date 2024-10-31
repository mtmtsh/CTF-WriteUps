# Reversing

## Twice the Trouble 66.45% (305 / 459)

以下のPythonファイルが配られた

```python
import math
import random

def xor_encrypt_decrypt(input_str, key):
    return ''.join(chr(ord(c) ^ key) for c in input_str)

def get_flag():
    # XOR-encoded flag
    encoded_flag = [92, 88, 72, 94, 89, 78, 66, 67, 118, 105, 61, 120, 111, 97, 62, 82, 121, 127, 61, 120, 111, 97, 62, 112]
    key = 13  # XOR key used for encoding
    flag = ''.join(chr(c ^ key) for c in encoded_flag)
    return flag

# Function to compare the magnitude of two numbers
def compare_numbers(num1, num2):
    if math.sqrt(num1**2) == 2 * abs(num2):
        print("The magnitude of the first number is exactly twice the magnitude of the second!")
        return True
    else:
        print("One of the numbers has a larger magnitude.")
        return False

def main():
    junk = [random.randint(1, 100) for _ in range(10)]

    try:
        num1 = float(input("Enter the first number: "))
        num2 = float(input("Enter the second number: "))

        # Compare numbers and reveal flag if the condition is met
        if compare_numbers(num1, num2):
            print(f"Congratulations! Here's the flag: {get_flag()}")
        else:
            print("Try again with different numbers.")

    except ValueError:
        print("Please enter valid numbers.")

if __name__ == "__main__":
    main()
```

2入力が`compare_numbers`関数で定義された条件を満たせば`get_flag`が呼び出される仕組み

1. `get_flag`を強制的に呼び出すように書き換え
2. `compare_numbers`を満たす入力を与える

の2パターンの解法が浮かんだ
1がお手軽だが、`compare_numbers`がそれほど複雑ではないので2の解法で解いた

`compare_numbers`によると、1つめの入力の２乗の平方根(つまり絶対値)が2つめの入力の2倍の絶対値になっていればよいことがわかるので、以下のように入力してFlagを得た

```
$ python3 magnitude.py
Enter the first number: 2
Enter the second number: 1
The magnitude of the first number is exactly twice the magnitude of the second!
Congratulations! Here's the flag: QUESTCON{d0ubl3_tr0ubl3}
```

## Unlock the Encrypted Flag

以下のpythonファイルと、`flag.txt.enc`が渡された

```python
### THIS FUNCTION WILL NOT HELP YOU FIND THE FLAG --LT ########################
def str_xor(secret, key):
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c, new_key_c) in zip(secret, new_key)])
###############################################################################

flag_enc = open('flag.txt.enc', 'rb').read()

def level_1_pw_check():
    user_pw = input("Please enter correct password for flag: ")

    pw_parts = ["ak98", "-=90", "adfjhgj321", "sleuth9000"]
    obfuscated_pw = "".join(pw_parts)

    if (user_pw.startswith("ak") and user_pw.endswith("9000")) and \
       (len(user_pw) == len(obfuscated_pw)):
        print("Welcome back... your flag, user:")
        decryption = str_xor(flag_enc.decode(), "utilitarian")
        print(decryption)
        return
    print("That password is incorrect")

level_1_pw_check()
```

先ほどと同様に、ユーザによる標準入力をチェックする`level_1_pw_check`があり、条件が満たされれば、`decryption = str_xor(flag_enc.decode(), "utilitarian")`によって`flag.txt.enc`が復号される
解法は先ほどと同様に2パターンある

1. `decryption = str_xor(flag_enc.decode(), "utilitarian")`を呼び出すように書きかえ
2. 条件を満たす入力をあたえる

2のパターンで解いた。「入力が`ak`で始まり、`9000`で終わる」かつ「入力文字列の長さが`obfuscated_pw`と等しい」を満たす入力であればよいので、そのような入力を与えるとFlagが得られた

## Unlock the Safe 64.49% (296 / 459)

以下のようなjavaファイルが渡された

```java
import java.io.*;
import java.util.*;

public class safeopener {
    public static void main(String args[]) throws IOException {
        BufferedReader keyboard = new BufferedReader(new InputStreamReader(System.in));
        Base64.Encoder encoder = Base64.getEncoder();
        String encodedkey = "";
        String key = "";
        int i = 0;
        boolean isOpen;

        while (i < 3) {
            System.out.print("Enter password for the safe: ");
            key = keyboard.readLine();

            encodedkey = encoder.encodeToString(key.getBytes());
            System.out.println(encodedkey);

            isOpen = openSafe(encodedkey);
            if (!isOpen) {
                System.out.println("You have  " + (2 - i) + " attempt(s) left");
                i++;
                continue;
            }
            break;
        }
    }

    public static boolean openSafe(String password) {
        String encodedkey = "cGwzYXMzX2wzdF9tM18xbnQwX3RoM19zYWYz"; // This is the encoded password

        if (password.equals(encodedkey)) {
            System.out.println("Sesame open");
            return true;
        } else {
            System.out.println("Password is incorrect\n");
            return false;
        }
    }
}
```

ユーザからの入力を受け取り、BAse64した結果を`encodedkey`に格納し、`openSafe(encodedkey)`関数を呼び出すことで`encodedkey`と`cGwzYXMzX2wzdF9tM18xbnQwX3RoM19zYWYz`を比較し、正しい場合に鍵が開くことが分かる

つまり、`cGwzYXMzX2wzdF9tM18xbnQwX3RoM19zYWYz`をBase64デコードすれば、金庫の鍵が得られる

これを実行するとFlagが得られた
