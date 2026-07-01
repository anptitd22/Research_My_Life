- [[#Hệ mật mã khóa đối xứng|Hệ mật mã khóa đối xứng]]
- [[#Hệ mật mã khóa bất đối xứng|Hệ mật mã khóa bất đối xứng]]

### Hệ mật mã khóa đối xứng

Là những hệ mật được sử dụng chung 1 khóa trong quá trình mã hóa và mã hóa. Do đó khóa phải được giữ bí mật tuyện đối.

Một số hệ mật mã khóa đối xứng hiện đại mà mình thấy hay được sử dụng là [DES](https://en.wikipedia.org/wiki/Data_Encryption_Standard), [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), [RC4](https://en.wikipedia.org/wiki/RC4), [RC5](https://en.wikipedia.org/wiki/RC5),...

**Hệ mật sẽ bao gồm:**

- Bản rõ (plaintext-M): bản tin được sinh ra bởi bên gửi
- Bản mật (ciphertext-C): bản tin che giấu thông tin của bản rõ, được gửi tới bên nhận qua một kênh không bí mật
- Khóa (Ks): nó là giá trị ngẫu nhiên và bí mật được chia sẻ giữa các bên trao đổi thông tin và được tạo ra từ:
    - Bên thứ 3 được tin cậy tạo và phân phối tới bên gửi và bên nhận
    - Hoặc, bên gửi tạo ra và chuyển cho bên nhận
- Mã hóa (encrypt-E): C = E(KS, M)
- Giải mã (decrypt): M = D(KS, C) = D(KS, E(KS, M))

**Cơ chế hoạt động**

- Người gửi sử dụng khóa chung (Ks) để mã hóa thông tin rồi gửi cho nguời nhận.
- Người nhận nhận được thông tin đó sẽ dùng chính khóa chung (Ks) để giải mã.

### Hệ mật mã khóa bất đối xứng

Ở hệ mật này thay vì nguời dùng dùng chung 1 khóa như ở hệ mật mã khóa đối xứng thì ở đây sẽ dùng 1 cặp khóa có tên là public key và private key.

Hệ mật mã khóa bất đối xứng mình thấy được dùng nhiều nhất là [RSA](https://en.wikipedia.org/wiki/RSA_\(cryptosystem\))

**Hệ mật sẽ bao gồm:**

- Bản rõ (plaintext-M): bản tin được sinh ra bởi bên gửi
- Bản mật (ciphertext-C): bản tin che giấu thông tin của bản rõ, được gửi tới bên nhận qua một kênh không bí mật
- Khóa: Bên nhận có 1 cặp khóa:
    - Khóa công khai (Kub) : công bố cho tất cả mọi người biết (kể cả hacker)
    - Khóa riêng (Krb) : bên nhận giữ bí mật, không chia sẻ cho bất kỳ ai
- Mã hóa (encrypt-E): C = E(Kub, M)
- Giải mã (decrypt): M = D(Krb, C) = D(Krb, E(Kub, M))

_Yêu cầu đối với cặp khóa (Kub, Krb) là:_

- Hoàn toàn ngẫu nhiên
- Có quan hệ về mặt toán học 1-1.
- Nếu chỉ có giá trị của Kub không thể tính được Krb.
- Krb phải được giữ mật hoàn toàn.

**Cơ chế hoạt động**

- Người gửi(A) gửi thông tin đã được mã hóa bằng khóa công khai (Kub) của người nhận(B) thông qua kênh truyền tin không bí mật
- Người nhận(B) nhận được thông tin đó sẽ giải mã bằng khóa riêng (Krb) của mình.
- Hacker cũng sẽ biết khóa công khai (Kub) của B tuy nhiên do không có khóa riêng (Krb) nên Hacker không thể xem được thông tin gửi
