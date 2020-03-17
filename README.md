# DB

## 00 Regenerate ss13me tables (data preparation)

### Penjelasan Fungsi Node:

- **SQLite Connector**: digunakan untuk membuat suatu koneksi pada SQLite database file dengan perantara JDBC.
- **DB Table Remover**: digunakan untuk menghapus tabel sesuai dengan nama tabel yang diinput. (sama halnya dengan _drop table_)
- **DB Table Creator**: digunakan untuk membuat suatu tabel dalam suatu koneksi database sesuai dengan format yang menjadi input. (dalam kasus ini tabel di-load melalui CSV Reader)
- **CSV Reader**: digunakan untuk membaca file dengan format CSV.
- **DB Writer**: digunakan untuk melakukan insert data tabel ke dalam database dan membuat tabel tersebut apabila belum ada sebelumnya.

### Penjelasan Flow Process:

1. Melakukan koneksi database menggunakan **SQLite Connector**
2. Menghapus existing table dengan **DB Table Remover**
3. Membuat tabel menggunakan **DB Table Creator** dimana datanya telah di-load terlebih dahulu menggunakan **CSV Reader** (atau Data reader lainnya)
4. Insert data ke dalam database menggunakan **DB Writer**

## 01 DB Connect

### Penjelasan Fungsi Node:

- **SQLite Connector**: digunakan untuk membuat suatu koneksi pada SQLite database file dengan perantara JDBC.
- **DB Table Selector**: digunakan untuk melakukan fungsi _SELECT_ dalam suatu _database connection_.
- **DB Reader**: digunakan untuk membaca _input query_ dan membuat tabel dalam bentuk hasil query.

### Penjelasan Flow Process:

1. Melakukan koneksi database menggunakan **SQLite Connector**
2. Melakukan select data dari tabel menggunakan **DB Table Selector**
3. Membaca hasil query menggunakan **DB Reader**

## 02 DB InDB Processing

### Penjelasan Fungsi Node:

- **SQLite Connector**: digunakan untuk membuat suatu koneksi pada SQLite database file dengan perantara JDBC.
- **DB Table Selector**: digunakan untuk melakukan fungsi _SELECT_ dalam suatu _database connection_.
- **DB Reader**: digunakan untuk membaca _input query_ dan membuat tabel dalam bentuk hasil query.
- **DB Column Filter**: digunakan untuk melakukan filter kolom antara kolom yang ingin ditampilkan dengan kolom yang tidak ingin ditampilkan.
- **DB Joiner**: digunakan untuk penggabungan 2 tabel dengan mengkombinasikan kolom dari keduanya.
- **DB Row Filter**: digunakan untuk melakukan filter dengan kondisi tertentu untuk menampilkan atau tidak suatu data dari baris tertentu.
- **DB GroupBy**: digunakan untuk mengelompokkan data sesuai dengan patokan terhadap kolom tertentu, dan dapat diolah menggunakan fungsi agregasi.
- **DB Sorter**: digunakan untuk melakukan sorting sesuai dengan kriteria yang diinginkan user.

### Penjelasan Flow Process:

0. Preprocess data

   1. Melakukan koneksi database menggunakan **SQLite Connector**
   2. Melakukan select data dari tabel _0511174000130_ss13pme_ dan _0511174000130_ss13hme_ menggunakan **DB Table Selector**
   3. Menghapus kolom PUMA* dan PWGTP* dari tabel _0511174000130_ss13pme_ dan _0511174000130_ss13hme_ menggunakan **DB Column Filter**

1. **Join ss13hme and ss13pme on SERIALNO**

   1. Menggabungkan hasil filter dari kedua tabel menggunakan **DB Joiner** sesuai dengan data _serialno_
   2. Membaca hasil query menggunakan **DB Reader**

1. **filter all rows from ss13pme where COW is NULL**

   1. Menghilangkan seluruh baris yang tidak memiliki data pada kolom _COW_ menggunakan **DB Row Filter**
   2. Membaca hasil query menggunakan **DB Reader**

1. **filter all rows from ss13pme where COW is NOT NULL**

   1. Menghilangkan seluruh baris yang memiliki data pada kolom _COW_ menggunakan **DB Row Filter**
   2. Membaca hasil query menggunakan **DB Reader**

1. **calculate average AGEP for the different SEX groups**

   1. Dalam konfigurasi "Groups" **DB GroupBy** memilih group column untuk kolom _sex_
   2. Dalam konfigurasi "Manual Aggregation" **DB GroupBy** memilih agregasi _AVG_ untuk kolom _agep_
   3. Membaca hasil query menggunakan **DB Reader**

1. **Sort the data rows by descending AGEP and extract top 10 only**
   1. Menggunakan **DB Sorter** dengan konfigurasi "Descending"
   2. Melakukan limit 10 data teratas menggunakan **DB Query** dengan SQL Statement (SELECT \* FROM #table# AS "table" limit 10)
   3. Membaca hasil query menggunakan **DB Reader**

## 03 DB Modelling

### Penjelasan Fungsi Node:

- **SQLite Connector**: digunakan untuk membuat suatu koneksi pada SQLite database file dengan perantara JDBC.
- **DB Table Selector**: digunakan untuk melakukan fungsi _SELECT_ dalam suatu _database connection_.
- **DB Reader**: digunakan untuk membaca _input query_ dan membuat tabel dalam bentuk hasil query.
- **DB Column Filter**: digunakan untuk melakukan filter kolom antara kolom yang ingin ditampilkan dengan kolom yang tidak ingin ditampilkan.
- **DB Row Filter**: digunakan untuk melakukan filter dengan kondisi tertentu untuk menampilkan atau tidak suatu data dari baris tertentu.
- **Number To String**: digunakan untuk mengubah data angka menjadi string dalam suatu kolom.
- **Decision Tree Learner**: digunakan untuk _process learning_ klasifikasi data dengan algoritma _Decision Tree_.
- **Decision Tree Predictor**: digunakan untuk melakukan _prediction_ klasifikasi data dengan algoritma _Decision Tree_.

### Penjelasan Flow Process

0. Preprocess data

   1. Melakukan koneksi database menggunakan **SQLite Connector**
   2. Melakukan select data dari tabel _0511174000130_ss13pme_ menggunakan **DB Table Selector**
   3. Menghapus kolom PUMA* dan PWGTP* dari tabel _0511174000130_ss13pme_ menggunakan **DB Column Filter**

1. **Train a decision tree on COW where COW is NOT NULL**

   1. Menghilangkan seluruh baris yang tidak memiliki data pada kolom _COW_ menggunakan **DB Row Filter**
   2. Konversi data pada kolom _COW_ menjadi string menggunakan **Number To String**
   3. Training data menggunakan **Decision Tree Learner**

1. **apply decision tree to predict COW value on rows with MISSING COW value**
   1. Menghilangkan seluruh baris yang memiliki data pada kolom _COW_ menggunakan **DB Row Filter** dan menghapus kolom _COW_ menggunakan **DB Column Filter**
   2. Membaca hasil query menggunakan **DB Reader**
   3. Melakukan prediction menggunakan **Decision Tree Predictor**

## 04 DB WritingToDB

### Penjelasan Fungsi Node:

- **SQLite Connector**: digunakan untuk membuat suatu koneksi pada SQLite database file dengan perantara JDBC.
- **DB Table Selector**: digunakan untuk melakukan fungsi _SELECT_ dalam suatu _database connection_.
- **DB Reader**: digunakan untuk membaca _input query_ dan membuat tabel dalam bentuk hasil query.
- **DB Column Filter**: digunakan untuk melakukan filter kolom antara kolom yang ingin ditampilkan dengan kolom yang tidak ingin ditampilkan.
- **DB Row Filter**: digunakan untuk melakukan filter dengan kondisi tertentu untuk menampilkan atau tidak suatu data dari baris tertentu.
- **Number To String**: digunakan untuk mengubah data angka menjadi string dalam suatu kolom.
- **Decision Tree Learner**: digunakan untuk _process learning_ klasifikasi data dengan algoritma _Decision Tree_.
- **Decision Tree Predictor**: digunakan untuk melakukan _prediction_ klasifikasi data dengan algoritma _Decision Tree_.
- **DB Connection Table Writer**: digunakan untuk membuat tabel baru dalam _database connection_ sesuai dengan tabel input
- **DB Writer**: digunakan untuk melakukan insert data tabel ke dalam database dan membuat tabel tersebut apabila belum ada sebelumnya.
- **DB Update**: digunakan untuk melakukan perubahan data dalam database dan membuat tabel baru berupa tabel dengan tambahan kolom status

### Penjelasan Flow Process

_Flow process yang dilakukan pada tahap ini sama dengan bagian 03 DB Modelling dengan tambahan sebagai berikut_

1. Menyimpan tabel awal _05111740000130_ss13pme_ original ke database dengan menggunakan **DB Connection Table Writer**
2. Menyimpan model hasil dari **Decision Tree Learner** ke dalam database menggunakan **DB Writer**
3. Mengubah data pada kolom COW yang berada di dalam database menggunakan **DB Update** dan dilakukan pengecekan kolom status dengan menggunakan **Row Filter** (dimana apabila status dari baris tersebut bernilai '1' berarti 'Update sukses' dan akan diabaikan, sehingga harapannya hasil query menghasilkan tabel kosong atau **seluruh update berhasil dilakukan**)

# Hadoop

## 00 Setup Hive Table

### Penjelasan Fungsi Node:

- **Create Temp Dir**: digunakan untuk membuat directory sementara saat dijalankan dan menyimpan directory tersebut dalam _flow variable_.
- **String Manipulation (Variable)**: digunakan untuk memanipulasi string yang tersimpan dalam variabel.
- **Create Local Big Data Environment**: digunakan untuk membuat environment big data lokal yang fungsional.
- **Table Reader**: digunakan untuk membaca data dalam format .table
- **DB Table Creator**: digunakan untuk membuat tabel baru pada suatu _database connection_
- **DB Loader**: digunakan untuk melakukan _loading data_ pada tabel dalam suatu _database connection_

### Penjelasan Flow Process

#### Setup pada KNIME

1. Membuat Local Big Data Environment menggunakan **Create Local Big Data Environment**
2. Penyesuaian nama directory dengan memanipulasi string directory menggunakan **String Manipulation (Variable)**
3. Membaca data tabel _05111740000130_ss13pme_ dan _05111740000130_ss13hme_ dengan format .table menggunakan **Table Reader**
4. Membuat/Memuat tabel yang telah dibaca ke dalam _Hive connection_ yang dibuat pada **Create Local Big Data Environment** menggunakan **DB Table Creator**
5. Melakukan _loading data_ dalam path HDFS dan _Hive connection_ menggunakan **DB Loader**

#### Uji coba menggunakan DBeaver

1. Membuat koneksi baru dengan membuka menu **New Database Connection**
2. Memilih database **Apache Hive**
3. Kembali ke KNIME, pada **Create Local Big Data Environment** buka **Hive Connection**
4. Kembali ke DBeaver, masukan port yang tertulis pada URL **Hive Connection** dan klik 'Finish'
5. Apabila koneksi telah berhasil, uji coba query tabel seperti **SELECT \* FROM 05111740000130_ss13pme LIMIT 10**
6. Apabila tabel berhasil di-load, maka hasil query akan muncul

## 01 Hive Modelling

_Node dan flow process yang dilakukan pada tahap ini sama dengan bagian DB/03 DB Modelling, perbedaannya pada Hive Modelling koneksi database yang digunakan adalah **Hive Connection** dan bukan **SQLite Connection**_

## 02 Hive WritingToDB

_Node dan flow process yang dilakukan pada tahap ini sama dengan bagian 01 Hive Modelling dengan tambahan sebagai berikut_

### Tambahan Node:

- **Concatenate**: digunakan untuk menggabungkan 2 komponen tabel (memiliki nama dan tipe yang sama), apabila terdapat nama yang berbeda, maka data pada baris tersebut dapat diisi dengan _Missing Value_
- **Create Temp Dir**: digunakan untuk membuat directory sementara saat dijalankan dan menyimpan directory tersebut dalam _flow variable_.
- **String Manipulation (Variable)**: digunakan untuk memanipulasi string yang tersimpan dalam variabel.
- **DB Table Creator**: digunakan untuk membuat tabel baru pada suatu _database connection_
- **DB Loader**: digunakan untuk melakukan _loading data_ pada tabel dalam suatu _database connection_

### Tambahan Proses

1. Menggabungkan hasil _prediction_ dengan tabel _learning_ menggunakan **Concatenate**
2. Memanipulasi variable directory sementara menggunakan **String Manipulation (Variable)** yang menghasilkan _HDFS compatible path_
3. Membuat tabel dengan data yang berasal dari **Concatenate** pada _Hive Connection_ menggunakan **DB Table Creator** dan di-load menggunakan **DB Loader**
