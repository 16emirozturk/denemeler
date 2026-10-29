# Güvenli Okul — Bildirim ve Takip Platformu (v2)

Okullarda akran zorbalığı bildirimlerini toplamak, **insan incelemesi** için önceliklendirmek ve süreci
izlenebilir kılmak için hazırlanmış bir Streamlit prototipidir. Karar veren bir sistem **değildir**.

## Çalıştırma
```bash
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .streamlit/secrets.toml.example .streamlit/secrets.toml
python tools/parola_hash.py                            # çıkan password_hash'i secrets.toml'a yazın
streamlit run app.py
```
Testler (ek bağımlılık gerekmez): `python -m unittest discover -s testler -v`

## Yapı
| Dosya | Görev |
|---|---|
| `app.py` | Arayüz ve sayfa akışı |
| `db.py` | SQLite katmanı: şema göçü, işlemler, istatistik, CSV, saklama süresi |
| `guvenlik.py` | Parola özeti/doğrulama, kişisel veri maskeleme, CSV enjeksiyon koruması |
| `hakaret_tespiti.py` | Kural tabanlı sinyal: kategori, öncelik, bağlam uyarısı |
| `degerlendirme.py` | Sentetik etiketli küme + precision/recall/F1 |
| `stil.py` | CSS (harici yazı tipi/CDN yok) |
| `testler/` | Birim testleri |

## v1 → v2: düzeltilen hatalar
- Türkçe karakterli kullanıcı adı/parola `hmac.compare_digest` ile `TypeError` veriyordu → UTF-8 bayt karşılaştırması.
- Sayfa düğmeleri `session_state["ana_menu"]` değerini widget oluştuktan sonra değiştiriyordu → `on_click` geri çağrıları.
- `st.success(); st.rerun()` mesajı yutuyordu → `flash` mekanizması.
- `clear_on_submit=True` doğrulama hatasında da yazılan metni siliyordu → yalnızca başarıda form sıfırlanır.
- `with sqlite3.connect()` bağlantıyı kapatmıyordu → her işlemde kapatan bağlam yöneticisi.
- Saatler sunucu saatindeydi (bulutta UTC) → Europe/Istanbul.
- `ilk_acilista_kayitlari_temizle` bayrak yoksa TÜM kayıtları siliyordu → kaldırıldı; yerine şema sürümü + göç.
- Algılayıcı "gerizekalı" yazımını kaçırıyordu (sözlükte ASCII) → Türkçe'ye duyarlı katlama.
- Bozuk Markdown (`** Gizlilik**`), boş ikon kutuları, çift "10." bölüm numarası, harici Google Fonts çağrısı.

## Yeni özellikler
Öncelik sinyali ve kuyruk sıralaması · kişisel veri maskeleme (T.C. no sağlama toplamı dâhil) · takip kodu ile
durum sorgulama · hatalı giriş kilidi + oturum zaman aşımı + PBKDF2 parola özeti · denetim kaydı ·
yönetici işlemlerinde "kim yaptı" bilgisi · CSV dışa aktarım (kimlik varsayılan hariç) · saklama süresi temizliği ·
anonim kayıtlarda zaman yuvarlama · yanıt süresi metrikleri · otomatik algılayıcı değerlendirmesi.

## Bilimsel doğrulama yol haritası (TÜBİTAK için)
1. Araştırma sorusu ve hipotezlerin yazılması (ör. "Kural tabanlı sinyal, yönetici inceleme sırasını iyileştirir mi?").
2. Etik kurul / okul ve veli izniyle, kimliksizleştirilmiş gerçek veya uzmanca üretilmiş bir veri kümesi.
3. En az iki uzmanın (rehber öğretmen/psikolojik danışman) bağımsız etiketlemesi; Cohen kappa raporu.
4. Eğitim/doğrulama/test ayrımı; kural tabanlı yöntemin başka bir yöntemle (ör. sınıflandırıcı) karşılaştırması.
5. Kullanılabilirlik testi (SUS) ve yanıt süresi ölçümü.
6. Yanlış alarm/kaçırma analizi ve sınırlılıkların açıkça raporlanması.

## Yayına almadan önce
KVKK (aydınlatma, hukuki sebep, saklama-imha, yurt dışı sunucu), kişi bazlı hesap + 2FA, HTTPS, şifreli yedek,
bağımsız güvenlik testi ve okul acil durum protokolü gereklidir. Acil hat numaralarını (`sabitler.py`) resmi
kaynaklardan yeniden doğrulayın.
