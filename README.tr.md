# ERPilot AI — Ürün ve Mimari Vitrini 🤖📊

[English](README.md) | **Türkçe**

**Şirketler için yapay zekâ destekli finans asistanı ve anomali tespit platformu. BTK Akademi Hackathon 2026 finans kategorisi için geliştirilmiştir.**

Sorunuzu doğal dille sorarsınız ("Ocak'ta en çok hangi departman harcadı?"); sistem gerçek finansal veriyi sorgular, grafiklerini çizer, her grafiğe anında yorum yazar ve olağandışı harcamaları işaretler.

> ℹ️ **Bu Vitrin Hakkında:**  
> Bu depo **ürün tanıtımı ve teknik portföy** içindir. Uygulama kaynak kodları (FastAPI backend, veri sorgulama motoru ve prompt guardrail katmanları) ayrı bir **private** depoda tutulur. Bu depoda sistem mimarisi, akış şemaları ve arayüz ekran görüntüleri yayımlanmaktadır.

---

## 📸 Ekran Görüntüleri ve Arayüz Turu

| Genel Bakış (Gelir, Gider, Kâr ve Veriyle Sohbet) | Bütçe ve Gerçekleşen Harcamalar |
|---|---|
| ![Genel bakış](docs/img/genel-tr.png) | ![Bütçe ve gerçekleşen](docs/img/butce-tr.png) |

| Alışılmadık Harcamalar (IQR Anomali Tespiti) | Veriye Sor (Doğal Dil ile Finansal Sorgulama) |
|---|---|
| ![Anomaliler](docs/img/anomali-tr.png) | ![Soru sorma](docs/img/sor-tr.png) |

| Koyu Tema (Dark Mode) | İngilizce Arayüz |
|---|---|
| ![Koyu tema](docs/img/genel-dark.png) | ![İngilizce genel bakış](docs/img/genel-en.png) |

---

## 🎯 Öne Çıkan Özellikler

- **Veriye Sor (Natural Language Querying):** Google Gemini modeli soruyu yapısal bir sorgu nesnesine (`QuerySpec`) çevirir; sayılar modelin halüsinasyonundan değil, doğrulanmış gerçek veriden gelir.
- **Güvenli Tasarım (Zero SQL/Code Injection):** Model asla doğrudan SQL veya Python kodu yazamaz; yalnızca backend'in kontrol ettiği katı `allowlist` kalıplarını doldurur. Konu dışı veya enjeksiyon amaçlı sorular mimari düzeyde reddedilir.
- **Anomali Tespiti (Outlier Detection):** Normal dağılım aralığının dışındaki harcamalar IQR (Interquartile Range) istatistiksel yöntemiyle anında tespit edilir ve kırmızıyla işaretlenir.
- **Yapay Zekâ Yorumları & Kalıcı Önbellek:** Her grafiğin altında kısa analitik açıklamalar üretilir. Gemini API anahtarı olmadığında dahi yerel deterministik motor aynı hesaplamalarla yorum yazar.
- **Çoklu Dil & Tema:** Türkçe ve İngilizce dil desteği, açık ve koyu tema seçenekleri.

---

## 🏗️ Sistem Mimarisi

```mermaid
flowchart TB
    subgraph FE[Frontend - SPA Arayüz]
        UI["6 Sayfalı Arayüz (Chart.js)<br/>Dark/Light Tema + TR/EN Desteği"]
    end
    subgraph BE[FastAPI Backend Katmanı]
        API["REST API Endpoint'leri<br/>/api/chat, /api/dashboard, /api/anomalies"]
        RL["Rate Limiter & TTL Cache"]
        GC["Gemini Entegrasyonu (Function Calling)"]
        QE["Query Engine (Allowlist Doğrulama)"]
        AN["Analitik & IQR Anomali Motoru"]
        DS["DataSource Soyutlama Katmanı"]
    end
    subgraph DATA[Veri Kaynakları]
        CSV[("CSV Demo Verisi")]
        SAP[("SAP S/4HANA (OData REST / BAPI)")]
    end

    UI -->|HTTPS REST| API
    API --> RL --> GC
    API --> AN
    GC --> QE
    QE --> DS
    AN --> DS
    DS --> CSV
    DS -.Gelecek Planı.-> SAP
```

### "Veriye Sor" Akışı (Function Calling + Guardrail Güvenliği)

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant F as Frontend
    participant B as FastAPI
    participant G as Gemini AI
    participant Q as Query Engine
    participant D as Veri Motoru (pandas)

    U->>F: Doğal dille finans sorusu sorar
    F->>B: POST /api/chat (mesaj, dil)
    B->>G: Soru + query_finance_data fonksiyon şeması
    alt Konu dışı soru (sohbet, genel kültür)
        G-->>B: Fonksiyon çağrılmaz
        B-->>F: Sabit ret ve yönlendirme cevabı
    else Finansal veri sorusu
        G-->>B: query_finance_data(QuerySpec)
        B->>Q: QuerySpec doğrula (katı Allowlist kontrolü)
        Q->>D: Güvenli pandas filtreleme
        D-->>Q: Doğrulanmış gerçek sayılar
        Q-->>B: Hesaplama sonucu
        B->>G: Yanıtı özetle (kullanıcının serbest metni olmadan)
        G-->>B: Doğal dil formatında yanıt
        B-->>F: Doğrulanmış yanıt ve grafik
    end
```

---

## 💻 Kullanılan Teknolojiler

| Katman | Teknoloji |
|---|---|
| **Backend** | Python, FastAPI, Pydantic, pandas, NumPy, SlowAPI |
| **Yapay Zekâ** | Google Gemini (Function Calling / Structured Outputs), Yerel Deterministik Yorum Motoru |
| **Arayüz (Frontend)** | Vanilla JavaScript, Chart.js, HTML5, CSS3, Modern Responsive Layout |
| **Veri Kaynağı** | Normalize Veri Modelleri, CSV (Gelecekte SAP S/4HANA OData Entegrasyonu) |
| **Mimari Dokümanları** | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) · [docs/DEMO.md](docs/DEMO.md) · [docs/pitch.md](docs/pitch.md) |

---

## 🔒 Kaynak Kodu Bildirimi

Bu depoda uygulamanın kaynak kodları, veri tabanı modelleri veya API anahtarları **bulunmaz**. Depo yalnızca mimari inceleme ve teknik vitrin niteliğindedir.
