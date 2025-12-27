#include <stdio.h>
#include <string.h>

//kullanıcı siteye girdiğinde mevcut filmler listelenir.İstenen film seçildikten sonra koltuk seçimine yönlendirilir
//koltuk seçiminden sonra üye girişi veya üye olma ekranına yönlendirilir.
//üye girişi yapıldıktan sonra ödeme adımına yönlendirilir.
//ödeme adımı başarıyla tanımlandıktan sonra rezervasyon oluşturulur ve film-tarih-seans-koltuk bilgisi ekrana yazdırılır.

#define MAX_FILM 10
#define MAX_KOLTUK 30
#define BILET_FIYAT 120
#define MAX_KULLANICI 10

typedef struct {
    char kullaniciAdi[50];
    char sifre[50];
} Kullanici;

typedef struct {
    char ad[50];
    char saat[10];
    int koltuklar[MAX_KOLTUK];
} Film;

void koltuklariGoster(Film film) {
    int i;
    printf("\nKoltuk Durumu (%s - %s):\n", film.ad, film.saat);
    for (i = 0; i < MAX_KOLTUK; i++) {
        printf("[%2d]%s ", i + 1, film.koltuklar[i] ? "Dolu" : "Bos ");
        if ((i + 1) % 5 == 0) printf("\n");
    }
    printf("\n");
}

void biletYazdir(const char *kullanici, Film film, int koltukNo) {
    FILE *dosya = fopen("bilet.txt", "w");
    int i;
    if (dosya == NULL) {
        printf("Bilet dosyasi olusturulamadi!\n");
        return;
    }

    fprintf(dosya, "===== SINEMA BILETI =====\n");
    fprintf(dosya, "Musteri: %s\n", kullanici);
    fprintf(dosya, "Film: %s\n", film.ad);
    fprintf(dosya, "Seans: %s\n", film.saat);
    fprintf(dosya, "Koltuk No: %d\n", koltukNo);
    fprintf(dosya, "Ucret: %d TL\n", BILET_FIYAT);
    fprintf(dosya, "==========================\n");
    fprintf(dosya, "Iyi seyirler dileriz!\n");

    fclose(dosya);
    printf("\nBilet dosyasi olusturuldu: bilet.txt\n");
}

void odemeYap() {
    int odemeYontemi;
    char kartNo[20], sonKullanma[10], cvv[5];
    
    printf("\n--- ODEME ADIMI ---\n");
    printf("Bilet Fiyati: %d TL\n", BILET_FIYAT);
    printf("Odeme Yontemi Seciniz:\n");
    printf("1. Kredi Karti\n");
    printf("2. Nakit\n");
    printf("Secim: ");
    scanf("%d", &odemeYontemi);

    if (odemeYontemi == 1) {
        printf("\nKredi Karti Bilgilerini Giriniz:\n");
        printf("Kart Numarasi (16 hane): ");
        scanf("%s", kartNo);
        printf("Son Kullanma Tarihi (AA/YY): ");
        scanf("%s", sonKullanma);
        printf("CVV (3 hane): ");
        scanf("%s", cvv);
        printf("\nOdeme islemi yapiliyor...\n");
        printf("Odeme basarili!\n");
    } else if (odemeYontemi == 2) {
        printf("\nNakit odemeyi sinema gisesinde yapabilirsiniz.\n");
    } else {
        printf("\nGecersiz odeme yontemi!\n");
    }
}

int kullaniciKayit(Kullanici kullanicilar[], int *sayac) {
    int i;
    char yeniKullanici[50], yeniSifre[50];
    
    if (*sayac >= MAX_KULLANICI) {
        printf("Kullanici sinirina ulasildi!\n");
        return -1;
    }

    printf("\n--- Yeni Kayit ---\n");
    printf("Kullanici adi: ");
    scanf("%s", yeniKullanici);
    printf("Sifre: ");
    scanf("%s", yeniSifre);

    for (i = 0; i < *sayac; i++) {
        if (strcmp(kullanicilar[i].kullaniciAdi, yeniKullanici) == 0) {
            printf("Bu kullanici adi zaten var!\n");
            return -1;
        }
    }

    strcpy(kullanicilar[*sayac].kullaniciAdi, yeniKullanici);
    strcpy(kullanicilar[*sayac].sifre, yeniSifre);
    (*sayac)++;

    printf("Kayit basarili!\n");
    return *sayac - 1;
}

int kullaniciGiris(Kullanici kullanicilar[], int sayac) {
    int i;
    char kullaniciAdi[50], sifre[50];
    
    printf("\n--- Giris Yap ---\n");
    printf("Kullanici adi: ");
    scanf("%s", kullaniciAdi);
    printf("Sifre: ");
    scanf("%s", sifre);

    for (i = 0; i < sayac; i++) {
        if (strcmp(kullanicilar[i].kullaniciAdi, kullaniciAdi) == 0 &&
            strcmp(kullanicilar[i].sifre, sifre) == 0) {
            printf("Giris basarili!\n");
            return i;
        }
    }
    printf("Hatali kullanici adi veya sifre!\n");
    return -1;
}

int main() {
    Film filmler[MAX_FILM] = {
        {"Interstellar", "14:00", {0}},
        {"Inception", "17:00", {0}},
        {"Oppenheimer", "20:00", {0}},
        {"The Dark Knight", "15:30", {0}},
        {"Dunkirk", "18:45", {0}},
        {"Tenet", "21:15", {0}},
        {"The Prestige", "13:20", {0}},
        {"Batman Begins", "16:10", {0}},
        {"The Dark Knight Rises", "19:30", {0}},
        {"Memento", "22:00", {0}}
    };
    Kullanici kullanicilar[MAX_KULLANICI];
    int kullaniciSayisi = 0;
    int aktifKullanici = -1;
    int filmSecim, koltuk, secimGiris;
    int i;

    printf("=== Sinema Rezervasyon Sistemi ===\n");
    printf("1. Giris Yap\n2. Kayit Ol\nSecim: ");
    scanf("%d", &secimGiris);

    if (secimGiris == 1) {
        aktifKullanici = kullaniciGiris(kullanicilar, kullaniciSayisi);
        if (aktifKullanici == -1) return 0;
    } else if (secimGiris == 2) {
        aktifKullanici = kullaniciKayit(kullanicilar, &kullaniciSayisi);
        if (aktifKullanici == -1) return 0;
    } else {
        printf("Gecersiz secim!\n");
        return 0;
    }

    printf("\nMevcut Filmler:\n");
    for (i = 0; i < MAX_FILM; i++) {
        printf("%2d. %-25s (%s)\n", i + 1, filmler[i].ad, filmler[i].saat);
    }

    printf("Film seciniz (1-%d): ", MAX_FILM);
    scanf("%d", &filmSecim);
    filmSecim--;

    if (filmSecim < 0 || filmSecim >= MAX_FILM) {
        printf("Gecersiz secim!\n");
        return 1;
    }

    koltuklariGoster(filmler[filmSecim]);

    printf("Rezervasyon yapmak istediginiz koltuk no (1-%d): ", MAX_KOLTUK);
    scanf("%d", &koltuk);
    koltuk--;

    if (koltuk < 0 || koltuk >= MAX_KOLTUK) {
        printf("Gecersiz koltuk numarasi!\n");
        return 1;
    }

    if (filmler[filmSecim].koltuklar[koltuk]) {
        printf("Bu koltuk zaten dolu!\n");
    } else {
        filmler[filmSecim].koltuklar[koltuk] = 1;
        printf("Rezervasyon basarili! (%s - Koltuk %d)\n",
               filmler[filmSecim].ad, koltuk + 1);

        odemeYap();
        biletYazdir(kullanicilar[aktifKullanici].kullaniciAdi,
                    filmler[filmSecim], koltuk + 1);

        printf("\nTesekkurler, %s! Iyi seyirler\n",
               kullanicilar[aktifKullanici].kullaniciAdi);
    }

    return 0;
}
