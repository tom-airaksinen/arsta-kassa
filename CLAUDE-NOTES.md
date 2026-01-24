# Utvecklingsanteckningar - Årsta AIK Kassaapp

## Swish QR-kod (implementerat)

### Tekniska detaljer
- **Swish-nummer:** 1236828156 (123 682 81 56)
- **Meddelande:** Sjöstadskaféet
- **QR-format:** `C<nummer>;<belopp>;<meddelande>;`
- **Svenska tecken:** Måste URL-enkodas med `encodeURIComponent()` för att Swish ska acceptera dem
- **QR-bibliotek:** qrcodejs (qrcode.min.js) - genererar canvas-baserade QR-koder lokalt
- **Error correction:** Level L (för att få plats med längre UTF-8-enkodade meddelanden)

### Varför lokal QR-generering?
Swish har ett API (`https://mpc.getswish.net/qrg-swish/api/v1/prefilled`) men det har CORS-restriktioner och kan inte anropas från webbläsare. Lokal generering med qrcodejs fungerar utan server.

### POC-filer (lokalt, ej committade)
- `swish-poc.html` - fungerade testversion
- `qr-debug.html` - felsökningsversion
- `test-qr.png` - test från Swish API via curl

---

## Framtida förbättringar

### Multi-device sync med Firebase Realtime Database

**Idé:** Separera kassa-device och display-device
- **Kassa-device:** Kassören lägger till/tar bort varor
- **Display-device:** Visar ordern och QR-koden vänd mot kunden

**Teknisk approach:**
1. Firebase Realtime Database (gratis tier räcker)
2. Kassa-device skriver till databasen vid varje ändring
3. Display-device lyssnar på ändringar och uppdaterar i realtid
4. Ingen backend krävs - Firebase SDK direkt i webbläsaren

**Firebase-konfiguration (exempel):**
```javascript
const firebaseConfig = {
  // Hämtas från Firebase Console
  apiKey: "...",
  databaseURL: "https://xxx.firebaseio.com",
  projectId: "xxx"
};

// Skriva (kassa-device)
firebase.database().ref('currentOrder').set(cartItems);

// Lyssna (display-device)
firebase.database().ref('currentOrder').on('value', (snapshot) => {
  cartItems = snapshot.val() || [];
  renderCart();
  updateSwishQR();
});
```

**Fördelar:**
- Realtidsuppdatering (~100ms latens)
- Gratis för låg trafik
- Ingen egen server behövs
- Fungerar på GitHub Pages

**Nackdelar:**
- Kräver Firebase-konto och konfiguration
- Internetuppkoppling krävs

---

## Swish Företag vs Swish Handel

För att ta emot betalningar via QR-kod behövs **Swish Företag** (ansöks via föreningens bank). Det räcker för denna användning - Swish Handel är mer avancerat och dyrt, för e-handel med callback/bekräftelse.

---

## Kontakt

Mejlutkast för att fråga föreningen om Swish finns i plan-filen:
`~/.claude/plans/cached-sprouting-lark.md`
