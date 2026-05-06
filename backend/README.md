# Exercițiu tehnic — Backend Developer

**SoftPrim Technology SRL** — probă tehnică pentru stagiul de Backend Developer.

---

## 1. Ce trebuie livrat

Un **API REST funcțional** pentru un mic sistem de catalog produse și plasare comenzi, conectat la o bază de date MySQL.

API-ul trebuie să pună la dispoziție trei endpoint-uri (detaliate în secțiunea 4) și să poată fi pornit local. Veți include un `README.md` propriu cu instrucțiunile de pornire.

**Termen de livrare:** 3 zile calendaristice de la primirea acestui exercițiu.

---

## 2. Cum se livrează

Două variante acceptate, la alegere:

- **Link către un repo Git** (GitHub, GitLab, Bitbucket — public sau privat)
- **Arhivă ZIP** trimisă pe email, cu numele `Nume_Prenume_backend.zip`

Indiferent de modul ales, livrarea trebuie să conțină:

- Codul sursă al aplicației
- Un `README.md` propriu (vezi secțiunea 5)
- Fișierul `setup.sql` (îl puteți copia ca atare din acest director)
- Eventuale fișiere de configurare necesare (ex. `.env.example`, `composer.json`, `package.json`)

---

## 3. Tehnologii

Stack-ul tehnic este la alegerea voastră, în limitele de mai jos:

| Componentă | Variante acceptate |
|------------|--------------------|
| Limbaj backend | PHP 7.4+ sau Node.js 18+ |
| Bază de date | MySQL 5.7+ sau MariaDB 10.3+ |
| Format API | JSON, encoding UTF-8 |

Puteți folosi un framework (Laravel, Slim, Express, Fastify etc.) sau puteți scrie codul fără framework — alegerea vă aparține. La fel și pentru biblioteca de acces la baza de date.

### Configurare bază de date

În acest director găsiți fișierul `setup.sql` cu schema și datele necesare. Pașii pentru a-l importa:

```bash
mysql -u root -p -e "CREATE DATABASE softprim_test CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p softprim_test < setup.sql
```

În urma importului veți avea trei tabele (`categories`, `products`, `orders`) populate cu 5 categorii și 20 de produse reale.

---

## 4. Endpoint-uri solicitate

Toate endpoint-urile întorc răspunsuri în format JSON și folosesc coduri HTTP corespunzătoare situației (200, 201, 400, 404, 500).

### 4.1. `GET /api/products`

**Rol:** Întoarce lista produselor, fiecare cu denumirea categoriei alături (obținută din `categories` printr-un JOIN).

**Filtru opțional:** parametrul `category_id` din query string (întreg) filtrează rezultatele după categoria respectivă.

**Exemplu de răspuns — `200 OK`:**

```json
[
  {
    "id": 1,
    "name": "Siguranță automată 1P+N 16A curba C",
    "price": 45.50,
    "stock": 120,
    "category_id": 1,
    "category_name": "Întrerupătoare automate"
  }
]
```

**Situații de gestionat:**
- dacă `category_id` este valid dar nu are produse asociate → array gol `[]`
- dacă `category_id` este invalid (text, număr negativ etc.) → `400 Bad Request`

---

### 4.2. `GET /api/products/{id}`

**Rol:** Întoarce detaliile unui singur produs, împreună cu denumirea categoriei.

**Exemplu de răspuns — `200 OK`:**

```json
{
  "id": 1,
  "name": "Siguranță automată 1P+N 16A curba C",
  "price": 45.50,
  "stock": 120,
  "category_id": 1,
  "category_name": "Întrerupătoare automate",
  "created_at": "2026-04-15 12:34:56"
}
```

**Situații de gestionat:**
- dacă `id` nu există în baza de date → `404 Not Found`
- dacă `id` este invalid (text, număr negativ) → `400 Bad Request`

---

### 4.3. `POST /api/orders`

**Rol:** Creează o comandă nouă pentru un produs existent și scade cantitatea din stocul produsului.

**Body-ul cererii:**

```json
{
  "product_id": 5,
  "quantity": 3,
  "customer_email": "client@exemplu.ro"
}
```

**Validări așteptate:**

| Câmp | Reguli |
|------|--------|
| `product_id` | Întreg pozitiv. Produsul trebuie să existe. |
| `quantity` | Întreg `> 0` și `<=` stocul disponibil. |
| `customer_email` | Format de email valid, maxim 150 caractere. |

**Comportament așteptat:**

Crearea comenzii și scăderea stocului trebuie să se facă împreună, ca o singură operație unitară: dacă adăugarea rândului în `orders` nu reușește din orice motiv, stocul produsului rămâne neschimbat. Modul de implementare (tranzacție SQL, lock etc.) îl alegeți voi.

Totalul comenzii (`total = price × quantity`) se calculează pe server, folosind prețul din baza de date — nu se preia din cererea clientului.

**Exemplu de răspuns — `201 Created`:**

```json
{
  "order_id": 42,
  "product_id": 5,
  "quantity": 3,
  "total": 525.00,
  "created_at": "2026-05-06 14:22:10"
}
```

**Situații de gestionat:**
- produs inexistent → `404 Not Found`
- stoc insuficient sau validări nereușite → `400 Bad Request`, cu un răspuns JSON care explică problema

---

## 5. README-ul vostru

Includeți în livrare un `README.md` propriu care explică oricui cum să pornească aplicația de la zero. Conținutul minim:

1. **Tehnologiile folosite** (limbaj + versiune, framework dacă e cazul, MySQL/MariaDB versiune testată)
2. **Pași de instalare** — comenzile concrete pentru instalarea dependențelor
3. **Configurare** — cum se completează datele de conectare la MySQL (variabile de mediu, fișier de configurare etc.)
4. **Pornire server** — comanda de pornire și portul implicit
5. **Exemple de apel** — cel puțin o comandă `curl` (sau echivalentă) pentru fiecare dintre cele trei endpoint-uri, împreună cu răspunsul așteptat

Dacă vreți să justificați o decizie tehnică, faceți-o tot în acest README, într-o secțiune separată.

---

## 6. Ce nu se cere în acest exercițiu

Pentru a păstra exercițiul concentrat pe esențial, nu este nevoie să implementați:

- autentificare, autorizare sau sesiuni
- frontend, pagini HTML sau template-uri server-side
- sistem de caching
- deploy public
- teste automatizate (acceptate dacă există, dar nu obligatorii)

Dacă totuși alegeți să adăugați lucruri peste cerințele de mai sus (ex. un `Dockerfile`, teste, documentație OpenAPI), menționați-le în README.

---

## 7. Contact

Pentru întrebări despre cerințe, înainte de a începe lucrul: adresa de email din anunțul de stagiu.
