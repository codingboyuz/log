Log fayllaringizda zararli harakatlarni aniqlash uchun quyidagi bosqichlarni bajaring:

### 1. **Aniq belgilarni qidiring**
Siz `grep -E` bilan quyidagi shubhali so‘zlarni qidirmoqdasiz:
   - `sqlmap` – SQL injection hujumlari uchun ishlatiladigan dastur.
   - `nmap` – Tarmoqlarni skanerlash uchun ishlatiladigan vosita.
   - `curl`, `wget`, `python-requests` – Avtomatlashtirilgan so‘rovlar yuboruvchi vositalar.
   - `49.232.107.95` – Shubhali IP-manzil.

Shu kabi loglar chiqsa, bu hujum belgilari bo‘lishi mumkin:
```log
192.168.1.10 - - [26/Feb/2025:12:34:56 +0000] "GET /?id=1%20AND%201=1-- HTTP/1.1" 200 1234 "-" "sqlmap/1.4.11"
203.0.113.5 - - [26/Feb/2025:12:35:10 +0000] "GET /admin HTTP/1.1" 404 5678 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine)"
49.232.107.95 - - [26/Feb/2025:12:35:20 +0000] "GET / HTTP/1.1" 200 1024 "-" "curl/7.68.0"
```

### 2. **IP-manzillarni tahlil qiling**
Ko‘p marotaba takrorlanayotgan yoki notanish IP-manzillarni aniqlash uchun:
```bash
awk '{print $1}' access_log.processed.11 | sort | uniq -c | sort -nr | head
```
Bu eng ko‘p so‘rov yuborgan IP-larni ko‘rsatadi.

### 3. **SQL Injection harakatlarini tekshiring**
Agar SQL injeksiyaga urinishlar bo‘lsa, so‘rovlar quyidagicha bo‘lishi mumkin:
```bash
grep -E "UNION|SELECT|INSERT|UPDATE|DELETE|DROP|OR 1=1|--" access_log.processed.11
```

### 4. **Ko‘p takrorlanuvchi foydalanuvchi agentlarini aniqlash**
Hujumchilar ko‘pincha avtomatlashtirilgan vositalarni ishlatadi. Foydalanuvchi agentlarini tekshirish uchun:
```bash
awk -F'"' '{print $6}' access_log.processed.11 | sort | uniq -c | sort -nr | head
```
Agar `sqlmap`, `curl`, `wget` kabi agentlar ko‘p bo‘lsa, bu hujum belgisi bo‘lishi mumkin.

### 5. **Muayyan URL yoki skriptga ko‘p so‘rov tushayotganini tekshiring**
```bash
awk '{print $7}' access_log.processed.11 | sort | uniq -c | sort -nr | head
```
Agar shubhali sahifalarga juda ko‘p so‘rov bo‘lsa (masalan, `/admin`, `/wp-login.php`, `/etc/passwd`), hujum ehtimoli bor.

### 6. **Foydalanuvchi nomlari bilan urinishlarni tekshirish**
Brute-force hujumlarini aniqlash uchun:
```bash
grep "POST /login" access_log.processed.11 | awk '{print $1}' | sort | uniq -c | sort -nr | head
```
Agar bir xil IP dan juda ko‘p login urinishlari bo‘lsa, bu brute-force hujum bo‘lishi mumkin.

### 7. **Port skanerlash yoki DDOS hujumlarini tekshirish**
Ko‘p so‘rov yuborayotgan IP-larni aniqlash uchun:
```bash
awk '{print $1}' access_log.processed.11 | sort | uniq -c | sort -nr | head -20
```
Agar bitta IP juda ko‘p so‘rov yuborgan bo‘lsa, bu hujum bo‘lishi mumkin.

---

### **Xulosa**
Agar yuqoridagi usullarda shubhali faollik topsangiz:
- Shubhali IP-larni firewall yoki `.htaccess` orqali bloklang.
- Web server loglarini avtomatik tahlil qilish uchun `fail2ban` yoki `mod_security` qo‘shing.
- Qo‘shimcha tekshiruv uchun `logwatch` yoki `goaccess` kabi vositalardan foydalaning.

Agar xohlasangiz, aniq loglarni ko‘rib chiqishim uchun menga ba'zi namunalar yuboring.
