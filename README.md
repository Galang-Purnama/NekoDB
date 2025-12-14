# NekodDB Client

NekodDB Client adalah library JavaScript/Node.js untuk berkomunikasi dengan server NekodDB melalui koneksi WebSocket. Library ini menyediakan antarmuka yang sederhana dan intuitif untuk melakukan operasi database seperti insert, read, update, dan delete (CRUD) serta manajemen collection.

## 📋 Daftar Isi

- [Instalasi](#instalasi)
- [Penggunaan Dasar](#penggunaan-dasar)
- [API Reference](#api-reference)
  - [Operasi Dokumen](#operasi-dokumen)
  - [Operasi Collection](#operasi-collection)
  - [Manajemen Koneksi](#manajemen-koneksi)
- [Contoh Penggunaan](#contoh-penggunaan)
- [Arsitektur Internal](#arsitektur-internal)
- [Error Handling](#error-handling)

## Instalasi

### Prasyarat

- Node.js v12 atau lebih tinggi

### Setup

```bash
npm install nekodb
```

### Import

```javascript
import { NekodDB } from 'nekodb';
```

## Penggunaan Dasar

### Inisialisasi Koneksi

```javascript
const db = new NekodDB({
  host: 'localhost:5000',    // Alamat host server
  username: 'your_username',  // Username untuk autentikasi
  password: 'your_password'   // Password untuk autentikasi
});
```

Koneksi akan otomatis dibuat saat instance dibuat. Jika koneksi terputus, client akan otomatis mencoba reconnect setiap 1500ms.

## API Reference

### Operasi Dokumen

#### `insert(collection, data)`

Menambahkan dokumen baru ke collection.

**Parameter:**
- `collection` (string): Nama collection
- `data` (Object): Data dokumen yang akan ditambahkan

**Return:** `Promise<Object|string>` - Response dari server

**Contoh:**
```javascript
const result = await db.insert('users', {
  name: 'John Doe',
  email: 'john@example.com',
  age: 25
});
console.log(result);
```

---

#### `list(collection)`

Mengambil semua dokumen dari collection.

**Parameter:**
- `collection` (string): Nama collection

**Return:** `Promise<Array|string>` - Array berisi semua dokumen

**Contoh:**
```javascript
const users = await db.list('users');
console.log(users);
```

---

#### `get(collection, id)`

Mengambil dokumen spesifik berdasarkan ID.

**Parameter:**
- `collection` (string): Nama collection
- `id` (string|number): ID dokumen yang dicari

**Return:** `Promise<Object|string>` - Dokumen yang ditemukan

**Contoh:**
```javascript
const user = await db.get('users', '12345');
console.log(user);
```

---

#### `update(collection, id, data)`

Memperbarui dokumen yang sudah ada.

**Parameter:**
- `collection` (string): Nama collection
- `id` (string|number): ID dokumen yang akan diupdate
- `data` (Object): Data baru untuk dokumen

**Return:** `Promise<Object|string>` - Response dari server

**Contoh:**
```javascript
const result = await db.update('users', '12345', {
  age: 26,
  email: 'newemail@example.com'
});
console.log(result);
```

---

#### `delete(collection, id)`

Menghapus dokumen dari collection.

**Parameter:**
- `collection` (string): Nama collection
- `id` (string|number): ID dokumen yang akan dihapus

**Return:** `Promise<string>` - Response dari server ('deleted' atau error message)

**Contoh:**
```javascript
const result = await db.delete('users', '12345');
console.log(result);
```

---

#### `search(collection, query)`

Mencari dokumen berdasarkan query filter.

**Parameter:**
- `collection` (string): Nama collection
- `query` (Object): Query filter untuk pencarian

**Return:** `Promise<Array|string>` - Array dokumen yang match

**Contoh:**
```javascript
const results = await db.search('users', {
  age: { $gte: 18 },
  name: 'John'
});
console.log(results);
```

---

### Operasi Collection

#### `count(collection)`

Menghitung jumlah dokumen dalam collection.

**Parameter:**
- `collection` (string): Nama collection

**Return:** `Promise<Object>` - Response dengan format `{ count: number }`

**Contoh:**
```javascript
const result = await db.count('users');
console.log(`Total users: ${result.count}`);
```

---

#### `deleteCollection(collection)`

Menghapus seluruh collection.

**Parameter:**
- `collection` (string): Nama collection yang akan dihapus

**Return:** `Promise<string>` - Response dari server ('collection-deleted' atau error message)

**Contoh:**
```javascript
const result = await db.deleteCollection('old_data');
console.log(result);
```

---

#### `listCollections()`

Mengambil daftar semua collection milik user.

**Return:** `Promise<Array|string>` - List collection atau error dari server

**Contoh:**
```javascript
const collections = await db.listCollections();
console.log(collections);
```

---

### Manajemen Koneksi

#### `close()`

Menutup koneksi WebSocket ke server dan menghentikan auto-reconnect.

**Return:** `void`

**Contoh:**
```javascript
db.close();
```

---

## Contoh Penggunaan

### Contoh Lengkap: Aplikasi User Management

```javascript
import { NekodDB } from './client.js';

// Inisialisasi client
const db = new NekodDB({
  host: 'localhost:5000',
  username: 'admin',
  password: 'password123'
});

async function main() {
  try {
    // 1. Insert dokumen baru
    console.log('1. Menambah user baru...');
    const insertResult = await db.insert('users', {
      name: 'Alice Smith',
      email: 'alice@example.com',
      age: 30
    });
    console.log('Insert result:', insertResult);

    // 2. Tampilkan semua users
    console.log('\n2. Menampilkan semua users...');
    const allUsers = await db.list('users');
    console.log('All users:', allUsers);

    // 3. Cari user berdasarkan kriteria
    console.log('\n3. Mencari users dengan age >= 25...');
    const searchResult = await db.search('users', { age: { $gte: 25 } });
    console.log('Search result:', searchResult);

    // 4. Update user
    console.log('\n4. Update user...');
    const updateResult = await db.update('users', 'user_id_here', {
      age: 31,
      city: 'Jakarta'
    });
    console.log('Update result:', updateResult);

    // 5. Hitung total users
    console.log('\n5. Menghitung total users...');
    const countResult = await db.count('users');
    console.log(`Total users: ${countResult.count}`);

    // 6. List semua collections
    console.log('\n6. Menampilkan semua collections...');
    const collections = await db.listCollections();
    console.log('Collections:', collections);

  } catch (error) {
    console.error('Error:', error);
  } finally {
    // Tutup koneksi
    db.close();
  }
}

main();
```

---

## Arsitektur Internal

### Flow Koneksi

```
┌─────────────────────────────────────────────────────────┐
│ Constructor                                             │
│ - Inisialisasi private fields                          │
│ - Panggil #init()                                      │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ #init()                                                 │
│ - Buat WebSocket connection                            │
│ - Setup event listeners (open, message, error, close)  │
│ - Buat connected promise                               │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ WebSocket Events                                        │
│ 'open'    → Resolve connected promise, start ping      │
│ 'message' → Buffer incoming data                       │
│ 'error'   → Reject connected promise                   │
│ 'close'   → Stop ping, reconnect otomatis              │
└─────────────────────────────────────────────────────────┘
```

### Mekanisme Pengiriman Data

1. **Method Call** (e.g., `insert()`)
   - Panggil `#request()` dengan parameter action, collection, data

2. **#request()**
   - Buat payload object dengan auth, action, collection, document, dll
   - Panggil `#sendPayload()`

3. **#sendPayload()**
   - Tunggu koneksi siap via `#ensureConnected()`
   - Konversi payload ke JSON binary
   - Kirim via WebSocket
   - Poll response hingga ada newline (`\n`)
   - Parse JSON response dan return

4. **Response Handling**
   - Server mengirim response sebagai JSON string
   - Client buffer data sampai ada newline
   - Parse dan resolve promise dengan data

### Mekanisme Heartbeat

**Ping Interval: 25 detik**

```javascript
#startPing() {
  this.#pingInterval = setInterval(() => {
    if (this.#ws && this.#ws.readyState === WebSocket.OPEN) {
      this.#ws.ping();
    }
  }, 25000); // 25 detik
}
```

**Fungsi:**
- Menjaga koneksi tetap aktif
- Detect disconnection lebih cepat
- Prevent server dari timeout

### Auto-Reconnection

Ketika koneksi terputus (event 'close'):
1. Stop ping interval
2. Set `#ws = null`
3. Jika `#shouldReconnect = true`, tunggu 1500ms
4. Panggil `#init()` untuk reconnect
5. Ulangi sampai berhasil

**Catatan:** Auto-reconnect dapat dimatikan dengan memanggil `close()`

---

## Private Fields (Enkapsulasi)

Library menggunakan ES2022 Private Fields (`#`) untuk encapsulation:

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| `#host` | string | Alamat server |
| `#username` | string | Username autentikasi |
| `#password` | string | Password autentikasi |
| `#ws` | WebSocket | Instance WebSocket |
| `#buffer` | string | Buffer respons server |
| `#connectedPromise` | Promise | Promise status koneksi |
| `#resolveConnected` | Function | Resolve connected promise |
| `#rejectConnected` | Function | Reject connected promise |
| `#shouldReconnect` | boolean | Flag auto-reconnect |
| `#pingInterval` | number | Interval ID untuk ping |

---

## Error Handling

### Koneksi Error

```javascript
const db = new NekodDB({
  host: 'invalid-host:5000',
  username: 'user',
  password: 'pass'
});

db.insert('users', {}).catch(error => {
  console.error('Connection error:', error.message);
});
```

### Invalid Response

Jika server mengirim response yang bukan JSON valid, client akan mengembalikan response sebagai string.

```javascript
const result = await db.get('users', '123');
console.log(typeof result); // Bisa 'object' atau 'string'
```

### Best Practice

```javascript
async function safeOperation() {
  try {
    const result = await db.insert('users', { name: 'John' });
    
    if (typeof result === 'string') {
      // Error message dari server
      console.error('Server error:', result);
    } else {
      // Success, result adalah object
      console.log('Success:', result);
    }
  } catch (error) {
    // Connection error
    console.error('Connection error:', error);
  }
}
```

---

## Signal Handling

Library secara otomatis menangani signal termination:

```javascript
process.once('SIGINT', () => {
  db.close();
  process.exit();
});

process.once('SIGTERM', () => {
  db.close();
  process.exit();
});
```

Ini memastikan koneksi ditutup dengan benar sebelum process berakhir.

---

## Lisensi

MIT
