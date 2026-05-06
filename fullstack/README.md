# Exercițiu tehnic — Full-Stack Web Developer

**SoftPrim Technology SRL** — probă tehnică pentru stagiul de Full-Stack Web Developer.

---

## 1. Ce trebuie livrat

O **mini-aplicație web funcțională** pentru afișarea unui catalog de produse, formată din două componente:

- Un **API REST** conectat la MySQL, cu două endpoint-uri (detaliate în secțiunea 4)
- O **interfață web** care afișează produsele și permite filtrarea după categorie (detaliată în secțiunea 5)

Backend-ul și frontend-ul rulează ca procese sau pagini independente, iar comunicarea între ele se face exclusiv prin apeluri HTTP.

Aplicația trebuie să poată fi pornită local. Veți include un `README.md` propriu cu instrucțiunile de pornire.

**Termen de livrare:** 3 zile calendaristice de la primirea acestui exercițiu.

---

## 2. Cum se livrează

Două variante acceptate, la alegere:

- **Link către un repo Git** (GitHub, GitLab, Bitbucket — public sau privat)
- **Arhivă ZIP** trimisă pe email, cu numele `Nume_Prenume_fullstack.zip`

Indiferent de modul ales, livrarea trebuie să conțină:

- Codul sursă complet (backend + frontend, organizate cum considerați necesar — preferabil în foldere separate)
- Un `README.md` propriu (vezi secțiunea 6)
- Fișierul `setup.sql` (îl puteți copia ca atare din acest director)
- Eventuale fișiere de configurare necesare (ex. `.env.example`, `composer.json`, `package.json`)

---

## 3. Tehnologii

Stack-ul tehnic este la alegerea voastră, în limitele de mai jos:

| Componentă | Variante acceptate |
|------------|--------------------|
| Limbaj backend | PHP 7.4+ sau Node.js 18+ |
| Bază de date | MySQL 5.7+ sau MariaDB 10.3+ |
| Frontend | HTML + CSS + JavaScript vanilla, sau React 18+, sau Vue 3 |
| Format API | JSON, encoding UTF-8 |

Folosirea unui framework backend (Laravel, Slim, Express, Fastify etc.), a unui framework de styling (Tailwind, Bootstrap sau CSS scris de mână) sau a unui build tool (Vite, Webpack, ori livrare ca `index.html` static) sunt decizii care vă aparțin.

### Configurare bază de date

În acest director găsiți fișierul `setup.sql` cu schema și datele necesare. Pașii pentru a-l importa:

```bash
mysql -u root -p -e "CREATE DATABASE softprim_test CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p softprim_test < setup.sql
```

În urma importului veți avea trei tabele (`categories`, `products`, `orders`) populate cu 5 categorii și 20 de produse reale. Pentru acest exercițiu folosiți doar `categories` și `products`.

---

## 4. Backend — Endpoint-uri solicitate

Ambele endpoint-uri întorc răspunsuri în format JSON, cu coduri HTTP corespunzătoare situației, și au antetele CORS configurate astfel încât să poată fi apelate de pe origin-ul pe care rulează frontend-ul în dezvoltare.

### 4.1. `GET /api/categories`

**Rol:** Întoarce toate categoriile, ordonate alfabetic după nume. Folosit de frontend pentru popularea filtrului.

**Exemplu de răspuns — `200 OK`:**

```json
[
  { "id": 5, "name": "Conectori și accesorii",   "slug": "conectori-accesorii" },
  { "id": 2, "name": "Contoare electrice",       "slug": "contoare-electrice" },
  { "id": 1, "name": "Întrerupătoare automate",  "slug": "intrerupatoare-automate" }
]
```

---

### 4.2. `GET /api/products`

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

## 5. Frontend — Interfață web

O singură pagină (statică, SPA sau cu randare pe server — la alegerea voastră) care îndeplinește următoarele:

### 5.1. Listare produse

- La încărcarea inițială se afișează toate produsele, sub formă de grid sau listă (la alegerea voastră).
- Pentru fiecare produs sunt vizibile: numele, denumirea categoriei, prețul (formatat cu 2 zecimale și sufix `RON`) și stocul disponibil.
- Produsele cu stoc 0 se afișează într-o formă vizual diferită (text estompat, badge "Stoc epuizat" sau alt indiciu vizual la alegere).

### 5.2. Filtrare după categorie

- Un control de selecție (dropdown, butoane sau tab-uri — la alegere) populat dinamic prin apel la `GET /api/categories`.
- O opțiune implicită "Toate categoriile" care afișează tot catalogul.
- Schimbarea selecției declanșează re-filtrarea listei prin apel la `GET /api/products?category_id=X`.

### 5.3. Stări UI

| Stare | Comportament așteptat |
|-------|-----------------------|
| **Încărcare** | La pornirea inițială și la fiecare schimbare de filtru, utilizatorul vede un indiciu vizual că datele se încarcă (spinner, skeleton, mesaj — la alegere). |
| **Eroare** | Dacă un apel API nu reușește, utilizatorul vede un mesaj clar — nu doar o eroare în consolă. Există posibilitatea de a reîncerca. |
| **Listă goală** | Dacă filtrul curent nu întoarce niciun produs, se afișează un mesaj explicit ("Nu sunt produse în această categorie" sau ceva similar). |

### 5.4. Responsive

Aplicația trebuie să fie utilizabilă și pe mobil (lățime ~375px), fără scroll orizontal și fără elemente care se rup vizual.

---

## 6. README-ul vostru

Includeți în livrare un `README.md` propriu care explică oricui cum să pornească aplicația de la zero. Conținutul minim:

1. **Tehnologiile folosite** (limbaj + versiune backend, framework frontend dacă e cazul, MySQL/MariaDB versiune testată)
2. **Pași de instalare** — comenzile concrete pentru instalarea dependențelor backend și frontend
3. **Configurare** — cum se completează datele de conectare la MySQL și URL-ul API-ului în frontend
4. **Pornire** — comanda pentru a porni backend-ul (cu portul implicit) și pașii pentru a accesa frontend-ul (deschidere fișier static, comandă `npm run dev` etc.)
5. **Exemple de apel API** — cel puțin o comandă `curl` pentru fiecare dintre cele două endpoint-uri
6. **Capturi de ecran** — 1–2 imagini cu starea finală a interfeței (desktop și/sau mobil)

Dacă vreți să justificați o decizie tehnică, faceți-o tot în acest README, într-o secțiune separată.

---

## 7. Ce nu se cere în acest exercițiu

Pentru a păstra exercițiul concentrat pe esențial, nu este nevoie să implementați:

- autentificare, autorizare, login sau sesiuni
- pagină de detalii produs, coș de cumpărături sau plasare comenzi din UI
- sistem de caching
- deploy public
- teste automatizate (acceptate dacă există, dar nu obligatorii)

Dacă totuși alegeți să adăugați lucruri peste cerințele de mai sus (ex. o pagină de detalii, un `Dockerfile`, teste, animații), menționați-le în README.

---

## 8. Contact

Pentru întrebări despre cerințe, înainte de a începe lucrul: adresa de email din anunțul de stagiu.
