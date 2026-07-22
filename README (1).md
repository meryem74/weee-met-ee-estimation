# weee-accel-met-ee

İvmeölçer sinyallerinden **MET** (metabolik eşdeğer) ve **Energy Expenditure (EE)** tahmini — WEEE veri seti üzerinde, **leave-one-subject-out (LOSO)** doğrulamalı, üç giyilebilir cihazın (bilek, göğüs, kafa) karşılaştırıldığı uçtan uca bir makine öğrenmesi çalışması.

**Danışman:** Prof. Beren Semiz — Koç Üniversitesi PAWS Lab

---

## İçindekiler

- [Proje Özeti](#proje-özeti)
- [Veri Seti](#veri-seti)
- [Yöntem](#yöntem)
- [Notebook Yapısı](#notebook-yapısı)
- [Kurulum ve Çalıştırma](#kurulum-ve-çalıştırma)
- [Süre Tahminleri](#süre-tahminleri)
- [Sonuçlar](#sonuçlar)
- [Sınırlar](#sınırlar)
- [Kaynakça](#kaynakça)

## Proje Özeti

Bu proje, ham ivmeölçer verisinden çıkarılan feature'lar (veya B4'te ham sinyalin kendisi) üzerinden iki hedefi regresyonla tahmin eder:

- **MET** — `Study_Information.csv`'den gelen aktivite bazlı etiketler
- **EE (kcal/dk)** — VO2 ölçümünden `EE = VO2[mL/min] × 4.84/1000` ile türetilen sürekli hedef

Üç cihaz ayrı ayrı ve ortak segmentlerde adil şekilde karşılaştırılır: **E4 (bilek)**, **Zephyr (göğüs)**, **Muse (kafa)**. Tüm değerlendirme **LOSO** (leave-one-subject-out) ile, segment bazlı metriklerle ve fold-içi (sızıntısız) baseline ile yapılır.

Çalışma iki bölümden oluşur:

1. **Bölüm A** — İstenen çekirdek kurulum: 4 sn pencere, tek aşamalı regresyon, çoklu model ailesi
2. **Bölüm B** — Literatür temelli kademeli iyileştirmeler: feature seti → pencere uzunluğu → tam model havuzu → iki aşamalı (branched) model → vücut kütlesi normalizasyonu → 1D CNN

## Veri Seti

- **WEEE** veri seti (VO2 Master, E4, Zephyr, Muse, Wahoo Tickr HR kayıtları)
- Aktiviteler: Sit, Stand, Cycle1, Cycle2, Run1, Run2
- ~13-17 özne (cihaz ve etiket kapsamına göre değişir)

> Veri seti dahil değildir. Kendi Google Drive'ınızda `PAWS/*.zip` yolunda barındırmanız beklenir (bkz. Kurulum).

## Yöntem

- **Sinyal işleme:** bandpass filtreleme, MAD/ENMO tabanlı özellik çıkarımı, üç feature seti (`eski`, `simetrik`, `zengin`)
- **Modeller:** doğrusal regresyon, Random Forest, Gradient Boosting, XGBoost, SVR, ANN (MLP)
- **Doğrulama:** LeaveOneGroupOut (özne bazlı), segment-bazlı metrikler, bootstrap güven aralığı, Bland–Altman uyum analizi
- **İleri analizler:** iki aşamalı (hiyerarşik) sınıflandırma + regresyon, feature importance (LOSO ortalamalı), vücut kütlesi/yağsız kütle (FFM) normalizasyonu, 1D CNN (ham 3 eksen sinyal)

## Notebook Yapısı

```
0.  Kurulum
1.  Sinyal yardımcıları ve feature setleri
2.  Aktivite aralıkları ve MET
3.  Cihaz okuyucular + feature tablosu
4.  EE hedefi (VO2), hizalama doğrulaması, vücut kompozisyonu
5.  Modelleme çekirdeği
6.  Tabloları kur + uçuş öncesi kontrol

BÖLÜM A — İstenen çekirdek çalışma
  A1. MET tahmini — cihaz × model
  A2. EE tahmini — cihaz × model
  A3. Adil cihaz karşılaştırması
  A4. En iyi EE kurulumu — Bland–Altman + bootstrap CI

BÖLÜM B — Literatür temelli kademeli iyileştirmeler
  B0.  Feature setinin etkisi
  B1.  Pencere uzunluğu (4 / 30 / 60 sn)
  B1b. Seçilen pencerede tam model havuzu
  B1c. Feature importance (LOSO ortalamalı)
  B2.  İki aşamalı (branched) model
  B2b. Sınıflandırma teşhisi
  B3.  Vücut kütlesinin doğru eklenmesi (yalnız EE)
  B4.  1D CNN (ham 3 eksen + FFM)

Özet — bütün adımlar tek tabloda
Sınırlar ve yorum notları
```

## Kurulum ve Çalıştırma

Bu notebook **Google Colab** üzerinde çalışacak şekilde tasarlanmıştır.

1. Veri setini `.zip` olarak Google Drive'da `MyDrive/PAWS/` altına yükleyin
2. Notebook'u Colab'de açın, Drive'ı mount edin (Hücre 2)
3. **CPU runtime** ile 0–6 arası tanım hücrelerini ve Bölüm A + B3'e kadar olan hücreleri sırayla çalıştırın
4. **B4 (1D CNN)** için: checkpoint hücresini çalıştırın → runtime'ı **T4 GPU**'ya çevirin → tanım hücrelerini yeniden çalıştırın → B4'ü çalıştırın

### Gereksinimler

```
numpy
pandas
scipy
scikit-learn
xgboost
tensorflow  (yalnız B4 için)
matplotlib
```

## Süre Tahminleri

| Aşama | Runtime | Süre |
|---|---|---|
| 0–6 (tanımlar + uçuş öncesi kontrol) | CPU | ~2 dk |
| A1 MET | CPU | ~47 dk |
| A2 EE | CPU | ~29 dk |
| A3 + A4 | CPU | ~30 dk |
| B0 | CPU | ~5 dk |
| B1 + B1b | CPU | ~6 dk |
| B2 + B2b | CPU | ~36 dk |
| B3 | CPU | ~46 dk |
| B4 | T4 GPU | ~20–40 dk |

## Sonuçlar

Notebook'un sonundaki "Özet" tablosunda tüm adımların MAE değerleri tek bakışta karşılaştırılabilir. Detaylı sayısal sonuçlar için notebook'u çalıştırın.

## Sınırlar

- Bisiklet aktivitesi bilek ivmeölçeri için yapısal olarak zordur
- EE'de grup düzeyinde kalibrasyon iyi olsa da bireysel kesinlik sınırlıdır
- Küçük örneklem (~13 özne) nedeniyle vücut kütlesi/demografik sonuçlar genellenemez
- VO2→kcal dönüşümü (4.84 sabiti) yaklaşık bir Weir denklemi kullanır

Detaylı tartışma için notebook'un son bölümüne bakın.

## Kaynakça

- Trost, S.G. et al. (2012) — pencere uzunluğu ve MET tahmini
- Crouter, S.E. — iki-regresyon (2-regression) modeli
- Ellis, R.J. (2014) — Random Forest ile MET tahmini
- Liu, S. (2011) — SVM tabanlı EE tahmini
- Staudenmayer, J. (2009); Montoye, A. (2017); Mackintosh, K. (2016) — ANN tabanlı yaklaşımlar
- Kim & Seong (2025) — WEEE veri setinde HR + pekiştirmeli öğrenme ile EE tahmini

---

*Bu proje akademik/araştırma amaçlıdır.*
