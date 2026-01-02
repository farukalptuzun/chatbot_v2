# ChatBotv2 — Belge Destekli Chatbot

Bu proje, **MongoDB** tabanlı hafıza ve **RAG (Retrieval Augmented Generation)** ile çalışan, bağlamsal anlama yeteneğine sahip bir **chatbot** uygulamasıdır.  
Sohbetler bağlamsaldır: Bot, önceki mesajları dikkate alır ve aynı ürün/konu üzerinde devam edebilir. Belgeleriniz, GUI üzerinden yüklenip **vektör veritabanına** işlenir.

> **Not:** `tools/unified_rag_gui.py` şu an **metin tabanlı** formatları (PDF/DOCX/Excel/CSV ve web sayfası metni) destekler. Görsel OCR (JPG/PNG) devre dışıdır.

---

## 📋 İçindekiler

1. [Genel Bakış](#genel-bakış)
2. [Teknolojiler ve Kütüphaneler](#teknolojiler-ve-kütüphaneler)
3. [Sistem Mimarisi](#sistem-mimarisi)
4. [İş Akışı (Turn Processing)](#iş-akışı-turn-processing)
5. [Ana Bileşenler](#ana-bileşenler)
6. [Veri Akışı](#veri-akışı)
7. [Özellikler](#özellikler)
8. [Kurulum](#kurulum)
9. [Kullanım](#kullanım)
10. [Sorun Giderme](#sorun-giderme)

---

## 🎯 Genel Bakış

Chatbot sistemi, modern NLP ve RAG teknolojilerini kullanarak doğal dilde sohbet edebilen, bağlamsal hafızaya sahip ve bilgi tabanından (Knowledge Base) bilgi çekerek yanıt üreten bir yapıdır.

### Temel Özellikler

- ✅ **Bağlamsal Anlama**: Önceki mesajları hatırlayarak bağlamı korur
- ✅ **RAG (Retrieval Augmented Generation)**: MongoDB'deki vektör veritabanından bilgi çeker
- ✅ **Çoklu Intent Desteği**: Ürün bilgisi, sipariş, stok sorgusu vb. farklı niyetleri anlar
- ✅ **Slot Filling**: Eksik bilgileri otomatik olarak sorarak tamamlar
- ✅ **Skill-based Architecture**: Modüler yapıda özel işlevler (skills) eklenebilir
- ✅ **Ürün Entegrasyonu**: MongoDB'deki ürün veritabanı ile entegre çalışır
- ✅ **Hafıza Yönetimi**: Rolling summary ile konuşma geçmişini özetler
- ✅ **Çoklu LLM Desteği**: OpenAI, Anthropic (Claude), Google Gemini
- ✅ **Hybrid Conversation History**: Son mesajlar tam, eski mesajlar özetlenmiş

---

## 🛠️ Teknolojiler ve Kütüphaneler

### Core Framework
- **Python 3.12.8**: Ana programlama dili
- **Pydantic 2.x**: Veri validasyonu ve şema yönetimi
- **FastAPI**: REST API sunucusu
- **Uvicorn**: ASGI web sunucusu

### Veritabanı ve Depolama
- **MongoDB**: Ana veritabanı (sohbet geçmişi, ürünler, knowledge base)
- **PyMongo**: MongoDB Python driver
- **Elasticsearch 8.x** (Opsiyonel): Gelişmiş arama özellikleri

### NLP ve LLM
- **OpenAI API**: GPT-4, GPT-4o-mini modelleri
- **Anthropic API**: Claude 3 (Haiku, Sonnet, Opus)
- **Google Gemini API**: Gemini Flash, Gemini Pro
- **Sentence Transformers**: Multilingual embedding modelleri
- **HuggingFace Transformers**: NLP modelleri

### RAG ve Embeddings
- **sentence-transformers/paraphrase-multilingual-mpnet-base-v2**: Çok dilli embedding modeli
- **PyTorch**: Deep learning framework
- **NumPy**: Sayısal hesaplamalar

### Dosya İşleme
- **PyMuPDF**: PDF okuma
- **python-docx**: DOCX dosya işleme
- **pandas**: Excel/CSV işleme
- **BeautifulSoup4**: Web scraping
- **Pillow**: Görsel işleme

### Diğer
- **python-dotenv**: Ortam değişkenleri yönetimi
- **httpx**: HTTP istemcisi (async)
- **rich**: Terminal formatlama

---

## 🏗️ Sistem Mimarisi

```
┌─────────────────────────────────────────────────────────────┐
│                    Kullanıcı Mesajı                          │
│                    (Web UI / CLI / API)                       │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  NLU (Natural Language Understanding)                       │
│  - Intent Classification (LLM)                               │
│  - Entity Extraction (Ürün adı, renk, beden vb.)            │
│  - Slot Extraction (Sipariş için gerekli alanlar)           │
│  - Memory Integration (Önceki slotlar)                       │
│  - Schema Validation                                         │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  Dialog Manager                                              │
│  - Context Building (Memory snapshot)                        │
│  - Query Rewriting (Focus rewrite)                           │
│  - Skill Selection (Registry-based)                          │
│  - Plan Generation (ResponsePlan)                            │
│  - Slot Filling Management                                   │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  RAG Retriever                                               │
│  - Vector Search (MongoDB)                                    │
│  - Query Expansion (Multi-query)                              │
│  - Product Search (MongoDB Products Collection)              │
│  - Knowledge Base Search                                     │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  Response Generator                                          │
│  - System Prompt Building                                    │
│  - User Prompt Building                                      │
│  - LLM Call (OpenAI/Anthropic/Gemini)                        │
│  - Post-processing (Normalization, Shortening)               │
│  - Policy Template Rendering                                  │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  Memory System                                               │
│  - Conversation History (Son N mesaj)                        │
│  - Conversation Summary (Eski mesajların özeti)              │
│  - Slot Persistence                                          │
│  - Active Orders (Product/Service)                            │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
                  Bot Yanıtı
```

---

## 🔄 İş Akışı (Turn Processing)

Her kullanıcı mesajı için aşağıdaki adımlar sırayla gerçekleşir:

### 1. **Mesaj Alınması**
- Kullanıcıdan gelen ham metin alınır
- Session ID ile hafıza yüklenir
- User ID kontrolü yapılır (multi-tenant desteği)

### 2. **NLU Analizi** (`nlu/analyzer.py`)

**Yapılanlar:**
- Metin temizleme ve normalize etme
- LLM ile intent classification (niyet tespiti)
- Entity extraction (ürün adı, renk, beden vb.)
- Slot extraction (sipariş için gerekli alanlar)
- Hafıza ile birleştirme (önceki slotlar)
- Şema doğrulama (coerce_and_validate)
- Ürün-türe özel attribute çıkarımı (kapasite, ağırlık, depolama vb.)

**Intent Örnekleri:**
- `ask_product_info`: Ürün bilgisi soruluyor
- `ask_price`: Fiyat soruluyor
- `ask_availability`: Stok durumu soruluyor
- `place_order`: Sipariş verilmek isteniyor
- `order_service`: Hizmet siparişi
- `book_appointment`: Randevu rezervasyonu
- `smalltalk_greeting`: Selamlama
- `bot_info`: Bot hakkında bilgi

**Çıktı:**
```python
NLUResult(
    intent=Intent.ask_product_info,
    intent_specific_info={
        "product_name": "25CrMo4",
        "color": "mavi",
        "size": "M"
    },
    accuracy=0.85
)
```

### 3. **Context Oluşturma** (`dialog/manager.py`)

**Yapılanlar:**
- Memory snapshot alınır (önceki konuşmalar)
- TurnContext oluşturulur
- Metin yeniden yazılır (focus rewrite):
  - Eğer follow-up mesajıysa, önceki ürün/grade ismini ekler
  - Örnek: "detay" → "25CrMo4 — detay"
- Conversation history hazırlanır (son N mesaj)
- Conversation summary kontrol edilir (eski mesajların özeti)

### 4. **Skill Proposals** (`dialog/registry.py`)

**Sistem:**
- Kayıtlı tüm skills kontrol edilir
- Her skill `can_handle()` ile uygunluğunu kontrol eder
- Uygun skills `propose()` ile öneri sunar
- Skorlama: `priority + 0.3 * accuracy - 0.1 * missing_slots`
- En yüksek skorlu skill seçilir

**Skill Örnekleri:**
- `EcommercePlaceOrderSkill`: Ürün siparişi
- `EcommerceAvailabilitySkill`: Stok sorgusu

### 5. **Plan Generation**

**Plan Tipleri:**
- `ANSWER_FROM_KB`: Knowledge base'den cevap
- `CREATE_ORDER`: Sipariş oluşturma
- `CREATE_APPOINTMENT`: Randevu oluşturma
- `GENERIC_INFO`: Genel bilgi
- `SLOTFILL`: Eksik bilgi toplama

### 6. **RAG Retrieval**

**Yapılanlar:**
- Query expansion (multi-query)
- Vector search (MongoDB)
- Product search (MongoDB Products Collection)
- Knowledge base search
- User ID filtresi (multi-tenant)

### 7. **Response Generation** (`dialog/generator.py`)

**Yapılanlar:**
- System prompt oluşturulur
- User prompt oluşturulur
- Conversation history eklenir
- LLM çağrısı yapılır (OpenAI/Anthropic/Gemini)
- Post-processing (normalization, shortening)
- Policy template rendering

### 8. **Memory Update**

**Yapılanlar:**
- Conversation message eklenir
- Conversation summary güncellenir (gerekirse)
- Slot'lar kaydedilir
- Active orders güncellenir

---

## 🧩 Ana Bileşenler

### NLU (Natural Language Understanding)

**Dosya:** `nlu/analyzer.py`

**Sorumluluklar:**
- Intent classification
- Entity extraction
- Slot extraction
- Schema validation

**Kullanılan LLM:**
- Intent classification için LLM kullanılır
- Model seçimi: `LLM_MODEL_CLASSIFIER` env variable

### Dialog Manager

**Dosya:** `dialog/manager.py`

**Sorumluluklar:**
- Turn processing orchestration
- Context building
- Skill selection
- Plan generation
- Slot filling management

**Ana Fonksiyon:**
```python
async def run_turn(text: str, session_id: str, user_id: Optional[str] = None) -> Dict[str, Any]
```

### Memory System

**Dosya:** `nlu/memory/memory_store.py`

**Sorumluluklar:**
- Conversation history yönetimi
- Conversation summary (rolling summary)
- Slot persistence
- Active orders (product/service)

**Veri Yapısı:**
```python
class DialogMemory(BaseModel):
    session_id: str
    turn: int
    conversation_messages: List[ConversationMessage]
    conversation_summary: str
    last_summary_turn: int
    active_product_order: Optional[ProductOrderFrame]
    active_service_order: Optional[ServiceOrderFrame]
    recent_entities: List[RecentEntity]
    missing_slot_warnings: Dict[str, int]
```

### RAG Retriever

**Dosya:** `services/retriever/mongo_retriever.py`

**Sorumluluklar:**
- Vector search (MongoDB)
- Query expansion (multi-query)
- Product search
- Knowledge base search

**Özellikler:**
- User ID filtresi (multi-tenant)
- Intent-aware query expansion
- Top-K retrieval

### Response Generator

**Dosya:** `dialog/generator.py`

**Sorumluluklar:**
- System prompt building
- User prompt building
- LLM call orchestration
- Post-processing
- Policy template rendering

**Özellikler:**
- Multi-provider support (OpenAI, Anthropic, Gemini)
- Conversation history integration
- KB-first approach (genel bilgi vermeme)
- Contextual interpretation (firma bağlamında yorumlama)

### Skills System

**Dosya:** `dialog/registry.py`, `skills/`

**Sorumluluklar:**
- Modüler işlev ekleme
- Skill selection
- Skill execution

**Mevcut Skills:**
- `EcommercePlaceOrderSkill`: Ürün siparişi
- `EcommerceAvailabilitySkill`: Stok sorgusu

---

## 📊 Veri Akışı

### 1. Belge Yükleme → Knowledge Base

```
PDF/DOCX/Excel/CSV/Web URL
    ↓
Text Extraction
    ↓
Chunking (Parçalama)
    ↓
Embedding (Vectorization)
    ↓
MongoDB Knowledge Collection
```

### 2. Kullanıcı Mesajı → Bot Yanıtı

```
User Message
    ↓
NLU Analysis
    ↓
Context Building
    ↓
Skill Selection / Plan Generation
    ↓
RAG Retrieval (KB Search)
    ↓
LLM Generation
    ↓
Post-processing
    ↓
Bot Response
```

### 3. Conversation History Management

```
Conversation Messages (List)
    ↓
Threshold Check (15 messages)
    ↓
Summarization (LLM)
    ↓
Conversation Summary (String)
    ↓
Recent Messages (Last 8) + Summary
```

---

## ✨ Özellikler

### 1. Bağlamsal Anlama
- Önceki mesajları hatırlar
- Aynı ürün/konu üzerinde devam edebilir
- Conversation history ile bağlam korunur

### 2. RAG (Retrieval Augmented Generation)
- MongoDB vektör veritabanından bilgi çeker
- Multi-query expansion ile daha iyi sonuçlar
- User ID filtresi ile multi-tenant desteği

### 3. Hybrid Conversation History
- Son 8 mesaj tam format
- Eski mesajlar LLM ile özetlenmiş
- Token optimizasyonu

### 4. KB-First Approach
- Genel bilgi vermez
- Sadece knowledge base'deki bilgileri kullanır
- Firma bağlamında yorumlama

### 5. Multi-LLM Support
- OpenAI (GPT-4, GPT-4o-mini)
- Anthropic (Claude 3 Haiku, Sonnet, Opus)
- Google Gemini (Flash, Pro)

### 6. Slot Filling
- Eksik bilgileri otomatik sorar
- Memory'de saklar
- Auto-stop detection

### 7. Product Integration
- MongoDB Products Collection
- Product search
- Price/availability queries

### 8. Appointment System
- Randevu rezervasyonu
- Slot filling (ad, telefon, e-posta)
- Appointment service integration

---

## 📦 Gereksinimler

- **Python 3.12.8** (test edilen sürüm)
- **MongoDB** (lokal kurulum ve çalışır durumda)
- `requirements.txt` içindeki Python paketleri

> Diğer Python sürümleriyle de çalışabilir, ancak **3.12.8** önerilir.

---

## 🚀 Kurulum

### 1. Sanal Ortam Oluşturma

```bash
# Python sürümünü kontrol et
python --version  # 3.12.8 olmalı

# Sanal ortam oluştur
python -m venv .venv

# Sanal ortamı etkinleştir
# Windows (PowerShell):
.venv\Scripts\Activate.ps1
# Windows (CMD):
.venv\Scripts\activate.bat
# macOS / Linux:
source .venv/bin/activate
```

### 2. Bağımlılıkları Yükleme

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 3. .env Dosyası Oluşturma

Proje kök dizininde `.env` dosyası oluşturun:

```env
# LLM Provider (openai, anthropic, gemini)
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-xxxx
LLM_MODEL=claude-3-haiku-20240307
LLM_CLASSIFIER=true

# MongoDB
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/chatbot_nlu?retryWrites=true&w=majority
DATABASE_NAME=chatbot_nlu

# RAG / Vektör veritabanı
RETRIEVER_BACKEND=mongo
STRICT_RAG_MODE=true
MONGO_VECTOR_COLLECTION=knowledge
MONGO_VECTOR_TEXT_FIELD=text
MONGO_VECTOR_EMB_FIELD=embedding
MONGO_VECTOR_MODE=local

# Embeddings
EMBEDDINGS_PROVIDER=hf
EMBEDDINGS_MODEL=sentence-transformers/paraphrase-multilingual-mpnet-base-v2
RETRIEVER_TOPK=5

# Yanıt üretimi
REPLY_MAX_TOKENS=1200
REPLY_CHAR_LIMIT=2200
REPLY_TEMPERATURE=0.2
REPLY_TONE=friendly

# Conversation History
CONVERSATION_SUMMARY_ENABLED=1
CONVERSATION_SUMMARY_THRESHOLD=15
RECENT_MESSAGES_COUNT=8

# Debug
DEBUG_MODE=false
```

### 4. MongoDB Veritabanı Oluşturma

```bash
mongosh
> use chatbot_nlu
```

### 5. Belgeleri Yükleme

```bash
python tools/unified_rag_gui.py
```

GUI'den belgeleri yükleyin (PDF, DOCX, Excel, CSV, Web URL).

### 6. Chatbot'u Başlatma

```bash
python main.py
```

---

## 💬 Kullanım

### CLI Kullanımı

```bash
python main.py
```

Terminal'de sohbet edebilirsiniz.

### Web UI

```bash
python tools/web_chat.py
```

Web arayüzü üzerinden sohbet edebilirsiniz.

### API Server

```bash
python tools/unified_rag_api.py
```

REST API sunucusu başlatılır.

### Örnek Sohbet Akışı

```
Kullanıcı: Merhaba
Bot: Merhaba! Size nasıl yardımcı olabilirim?

Kullanıcı: 25CrMo4 hakkında bilgi verir misiniz?
Bot: 25CrMo4, krom-molibden alaşımlı çelik... [KB'den bilgi]

Kullanıcı: Fiyatı nedir?
Bot: 25CrMo4 için liste fiyatı 150 TL. [Bağlam korundu]

Kullanıcı: Bu ürünü satın almak istiyorum
Bot: Sipariş için gerekli bilgileri topluyorum... [Slot filling]
```

---

## 🗄️ Veritabanı Yapısı

### MongoDB Collections

1. **knowledge**: Vektör veritabanı (belgeler, embeddings)
2. **products**: Ürün veritabanı
3. **files**: Yüklenen dosyalar
4. **behaviour**: Bot davranış ayarları (user_id ile)
5. **manipulation**: Manipülasyon talimatları (user_id ile)
6. **conversations**: Sohbet geçmişi (opsiyonel)

### Veri Modelleri

**DialogMemory:**
- `session_id`: Oturum ID
- `conversation_messages`: Son mesajlar
- `conversation_summary`: Eski mesajların özeti
- `active_product_order`: Aktif ürün siparişi
- `active_service_order`: Aktif hizmet siparişi

---

## 🛠️ Sorun Giderme

### MongoDB Bağlantı Hatası
- `MONGODB_URI` doğru mu?
- MongoDB servisi çalışıyor mu?
- Network bağlantısı var mı?

### API Anahtarı Hatası
- `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` / `GEMINI_API_KEY` dolduruldu mu?
- API anahtarı geçerli mi?

### Belge İndeksleme Sorunu
- GUI'den yükleme tamamlandı mı?
- Konsolda hata var mı?
- MongoDB'de `knowledge` collection'ı var mı?

### venv Etkin Değil
- Terminal başında `(.venv)` görünüyor mu?
- `which python` / `where python` çıktısı `.venv` içini gösteriyor mu?

### Türkçe Karakter Sorunu
- Python 3.12.8 kullanın
- UTF-8 terminal kullanın
- `.env` dosyası UTF-8 encoding ile kaydedildi mi?

### LLM Yanıt Vermiyor
- `DEBUG_MODE=true` ile logları kontrol edin
- API quota'sı dolmuş olabilir
- Model adı doğru mu?

---

## 📁 Proje Yapısı

```
.
├─ main.py                      # Chatbot giriş noktası
├─ requirements.txt             # Bağımlılıklar
├─ .env                         # Ortam değişkenleri
├─ dialog/                      # Dialog yönetimi
│  ├─ manager.py               # Ana dialog manager
│  ├─ generator.py             # Yanıt üretici
│  ├─ plan.py                  # Plan modelleri
│  ├─ registry.py              # Skill registry
│  └─ slotfilling.py           # Slot filling
├─ nlu/                         # Natural Language Understanding
│  ├─ analyzer.py              # NLU analiz motoru
│  ├─ intents.py               # Intent tanımları
│  ├─ llm_providers.py         # LLM provider'ları
│  └─ memory/                   # Hafıza sistemi
│     ├─ memory_store.py       # Hafıza yönetimi
│     ├─ models.py             # Hafıza modelleri
│     └─ summary.py            # Konuşma özetleme
├─ services/                    # Servisler
│  ├─ retriever/                # RAG retriever
│  │  ├─ mongo_retriever.py    # MongoDB retriever
│  │  └─ query_expansion.py    # Query expansion
│  ├─ products_repo.py         # Ürün repository
│  └─ order_service.py         # Sipariş servisi
├─ skills/                      # Skills (modüler işlevler)
│  ├─ ecommerce_place_order.py
│  └─ ecommerce_availability.py
└─ tools/                       # Araçlar
   ├─ unified_rag_gui.py       # Belge yükleme GUI
   ├─ unified_rag_api.py       # REST API
   └─ web_chat.py              # Web UI
```

---

## 📜 Lisans

Bu proje ekip içi kullanım için hazırlanmıştır. Harici dağıtım/lisans bilgisi eklenecektir.

---

## 🤝 Katkıda Bulunma

1. Fork yapın
2. Feature branch oluşturun (`git checkout -b feature/amazing-feature`)
3. Commit yapın (`git commit -m 'Add amazing feature'`)
4. Push yapın (`git push origin feature/amazing-feature`)
5. Pull Request açın

---

## 📞 İletişim

Sorularınız için issue açabilir veya ekip ile iletişime geçebilirsiniz.
