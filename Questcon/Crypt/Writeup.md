# Cryptography

## ENIGMA 65.58% (301 / 459)

Enigma Settings: Rotors: I, II, III Ring Settings: D, D, D Initial Rotor Positions: A, B, C Reflector: B Plugboard Connections: A ↔ G, B ↔ H

Ciphertext: ymnjp znmjo gteqj cjwwh qljtd nprmp g

以上の設定が与えられるので、[DenCode](https://dencode.com/ja/cipher/enigma)の解読ツールに設定すると、以下の文字列が出現

```
berli nhads ecret sbene athhi schar m
```

これを全て結合し、適切なところで区切りなおすと、以下のようになる

```
berlin had secrets beneath his charm
```

エニグマの問題らしい文章
これがFlagだった
