# OBSCURA v3.0

> **Text Encoder · Decoder · Hasher · Code Obfuscator**  
> Single-file web app — semua proses berjalan 100% di browser, tidak ada data yang dikirim ke server.

---

## 🖥️ Demo

Buka file `obscura-v3.html` langsung di browser — tidak perlu server, tidak perlu install apapun.

---

## ✨ Fitur

### ⬆ Encode / ⬇ Decode
Konversi teks dua arah menggunakan 6 algoritma:

| Algoritma | Keterangan |
|-----------|------------|
| **Base64** | Encoding standar, support Unicode penuh |
| **URL** | Percent-encoding untuk URL |
| **Hex** | Byte array dalam format hexadecimal |
| **Binary** | Byte array dalam format biner 8-bit |
| **HTML Entities** | Escape karakter HTML (`&amp;`, `&lt;`, dll.) |
| **Unicode** | Escape sequence `\uXXXX` |

---

### ⚡ Hash
Generate hash satu arah (tidak bisa di-decode):

| Algoritma | Output | Keterangan |
|-----------|--------|------------|
| **MD5** | 128-bit / 32 hex chars | Pure JS, UTF-8 aware |
| **SHA-1** | 160-bit / 40 hex chars | Web Crypto API |
| **SHA-256** | 256-bit / 64 hex chars | Web Crypto API |
| **SHA-512** | 512-bit / 128 hex chars | Web Crypto API |

---

### 🔐 Cipher
Enkripsi/dekripsi klasik berbasis teks:

| Cipher | Keterangan |
|--------|------------|
| **Caesar** | Shift huruf dengan nilai custom (1–25) |
| **ROT-13** | Caesar shift = 13, reversible |
| **Atbash** | Mirror alfabet (A↔Z, B↔Y, ...) |
| **Reverse** | Balik urutan karakter |
| **Morse** | Encode teks → Morse, auto-detect untuk decode |

> **Auto-detect Morse:** jika input hanya berisi `.`, `-`, `/`, dan spasi, otomatis masuk mode decode.

---

### 🛡 Code Obfuscator
Obfuskasi JavaScript dengan hingga 6 layer proteksi yang bisa dikombinasikan:

| Layer | Keterangan |
|-------|------------|
| **Name Mangling** | Rename semua variabel & fungsi → `_0x0000`, `_0x0001`, ... |
| **String Encryption** | Encode string literal ke hex escape `\xNN` secara acak |
| **Dead Code Injection** | Sisipkan snippet kode palsu yang tidak pernah dieksekusi |
| **Control Flow Wrap** | Bungkus seluruh kode dalam IIFE state-machine dengan switch/while |
| **Minify** | Hapus whitespace, komentar, kolaps operator |
| **Anti-Tamper Header** | Tambah FNV-1a integrity check di atas kode (browser-only) |

> Output tetap **100% fungsional secara logika** — hanya susah dibaca manusia.

#### Urutan pipeline obfuskasi:
```
Input → Minify → String Encryption → Name Mangling
      → Dead Code Injection → Control Flow Wrap → Anti-Tamper
```

#### Stats bar:
Setelah obfuskasi, ditampilkan:
- Ukuran original vs hasil
- Perubahan ukuran (%)
- Layer yang diterapkan

---

## 🐛 Bug Fixes (v2.1 → v3.0)

Versi ini memperbaiki **9 bug** dari versi sebelumnya:

| # | Bug | Fix |
|---|-----|-----|
| 1 | **MD5 output salah** — nibble swap (`x[1]+x[0]`) bukan byte reversal | Ganti ke `.reverse()` |
| 2 | **Anti-tamper no-op** — body `if` kosong `{}` | Tambah `innerHTML=''` + `throw Error` |
| 3 | **Control flow urutan terbalik** — chunk dieksekusi dalam urutan acak | Redesign label mapping |
| 4 | **`chunk` variable tidak dipakai** — backtick escaping non-functional | Pakai variable yang sudah di-escape |
| 5 | **Minifier merusak string** — regex buta menimpa isi string literal | Rewrite 3-pass dengan placeholder extraction |
| 6 | **Dead code masuk object literal** — trigger di `}` tanpa cek depth | Full bracket depth tracking, hanya inject setelah `;` di depth 0 |
| 7 | **Dead code memotong generated switch** — split mid-expression | Rewrite `splitCodeChunks` dengan statement collector |
| 8 | **Morse decode hilang** — `MORSE_REV` didefinisikan tapi tidak dipakai | Tambah jalur decode dengan auto-detect |
| 9 | **Name mangler skip single-char var** — `i`, `j`, `n` tidak di-mangle | Hapus batasan `length > 1` |

---

## 🚀 Cara Pakai

### Browser biasa
```
Buka obscura-v3.html → pilih mode → input teks → klik ▶ Run
```

### Android (Kiwi Browser)
```
Buka file → Developer Tools → Console
Paste output obfuscated langsung untuk test
```

### Node.js (tanpa Anti-Tamper)
```bash
node output.js
```

### Node.js (dengan Anti-Tamper aktif)
```bash
node -e "var document={currentScript:null,body:{innerHTML:''}};$(cat output.js)"
```

---

## 📋 Cara Test Obfuscator

Gunakan test code ini untuk verifikasi output masih fungsional:

```javascript
var message = "Hello, Obscura!";
var numbers = [1, 2, 3, 4, 5];
var result = 0;

function add(a, b) { return a + b; }
function multiply(x, y) { return x * y; }

for (var i = 0; i < numbers.length; i++) {
  result = add(result, numbers[i]);
}

var label = result > 10 ? "big" : "small";
var info = "sum >= 10? value is: " + result; // string dengan operator — stress test minifier

function greet(name) {
  var prefix = "Hello";
  return prefix + ", " + name + "!";
}

var person = {
  name: "Alice",
  age: 25,
  speak: function() { return "My name is " + this.name; }
};

console.log("sum:", result);           // expected: 15
console.log("label:", label);          // expected: "big"
console.log("info:", info);            // expected: "sum >= 10? value is: 15"
console.log("greet:", greet("World")); // expected: "Hello, World!"
console.log("multiply:", multiply(3,4)); // expected: 12
console.log("speak:", person.speak()); // expected: "My name is Alice"
```

**Expected output setelah obfuskasi:**
```
sum: 15
label: big
info: sum >= 10? value is: 15
greet: Hello, World!
multiply: 12
speak: My name is Alice
```

---

## ⚙️ Technical Details

- **Single file** — satu `.html`, tidak ada dependency eksternal selain Google Fonts
- **Zero server** — tidak ada backend, tidak ada API call, tidak ada tracking
- **MD5** — implementasi pure JavaScript, UTF-8 aware via `TextEncoder`
- **SHA-1/256/512** — menggunakan native Web Crypto API (`crypto.subtle.digest`)
- **Minifier** — 3-pass: ekstrak string → minify → restore (aman untuk string yang mengandung operator)
- **Dead code injector** — bracket + string depth aware, hanya inject di top-level statement
- **Control flow splitter** — kumpulkan top-level statements via depth tracking, baru di-group jadi N chunk
- **Anti-tamper** — FNV-1a hash check via `document.currentScript.text` (browser-only)

---

## 📁 Struktur File

```
obscura-v3.html          ← seluruh aplikasi dalam 1 file
README.md                ← dokumentasi ini
```

---

## ⚠️ Catatan

- **Anti-Tamper** hanya berfungsi di browser (membutuhkan `document.currentScript`). Uncheck jika output akan dijalankan di Node.js.
- **Control Flow Wrap** tidak kompatibel dengan kode yang mengandalkan `return` di top-level (hanya cocok untuk kode yang dibungkus dalam fungsi).
- Obfuscator ini bersifat **deterrence** — mempersulit pembacaan manusia, bukan enkripsi kriptografis. Kode yang ter-obfuskasi masih bisa di-reverse dengan cukup waktu.

---

## 📄 License

Free to use. No warranty. All processing is client-side.
