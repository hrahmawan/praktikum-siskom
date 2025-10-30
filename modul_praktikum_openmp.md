# Modul Praktikum: Pemrograman Multicore dengan OpenMP

## Durasi Praktikum
2 jam

## Tujuan Pembelajaran
Mahasiswa memahami dasar-dasar pemrograman paralel menggunakan OpenMP serta mampu membandingkan performa antara program sekuensial dan paralel.

---

## 1. Konsep Dasar OpenMP

OpenMP (**Open Multi-Processing**) adalah API (Application Programming Interface) yang mendukung pemrograman paralel pada sistem berbasis shared memory (memori bersama). OpenMP memungkinkan programmer menulis kode paralel dengan cara menambahkan **directive** khusus pada bahasa C, C++, atau Fortran.

### Fitur utama OpenMP
- Model **fork-join parallelism**
- **Shared memory architecture**
- Penggunaan **pragma directives** seperti `#pragma omp parallel` dan `#pragma omp for`
- Pengendalian jumlah thread dengan `OMP_NUM_THREADS`

### Arsitektur Dasar
Model eksekusi OpenMP menggunakan **fork-join**:
1. Program dimulai dalam satu thread utama (master thread).
2. Ketika mencapai *parallel region*, thread utama akan membuat beberapa *worker threads*.
3. Setelah bagian paralel selesai, semua thread bergabung kembali menjadi satu.

---

## 2. Kompilasi dan Eksekusi Program OpenMP

Untuk mengompilasi program OpenMP di Linux:
```bash
gcc -fopenmp nama_program.c -o nama_program
```

Menjalankan dengan jumlah thread tertentu (Linux):
```bash
export OMP_NUM_THREADS=8
./nama_program
```

---

## 3. Contoh Program Dasar OpenMP

### Contoh 1 — Hello World Paralel
```c
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        int id = omp_get_thread_num();
        printf("Hello from thread %d\n", id);
    }
    return 0;
}
```

### Contoh 2 — Menjumlahkan Elemen Array (Paralel)
```c
#include <stdio.h>
#include <omp.h>
#define N 10000000

int main() {
    static long long a[N];
    long long sum = 0;
    for (int i = 0; i < N; i++) a[i] = 1;

    double start = omp_get_wtime();

    #pragma omp parallel for reduction(+:sum)
    for (int i = 0; i < N; i++)
        sum += a[i];

    double end = omp_get_wtime();
    printf("Sum = %lld\n", sum);
    printf("Waktu eksekusi: %f detik\n", end - start);
}
```

---

## 4. Studi Kasus: Perkalian Matriks Besar

### 💻 Program Sekuensial
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define N 1500

int main() {
    static double A[N][N], B[N][N], C[N][N];

    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++) {
            A[i][j] = 1.0;
            B[i][j] = 2.0;
        }

    clock_t start = clock();
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++)
            for (int k = 0; k < N; k++)
                C[i][j] += A[i][k] * B[k][j];
    clock_t end = clock();

    double time_taken = (double)(end - start) / CLOCKS_PER_SEC;
    printf("Waktu eksekusi (sekuensial): %f detik\n", time_taken);
}
```

### Program Paralel (OpenMP)
```c
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

#define N 1500

int main() {
    static double A[N][N], B[N][N], C[N][N];

    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++) {
            A[i][j] = 1.0;
            B[i][j] = 2.0;
        }

    double start = omp_get_wtime();

    #pragma omp parallel for collapse(2)
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++)
            for (int k = 0; k < N; k++)
                C[i][j] += A[i][k] * B[k][j];

    double end = omp_get_wtime();
    printf("Waktu eksekusi (OpenMP): %f detik\n", end - start);
}
```

### Eksperimen dan Perbandingan

| Jumlah Thread | Waktu Sekuensial (s) | Waktu Paralel (s) | Speedup |
|----------------|-----------------------|--------------------|----------|
| 1              | 10.5                  | 10.4               | 1.0x     |
| 2              | 10.5                  | 5.3                | 1.98x    |
| 4              | 10.5                  | 2.8                | 3.75x    |
| 8              | 10.5                  | 1.6                | 6.56x    |

> **Kesimpulan:** Dengan memperbesar ukuran matriks (`N`), waktu eksekusi meningkat pada program sekuensial, tetapi penurunan signifikan terjadi pada program paralel OpenMP.

---

## 5. Tugas Praktikum

1. Modifikasi program perkalian matriks untuk:
   - Mencetak jumlah thread yang digunakan.
   - Mengukur waktu tiap bagian loop.
2. Uji dengan ukuran matriks berbeda: 1000, 2000, 4000.
3. Buat grafik *speedup* terhadap jumlah thread.
4. Laporkan hasil pengujian dalam format tabel dan analisis singkat.

---

## 📚 Referensi
- OpenMP API Specification v5.2 — [https://www.openmp.org/specifications/](https://www.openmp.org/specifications/)
- Chapman, Barbara et al., *Using OpenMP: Portable Shared Memory Parallel Programming*, MIT Press, 2007.
- TutorialsPoint: [https://www.tutorialspoint.com/openmp/index.htm](https://www.tutorialspoint.com/openmp/index.htm)

