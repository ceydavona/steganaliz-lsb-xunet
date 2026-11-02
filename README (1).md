# 🔍 Steganaliz: LSB Steganografi Tespiti — Xu-Net CNN

> Dijital görüntülerin içine piksel seviyesinde gizlenmiş veriyi derin öğrenme ile tespit eden siber savunma sistemi.

![Python](https://img.shields.io/badge/Python-3.10-blue)
![PyTorch](https://img.shields.io/badge/PyTorch-2.0-orange)
![Accuracy](https://img.shields.io/badge/Accuracy-91.0%25-green)
![AUC](https://img.shields.io/badge/AUC--ROC-0.9850-brightgreen)
![Alan](https://img.shields.io/badge/Alan-Siber%20Savunma-red)

---

## 📌 Proje Hakkında

Siber saldırganlar, standart güvenlik duvarları ve antivirüs yazılımları multimedya trafiğini doğrudan geçirdiği için verileri masum görünen görüntü dosyalarının içine gizleyerek sızdırabilir. Steganografi, şifrelemeden farklı olarak verinin **varlığını** tamamen kamufle eder.

Bu projede **LSB (Least Significant Bit)** yöntemiyle görüntülere gizlenmiş veriyi tespit eden **Xu-Net CNN** mimarisi sıfırdan kodlanmış ve ALASKA2 veri seti üzerinde eğitilmiştir. Çalışma üç deneyden oluşur:

1. **Xu-Net / LSB** — ana deney
2. **Xu-Net / MSB** — gizleme yönteminin görünürlüğünün tespit başarısına etkisi
3. **SimpleCNN / LSB** — HPF filtresi çıkarılarak yapılan ablasyon deneyi

**LSB Mantığı:** Her pikselin son biti değiştirilerek gizli veri saklanır. Maksimum değişim **1 birim** — insan gözü göremez, Xu-Net görebilir.

---

## 📊 Sonuçlar

### Xu-Net / LSB (ana sonuç, 600 test görüntüsü)

| Metrik | Değer |
|---|---|
| **Test Accuracy** | **%91.0** |
| **AUC-ROC** | **0.9850** |
| Cover Precision / Recall | 0.92 / 0.90 |
| Stego Precision / Recall | 0.90 / 0.92 |
| F1-Score (her iki sınıf) | 0.91 |
| Confusion Matrix | 270 / 30 (Cover), 24 / 276 (Stego) |

### Deney karşılaştırması

| Model | Yöntem | Accuracy | AUC-ROC | Yorum |
|---|---|---|---|---|
| Xu-Net | LSB | %91.0 | 0.9850 | Güçlü tespit |
| Xu-Net | MSB | %100.0 | 1.0000 | Piksel farkı 128 → kolay tespit |
| SimpleCNN (HPF yok) | LSB | %50.0 | 0.5010 | Rastgele tahmin, öğrenme yok |

**Bulgular**
- **LSB ve MSB:** Gizleme yöntemi ne kadar görünürse tespit o kadar kolaydır. MSB'de maksimum piksel farkı 128'dir ve model 1. epoch'tan itibaren %100'e ulaşır; LSB'de fark yalnızca 1'dir ve doğruluk %91'dir.
- **HPF filtresinin katkısı:** Aynı veri ve hiperparametrelerle HPF olmadan eğitilen SimpleCNN tüm eğitim boyunca %50'de kalmıştır. HPF'nin katkısı yaklaşık **+41 puan** doğruluktur.
- **Eğitim kararlılığı:** LSB eğitiminde validation eğrisi bazı epoch'larda dalgalanmaktadır; en iyi validation ağırlıkları kaydedilerek final model bu dalgalanmadan bağımsız değerlendirilmiştir.

---

## 🗂️ Veri Seti

**ALASKA2** — IEEE WIFS 2020 Yarışması (Troyes Teknoloji Üniversitesi)

50 farklı kamera modeliyle çekilmiş gerçek hayat fotoğraflarından oluşur.

- 75.000 Cover (temiz) görüntü
- 75.000 JMiPOD, 75.000 JUNIWARD, 75.000 UERD stego görüntü
- Bu projede görüntüler 256×256 gri tonlamaya çevrilmiştir

Bu projede 2.000 Cover görüntü seçilmiş, bunlardan kendi LSB ve MSB stego görüntülerimiz üretilmiştir (train %70 / val %15 / test %15, seed = 42). JPEG sıkıştırması LSB izlerini bozduğu için görüntüler kayıpsız **PNG** formatına ve **gri tonlamaya** dönüştürülmüştür.

**Kaggle:** [ALASKA2 Image Steganalysis](https://www.kaggle.com/competitions/alaska2-image-steganalysis)

---

## 🧠 Model: Xu-Net CNN

Xu-Net, 2016 yılında Xu ve ark. tarafından **steganaliz için özel** tasarlanmış bir CNN mimarisidir. Genel nesne tanıma modellerinden (ResNet, VGG vb.) farklı olarak görüntünün görsel içeriğini değil, piksel seviyesindeki gürültü örüntülerini analiz eder.

```
Giriş (256×256) → HPF Filtre → Conv+BN+Tanh ×2 → Conv+BN+ReLU ×3 → Global AvgPool → FC(256) → Çıkış(2)
```

**HPF (Yüksek Geçiren Filtre):** Görüntünün genel görünümünü bastırıp LSB değişikliklerini ön plana çıkaran sabit filtre. Ağırlıkları eğitim sırasında değişmez.

```
[-1,  2, -2,  2, -1]
[ 2, -6,  8, -6,  2]
[-2,  8,-12,  8, -2]
[ 2, -6,  8, -6,  2]
[-1,  2, -2,  2, -1]
```

| Parametre | Değer |
|---|---|
| Toplam | 48.459 |
| Eğitilen | 48.434 |
| Sabit (HPF) | 25 |

Eğitim: Adam, lr = 0.0001, ReduceLROnPlateau, batch = 32, 20 epoch, CrossEntropyLoss.

---

## 🔄 Proje Akışı

```
ALASKA2 Veri Seti (75.000 görüntü)
         ↓
  2.000 Cover Görüntü Seçimi
         ↓
  LSB ve MSB Steganografi Uygulama
         ↓
  Train / Val / Test Bölünmesi (%70 / %15 / %15)
         ↓
  Xu-Net Eğitimi (LSB, MSB) + SimpleCNN Eğitimi (LSB)
         ↓
  Test & Metrikler (Accuracy, AUC-ROC, F1, Confusion Matrix)
         ↓
  Grad-CAM Görselleştirme + Yanlış Tahmin Analizi
         ↓
  Genel Model/Yöntem Karşılaştırması
```

---

## ⚙️ Kurulum & Çalıştırma

### Google Colab (Önerilen)

1. `Steganaliz_CNN.ipynb` dosyasını Colab'e yükle
2. **Runtime → Change runtime type → T4 GPU** seç
3. Kaggle hücresine kendi Kaggle bilgilerini gir
4. **Runtime → Run all**

### Gereksinimler

```
pip install torch torchvision
pip install kaggle
pip install grad-cam
pip install seaborn scikit-learn
pip install tqdm Pillow matplotlib
```

---

## 📁 Dosya Yapısı

```
steganaliz-lsb-xunet/
│
├── Steganaliz_CNN.ipynb      # Ana notebook (LSB, MSB, SimpleCNN, Grad-CAM)
├── Steganaliz_Rapor.docx     # Akademik proje raporu
├── README.md                 # Bu dosya
└── outputs/                  # Çıktı görselleri (PNG)
```

---

## ⚠️ Güvenlik Notu

Notebook'taki Kaggle API bilgilerini **asla** gerçek değerleriyle GitHub'a yükleme. Kaggle hücresindeki bilgileri placeholder olarak bırak:

```
KAGGLE_USERNAME = "KAGGLE_KULLANICI_ADINIZ"
KAGGLE_KEY      = "KAGGLE_API_KEYINIZ"
```

---

## 📚 Referanslar

- Buluş, E. (2025). Investigation of steganalysis performance of selected steganography methods with deep learning models. *Journal of the Faculty of Engineering and Architecture of Gazi University*, 40(1), 443-453.
- Xu, G., Wu, H. Z., & Shi, Y. Q. (2016). Structural design of convolutional neural networks for steganalysis. *IEEE Signal Processing Letters*, 23(5), 708-712.
- Cogranne, R. et al. (2020). ALASKA#2: Challenging Academic Research on Steganalysis with Realistic Images. *IEEE WIFS*.

---

## 👤 Geliştirici

**Ceyda** — Siber Güvenlik & Yazılım Geliştirici | Ankara, Türkiye

[![GitHub](https://img.shields.io/badge/GitHub-ceydavona-black)](https://github.com/ceydavona)

*Bu proje akademik amaçlı geliştirilmiştir.*
