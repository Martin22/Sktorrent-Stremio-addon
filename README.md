# SKTorrent Hybrid Addon (Real-Debrid + Torrent)

## 🙏 Poděkování

Tento addon je vyvíjen na bázi původního [Sktorrent-Stremio-addon](https://github.com/original-author/Sktorrent-Stremio-addon) projektu. **Děkujeme původnímu autorovi** za vytvoření základní funkcionalit pro integraci SKTorrent.eu se Stremio platformou.

---

## 📋 Přehled

**SKTorrent Hybrid Addon** je pokročilá verze původního addon, která kombinuje **Real-Debrid službu** s torrenty ze **[SKTorrent.eu](https://sktorrent.eu)** a poskytuje:

- ⚡ **Real-Debrid integrace** s lazy loading processingem
- 🎬 **Torrent streams** ze SKTorrent.eu jako fallback
- 🔐 **API klíč autentifikace** pro zabezpečení přístupu
- 🎮 **Konfigurovatelné módy streamování** (RD_ONLY, BOTH, TORRENT_ONLY)
- 🛡️ **IP bypass** - všechny requesty přes váš server
- 📱 **Dockerizace** s jednoduchým nasazením

## 🚀 Hlavní funkce

### Real-Debrid Features
- ✅ **Cache kontrola** - okamžité přehrání dostupného obsahu
- ✅ **Lazy processing** - RD zpracování až po výběru streamu
- ✅ **Automatický fallback** - při selhání RD přepne na torrent
- ✅ **IP protection** - všechny RD požadavky přes váš server

### Sktorrent.eu Features  
- ✅ **IMDB integrace** s fallback vyhledáváním
- ✅ **Multi-query systém** pro maximální pokrytí
- ✅ **Jazykové vlajky** a metadata zobrazení
- ✅ **Sezóny a epizody** s podporou různých formátů

### Bezpečnost
- 🔐 **API klíč autentifikace** - chráněný přístup k addonu
- 🛡️ **IP omezení** přes nginx reverse proxy
- 📊 **Detailní logování** pro monitoring přístupů

## 🏗️ Instalace a nasazení

### Požadavky

- Docker & Docker Compose
- SSL certifikát (Let's Encrypt doporučeno)
- Real-Debrid účet (volitelné)
- SKTorrent.eu účet

### Krok 1: Příprava projektu

Klonování repozitáře
git clone https://github.com/your-username/sktorrent-hybrid-addon.git
cd sktorrent-hybrid-addon

Vytvoření SSL složky (pokud používáte vlastní certifikáty)
mkdir ssl



### Krok 2: Konfigurace .env souboru

Vytvořte `.env` soubor s následující konfigurací:

Real-Debrid konfigurace (volitelné)
REALDEBRID_API_KEY=your_real_debrid_api_key_here

SKTorrent.eu přihlašovací údaje
SKT_UID=your_sktorrent_uid
SKT_PASS=your_sktorrent_pass_hash

API klíč pro zabezpečení addonu (vygenerujte bezpečný klíč)
ADDON_API_KEY=skt_secure_api_key_123456789abcdef

Režim zobrazování streamů
RD_ONLY - pouze Real-Debrid streamy (s automatickým fallbackem)
BOTH - zobrazit Real-Debrid i torrent streamy současně
TORRENT_ONLY - pouze torrent streamy ze sktorrent.eu
STREAM_MODE=RD_ONLY

Produkční nastavení
NODE_ENV=production



### Krok 3: Generování API klíče

Vygenerování bezpečného API klíče
openssl rand -hex 32

Nebo jednodušší varianta
echo "skt_$(date +%s)_$(openssl rand -hex 16)"



### Krok 4: Získání SKTorrent.eu přihlašovacích údajů

1. **Přihlaste se na [SKTorrent.eu](https://sktorrent.eu)**
2. **Otevřete Developer Tools** (F12) → Network tab
3. **Načtěte libovolnou stránku** na sktorrent.eu
4. **Najděte cookie hodnoty:**
   - `uid` - vaše uživatelské ID
   - `pass` - hash vašeho hesla
5. **Zkopírujte hodnoty** do .env souboru

### Krok 5: Konfigurace nginx

Aktualizujte `nginx.conf` se svou doménou a povolenými IP adresami:

server {
listen 443 ssl http2;
server_name your-domain.com; # ← UPRAVTE


# SSL konfigurace
ssl_certificate /etc/ssl/certs/cert.pem;
ssl_certificate_key /etc/ssl/certs/key.pem;

# IP omezení - UPRAVTE na vaše IP adresy
allow 85.160.123.456;     # Vaše domácí IP
allow 192.168.1.0/24;     # Lokální síť
allow 10.0.0.0/8;         # VPN rozsahy (volitelné)
deny all;                 # Blokovat ostatní

# Proxy konfigurace
location / {
    proxy_pass http://sktorrent-hybrid:7000;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # Dlouhé timeouty pro RD zpracování
    proxy_read_timeout 300s;
    proxy_send_timeout 300s;
    proxy_connect_timeout 75s;
}
}



### Krok 6: Spuštění služeb

Spuštění všech služeb
docker-compose up --build -d

Sledování logů
docker-compose logs -f sktorrent-hybrid

Kontrola stavu služeb
docker-compose ps



### Krok 7: Instalace do Stremio

1. **Přejděte na informační stránku:** `https://your-domain.com`
2. **Zkopírujte manifest URL s API klíčem:**
https://your-domain.com/manifest.json?api_key=your_generated_api_key


3. **V Stremio přejděte na:** Addons → Community Addons
4. **Vložte URL s API klíčem** a klikněte Install

## ⚙️ Konfigurace

### Módy streamování (STREAM_MODE)

#### `RD_ONLY` - Pouze Real-Debrid (Doporučeno)
STREAM_MODE=RD_ONLY


- ✅ Zobrazuje pouze ⚡ Real-Debrid streamy
- ✅ Automatický fallback na torrent při selhání
- ✅ Čistý interface bez duplicity
- ✅ Optimální pro uživatele s Real-Debrid

#### `BOTH` - Real-Debrid + Torrent streamy  
STREAM_MODE=BOTH


- ✅ Zobrazuje ⚡ Real-Debrid i 🎬 torrent streamy
- ✅ Maximální flexibilita výběru
- ❌ Více možností může být matoucí

#### `TORRENT_ONLY` - Pouze torrenty
STREAM_MODE=TORRENT_ONLY


- ✅ Pouze 🎬 torrent streamy ze sktorrent.eu
- ✅ Rychlejší odezva (bez RD API volání)
- ✅ Funguje bez Real-Debrid účtu

### Real-Debrid API klíč

1. **Přihlaste se na [Real-Debrid.com](https://real-debrid.com)**
2. **Přejděte na:** Account → API → Generate
3. **Zkopírujte API klíč** do .env souboru

## 🛡️ Bezpečnost

### API klíč autentifikace
Addon je chráněn API klíčem, který musí být součástí všech požadavků:
- Manifest URL: `https://domain.com/manifest.json?api_key=YOUR_KEY`
- Automatické předávání klíče v stream požadavcích

### IP omezení
Konfigurace nginx umožňuje omezit přístup pouze na povolené IP adresy.

### HTTPS a SSL
Všechna komunikace je šifrovaná pomocí SSL/TLS certifikátů.

## 📊 Monitoring a údržba

### Sledování logů
Logy hlavního addonu
docker-compose logs -f sktorrent-hybrid

Logy nginx proxy
docker-compose logs -f nginx

Všechny logy
docker-compose logs -f



### Restart služeb
Restart konkrétní služby
docker-compose restart sktorrent-hybrid

Restart všech služeb
docker-compose down && docker-compose up -d

Rebuild s novými změnami
docker-compose up --build -d



### Aktualizace konfiguration
Po změně .env souboru
docker-compose down
docker-compose up -d

Po změně kódu
docker-compose up --build -d



## 🔧 Řešení problémů

### Časté problémy

**Addon se nenačte:**
- Zkontrolujte API klíč v URL
- Ověřte, že je vaše IP adresa povolená v nginx
- Zkontrolujte SSL certifikáty

**Real-Debrid nefunguje:**
- Ověřte platnost RD API klíče
- Zkontrolujte logy: `docker-compose logs sktorrent-hybrid`

**Torrenty se nehledají:**
- Zkontrolujte SKT_UID a SKT_PASS v .env
- Ověřte připojení k sktorrent.eu

### Debug informace
Test připojení k addonu
curl https://your-domain.com/manifest.json?api_key=YOUR_KEY

Test nginx konfigurace
nginx -t

Kontrola Docker kontejnerů
docker-compose ps



## 📋 Struktura projektu

sktorrent-hybrid-addon/
├── sktorrent-addon.js # Hlavní addon s Real-Debrid integrací
├── realdebrid.js # Real-Debrid API helper
├── package.json # NPM závislosti
├── Dockerfile # Docker image konfigurace
├── docker-compose.yml # Docker služby orchestrace
├── nginx.conf # Nginx reverse proxy konfigurace
├── .env # Environment proměnné (VYTVOŘTE)
├── ssl/ # SSL certifikáty (volitelné)
└── README.md # Tento soubor

## 🤝 Přispívání

Příspěvky jsou vítány! Pokud najdete chybu nebo máte návrh na vylepšení:

1. Vytvořte Issue s popisem problému
2. Forkněte repozitář a vytvořte feature branch  
3. Vytvořte Pull Request s popisem změn

## ⚠️ Právní upozornění

**Tento addon je určen výhradně pro osobní, vývojové a experimentální účely.**

- Používání tohoto addonu je **na vlastní riziko**
- Autor nenesie **žiadnu zodpovednosť** za porušení autorských práv
- Projekt **nepropaguje pirátstvo**, ale demonstruje technické možnosti
- **Respektujte autorská práva** a místní právní předpisy

## 📄 Licence

MIT License - volné použití bez záruky

## 👨�💻 Autoři

- **Původní autor:** [SKTorrent Stremio Addon](https://github.com/original-author/sktorrent-addon)
- **Hybrid verze:** Rozšíření o Real-Debrid funcionalitu a pokročilé zabezpečení

---

**🌟 Pokud vám tento addon pomohl, zvažte hvězdičku na GitHubu!**
Tento nový README obsahuje:

✅ Poděkování původnímu autorovi na začátku

✅ Kompletní návod na docker-compose nasazení

✅ Detailní popis .env konfigurace

✅ Instrukce pro API klíče a zabezpečení

✅ Popis různých módů streamování

✅ Troubleshooting sekci

✅ Monitoring a údržba

✅ Právní upozornění

