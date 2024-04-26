# SOAL 4
Dikerjakan oleh Raditya Hardian Santoso (5027231033)


Kode yang disajikan adalah implementasi dari sebuah sistem manajemen aplikasi sederhana dalam bahasa C. Skrip utama yang disertakan adalah:


- `setup.c`: Sebuah file dalam bahasa pemrograman C yang digunakan untuk menginisialisasi dan menyiapkan lingkungan atau konfigurasi awal sebelum program utama dijalankan
- `file.conf`: Sebuah file config yang berisi config yang nantinya bisa di-run oleh hasil compile setup.c


Dalam soal ini, kita diminta untuk membuat program yang memungkinkan pengguna untuk meluncurkan aplikasi dengan jumlah jendela yang ditentukan dan memberikan kemampuan untuk menghentikan aplikasi yang sedang berjalan.


## setup.c


Isi skrip saya :


```bash
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
#include <signal.h>


#define MAX_APPS 10




typedef struct {
    char app_name[100];
    int num_windows;
    pid_t process_ids[10];
    int num_processes;
} Application;


Application applications[MAX_APPS];
int num_applications = 0;


void parse_arguments(int argc, char *argv[]) {
    int i;
    for (i = 2; i < argc; i += 2) {
        strcpy(applications[num_applications].app_name, argv[i]);
        applications[num_applications].num_windows = atoi(argv[i + 1]);
        applications[num_applications].num_processes = 0;
        num_applications++;
    }
}


void read_configuration_file(char *filename) {
    FILE *file_pointer = fopen(filename, "r");
    if (file_pointer == NULL) {
        printf("Gagal membuka file konfigurasi\n");
        return;
    }


    char line[100];
    while (fgets(line, sizeof(line), file_pointer)) {
        char *token = strtok(line, " ");
        if (token != NULL) {
            strcpy(applications[num_applications].app_name, token);
            token = strtok(NULL, " ");
            if (token != NULL) {
                applications[num_applications].num_windows = atoi(token);
                applications[num_applications].num_processes = 0;
                num_applications++;
            }
        }
    }


    fclose(file_pointer);
}


void launch_applications() {
    int i, j;
    for (i = 0; i < num_applications; i++) {
        for (j = 0; j < applications[i].num_windows; j++) {
            pid_t pid = fork();
            if (pid == 0) {
                char *args[] = {applications[i].app_name, NULL};
                execvp(applications[i].app_name, args);
                printf("Gagal membuka %s\n", applications[i].app_name);
                exit(1);
            } else {
                applications[i].process_ids[applications[i].num_processes++] = pid;
            }
        }
    }
}


void terminate_applications(char *filename) {
    int i, j;
    if (filename == NULL) {
        for (i = 0; i < num_applications; i++) {
            for (j = 0; j < applications[i].num_processes; j++) {
                kill(applications[i].process_ids[j], SIGTERM);
            }
        }
    } else {
        for (i = 0; i < num_applications; i++) {
            if (strcmp(applications[i].app_name, filename) == 0) {
                for (j = 0; j < applications[i].num_processes; j++) {
                    kill(applications[i].process_ids[j], SIGTERM);
                }
                break;
            }
        }
    }


    for (i = 0; i < num_applications; i++) {
        for (j = 0; j < applications[i].num_processes; j++) {
            waitpid(applications[i].process_ids[j], NULL, 0);
        }
        applications[i].num_processes = 0;
    }


    num_applications = 0;
}


void signal_handler(int sig) {
    terminate_applications(NULL);
    exit(0);
}


int main(int argc, char *argv[]) {
    if (argc < 2) {
        printf("Penggunaan: %s -o <app1> <num1> <app2> <num2>... atau %s -f <file.conf> atau %s -k [<file.conf>]\n", argv[0], argv[0], argv[0]);
        return 1;
    }


    signal(SIGINT, signal_handler);
    signal(SIGTERM, signal_handler);


    if (strcmp(argv[1], "-o") == 0) {
        parse_arguments(argc, argv);
        launch_applications();
    } else if (strcmp(argv[1], "-f") == 0) {
        if (argc < 3) {
            printf("Penggunaan: %s -f <file.conf>\n", argv[0]);
            return 1;
        }
        read_configuration_file(argv[2]);
        launch_applications();
    } else if (strcmp(argv[1], "-k") == 0) {
        if (argc == 2) {
            terminate_applications(NULL);
        } else {
            terminate_applications(argv[2]);
        }
    } else {
        printf("Argumen tidak valid\n");
        return 1;
    }


    return 0;
}

```
## Penjelasan Struktur Kode 

1. Header File dan Deklarasi Awal
  Kode dimulai dengan inklusi beberapa header file standar C yang diperlukan untuk operasi sistem (stdio, stdlib, string, unistd, sys/wait, signal).
  Konstanta MAX_APPS didefinisikan untuk menetapkan jumlah maksimum aplikasi yang dapat dikelola.
2. Definisi Struktur Data
  Application: Struktur data yang merepresentasikan sebuah aplikasi.
    app_name: Nama aplikasi (string).
    num_windows: Jumlah jendela aplikasi yang akan diluncurkan.
    process_ids: Array pid_t yang menyimpan ID proses dari aplikasi yang diluncurkan.
    num_processes: Jumlah proses yang sedang berjalan untuk aplikasi tersebut.
3. Fungsi parse_arguments
  Fungsi ini digunakan untuk membaca dan menguraikan argumen baris perintah yang diberikan kepada program.
  Argumen baris perintah dipecah berdasarkan pola tertentu untuk menentukan aplikasi mana yang harus       diluncurkan dan dengan berapa banyak jendela.
  Informasi ini disimpan dalam array applications.
4. Fungsi read_configuration_file
  Fungsi ini digunakan untuk membaca konfigurasi aplikasi dari sebuah file.
  Setiap baris dalam file konfigurasi berisi nama aplikasi dan jumlah jendela yang harus diluncurkan.
  Informasi ini juga disimpan dalam array applications.
5. Fungsi launch_applications
  Fungsi ini mengelola peluncuran aplikasi berdasarkan informasi yang telah diperoleh dari argumen baris   perintah atau file konfigurasi.
  Setiap aplikasi diluncurkan dalam loop menggunakan fork().
  Proses anak (child process) akan mengeksekusi aplikasi dengan bantuan execvp.
  PID dari setiap proses aplikasi disimpan untuk pemantauan dan penghentian nantinya.
6. Fungsi terminate_applications
  Fungsi ini digunakan untuk menghentikan aplikasi-aplikasi yang sedang berjalan.
  Dapat dijalankan untuk menghentikan semua aplikasi atau hanya aplikasi tertentu berdasarkan nama.
7. Fungsi signal_handler dan main
   signal_handler: Fungsi penangan sinyal yang akan memastikan bahwa aplikasi dapat dihentikan dengan aman saat menerima sinyal seperti SIGINT (CTRL+C) atau SIGTERM.
   main: Fungsi utama yang menangani logika dasar program.
      Memeriksa argumen baris perintah untuk menentukan tindakan yang diperlukan (peluncuran aplikasi, baca dari file konfigurasi, atau penghentian aplikasi).
      Mengatur penangan sinyal dan memanggil fungsi-fungsi yang sesuai berdasarkan argumen yang diberikan.

## file.conf

Isi skrip saya :


```bash
firefox 2
wireshark 3
```


Kegunaan kedua kode tadi : 


Kode ini berguna untuk mengelola peluncuran dan penghentian aplikasi secara otomatis berdasarkan konfigurasi yang diberikan. Pengguna dapat menjalankan aplikasi dengan jumlah jendela yang diinginkan atau membaca konfigurasi dari file eksternal untuk mengotomatiskan proses peluncuran aplikasi.


### hasil run:
test case 1 :








Agar aggregate_minutes_to_hourly_log.sh dapat dijalankan tiap jam, kita dapat menambahkan line berikut pada crontab:


```bash
0 * * * * /{path aggregate_minutes_to_hourly_log.sh}
```


# Kendala yang dialami 
Kendala yang saya alami selama pengerjaan soal nomor 4 ini adalah hasil dari eksekusi aggregate per-jamnya yang memiliki nilai tidak wajar yaitu memiliki nilai memory satuan yang harusnya ribuan, seperti berikut : 


![alt text](https://cdn.discordapp.com/attachments/1176233896292122634/1221504764857028608/image.png?ex=6612d1f6&is=66005cf6&hm=ca3e8774192fa09367ce9ed864cf1756ccffc99fc2c3adb462e53d329592f91b&)


yang harusnya seperti ini : 


![alt text](https://media.discordapp.net/attachments/1176233896292122634/1223602120142360708/image.png?ex=661a7346&is=6607fe46&hm=69e0e547b134883cb8dc5219cf313645c88033e5346d2d5035f88e00740fbf36&=&format=webp&quality=lossless&width=880&height=175)


Sudah saya coba membuat berbagai logika, tetapi hasilnya antara aggregate tidak muncul. Atau hasilnya menjadi terlalu besar.



