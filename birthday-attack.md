## 誕生日攻撃のCTF問題

この問題は、誕生日攻撃（Birthday Attack）の原理を理解し、実際に攻撃を成功させることを目的としています。

-----

### 問題名：プレゼントは早い者勝ち！

#### 問題文

オンラインショップ「Giftopia」で、先着100名様限定の特別なクーポンが配布されています。
このクーポンは、あなたのメールアドレスのハッシュ値と一致するハッシュ値を持つ人が、すでにクーポンを獲得している場合に限り、あなたも同じクーポンを獲得できるという、少し変わったルールで運用されています。

具体的には、以下の条件を満たせばクーポンを獲得できます。

1.  あなたが入力したメールアドレスをSHA-256でハッシュ化する。
2.  そのハッシュ値の**下位6桁**が、すでにクーポンを獲得した誰かのメールアドレスのハッシュ値の**下位6桁**と一致する。

このショップには、すでに多くのユーザーが登録しています。
あなたの目的は、誰かと下位6桁のハッシュ値が一致するメールアドレスをできるだけ早く見つけ出し、クーポンを獲得することです。

#### 課題

以下のPythonコードを使って、クーポンを獲得できるメールアドレスを見つけてください。

```python
import hashlib
import random
import string

def generate_random_email():
    """ランダムな文字列のメールアドレスを生成する"""
    chars = string.ascii_lowercase + string.digits
    username = ''.join(random.choice(chars) for _ in range(10))
    domain = "example.com"
    return f"{username}@{domain}"

def get_hash_suffix(email):
    """メールアドレスのSHA-256ハッシュの下位6桁を返す"""
    sha256 = hashlib.sha256()
    sha256.update(email.encode('utf-8'))
    full_hash = sha256.hexdigest()
    return full_hash[-6:]

def check_for_collision(email_address, existing_hashes):
    """ハッシュ値の下位6桁が既存のハッシュと衝突するかチェックする"""
    current_suffix = get_hash_suffix(email_address)
    if current_suffix in existing_hashes:
        return True, current_suffix
    return False, None

# --- ここからがあなたの課題 ---

# 既にクーポンを獲得したユーザーのハッシュ値のデータベース（架空）
# このリストのユーザー数は実際にはもっと多いと仮定してください。
# このデータを使って、あなたのメールアドレスと衝突するかどうかを検証します。
existing_hashes = {
    '2a1b3f', '7d4e5a', '9c2f8b', '1e6d9a', '3b7c2d', 
    '5f8a1c', '8d2e3f', 'a7b1c3', 'b9e4f5', 'c2d3e4'
    # ...実際にはもっと多くのハッシュが格納されている
}

# 以下のコードを完成させて、ハッシュの衝突を検知してください。
# ヒント：誕生日攻撃の原理を応用し、効率的に衝突を見つける方法を考えてみましょう。

# collision_found = False
# while not collision_found:
#     # ここにあなたのコードを記述してください
#     # 1. ランダムなメールアドレスを生成する
#     # 2. そのハッシュ値の下位6桁を計算する
#     # 3. existing_hashesと衝突するかチェックする
#     # 4. 衝突したらループを終了する
#     pass # この行を削除してコードを記述してください

# print("衝突するハッシュが見つかりました！")
# print(f"生成したメールアドレス: {found_email}")
# print(f"見つかったハッシュの下位6桁: {found_suffix}")
# print(f"既存のハッシュの下位6桁: {found_suffix}")
# print("おめでとうございます！クーポンを獲得しました！")
```

#### 解答のヒント

  * 誕生日攻撃は、あるハッシュ値と一致するものを探すのではなく、**2つの異なる入力が同じハッシュ値（の一部）になる**ことを利用する攻撃です。
  * この問題では、**誰かとハッシュの下位6桁が一致**すればよいので、これはまさに誕生日攻撃のシナリオです。
  * ハッシュ値の下位6桁は、$16^6 = 16,777,216$ 通りの可能性があります。
  * しかし、誕生日攻撃の原理を使うと、ランダムに生成した値を約 $\\sqrt{N}$ 回試行すれば衝突が見つかる可能性が高くなります。この問題では、約 $\\sqrt{16,777,216} \\approx 4,096$ 回の試行で衝突が発生する可能性が高いことを示唆しています。
  * コードのループ内で、効率よく衝突を見つけるためのロジックを実装しましょう。

-----

### 解答コード例

```python
import hashlib
import random
import string

def generate_random_email():
    """ランダムな文字列のメールアドレスを生成する"""
    chars = string.ascii_lowercase + string.digits
    username = ''.join(random.choice(chars) for _ in range(10))
    domain = "example.com"
    return f"{username}@{domain}"

def get_hash_suffix(email):
    """メールアドレスのSHA-256ハッシュの下位6桁を返す"""
    sha256 = hashlib.sha256()
    sha256.update(email.encode('utf-8'))
    full_hash = sha256.hexdigest()
    return full_hash[-6:]

def check_for_collision(email_address, existing_hashes):
    """ハッシュ値の下位6桁が既存のハッシュと衝突するかチェックする"""
    current_suffix = get_hash_suffix(email_address)
    if current_suffix in existing_hashes:
        return True, current_suffix
    return False, None

# 既にクーポンを獲得したユーザーのハッシュ値のデータベース（架空）
existing_hashes = {
    '2a1b3f', '7d4e5a', '9c2f8b', '1e6d9a', '3b7c2d', 
    '5f8a1c', '8d2e3f', 'a7b1c3', 'b9e4f5', 'c2d3e4'
}

print("クーポン獲得のためのメールアドレスを探索中...")

collision_found = False
found_email = None
found_suffix = None

# 誕生日攻撃の原理を応用し、衝突が見つかるまで試行を繰り返す
while not collision_found:
    # 1. ランダムなメールアドレスを生成する
    new_email = generate_random_email()

    # 2. そのハッシュ値の下位6桁を計算する
    is_collision, suffix = check_for_collision(new_email, existing_hashes)

    # 3. 衝突したかチェック
    if is_collision:
        collision_found = True
        found_email = new_email
        found_suffix = suffix
        
print("---")
print("衝突するハッシュが見つかりました！")
print(f"生成したメールアドレス: {found_email}")
print(f"見つかったハッシュの下位6桁: {found_suffix}")
print(f"既存のハッシュの下位6桁: {found_suffix}")
print("おめでとうございます！クーポンを獲得しました！")

```

#### 重要なポイント

  * このCTF問題では、**ハッシュ値の下位6桁**という非常に限定的な条件にすることで、誕生日攻撃が現実的な時間で成功するように設定しています。
  * 現実世界でSHA-256の完全なハッシュ衝突を見つけることは、現在の計算能力では事実上不可能です。しかし、このようにハッシュ値の一部を使うことで、攻撃の成功確率を飛躍的に上げることができます。
  * この問題を通じて、ハッシュ関数の一部だけを使用したり、出力長を短くしたりすることのセキュリティ上のリスクを学ぶことができます。
