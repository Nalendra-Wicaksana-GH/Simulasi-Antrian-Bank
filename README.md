#include <iostream>
#include <cstdlib>
#include <ctime>
#include <algorithm>

using namespace std;

#define DURASI_SIMULASI 28800 // Total waktu operasi bank 8 jam 
#define MAKS_WAKTU_KEDATANGAN 600 // Waktu kedatangan antara 1-600 detik/pelanggan
#define MAKS_WAKTU_LAYANAN 600 // Waktu pelayanan teller 1-600 detik

struct Pelanggan {
    int id;
    int waktu_kedatangan;
    int waktu_layanan;
};

// dari sini
class Antrian {
private:
    Pelanggan* data;
    int depan, belakang, ukuran, kapasitas;

public:
    Antrian(int kapasitas) : kapasitas(kapasitas), depan(0), belakang(kapasitas - 1), ukuran(0) {
        data = new Pelanggan[kapasitas];
    }
    ~Antrian() {
        delete[] data;
    }

    bool penuh() const {
        return ukuran == kapasitas;
    }

    bool kosong() const {
        return ukuran == 0;
    }

    void enqueue(const Pelanggan& p) {
        if (penuh()) return;
        belakang = (belakang + 1) % kapasitas;
        data[belakang] = p;
        ukuran++;
    }

    Pelanggan dequeue() {
        if (kosong()) {
            cerr << "Antrian kosong!" << endl;
            exit(EXIT_FAILURE);
        }
        Pelanggan p = data[depan];
        depan = (depan + 1) % kapasitas;
        ukuran--;
        return p;
    }
};
// sampai sini aku belum begitu paham fungsinya

void simulasiBank(int jumlah_teller) {
    Antrian antrian(100);
    int waktu_kedatangan_berikutnya = rand() % MAKS_WAKTU_KEDATANGAN + 1;

    int* waktu_selesai_teller = new int[jumlah_teller](); // waktu selesai tiap teller
    int total_pelanggan = 0;
    int total_waktu_tunggu = 0;
    int max_waktu_tunggu = 0;
    int* waktu_sibuk_teller = new int[jumlah_teller]();
    int pelanggan_dilayani = 0;

    for (int waktu = 0; waktu < DURASI_SIMULASI; waktu++) {
        // Proses kedatangan pelanggan
        while (waktu == waktu_kedatangan_berikutnya && waktu_kedatangan_berikutnya < DURASI_SIMULASI) {
            Pelanggan p{ total_pelanggan++, waktu, rand() % MAKS_WAKTU_LAYANAN + 1 };
            if (!antrian.penuh()) {
                antrian.enqueue(p);
            }
            waktu_kedatangan_berikutnya = waktu + rand() % MAKS_WAKTU_KEDATANGAN + 1;
        }
        // Proses pelayanan teller
        for (int i = 0; i < jumlah_teller; i++) {
            if (waktu_selesai_teller[i] <= waktu && !antrian.kosong()) {
                Pelanggan p = antrian.dequeue();
                int waktu_layanan_aktual = min(p.waktu_layanan, DURASI_SIMULASI - waktu);
                waktu_selesai_teller[i] = waktu + waktu_layanan_aktual;
                waktu_sibuk_teller[i] += waktu_layanan_aktual;

                int waktu_tunggu = max(0, waktu - p.waktu_kedatangan);
                total_waktu_tunggu += waktu_tunggu;
                if (waktu_tunggu > max_waktu_tunggu) max_waktu_tunggu = waktu_tunggu;
                pelanggan_dilayani++;
            }
        }
    }

    int total_sibuk = 0;
    for (int i = 0; i < jumlah_teller; i++) {
        total_sibuk += waktu_sibuk_teller[i];
    }

    float utilisasi_rata = (float)total_sibuk / (jumlah_teller * DURASI_SIMULASI) * 100;
    float rata_waktu_tunggu = pelanggan_dilayani > 0 ? (float)total_waktu_tunggu / pelanggan_dilayani : 0;

    cout << "\n=== Hasil Simulasi (" << jumlah_teller << " Teller) ===\n";
    cout << "Total Pelanggan: " << total_pelanggan << endl;
    cout << "Pelanggan Dilayani: " << pelanggan_dilayani << endl;
    cout << "Rata-rata Waktu Tunggu: " << rata_waktu_tunggu << " detik\n";
    cout << "Waktu Tunggu Maksimum: " << max_waktu_tunggu << " detik\n";
    cout << "Utilisasi Teller: " << utilisasi_rata << " %\n";

    delete[] waktu_selesai_teller;
    delete[] waktu_sibuk_teller;
}

int main() {
    srand(time(nullptr));

    cout << "Simulasi Bank (detik)\n";
    cout << "Durasi Simulasi: " << DURASI_SIMULASI << " detik\n";
    cout << "Interval Kedatangan Maksimum: " << MAKS_WAKTU_KEDATANGAN << " detik\n";
    cout << "Waktu Layanan Maksimum: " << MAKS_WAKTU_LAYANAN << " detik\n\n";

    for (int teller = 1; teller <= 4; teller++) {
        simulasiBank(teller);
    }

    return 0;
}
