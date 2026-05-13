#tool #must-know #crypto #lab #cmd

# 🔧 OpenSSL — практическое руководство для специалиста по ИБ

> [!abstract] Суть одной фразой
> OpenSSL — это криптографический инструментарий с открытым исходным кодом, реализующий протоколы [[TLS]] и [[SSL]], а также предоставляющий доступ к обширному набору криптографических функций из командной строки. Для специалиста по ИБ это «швейцарский нож» для работы с сертификатами, ключами, шифрованием и диагностикой защищённых соединений.

OpenSSL выполняет три главные функции для ИБ-специалиста:
1.  **Управление ключами и сертификатами:** создание [[RSA]], [[ECC]] ключей, запросов на подпись (CSR), самоподписанных сертификатов [[X.509]].
2.  **Криптографические операции:** [[Шифрование]], [[Хеширование]], создание и проверка [[Электронная подпись|электронной подписи]].
3.  **Диагностика TLS/SSL:** тестирование защищённых соединений, просмотр сертификатов серверов, проверка цепочек доверия [[Инфраструктура открытых ключей (PKI)]].

Знание этих команд необходимо для аттестации, пентеста и повседневного администрирования.

---

## 1. Работа с ключами

### 🔑 Генерация закрытого ключа RSA

> openssl genrsa -out private.key 2048

> openssl genrsa -aes256 -out private_encrypted.key 4096

- `2048` / `4096` — длина ключа. Для новых систем всегда используй **не менее 2048 бит** для RSA.

### 🔑 Генерация ключа ECC

> openssl ecparam -genkey -name prime256v1 -out private_ecc.key

- **ECC** предпочтительнее RSA для новых систем: быстрее и требует меньше ресурсов при том же уровне стойкости. Ключ 256 бит ECC ≈ 3072 бит RSA.

### 🔑 Извлечение открытого ключа

> openssl rsa -in private.key -pubout -out public.key

> openssl ec -in private_ecc.key -pubout -out public_ecc.key

---

## 2. Работа с сертификатами X.509 и PKI

### 📜 Создание самоподписанного сертификата

> openssl req -x509 -newkey rsa:4096 -keyout ca_key.pem -out ca_cert.pem -days 365 -nodes

> openssl x509 -in certificate.crt -text -noout

- `-x509` — указывает на создание самоподписанного сертификата.
- `-nodes` — **не шифровать** закрытый ключ (No DES). Для продакшена лучше убрать этот флаг.
- `-days 365` — срок действия сертификата.

### 📜 Создание сертификата, подписанного вашим УЦ (CA)

**Шаг 1: Создайте запрос на сертификат (CSR)**

> openssl req -new -newkey rsa:2048 -nodes -keyout server_key.pem -out server_req.csr

**Шаг 2: Подпишите запрос вашим корневым сертификатом (CA)**

> openssl x509 -req -in server_req.csr -CA ca_cert.pem -CAkey ca_key.pem -CAcreateserial -out server_cert.pem -days 365

### 📜 Проверка цепочки сертификатов

> openssl verify -CAfile ca_cert.pem server_cert.pem

---

## 3. Диагностика TLS/SSL-соединений

### 🌐 Подключение к серверу и просмотр его сертификата

> openssl s_client -connect example.com:443 -servername example.com -tls1_2

> openssl s_client -connect example.com:443 -showcerts

### 🌐 Проверка поддержки конкретных протоколов

> openssl s_client -connect example.com:443 -tls1_3

> openssl s_client -connect example.com:443 -ssl3

### 🌐 Проверка срока действия сертификата

> echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

---

## 4. Криптографические операции

### ✂️ Хеширование

> openssl dgst -sha256 file.txt

> echo -n "Hello World" | openssl dgst -sha256

### 🔐 Симметричное шифрование

> openssl enc -aes-256-cbc -pbkdf2 -in secret.txt -out secret.enc

> openssl enc -d -aes-256-cbc -pbkdf2 -in secret.enc -out decrypted.txt

### 🔐 Асимметричное шифрование

> openssl pkeyutl -encrypt -in plain.txt -pubin -inkey public.key -out encrypted.bin

> openssl pkeyutl -decrypt -in encrypted.bin -inkey private.key -out decrypted.txt

### 🖊️ Электронная подпись

> openssl dgst -sha256 -sign private.key -out signature.bin document.txt

> openssl dgst -sha256 -verify public.key -signature signature.bin document.txt

---

## 🧠 Значимость для меня как специалиста по ИБ

- Я использую OpenSSL как основной инструмент для создания и анализа сертификатов, генерации ключей и проведения криптографических операций.
- При аудите и пентесте я проверяю поддержку протоколов и шифров, валидность сертификатов и их цепочек с помощью команд `s_client` и `verify`.
- При настройке серверов я умею быстро сгенерировать самоподписанный сертификат для тестового стенда или сформировать корректный CSR для отправки в центр сертификации.
- Я всегда помню о безопасности: не передаю пароли к ключам в открытом виде, использую `-pbkdf2` для шифрования файлов и храню закрытые ключи в недоступном для посторонних месте.

## 💬 Ответ на собеседовании (кратко)

> «OpenSSL — это основной криптографический инструмент командной строки для специалиста по ИБ. С его помощью я генерирую RSA и ECC ключи, создаю и анализирую X.509 сертификаты, проверяю цепочки доверия, настраиваю TLS-соединения и выполняю криптографические операции — шифрование, хеширование, электронную подпись. Для диагностики серверов обязательно использую `s_client` с проверкой поддерживаемых протоколов, а для аудита — `verify` для валидации сертификатов».

## ❓ Самопроверка

1.  **Как сгенерировать ECC-ключ?**
    👉 `openssl ecparam -genkey -name prime256v1 -out private.key`

2.  **Как проверить цепочку сертификата?**
    👉 `openssl verify -CAfile root.pem cert.pem`

3.  **Как быстро проверить срок действия сертификата на сервере?**
    👉 `echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null | openssl x509 -noout -dates`

4.  **В чём отличие RSA от ECC и какой выбрать?**
    👉 ECC предпочтительнее: он быстрее и требует меньше ресурсов при том же уровне стойкости (ключ 256 бит ECC ≈ 3072 бит RSA).