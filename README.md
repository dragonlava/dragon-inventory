#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FILENAME "toko.txt"
#define TEMP_FILENAME "temp.txt"
#define MAX_LINE 256

typedef struct {
    char id[10];
    char nama[50];
    char kategori[30];
    double harga;
    int stok;
} Produk;

// Fungsi Prototipe
void tampilkanMenu();
void tambahProduk();
void tampilkanProduk();
void updateProduk();
void hapusProduk();
void transaksiJualBeli();
int produkSudahAda(const char *id);
void clearBuffer();

// Fungsi Utama
int main() {
    int pilihan;

    printf("==========================================\n");
    printf("Selamat Datang di Manajemen Toko Elektronik\n");
    printf("==========================================\n");

    do {
        tampilkanMenu();
        printf("Masukkan pilihan Anda: ");
        if (scanf("%d", &pilihan) != 1) {
            printf("Input tidak valid. Silakan masukkan angka.\n");
            clearBuffer();
            continue;
        }

        switch (pilihan) {
            case 1:
                tambahProduk();
                break;
            case 2:
                tampilkanProduk();
                break;
            case 3:
                updateProduk();
                break;
            case 4:
                hapusProduk();
                break;
            case 5:
                transaksiJualBeli();
                break;
            case 6:
                printf("Keluar dari program. Terima kasih!\n");
                break;
            default:
                printf("Pilihan tidak valid. Silakan coba lagi.\n");
        }
    } while (pilihan != 6);

    return 0;
}

// Fungsi Menampilkan Menu
void tampilkanMenu() {
    printf("\n========== Menu =========\n");
    printf("1. Tambah Produk Baru\n");
    printf("2. Tampilkan Daftar Produk\n");
    printf("3. Update Informasi Produk\n");
    printf("4. Hapus Produk\n");
    printf("5. Transaksi Jual Beli\n");
    printf("6. Keluar\n");
    printf("========================\n");
}

// Fungsi Menambah Produk Baru
void tambahProduk() {
    Produk produk;
    FILE *file = fopen(FILENAME, "a");
    if (!file) {
        perror("Gagal membuka file");
        return;
    }

    printf("Masukkan ID Produk: ");
    scanf("%s", produk.id);

    if (produkSudahAda(produk.id)) {
        printf("ID sudah terdaftar. Silakan gunakan ID lain.\n");
        fclose(file);
        return;
    }

    printf("Masukkan Nama Produk: ");
    clearBuffer();
    fgets(produk.nama, sizeof(produk.nama), stdin);
    strtok(produk.nama, "\n");

    printf("Masukkan Kategori Produk: ");
    fgets(produk.kategori, sizeof(produk.kategori), stdin);
    strtok(produk.kategori, "\n");

    printf("Masukkan Harga Produk: ");
    if (scanf("%lf", &produk.harga) != 1 || produk.harga < 0) {
        printf("Input harga tidak valid. Harus berupa angka positif.\n");
        clearBuffer();
        fclose(file);
        return;
    }

    printf("Masukkan Stok Produk: ");
    if (scanf("%d", &produk.stok) != 1 || produk.stok < 0) {
        printf("Input stok tidak valid. Harus berupa angka positif.\n");
        clearBuffer();
        fclose(file);
        return;
    }

    fprintf(file, "%s|%s|%s|%.2lf|%d\n", produk.id, produk.nama, produk.kategori, produk.harga, produk.stok);
    printf("Produk berhasil ditambahkan!\n");
    fclose(file);
}

// Fungsi Menampilkan Produk
void tampilkanProduk() {
    FILE *file = fopen(FILENAME, "r");
    if (!file) {
        perror("Gagal membuka file");
        return;
    }

    char line[MAX_LINE];
    printf("\nDaftar Produk:\n");
    printf("============================================================\n");
    printf("ID       | Nama                     | Kategori     | Harga      | Stok\n");
    printf("============================================================\n");
    while (fgets(line, sizeof(line), file)) {
        Produk produk;
        sscanf(line, "%[^|]|%[^|]|%[^|]|%lf|%d", produk.id, produk.nama, produk.kategori, &produk.harga, &produk.stok);
        printf("%-8s | %-24s | %-12s | %-10.2lf | %-4d\n", produk.id, produk.nama, produk.kategori, produk.harga, produk.stok);
    }
    fclose(file);
}

void updateProduk() {
    char id[10];
    Produk produkBaru;
    int ditemukan = 0;

    printf("Masukkan ID Produk yang ingin diupdate: ");
    scanf("%s", id);

    FILE *file = fopen(FILENAME, "r");
    FILE *temp = fopen(TEMP_FILENAME, "w");

    if (!file || !temp) {
        perror("Gagal membuka file");
        if (file) fclose(file);
        if (temp) fclose(temp);
        return;
    }

    char line[MAX_LINE];
    while (fgets(line, sizeof(line), file)) {
        Produk produk;
        sscanf(line, "%[^|]|%[^|]|%[^|]|%lf|%d", produk.id, produk.nama, produk.kategori, &produk.harga, &produk.stok);

        if (strcmp(produk.id, id) == 0) {
            ditemukan = 1;
            printf("\nProduk ditemukan!\n");
            printf("ID Produk     : %s\n", produk.id);
            printf("Nama Produk   : %s\n", produk.nama);
            printf("Kategori      : %s\n", produk.kategori);
            printf("Harga         : %.2lf\n", produk.harga);
            printf("Stok          : %d\n", produk.stok);

            // Input pembaruan
            printf("\nMasukkan pembaruan (kosongkan jika tidak ingin mengubah):\n");

            printf("Nama Produk [%s]: ", produk.nama);
            clearBuffer();
            fgets(produkBaru.nama, sizeof(produkBaru.nama), stdin);
            if (strlen(produkBaru.nama) > 1)
                strtok(produkBaru.nama, "\n");
            else
                strcpy(produkBaru.nama, produk.nama);

            printf("Kategori [%s]: ", produk.kategori);
            fgets(produkBaru.kategori, sizeof(produkBaru.kategori), stdin);
            if (strlen(produkBaru.kategori) > 1)
                strtok(produkBaru.kategori, "\n");
            else
                strcpy(produkBaru.kategori, produk.kategori);

            printf("Harga [%.2lf]: ", produk.harga);
            char hargaInput[20];
            fgets(hargaInput, sizeof(hargaInput), stdin);
            if (strlen(hargaInput) > 1)
                produkBaru.harga = atof(hargaInput);
            else
                produkBaru.harga = produk.harga;

            printf("Stok [%d]: ", produk.stok);
            char stokInput[10];
            fgets(stokInput, sizeof(stokInput), stdin);
            if (strlen(stokInput) > 1)
                produkBaru.stok = atoi(stokInput);
            else
                produkBaru.stok = produk.stok;

            // Salin ID
            strcpy(produkBaru.id, produk.id);

            // Tulis data yang diperbarui ke file sementara
            fprintf(temp, "%s|%s|%s|%.2lf|%d\n", produkBaru.id, produkBaru.nama, produkBaru.kategori, produkBaru.harga, produkBaru.stok);
            printf("\nProduk berhasil diperbarui!\n");
        } else {
            // Tulis data yang tidak diubah ke file sementara
            fprintf(temp, "%s|%s|%s|%.2lf|%d\n", produk.id, produk.nama, produk.kategori, produk.harga, produk.stok);
        }
    }

    fclose(file);
    fclose(temp);

    if (ditemukan) {
        remove(FILENAME);
        rename(TEMP_FILENAME, FILENAME);
    } else {
        remove(TEMP_FILENAME);
        printf("Produk dengan ID %s tidak ditemukan.\n", id);
    }
}

void hapusProduk() {
    char id[10];
    printf("Masukkan ID Produk yang akan dihapus: ");
    scanf("%s", id);

    FILE *file = fopen(FILENAME, "r");
    FILE *temp = fopen("temp.txt", "w");
    if (!file || !temp) {
        perror("Gagal membuka file");
        return;
    }

    char line[MAX_LINE];
    int ditemukan = 0;
    while (fgets(line, sizeof(line), file)) {
        Produk produk;
        sscanf(line, "%[^|]|%[^|]|%[^|]|%lf|%d", produk.id, produk.nama, produk.kategori, &produk.harga, &produk.stok);

        if (strcmp(produk.id, id) != 0) {
            fprintf(temp, "%s|%s|%s|%.2lf|%d\n", produk.id, produk.nama, produk.kategori, produk.harga, produk.stok);
        } else {
            ditemukan = 1;
        }
    }

    fclose(file);
    fclose(temp);

    if (ditemukan) {
        remove(FILENAME);
        rename("temp.txt", FILENAME);
        printf("Produk berhasil dihapus!\n");
    } else {
        remove("temp.txt");
        printf("Produk dengan ID %s tidak ditemukan.\n", id);
    }
}
// Fungsi Transaksi Jual Beli
void transaksiJualBeli() {
    char id[10];
    int quantity;

    printf("Masukkan ID Produk yang ingin dijual: ");
    scanf("%s", id);

    FILE *file = fopen(FILENAME, "r");
    FILE *temp = fopen(TEMP_FILENAME, "w");
    if (!file || !temp) {
        perror("Gagal membuka file");
        if (file) fclose(file);
        if (temp) fclose(temp);
        return;
    }

    char line[MAX_LINE];
    int ditemukan = 0;
    while (fgets(line, sizeof(line), file)) {
        Produk produk;
        sscanf(line, "%[^|]|%[^|]|%[^|]|%lf|%d", produk.id, produk.nama, produk.kategori, &produk.harga, &produk.stok);

        if (strcmp(produk.id, id) == 0) {
            ditemukan = 1;

            printf("Nama Produk: %s\n", produk.nama);
            printf("Harga per unit: %.2lf\n", produk.harga);
            printf("Stok saat ini: %d\n", produk.stok);

            printf("Masukkan jumlah yang ingin dijual: ");
            if (scanf("%d", &quantity) != 1 || quantity <= 0) {
                printf("Jumlah tidak valid. Transaksi dibatalkan.\n");
                clearBuffer();
                break;
            }

            if (quantity > produk.stok) {
                printf("Stok tidak mencukupi. Transaksi dibatalkan.\n");
                break;
            }

            produk.stok -= quantity;
            double totalHarga = produk.harga * quantity;
            printf("Total harga: %.2lf\n", totalHarga);

            double uangDibayar;
            printf("Masukkan jumlah uang yang dibayar: ");
            if (scanf("%lf", &uangDibayar) != 1 || uangDibayar <= 0) {
                printf("Input tidak valid. Transaksi dibatalkan.\n");
                clearBuffer();
                break;
            }

            if (uangDibayar < totalHarga) {
                printf("Uang yang dibayar kurang. Transaksi dibatalkan.\n");
                break;
            }

            double kembalian = uangDibayar - totalHarga;
            printf("Transaksi berhasil! Kembalian: %.2lf\n", kembalian);
        }

        fprintf(temp, "%s|%s|%s|%.2lf|%d\n", produk.id, produk.nama, produk.kategori, produk.harga, produk.stok);
    }

    fclose(file);
    fclose(temp);

    if (ditemukan) {
        remove(FILENAME);
        rename(TEMP_FILENAME, FILENAME);
        printf("Stok telah diperbarui.\n");
    } else {
        remove(TEMP_FILENAME);
        printf("Produk dengan ID %s tidak ditemukan.\n", id);
    }
}


// Fungsi Mengecek ID Produk Sudah Ada
int produkSudahAda(const char *id) {
    FILE *file = fopen(FILENAME, "r");
    if (!file) return 0;

    char line[MAX_LINE];
    while (fgets(line, sizeof(line), file)) {
        Produk produk;
        sscanf(line, "%[^|]|%[^|]|%[^|]|%lf|%d", produk.id, produk.nama, produk.kategori, &produk.harga, &produk.stok);
        if (strcmp(produk.id, id) == 0) {
            fclose(file);
            return 1;
        }
    }

    fclose(file);
    return 0;
}

// Fungsi Membersihkan Buffer
void clearBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}
