from flask import Flask, request, jsonify
import random
import string

app = Flask(__name__)

# Şifre oluşturma fonksiyonu
def generate_password(inputs):
    special_chars = "!@#$%^&*()-_=+"
    parts = []

    for value in inputs.values():
        if len(value) >= 3:
            parts.append(value[:3].capitalize())
        else:
            parts.append(value.capitalize())

    parts.append(str(random.randint(10, 99)))
    parts.append(random.choice(special_chars))
    random.shuffle(parts)
    password = ''.join(parts)
    return password

# Şifre analiz fonksiyonu
def analyze_password(password, inputs):
    issues = []
    score = 0

    if len(password) >= 12:
        score += 1
    else:
        issues.append("Şifre çok kısa.")

    if any(c.isupper() for c in password):
        score += 1
    else:
        issues.append("Büyük harf eksik.")

    if any(c.islower() for c in password):
        score += 1
    else:
        issues.append("Küçük harf eksik.")

    if any(c.isdigit() for c in password):
        score += 1
    else:
        issues.append("Rakam eksik.")

    if any(c in string.punctuation for c in password):
        score += 1
    else:
        issues.append("Özel karakter eksik.")

    common_words = [v.lower() for v in inputs.values()]
    if any(word in password.lower() for word in common_words):
        issues.append("Şifre tahmin edilebilir kelimeler içeriyor.")
        score -= 1

    is_secure = score >= 4
    return is_secure, issues

# API endpoint
@app.route('/generate-password', methods=['POST'])
def password_api():
    data = request.get_json()

    if not data or not isinstance(data, dict):
        return jsonify({"error": "Geçerli kategori-yanıt verileri JSON formatında gönderilmelidir."}), 400

    attempt = 1
    while attempt <= 10:
        password = generate_password(data)
        is_secure, issues = analyze_password(password, data)

        if is_secure:
            return jsonify({
                "password": password,
                "secure": True,
                "attempt": attempt,
                "issues": [],
                "message": "Şifre güvenli bulundu!"
            })
        else:
            attempt += 1

    return jsonify({
        "password": password,
        "secure": False,
        "attempt": attempt,
        "issues": issues,
        "message": "Güvenli şifre oluşturulamadı. Lütfen daha güçlü bilgiler sağlayın."
    })

if __name__ == '__main__':
    app.run(debug=True)
