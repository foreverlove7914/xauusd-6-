# XAUT/USDT Perpetual — Live Dashboard

Dashboard live real-time untuk XAUT/USDT Perpetual (Tether Gold, Binance Futures), dibina 100% static (HTML + JS tulen) — tiada backend, tiada build step. Terus boleh host di **GitHub Pages**.

## Ciri-ciri

Interval tersedia: **1m, 5m, 15m, 1h, 4h, 1D, 1W**. Weekly (1W) berguna untuk baca trend besar/struktur pasaran sebelum turun ke timeframe kecil cari entry — S/R, Fibonacci, dan whale wall semua berfungsi sama di semua interval.

| Ciri | Cara ia dikira |
|---|---|
| Chart candlestick live | WebSocket `kline` stream, Binance Futures |
| 24h High / Low / Volume | WebSocket `ticker` stream |
| Support & Resistance (kuat/lemah) | Pivot swing high/low + clustering. "Kuat" = ≥3 sentuhan, "Lemah" = <3 sentuhan |
| Entry Buy/Sell | Reaksi candle bullish/bearish di level S/R |
| Re-entry | Level yang sama disentuh & confirm buat kali kedua |
| Breakout | Candle *close* menembusi level S/R > 0.15×ATR berbanding candle sebelumnya — ditanda ungu (`BO`) di chart & diberi keutamaan berbanding entry/re-entry biasa |
| Pullback | Selepas breakout, harga patah balik retest level yang baru dipecah (dalam 12 candle, ±0.15%) dan candle confirm sambung arah asal — ditanda kuning (`PB`) |
| TP / SL | Reversal: TP = level S/R bertentangan, SL = ATR(14)×0.6 di luar level. Breakout: TP = ATR×3.2 ke arah breakout, SL = kembali ke belakang level yang pecah |
| Big trader pending order (whale wall) | Order book depth (`depth20`) — level dengan qty > mean + 1.5×std dev ditandakan sebagai "wall" |
| Fibonacci Retracement | Auto-drawn dari swing high/low ketara dalam 150 candle terkini (0%, 23.6%, 38.2%, 50%, 61.8%, 78.6%, 100%) — arah (uptrend/downtrend) dikesan automatik ikut extreme mana yang lebih baru; 61.8% (golden ratio) ditanda emas |
| Confluence (★) | Bila level Fib jatuh dalam 0.3% dari satu level S/R, kedua-duanya digabung jadi **satu** garis emas terang berlabel `★ Fib xx% + S/R` — mengelakkan dua label bertindih dan menandakan zon yang lebih kuat sebab disokong dua indicator sekali |

Untuk kurangkan kesesakan label di paksi harga, hanya level Fib utama (0%, 38.2%, 50%, 61.8%, 100%) yang dilabel di chart; 23.6% dan 78.6% tetap dilukis (garis nipis) tapi tanpa label supaya tak bertindih dengan label lain.

Chart memaparkan **semua** event (Buy/Sell/Re-entry/Breakout/Pullback) dari sejarah candle yang dimuat — bukan setakat signal terkini — lengkap dengan legend kecil di sudut kiri-atas chart. Panel "Signal Terkini" di sidebar hanya papar detail (Entry/TP/SL/R:R) untuk event paling baharu sahaja.

Layout responsif — pada skrin sempit (mobile), chart akan duduk di atas (bukan tersepit tepi) dan panel sidebar disusun di bawahnya, boleh scroll.

## ⚠️ Kalau status tunjuk "Terputus" / data "--"

Punca paling biasa: fail dibuka terus dari storan tempatan (`content://...` / `file://...`) bukan melalui `http(s)://`. Sesetengah browser mobile sekat `fetch`/`WebSocket` dari fail lokal atas sebab keselamatan. Penyelesaian:
1. Jalankan `python3 -m http.server` dan buka `http://localhost:8080`, **atau**
2. Deploy terus ke GitHub Pages (langkah di bawah) — sekali live di `https://...`, ia akan connect terus.

Jika masih terputus selepas itu, kemungkinan `fapi.binance.com` / `fstream.binance.com` disekat oleh rangkaian/ISP anda — cuba guna VPN.

## ⚠️ Disclaimer penting

Semua entry/re-entry/TP/SL/S&R di sini dijana oleh **algoritma heuristik ringkas** (pivot + ATR), **bukan** sistem prediktif yang terbukti profit dan **bukan nasihat kewangan**. Ia satu alat bantu visual sahaja. Sentiasa buat analisis dan pengesahan sendiri sebelum membuat sebarang keputusan trading. Leverage/perpetual futures berisiko tinggi — anda boleh kehilangan lebih daripada modal asal.

## Jalankan secara tempatan

Tiada build step diperlukan — cuma buka fail terus, atau serve secara statik:

```bash
# guna Python
python3 -m http.server 8080
# atau guna Node
npx serve .
```

Kemudian buka `http://localhost:8080`.

> Nota: browser perlu boleh akses `fapi.binance.com` dan `fstream.binance.com`. Jika Binance disekat di lokasi/rangkaian anda, guna VPN, atau tukar `REST_BASE`/`WS_BASE` dalam `index.html` kepada exchange lain yang senaraikan XAUT perpetual (contoh Bybit) — struktur kod sama, cuma format mesej WebSocket perlu disesuaikan.

## Deploy ke GitHub Pages

```bash
git init
git add .
git commit -m "Initial commit: XAUT/USDT live dashboard"
git branch -M main
git remote add origin https://github.com/<username>/<repo>.git
git push -u origin main
```

Kemudian di GitHub:
1. Buka repo → **Settings** → **Pages**
2. Bawah **Build and deployment**, pilih **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)` → **Save**
4. Tunggu ~1 minit, dashboard akan live di `https://<username>.github.io/<repo>/`

## Struktur fail

```
xau-dashboard/
├── index.html                    ← dashboard web penuh (chart, WebSocket, S/R, Fib, signal, whale wall)
├── xaut_sr_fib_signals.pine      ← versi Pine Script v6 untuk dipasang terus di TradingView
└── README.md
```

## Guna di TradingView (Pine Script)

`xaut_sr_fib_signals.pine` ialah port dari logik dashboard web ni ke Pine Script v6, supaya boleh jalan terus atas chart TradingView sendiri (contoh chart XAUTUSDT/PAXGUSDT).

**Cara pasang:**
1. Buka TradingView → buka chart symbol yang anda nak (contoh `XAUTUSDT`)
2. Klik **Pine Editor** (tab bawah skrin)
3. Padam kandungan default, **copy-paste** semua isi kandungan `xaut_sr_fib_signals.pine` ke dalam editor
4. Klik **Add to Chart**
5. Kalau ada garis merah (error) dalam console Pine Editor bawah, copy mesej error tu dan hantar kat saya — saya boleh baiki terus (saya tak boleh jalankan Pine Script dalam sandbox saya sendiri untuk test, jadi ada sedikit risiko silap sintaks halus yang cuma nampak bila betul-betul di-compile TradingView)
6. Untuk dapat notifikasi automatik, klik ikon 🔔 (Alert) di chart → pilih indicator ni → pilih kondisi (Buy Entry / Sell Entry / Breakout / Re-entry / Pullback)

**Ciri yang sama dengan dashboard web:**
- Support/Resistance kuat/lemah (auto-cluster dari swing pivot)
- Fibonacci Retracement + Confluence (★ bila Fib bertindih S/R)
- Buy / Sell / Re-entry / Breakout / Pullback markers
- TP/SL (ATR-based)
- Panel "Signal Terkini" (table di penjuru kanan-atas chart)
- Alert bawaan TradingView untuk semua jenis signal

**Ciri yang TAK ada (had platform TradingView, bukan had kod):**
- ❌ **Big trader whale wall** — Pine Script tiada akses kepada data order book (Level 2), walaupun plan berbayar. Ni sebab utama dashboard web berasingan ni masih berguna — ia baca terus dari Binance order book yang Pine tak boleh capai.

Semua input (toleransi cluster, ATR period, breakout threshold, dll) boleh ditala terus dalam panel settings indicator (klik ikon gear ⚙️ selepas add to chart) — tak perlu edit kod.


## Nak ubah suai

Semua parameter tuning ada di bahagian atas `<script>` dalam `index.html`:

```js
const SWING_LOOKBACK = 3;        // ketat/longgar pengesanan swing point
const SR_TOLERANCE_PCT = 0.0012; // toleransi cluster S/R (0.12%)
const MAX_SR_LEVELS = 6;         // berapa banyak level S/R dipaparkan
const ATR_PERIOD = 14;           // period ATR untuk SL sizing
```

Tukar `SYMBOL` (baris atas) jika nak pasangan lain yang ada di Binance Futures (contoh `btcusdt`, `xauusdt`).
