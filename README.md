# 🔍 Steganaliz: LSB Steganografi Tespiti — Xu-Net CNN

> Dijital görüntülerin içine piksel seviyesinde gizlenmiş veriyi derin öğrenme ile tespit eden siber savunma sistemi.

![Python](https://img.shields.io/badge/Python-3.10-blue)
![PyTorch](https://img.shields.io/badge/PyTorch-2.0-orange)
![Accuracy](https://img.shields.io/badge/Accuracy-92.92%25-green)
![AUC](https://img.shields.io/badge/AUC--ROC-0.9883-brightgreen)
![Alan](https://img.shields.io/badge/Alan-Siber%20Savunma-red)

---

## 📌 Proje Hakkında

Siber saldırganlar, standart güvenlik duvarları ve antivirüs yazılımları multimedya trafiklerini doğrudan geçirdiği için verileri masum görünen görüntü dosyalarının içine gizleyerek sızdırabilir. Steganografi, şifrelemeden farklı olarak verinin **varlığını** tamamen kamufle eder.

Bu projede **LSB (Least Significant Bit)** yöntemiyle görüntülere gizlenmiş veriyi tespit eden **Xu-Net CNN** mimarisi sıfırdan kodlanmış ve ALASKA2 veri seti üzerinde eğitilmiştir.

**LSB Mantığı:** Her pikselin son biti değiştirilerek gizli veri saklanır. Maksimum değişim **1 birim** — insan gözü göremez, Xu-Net görebilir.

---

## 📊 Sonuçlar

| Metrik | Değer |
|--------|-------|
| **Test Accuracy** | **%92.92** |
| **AUC-ROC** | **0.9883** |
| Cover Precision / Recall | 0.9355 / 0.9220 |
| Stego Precision / Recall | 0.9231 / 0.9365 |
| Macro F1-Score | 0.9292 |
| Test Görüntü Sayısı | 600 |

---

## 🗂️ Veri Seti

**ALASKA2** — IEEE WIFS 2020 Yarışması (Troyes Teknoloji Üniversitesi)

50 farklı akıllı telefon kamerasıyla çekilmiş gerçek hayat fotoğraflarından oluşur.

- 75,000 Cover (temiz) görüntü
- 75,000 JMiPOD stego görüntü
- 75,000 JUNIWARD stego görüntü
- 75,000 UERD stego görüntü
- Görüntü boyutu: 512×512 piksel

Bu projede Cover görüntülerden kendi LSB stego görüntülerimiz üretilmiştir. JPEG sıkıştırması LSB izlerini bozduğu için tüm görüntüler kayıpsız **PNG** formatına ve **Gri Tonlamaya** dönüştürülmüştür.

**Kaggle:** [ALASKA2 Image Steganalysis](https://www.kaggle.com/competitions/alaska2-image-steganalysis)

---

## 🧠 Model: Xu-Net CNN

Xu-Net, 2016 yılında Guanshuo Xu tarafından **steganaliz için özel** tasarlanmış bir CNN mimarisidir. Genel nesne tanıma modellerinden (ResNet, VGG vb.) farklı olarak görüntünün görsel içeriğini değil, piksel seviyesindeki gürültü örüntülerini analiz eder.

```
Giriş (256×256) → HPF Filtre → Conv+BN+Tanh ×2 → Conv+BN+ReLU ×3 → AvgPool → FC(256) → Çıkış(2)
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
|-----------|-------|
| Toplam | 48,459 |
| Eğitilen | 48,434 |
| Sabit (HPF) | 25 |

---

## 🔄 Proje Akışı

```
ALASKA2 Veri Seti (75,000 görüntü)
         ↓
  2,000 Cover Görüntü Seçimi
         ↓
  LSB Steganografi Uygulama
  (2,000 Stego Görüntü Üretimi)
         ↓
  Train/Val/Test Bölünmesi
  (%70 / %15 / %15)
         ↓
  Xu-Net CNN Eğitimi (20 Epoch)
  lr=0.0001, Adam, ReduceLROnPlateau
         ↓
  Test & Metrikler
  (Accuracy, AUC-ROC, F1, CM)
         ↓
  Grad-CAM Görselleştirme
```

---

## 🖼️ Çıktı Görselleri

| # | Görsel | Açıklama |
|---|--------|----------|
| 01 | Cover vs Stego | İnsan gözü farkı göremez |
| 02 | Sınıf Dağılımı | ALASKA2 veri seti istatistikleri |
| 03 | Temel Görüntü İşleme | Gri, blur, sharpen, kanallar |
| 04 | Histogramlar | RGB ve gri seviye dağılımları |
| 05 | LSB Piksel Analizi | Fark haritası, bit dağılımı |
| 06 | LSB Fark Haritası | Maks fark: 1 piksel |
| 07 | Bölüm Dağılımı | Train/Val/Test görüntü sayıları |
| 08 | Xu-Net Mimarisi | Katman akış diyagramı |
| 09 | Eğitim Grafikleri | Loss ve Accuracy eğrileri |
| 10 | Confusion Matrix | TP/FP/TN/FN dağılımı |
| 11 | ROC Eğrisi | AUC = 0.9883 |
| 12 | Grad-CAM | Model nereye baktı? |
| 13 | Yanlış Tahminler | FP ve FN örnekleri |
| 14 | Sonuç Tablosu | Tüm metrikler özeti |

---

## ⚙️ Kurulum & Çalıştırma

### Google Colab (Önerilen)

1. `Steganaliz_CNN_FINAL_v2.ipynb` dosyasını Colab'e yükle
2. **Runtime → Change runtime type → T4 GPU** seç
3. 3. hücreye Kaggle bilgilerini gir
4. **Runtime → Run all**

### Gereksinimler

```bash
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
├── Steganaliz_CNN_FINAL_v2.ipynb   # Ana notebook
├── README.md                        # Bu dosya
└── outputs/                         # Çıktı görselleri (14 PNG)
    ├── 01_cover_vs_stego.png
    ├── 02_sinif_dagilimi.png
    ├── 03_temel_goruntu.png
    ├── 04_histogramlar.png
    ├── 05_lsb_analiz.png
    ├── 06_lsb_fark_haritasi.png
    ├── 07_bolum_dagilim.png
    ├── 08_xunet_mimari.png
    ├── 09_egitim_grafigi.png
    ├── 10_confusion_matrix.png
    ├── 11_roc_egrisi.png
    ├── 12_gradcam.png
    ├── 13_yanlis_tahminler.png
    └── 14_sonuc_tablosu.png
```

---

## ⚠️ Güvenlik Notu

Notebook'taki Kaggle API bilgilerini **asla** gerçek değerleriyle GitHub'a yükleme. 3. hücredeki bilgileri şu şekilde placeholder olarak bırak:

```python
KAGGLE_USERNAME = "KAGGLE_KULLANICI_ADINIZ"
KAGGLE_KEY      = "KAGGLE_API_KEYINIZ"
```

---

## 📚 Referanslar

- Buluş, E. (2025). Investigation of steganalysis performance of selected steganography methods with deep learning models. *Journal of the Faculty of Engineering and Architecture of Gazi University*, 40(1), 443-453.
- Xu, G., Wu, H. Z., & Shi, Y. Q. (2016). Structural design of convolutional neural networks for steganalysis. *IEEE Signal Processing Letters*, 23(5), 708-712.

---

## 👤 Geliştirici

**Ceyda** — Siber Güvenlik & Yazılım Geliştirici | Ankara, Türkiye

[![GitHub](https://img.shields.io/badge/GitHub-ceydavona-black)](https://github.com/ceydavona)

---

*Bu proje akademik amaçlı geliştirilmiştir.*
