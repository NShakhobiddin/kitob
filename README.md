# Kitob Mag'zi

Psixologiya va o'z-o'zini anglash kitoblarining **mag'zini**, amaliy tavsiyalarini va
interaktiv mashqlarini sodda o'zbek tilida yetkazuvchi Telegram WebApp.

- Serversiz — faqat statik fayllar, GitHub Pages'da ishlaydi
- Bitta `index.html` (React CDN orqali, build kerak emas)
- Kitoblar `books/` papkasidagi JSON fayllardan yuklanadi — **yangi kitob qo'shish uchun frontend o'zgarmaydi**
- Javoblar `localStorage`'da, foydalanuvchi qurilmasida saqlanadi
- Telegram SDK bo'lmasa (oddiy brauzer) ham to'liq ishlaydi

## Fayl strukturasi

```
.
├── index.html          ← butun ilova (React + CSS + mantiq)
├── books/
│   ├── index.json      ← kitoblar ro'yxati
│   └── namuna.json     ← namuna kitob ("Ichki Kuzatuvchi")
└── README.md
```

## Lokal ishga tushirish

`fetch` ishlatilgani uchun faylni to'g'ridan-to'g'ri (`file://`) ochib bo'lmaydi —
kichik server kerak:

```bash
python3 -m http.server 8000
```

So'ng brauzerda oching: <http://localhost:8000>

## GitHub Pages'ga joylash

1. Fayllarni GitHub repozitoriyaga push qiling (`index.html` **repo ildizida** turishi kerak).
2. Repo → **Settings** → **Pages**.
3. **Source**: `Deploy from a branch`.
4. **Branch**: kerakli branch (masalan `main`), papka: `/ (root)` → **Save**.
5. 1–2 daqiqadan so'ng sayt shu manzilda ochiladi:
   `https://<foydalanuvchi>.github.io/<repo>/`

> Sayt HTTPS'da bo'lishi shart — Telegram faqat HTTPS manzilni qabul qiladi.
> GitHub Pages HTTPS'ni o'zi beradi.

Kesh sababli yangilanish darrov ko'rinmasa, brauzerda `Ctrl+Shift+R` bosing.

## Telegram botga ulash

### 1. Bot yaratish

[@BotFather](https://t.me/BotFather) → `/newbot` → nom va username bering → tokenni saqlab qo'ying.

### 2. Menyu tugmasi qo'shish (eng sodda yo'l)

BotFather'da:

```
/mybots → botni tanlang → Bot Settings → Menu Button → Configure menu button
```

So'ng GitHub Pages manzilini yuboring:

```
https://<foydalanuvchi>.github.io/<repo>/
```

va tugma nomini yozing, masalan: `📖 Kitoblar`

Endi bot chatidagi menyu tugmasi WebApp'ni ochadi.

### 3. Chat ichida tugma sifatida (ixtiyoriy)

Botdan xabar bilan birga tugma yuborish (server/skript orqali):

```json
{
  "chat_id": 123456789,
  "text": "Kitob mag'zini oching:",
  "reply_markup": {
    "inline_keyboard": [[
      { "text": "📖 Kitob Mag'zi",
        "web_app": { "url": "https://<foydalanuvchi>.github.io/<repo>/" } }
    ]]
  }
}
```

Bu so'rovni `https://api.telegram.org/bot<TOKEN>/sendMessage` manziliga POST qiling.

### 4. Web App'ni BotFather'da ro'yxatdan o'tkazish (ixtiyoriy)

`/newapp` buyrug'i orqali ilovaga qisqa nom, rasm va to'g'ridan-to'g'ri havola olish mumkin.

## Yangi kitob qo'shish

1. `books/<id>.json` faylini quyidagi sxema bo'yicha yarating.
2. `books/index.json` ichidagi `kitoblar` ro'yxatiga bitta yozuv qo'shing.
3. Push qiling — tamom. `index.html` ga tegish shart emas.

### `books/index.json`

```json
{
  "kitoblar": [
    {
      "id": "namuna",
      "nomi": "Kitob nomi",
      "muallif": "Muallif ismi",
      "rang": "#6b4c9a",
      "tavsif": "Bir jumlalik qisqa tavsif"
    }
  ]
}
```

`id` — fayl nomi bilan bir xil bo'lishi kerak (`books/namuna.json`).
`rang` — muqova gradientining asosiy rangi.

### `books/{id}.json`

```json
{
  "id": "namuna",
  "nomi": "Kitob nomi",
  "muallif": "Muallif ismi",
  "kirish": "2-3 jumlada kitob nima haqida",
  "bolimlar": [
    {
      "sarlavha": "Bo'lim nomi",
      "magiz": ["Asosiy g'oya 1 (sodda tilda)", "Asosiy g'oya 2"],
      "tavsiyalar": ["Amaliy tavsiya 1", "Amaliy tavsiya 2"],
      "mashqlar": []
    }
  ]
}
```

## Mashq turlari

Har bir mashq `bolimlar[].mashqlar[]` ichida turadi va `turi` maydoni bilan aniqlanadi.

| Turi | Maydonlar | Xatti-harakati |
|---|---|---|
| `jurnal` | `savol` | Textarea; yozilgani avtomatik saqlanadi, so'zlar soni ko'rsatiladi |
| `checklist` | `sarlavha`, `bandlar[]` | Belgilangan bandlar saqlanadi, progress foizi chiziladi |
| `slider` | `savol`, `min`, `max`, `min_yorliq`, `max_yorliq` | Qiymat saqlanadi; oldingi qiymat bo'lsa "O'tgan safar: X" chiqadi |
| `taymer` | `daqiqa`, `korsatma` | Boshlash / pauza / qayta, aylana progress, tugaganda haptic + xabar |
| `test` | `savol`, `variantlar[]`, `izohlar[]` | Variant tanlanganda mos izoh ochiladi (to'g'ri/noto'g'ri yo'q) |

```json
{ "turi": "jurnal", "savol": "Yozma savol matni" }

{ "turi": "checklist", "sarlavha": "Ro'yxat nomi", "bandlar": ["band 1", "band 2"] }

{ "turi": "slider", "savol": "Savol", "min": 1, "max": 10,
  "min_yorliq": "Past", "max_yorliq": "Yuqori" }

{ "turi": "taymer", "daqiqa": 5, "korsatma": "Mashq davomida nima qilish kerak" }

{ "turi": "test", "savol": "Savol",
  "variantlar": ["A", "B", "C"],
  "izohlar": ["A tanlansa izoh", "B izoh", "C izoh"] }
```

`izohlar` massivi `variantlar` bilan bir tartibda bo'lishi kerak.

## Saqlash mantiqi

Javoblar `localStorage`'da quyidagi kalit bilan saqlanadi:

```
km_{kitobId}_{bolimIndeksi}_{mashqIndeksi}
```

Masalan, `namuna` kitobining 1-bo'limidagi 2-mashq → `km_namuna_0_1`.

⚠️ Bo'lim yoki mashqning **tartibini o'zgartirsangiz**, foydalanuvchining eski javoblari
boshqa mashqqa tegishli bo'lib qoladi. Mavjud kitobni tahrirlashda yangi mashqni
oxiriga qo'shgan ma'qul.

Ma'lumot faqat foydalanuvchi qurilmasida qoladi — hech qayerga yuborilmaydi.

## Telegram integratsiyasi

`index.html` ichida SDK xavfsiz o'ram (`TG`, `haptic`, `backButton`) orqali ishlatiladi:

- `ready()` va `expand()` — ilova ochilganda
- Header/fon rangi ilova mavzusiga moslanadi
- `BackButton` — kitob ichida ko'rsatiladi, chiqishda yashiriladi
- `HapticFeedback` — kitob ochish, akkordeon, checklist, slider, test va taymer tugashida
- SDK topilmasa barcha chaqiruvlar jimgina o'tkazib yuboriladi — oddiy brauzerda xato bermaydi

## Sinovdan o'tkazilgan

Namuna kitob barcha 5 mashq turini o'z ichiga oladi. Chromium (380px mobil ekran) da
tekshirilgan: har bir mashq turi ishlaydi, `localStorage` saqlaydi va sahifa
yangilangandan keyin javoblarni tiklaydi, taymer oxirigacha sanaydi, Telegram SDK
bo'lmagan holatda ham JS xatosi chiqmaydi.
