# ПР №5-7. Розробка безпечних мережевих застосунків

> Це **поради**, а не інструкція; можливі неточності. Перевіряйте офіційну документацію.

## Компоненти
- **server/** FastAPI: TLS, bcrypt‑паролі, JWT (HS256), опційний TOTP (pyotp), rate‑limit, JSON‑логи.
- **client/** CLI на requests: echo http/https, login/JWT, MFA, brute‑force тест, бенчмарки.
- **tools/** analyze_logs.py — простий аналізатор логів.
- **.github/workflows/security.yml** — Bandit + pip‑audit.

## Швидкий старт
```bash
pip install -r requirements.txt

# Сертифікати
bash server/scripts/gencerts.sh

# Сервер (TLS)
uvicorn server.app:app --host 127.0.0.1 --port 8443 \
  --ssl-keyfile server/certs/server.key --ssl-certfile server/certs/server.crt

# Базовий HTTP echo (крок 2): інший процес без TLS
uvicorn server.app:app --host 127.0.0.1 --port 8080

# Клієнт: echo HTTP
python client/client.py echo --http --msg "hi"

# Клієнт: echo HTTPS з перевіркою CA
python client/client.py echo --https --ca server/certs/ca.crt --msg "hi"

# Логін і захищений запит
python client/client.py login --user alice --password secret --ca server/certs/ca.crt
python client/client.py whoami --token <JWT> --ca server/certs/ca.crt

# Підключити TOTP MFA
python client/client.py mfa-setup --user alice --password secret --ca server/certs/ca.crt
python client/client.py mfa-login --user alice --password secret --otp 123456 --ca server/certs/ca.crt

# Брутфорс/Rate-limit тест
python client/client.py brute --user alice --wrong 20 --ca server/certs/ca.crt

# Бенчмарки HTTP vs HTTPS
python client/bench.py --n 200 --https --ca server/certs/ca.crt
python client/bench.py --n 200
```

## Звіт: що додати
- Топологія вузлів і IP, порти.
- TLS параметри (версія, cipher, SAN) і результат верифікації на клієнті.
- Політика JWT (тривалість, алгоритм), сценарії 401/403/expired.
- MFA (секрет, URI, валідація OTP), ризики.
- Політика rate‑limit і результати тестів (429), вплив на UX.
- Витяги з логів та результат `tools/analyze_logs.py`.
- Висновки продуктивності HTTP vs HTTPS.
