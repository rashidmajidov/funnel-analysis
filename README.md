# Funnel Analysis — E-commerce

Bu layihə çoxkateqoriyalı bir e-ticarət platformasının (Yanvar–Fevral 2020) istifadəçi davranış datası üzərində view → cart → purchase konversiya varonkasının (funnel) analiz edilməsini əhatə edir.

## Dataset

- **Mənbə:** eCommerce behavior data (multi-category store) — Yanvar və Fevral 2020 aylıq event logları
- **Həcm:** ~8.4M sətir, 9 sütun (~578 MB)
- **Sütunlar:** `event_time`, `event_type` (view / cart / remove_from_cart / purchase), `product_id`, `category_id`, `category_code`, `brand`, `price`, `user_id`, `user_session`
- Data faylları həcmcə böyük olduğu üçün (100 MB limitini keçdiyi üçün) repoya əlavə edilməyib.
- **Data linki:** [(https://www.kaggle.com/datasets/mkechinov/ecommerce-events-history-in-cosmetics-shop)]

## Layihənin Strukturu

```
funnel-analysis/
│
├── data/
│   ├── raw/                        # 2020-Jan.csv, 2020-Feb.csv (Drive linkindən yüklənməli)
│   ├── merged/
│   │   └── merged_data.csv         # Birləşdirilmiş xam data
│   └── processed/
│       └── cleaned_data.parquet    # Təmizlənmiş dataset
│
├── notebooks/
│   ├── 01_data_merging.ipynb       # Yanvar+Fevral datasının birləşdirilməsi
│   ├── 02_data_cleaning.ipynb      # Data təmizləmə
│   └── 03_eda.ipynb                # Funnel analizi və insight-lar
│
├── reports/
│   └── figure/
│       └── e-commerce funnel & conversion analysis.png
│
└── README.md
```

## Görülən İşlər

1. **Birləşdirmə (`01_data_merging.ipynb`):** Yanvar və Fevral aylıq CSV faylları bir dataset-də birləşdirilib.
2. **Təmizləmə (`02_data_cleaning.ipynb`):** `event_time` `datetime`-a çevrilib, boş/təkrarlanan sətirlər yoxlanılıb və silinib, yekun dataset `.parquet` formatında (sıxılmış, sürətli oxunan) saxlanılıb.
3. **Analiz (`03_eda.ipynb`):**
   - Hadisə növlərinin (view/cart/remove_from_cart/purchase) ümumi paylanması
   - Əsas funnel mərhələləri üzrə unikal istifadəçi sayı və konversiya dərəcələri (CR)
   - Top 8 kateqoriya üzrə funnel və CR müqayisəsi
   - Ay üzrə (Yanvar vs Fevral) funnel dinamikası
   - Top 10 brend üzrə view→purchase konversiyası
   - Alış qiymətlərində outlier təhlili (IQR metodu)
   - View → Purchase arası keçən vaxtın (time-to-purchase) hesablanması
   - Çox baxılan, lakin az satılan ("underperforming") məhsulların aşkarlanması

## Əsas Tapıntılar

**Ümumi Funnel:**
| Mərhələ | Unikal İstifadəçi | Konversiya |
|---|---|---|
| View | 716,987 | — |
| Cart | 162,035 | View→Cart: 22.60% |
| Purchase | 49,473 | Cart→Purchase: 30.53% |
| **Ümumi (View→Purchase)** | | **6.90%** |

- Bütün event-lərin 15.24%-i (1.21M) `remove_from_cart`-dır — səbətə atılanların 70%-ə yaxını alınmadan tərk edilir və ya çıxarılır.
- **Aylıq dinamika:** ümumi CR Yanvardan (7.09%) Fevrala (6.79%) -0.30% düşüb; mütləq ədədlərdə də view, cart və purchase sayları azalıb (purchase: 28,220 → 25,759) — sadəcə mövsümi tərəddüd deyil, ümumi fəallıqda geriləmə müşahidə olunur.
- **Kateqoriyalar üzrə:** `apparel.glove` (38.91%) və `stationery.cartrige` (23.77%) ən yüksək CR-ə malikdir; `accessories.bag` (1.43%) və `furniture.living_room.cabinet` (2.21%) isə yüksək trafikə baxmayaraq demək olar ki, konversiya vermir.
- **Brendlər üzrə:** `bpw.style` az trafiklə (13,376 baxış) ən yüksək CR-i (19.38%) göstərir; `estel` isə yüksək trafikinə (40,234 baxış) baxmayaraq ən aşağı CR-ə (9.57%) malikdir.
- **Qiymət davranışı:** baxılan məhsulların orta qiyməti alınanlardan xeyli yüksəkdir — istifadəçilər baha məhsullara baxır, lakin ucuz olanları alır. Alışların 6.91%-i (34,919 ədəd) IQR sərhədlərindən kənar outlier-dir.
- **Time-to-purchase:** view-dan purchase-a qədər median vaxt ~1 saat 8 dəqiqədir, lakin ortalama 2 gün 6 saatdır — bu fərq, əksər alışların tez baş verdiyini, lakin bir qisim istifadəçinin qərar üçün xeyli vaxt apardığını göstərir.
- Qlobal ortalama CR 12.67% (məhsul səviyyəsində), lakin minimum 100 baxışı olan çoxlu məhsulun heç bir satışı yoxdur (0% CR) — bunlar "çox axtarılan, amma alınmayan" məhsul qrupunu təşkil edir.

## Təklif Olunan Həllər

- **Səbət tərki (cart abandonment):** 24-48 saat pəncərəsində retargeting email/push bildirişləri; checkout prosesinin sadələşdirilməsi (guest checkout, autofill, ödəniş metodu optimallaşdırması).
- **Zəif performanslı kateqoriyalar** (`accessories.bag`, `furniture.living_room.cabinet`): məhsul səhifəsi optimallaşdırması (şəkil, təsvir, rəylər) və ya retargeting kampaniyaları; yüksək-CR kateqoriyaların (glove, stationery) homepage/banner-lərdə önə çıxarılması.
- **Brend səviyyəsində:** `bpw.style` kimi yüksək-CR, aşağı-görünürlük brendlərinə əlavə trafik/promo büdcəsi yönləndirmək; `estel` üçün məhsul səhifəsi optimallaşdırması və retargeting.
- **Aylıq geriləmə üçün:** mövsümi kampaniyalar, endirimlər, loyallıq proqramları ilə fəallığı artırmaq; azalmanın davam edib-etmədiyini izləmək üçün əlavə aylar üzrə monitorinq.

## İstifadə Olunan Texnologiyalar

- Python 3 (pandas, numpy, matplotlib, seaborn, plotly)
- Jupyter Notebook
- Parquet (sürətli, sıxılmış data saxlama formatı)

## Necə İşə Salmaq

```bash
uv sync
```

1. Yuxarıdakı linkindən `2020-Jan.csv` və `2020-Feb.csv` fayllarını yükləyib `data/raw/` qovluğuna yerləşdirin.
2. Notebook-ları ardıcıl işə salın:

```bash
uv run jupyter notebook notebooks/01_data_merging.ipynb
```
