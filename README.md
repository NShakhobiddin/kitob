# Kitob Mag'zi

Psixologiya va o'z-o'zini anglash kitoblarining **mag'zini**, amaliy tavsiyalarini va
interaktiv mashqlarini sodda o'zbek tilida yetkazuvchi Telegram WebApp.

- Serversiz — faqat statik fayllar, GitHub Pages'da ishlaydi
- Bitta `index.html` (React CDN orqali, build kerak emas)
- Kitoblar `books/` papkasidagi JSON fayllardan yuklanadi — **yangi kitob qo'shish uchun frontend o'zgarmaydi**
- Javoblar `localStorage`'da, foydalanuvchi qurilmasida saqlanadi
- Telegram SDK bo'lmasa (oddiy brauzer) ham to'liq ishlaydi

## Dizayn tizimi

Och fon, oq kartalar va to'q ko'k (navy) urg'u — zamonaviy mobil ilova uslubi.

| Token | Qiymat | Qayerda |
|---|---|---|
| `--fon` | `#eef1f8` | sahifa foni |
| `--karta` | `#ffffff` | kartalar, inputlar |
| `--asos` | `#1e3a8a` | urg'u rangi, ikonkalar, progress |
| `--asos-tim` | `#152a63` | gradientning to'q uchi |
| `--asos-och` | `#e7ecfb` | nishonlar, ikonka fonlari |
| `--matn` / `--matn-xira` | `#16204a` / `#7b86a8` | asosiy / ikkilamchi matn |

- Shrift: **Poppins** (Google Fonts), tizim sans-serifga tushib qoladi
- Radiuslar: kartalar 18–24px, tugmalar va nishonlar 999px
- Soyalar yumshoq va ko'kimtir: `0 6px 22px rgba(30,58,138,.07)`
- Kitob muqovasi va sahifa hero'si kitobning `rang` maydonidan gradient yasaydi
- Pastki navigatsiya: **Kitoblar · Jarayon · Kundalik · Eslatma** — to'rttasi ham
  haqiqiy sahifa, `localStorage`'dagi ma'lumotdan hisoblanadi
- Kitob ichida har bir bo'lim ikkita bo'limchaga bo'lingan: **💠 Mag'iz** (mag'iz +
  amaliy tavsiyalar) va **🌙 Mashqlar**. Bo'lim sarlavhasida bajarilgan mashq
  hisobi ko'rinadi
- 380px mobil ekranga moslangan, `prefers-reduced-motion` hurmat qilinadi

## Fayl strukturasi

```
.
├── index.html          ← butun ilova (React + CSS + mantiq)
├── books/
│   ├── index.json      ← kitoblar ro'yxati
│   ├── namuna.json     ← namuna kitob ("Ichki Kuzatuvchi")
│   ├── lucid-tush.json ← Lucid tush amaliyoti (kundalik + eslatmali bo'lim)
│   ├── intuitsiya.json ← Norbekov turkumidagi 6-kitob
│   └── *.json          ← qolgan kitoblar
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
      "turkum": "Turkum nomi (ixtiyoriy)",
      "rang": "#6b4c9a",
      "tavsif": "Bir jumlalik qisqa tavsif"
    }
  ]
}
```

`id` — fayl nomi bilan bir xil bo'lishi kerak (`books/namuna.json`).
`rang` — muqova gradientining asosiy rangi.

`turkum` — **ixtiyoriy**. Bir xil `turkum` qiymatiga ega kitoblar bosh sahifada va
Jarayon sahifasida bitta guruh sarlavhasi ostida chiqadi (masalan «Tentakning
tajribasi» turkumidagi 4 ta kitob). Turkumsiz kitoblar «Boshqa kitoblar» ostiga
tushadi va ro'yxat oxirida turadi.

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
| `kundalik` | `sarlavha`, `korsatma`, `joy_matn`, `belgi_matn` | Sanali ko'p yozuvli kundalik; belgilar sanaladi, ketma-ket kunlar seriyasi ko'rsatiladi |
| `eslatma` | `sarlavha`, `matn`, `korsatma`, `vaqtlar[]` | Kunlik eslatma vaqtlari; ilova ochiq turganda bildirishnoma, `.ics` orqali telefon kalendariga eksport |

```json
{ "turi": "jurnal", "savol": "Yozma savol matni" }

{ "turi": "checklist", "sarlavha": "Ro'yxat nomi", "bandlar": ["band 1", "band 2"] }

{ "turi": "slider", "savol": "Savol", "min": 1, "max": 10,
  "min_yorliq": "Past", "max_yorliq": "Yuqori" }

{ "turi": "taymer", "daqiqa": 5, "korsatma": "Mashq davomida nima qilish kerak" }

{ "turi": "test", "savol": "Savol",
  "variantlar": ["A", "B", "C"],
  "izohlar": ["A tanlansa izoh", "B izoh", "C izoh"] }

{ "turi": "kundalik", "sarlavha": "Tush kundaligi",
  "korsatma": "Qanday to'ldirish kerakligi haqida ko'rsatma",
  "joy_matn": "Textarea placeholder",
  "belgi_matn": "Belgilar maydoni placeholder" }

{ "turi": "eslatma", "sarlavha": "Reallik tekshiruvi",
  "matn": "Eslatma chiqqanda ko'rinadigan matn",
  "korsatma": "Vaqtlarni qanday tanlash kerakligi haqida ko'rsatma",
  "vaqtlar": ["09:20", "14:10", "19:00"] }
```

`izohlar` massivi `variantlar` bilan bir tartibda bo'lishi kerak.

### `kundalik` haqida

Oddiy `jurnal`dan farqi — bitta matn emas, **sanali yozuvlar ro'yxati**. Foydalanuvchi
har safar yangi yozuv qo'shadi, eskilarini ochib o'qiydi yoki o'chiradi. Vergul bilan
kiritilgan belgilar barcha yozuvlar bo'yicha sanaladi va eng ko'p takrorlangan 8 tasi
nishon sifatida ko'rsatiladi — tush belgilarini topish shu orqali ishlaydi. Yozuvlar
ketma-ket kunlar seriyasi ham hisoblanadi.

### `eslatma` haqida

`vaqtlar[]` — standart vaqtlar; foydalanuvchi ularni o'chirib, o'zinikini qo'sha oladi.
Eslatma yoqilganda ilova har 15 soniyada vaqtni tekshiradi va mos kelganda:

1. haptic beradi va ilova ichida xabar ko'rsatadi;
2. `Notification` API mavjud va ruxsat berilgan bo'lsa, tizim bildirishnomasini chiqaradi.

**Cheklov:** brauzer bildirishnomasi faqat ilova ochiq turganda ishlaydi (serversiz
ilovada push yuboradigan joy yo'q). Shuning uchun har bir eslatmada **«Kalendarga
qo'shish»** tugmasi bor — u `RRULE:FREQ=DAILY` va `VALARM` bilan `.ics` fayl yaratadi,
foydalanuvchi uni telefon kalendariga qo'shsa, eslatma ilovadan mustaqil ishlaydi.
`Notification` yo'q bo'lsa yoki ruxsat berilmagan bo'lsa ham ilova xato bermaydi —
eslatma faqat ilova ichida ko'rinadi.

## Saqlash mantiqi

Javoblar `localStorage`'da quyidagi kalit bilan saqlanadi:

```
km_{kitobId}_{bolimIndeksi}_{mashqIndeksi}
```

Masalan, `namuna` kitobining 1-bo'limidagi 2-mashq → `km_namuna_0_1`.

⚠️ Bo'lim yoki mashqning **tartibini o'zgartirsangiz**, foydalanuvchining eski javoblari
boshqa mashqqa tegishli bo'lib qoladi. Mavjud kitobni tahrirlashda yangi mashqni
oxiriga qo'shgan ma'qul.

Bundan tashqari `km_oxirgi` kaliti oxirgi ochilgan kitob `id`sini saqlaydi — bosh
sahifada o'sha kitob birinchi bo'lib, to'q ko'k karta ko'rinishida chiqadi.

**Jarayon foizi** shu javoblardan hisoblanadi. Mashq "bajarilgan" deb belgilanadi:
`jurnal` — matn bo'sh emas; `checklist` — barcha bandlar belgilangan; `slider` —
qiymat saqlangan; `test` — variant tanlangan; `taymer` — kamida bir marta oxirigacha
sanagan; `kundalik` — kamida bitta yozuv; `eslatma` — yoqilgan.

Ma'lumot faqat foydalanuvchi qurilmasida qoladi — hech qayerga yuborilmaydi.

## Telegram integratsiyasi

`index.html` ichida SDK xavfsiz o'ram (`TG`, `haptic`, `backButton`) orqali ishlatiladi:

- `ready()` va `expand()` — ilova ochilganda
- Header/fon rangi ilova mavzusiga moslanadi
- `BackButton` — kitob ichida ko'rsatiladi, chiqishda yashiriladi
- `HapticFeedback` — kitob ochish, akkordeon, checklist, slider, test va taymer tugashida
- Foydalanuvchi ismi (`initDataUnsafe.user.first_name`) salomlashuvda ishlatiladi, bo'lmasa umumiy matn
- SDK topilmasa barcha chaqiruvlar jimgina o'tkazib yuboriladi — oddiy brauzerda xato bermaydi

## Sinovdan o'tkazilgan

Namuna kitob 5 ta asosiy mashq turini, `lucid-tush` kitobi esa qolgan ikkitasini
(`kundalik`, `eslatma`) ham o'z ichiga oladi. Chromium (380px mobil ekran) da
tekshirilgan: har bir mashq turi ishlaydi, `localStorage` saqlaydi va sahifa
yangilangandan keyin javoblarni tiklaydi, taymer oxirigacha sanaydi, kundalik yozuvlari
qo'shiladi/o'chiriladi va belgilar sanaladi, eslatma vaqti kelganda ishga tushadi,
`.ics` fayl to'g'ri tarkib bilan yuklanadi, taymer tugagach bajarilgani saqlanib
jarayon foizi jonli yangilanadi, pastki navigatsiyaning to'rt sahifasi ham ochiladi,
kitoblar turkum bo'yicha guruhlanadi, bo'limchalar bir-biriga to'g'ri almashadi,
Telegram SDK va `Notification` bo'lmagan holatda ham JS xatosi chiqmaydi.
