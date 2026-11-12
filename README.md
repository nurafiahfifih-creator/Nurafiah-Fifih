<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>CBT Informatika Kelas X</title>
  <style>
    * { box-sizing: border-box; user-select: none; -webkit-user-select: none; }
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f0f2f5; margin: 0; padding: 0; color: #333; }
    
    /* Header & Timer Sticky */
    header { position: sticky; top: 0; background: #1a73e8; color: white; padding: 12px 16px; display: flex; justify-content: space-between; align-items: center; z-index: 100; box-shadow: 0 2px 5px rgba(0,0,0,0.2); }
    .timer { font-weight: bold; background: #fff; color: #d93025; padding: 6px 12px; border-radius: 20px; font-size: 15px; }
    .pelanggaran-badge { background: #f9ab00; color: #202124; font-size: 12px; padding: 4px 8px; border-radius: 12px; font-weight: bold; }

    .container { max-width: 650px; margin: 15px auto; padding: 15px; }
    .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.12); margin-bottom: 15px; }
    
    /* Form Input */
    .form-group { margin-bottom: 15px; }
    .form-group label { display: block; font-weight: 600; margin-bottom: 6px; font-size: 14px; }
    .form-group input { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 8px; font-size: 16px; }

    /* Soal Card */
    .soal-meta { display: flex; justify-content: space-between; font-size: 12px; color: #5f6368; margin-bottom: 8px; text-transform: uppercase; font-weight: bold; }
    .soal-text { font-size: 15px; font-weight: 500; line-height: 1.5; margin-bottom: 15px; }
    pre { background: #272822; color: #f8f8f2; padding: 10px; border-radius: 6px; overflow-x: auto; font-size: 13px; font-family: monospace; }

    /* Opsi Pilihan */
    .option-label { display: flex; align-items: flex-start; padding: 10px 12px; border: 1px solid #dadce0; border-radius: 8px; margin-bottom: 8px; cursor: pointer; transition: 0.2s; font-size: 14px; }
    .option-label:active { background: #e8f0fe; }
    .option-label input { margin-right: 12px; margin-top: 3px; transform: scale(1.2); }

    /* Tombol */
    .btn { width: 100%; padding: 14px; border: none; border-radius: 8px; font-size: 16px; font-weight: bold; cursor: pointer; }
    .btn-primary { background: #1a73e8; color: white; }
    .btn-success { background: #1e8e3e; color: white; margin-top: 15px; }
    
    /* Dialog Peringatan Kecurangan */
    .alert-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.85); z-index: 999; justify-content: center; align-items: center; padding: 20px; }
    .alert-box { background: white; padding: 24px; border-radius: 12px; text-align: center; max-width: 400px; width: 100%; }
    .alert-box h3 { color: #d93025; margin-top: 0; }
  </style>
</head>
<body oncontextmenu="return false;">

  <!-- Screen Alert Kecurangan -->
  <div id="cheatAlert" class="alert-overlay">
    <div class="alert-box">
      <h3>⚠️ PERINGATAN KECURANGAN!</h3>
      <p id="cheatMsg">Anda terdeteksi keluar dari layar ujian atau membuka aplikasi lain!</p>
      <p>Sisa toleransi pelanggaran: <b id="sisaPelanggaran">3</b> kali lagi.</p>
      <button class="btn btn-primary" onclick="tutupPeringatan()">KEMBALI KE UJIAN</button>
    </div>
  </div>

  <!-- Form Login Siswa -->
  <div class="container" id="loginBox">
    <div class="card">
      <h2 style="text-align: center; color: #1a73e8; margin-top: 0;">CBT INFORMATIKA X</h2>
      <p style="font-size: 13px; color: #666; text-align: center;">Ujian berlangsung selama 60 menit. Dilarang berpindah aplikasi!</p>
      <div class="form-group">
        <label>Nama Lengkap Siswa</label>
        <input type="text" id="siswaNama" placeholder="Masukkan Nama Lengkap" required>
      </div>
      <div class="form-group">
        <label>Kelas</label>
        <input type="text" id="siswaKelas" placeholder="Contoh: X MIPA 1 / X-A" required>
      </div>
      <div class="form-group">
        <label>Nomor Induk Siswa (NIS)</label>
        <input type="number" id="siswaNIS" placeholder="Masukkan NIS" required>
      </div>
      <button class="btn btn-primary" onclick="mulaiUjian()">MULAI UJIAN (FULLSCREEN)</button>
    </div>
  </div>

  <!-- Area Soal Ujian -->
  <div id="examBox" style="display: none;">
    <header>
      <span class="pelanggaran-badge" id="badgePelanggaran">Pelanggaran: 0/3</span>
      <div class="timer" id="timerDisplay">60:00</div>
    </header>

    <div class="container">
      <form id="quizForm">
        <div id="soalContainer"></div>
        <button type="button" class="btn btn-success" onclick="konfirmasiKirim()">SELESAI & KUMPULKAN</button>
      </form>
    </div>
  </div>

  <script>
    // MASUKKAN URL GOOGLE APPS SCRIPT ANDA DI SINI
    const GOOGLE_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbz-ghL81u5GGLOmrNZXlADDelHslqkDYnk1tG1ABa8OYgEgYfC4c6BAalZ0CxYKwcwxyg/execDA";

    let waktuUjianDetik = 60 * 60; // 60 menit
    let timerInterval = null;
    let pelanggaran = 0;
    const MAKS_PELANGGARAN = 3;
    let ujianSelesai = false;

    // BANK 40 SOAL INFORMATIKA KELAS 10 (TAKSONOMI SOLO)
    const bankSoal = [
      // Topik 1: Informatika & Kemampuan Generik (5 Soal)
      { id: 1, topic: "Informatika & Generik", solo: "Unistructural", q: "Salah satu pilar kemampuan generik dalam pembelajaran abad ke-21 yang menekankan kerja sama tim adalah...", opt: ["Komputasi", "Kolaborasi", "Algoritma", "Kompilasi"], ans: 1 },
      { id: 2, topic: "Informatika & Generik", solo: "Unistructural", q: "Media penyampaian informasi visual berupa ringkasan teks dan grafik yang menarik disebut...", opt: ["Infografis", "Pseudocode", "Flowchart", "Logbook"], ans: 0 },
      { id: 3, topic: "Informatika & Generik", solo: "Multistructural", q: "Komponen utama yang harus ada pada slide presentasi tugas informatika yang efektif mencakup...", opt: ["Hanya paragraf panjang dan detail", "Teks ringkas berbobot, visual pendukung relevan, alur logis", "Animasi warna-warni tanpa batas topik", "Format kode mentah tanpa rangkuman"], ans: 1 },
      { id: 4, topic: "Informatika & Generik", solo: "Relational", q: "Hubungan mendasar antara berpikir komputasional dengan disiplin ilmu informatika adalah...", opt: ["Informatika hanya perangkat lunak, berpikir komputasional perangkat keras", "Berpikir komputasional adalah pondasi pola pikir untuk menyelesaikan masalah berbantuan komputer", "Keduanya tidak berhubungan satu sama lain", "Informatika menggantikan seluruh aktivitas berpikir manusia"], ans: 1 },
      { id: 5, topic: "Informatika & Generik", solo: "Extended Abstract", q: "Jika sebuah tim proyek informatika gagal menyepakati pembagian tugas akibat perbedaan pemahaman teknis, pendekatan generik paling solutif adalah...", opt: ["Menyerahkan seluruh proyek pada satu programmer inti", "Membuat kesepakatan target (milestone), matriks pembagian peran, dan log komunikasi berkala", "Menghapus bagian yang sulit dari proyek", "Membatalkan seluruh pengerjaan modul"], ans: 1 },

      // Topik 2: Berpikir Komputasional (Searching, Sorting, Stack, Queue) (12 Soal)
      { id: 6, topic: "Berpikir Komputasional", solo: "Unistructural", q: "Prinsip kerja struktur data Antrean (Queue) adalah...", opt: ["LIFO (Last In First Out)", "FIFO (First In First Out)", "Random In Out", "Priority In Last Out"], ans: 1 },
      { id: 7, topic: "Berpikir Komputasional", solo: "Unistructural", q: "Prinsip dasar pada struktur tumpukan (Stack) di mana elemen terakhir dimasukkan akan keluar pertama disebut...", opt: ["FIFO", "LIFO", "Tree", "Graph"], ans: 1 },
      { id: 8, topic: "Berpikir Komputasional", solo: "Unistructural", q: "Syarat mutlak agar algoritma Binary Search dapat dijalankan pada sekumpulan data adalah...", opt: ["Data harus dalam format acak", "Data harus sudah dalam keadaan terurut", "Data harus bertipe float", "Jumlah elemen data wajib ganjil"], ans: 1 },
      { id: 9, topic: "Berpikir Komputasional", solo: "Multistructural", q: "Operasi dasar untuk menambah data dan menghapus data pada struktur Stack berturut-turut adalah...", opt: ["Enqueue dan Dequeue", "Push dan Pop", "Insert dan Drop", "Append dan Popleft"], ans: 1 },
      { id: 10, topic: "Berpikir Komputasional", solo: "Multistructural", q: "Dua metode algoritma pengurutan (sorting) yang umum dipelajari di kelas 10 adalah...", opt: ["Binary Search & Linear Search", "Bubble Sort & Insertion Sort", "Breadth First & Depth First", "Push & Pop"], ans: 1 },
      { id: 11, topic: "Berpikir Komputasional", solo: "Relational", q: "Terdapat data terurut: [3, 8, 12, 19, 25, 30, 42]. Jika mencari nilai 25 dengan Binary Search, perbandingan pertama dilakukan dengan nilai...", opt: ["3", "19", "25", "42"], ans: 1 },
      { id: 12, topic: "Berpikir Komputasional", solo: "Relational", q: "Fitur tombol 'Undo' pada text editor dan riwayat 'Back' di browser web paling tepat dimodelkan menggunakan...", opt: ["Queue (Antrean)", "Stack (Tumpukan)", "Bubble Sort", "Linear Search"], ans: 1 },
      { id: 13, topic: "Berpikir Komputasional", solo: "Relational", q: "Diberikan antrean cetak printer: Dokumen A, lalu Dokumen B, lalu Dokumen C. Dokumen yang selesai dicetak pertama adalah...", opt: ["Dokumen C", "Dokumen A", "Dokumen B", "Semua bersamaan"], ans: 1 },
      { id: 14, topic: "Berpikir Komputasional", solo: "Relational", q: "Sebuah array [5, 2, 8, 1] diurutkan menaik dengan Bubble Sort. Kondisi susunan array setelah iterasi putaran pertama (Pass 1) adalah...", opt: ["1, 2, 5, 8", "2, 5, 1, 8", "2, 1, 5, 8", "5, 2, 1, 8"], ans: 1 },
      { id: 15, topic: "Berpikir Komputasional", solo: "Relational", q: "Diberikan stack kosong. Operasi: Push(7), Push(3), Pop(), Push(9), Push(2), Pop(). Nilai di puncak stack sekarang adalah...", opt: ["7", "9", "3", "2"], ans: 1 },
      { id: 16, topic: "Berpikir Komputasional", solo: "Extended Abstract", q: "Kasus antrean nasabah bank dengan nasabah Prioritas VIP membutuhkan modifikasi struktur data Queue konvensional menjadi...", opt: ["Stack murni", "Priority Queue berdasarkan atribut tingkat urgensi", "Linear Search Array", "Hash Map statis"], ans: 1 },
      { id: 17, topic: "Berpikir Komputasional", solo: "Extended Abstract", q: "Ketika data berjumlah jutaan dan sering diperbarui acak, Binary Search tidak selalu lebih cepat dibanding Linear Search jika...", opt: ["Biaya waktu pengurutan (sorting) data awal jauh lebih besar dibanding pencarian itu sendiri", "Linear Search menggunakan memori lebih besar", "Komputer mengalami overheat", "Binary Search hanya untuk string"], ans: 0 },

      // Topik 3: Sistem Komputer (7 Soal)
      { id: 18, topic: "Sistem Komputer", solo: "Unistructural", q: "Komponen CPU yang bertanggung jawab melakukan operasi aritmatika dan logika adalah...", opt: ["Control Unit (CU)", "Arithmetic Logic Unit (ALU)", "Register", "Cache"], ans: 1 },
      { id: 19, topic: "Sistem Komputer", solo: "Unistructural", q: "Memori utama yang bersifat volatil (hilang data saat listrik mati) adalah...", opt: ["ROM", "Hard Disk", "RAM", "Flashdisk"], ans: 2 },
      { id: 20, topic: "Sistem Komputer", solo: "Multistructural", q: "Contoh perangkat keras yang berfungsi sebagai alat masukan (Input Device) sekaligus keluaran (Output Device) adalah...", opt: ["Keyboard dan Mouse", "Layar Sentuh (Touchscreen)", "Monitor LED", "Speaker"], ans: 1 },
      { id: 21, topic: "Sistem Komputer", solo: "Multistructural", q: "Nilai bilangan biner 1011 dalam format desimal bernilai...", opt: ["9", "11", "13", "15"], ans: 1 },
      { id: 22, topic: "Sistem Komputer", solo: "Relational", q: "Mengapa sistem operasi (OS) menjadi penghubung penting antara hardware dan software aplikasi?", opt: ["Karena OS mengalokasikan resource hardware secara adil dan menerjemahkan instruksi program", "Karena OS mematikan sirkuit saat aplikasi error", "OS hanya mempercantik tampilan grafis", "OS bertugas merakit hardware secara fisik"], ans: 0 },
      { id: 23, topic: "Sistem Komputer", solo: "Relational", q: "Jika sebuah komputer mengalami lag saat membuka banyak aplikasi sekaligus padahal CPU rendah, indikasi masalah terbesarnya adalah...", opt: ["Kapasitas RAM tidak mencukupi sehingga terjadi swapping berat", "Monitor kehilangan refresh rate", "Kabel power supply kendor", "Mouse kehabisan baterai"], ans: 0 },
      { id: 24, topic: "Sistem Komputer", solo: "Extended Abstract", q: "Kombinasi teknologi penyimpanan SSD (Solid State Drive) menggantikan HDD konvensional memberikan lompatan performa sistem karena...", opt: ["Tidak ada komponen mekanik bergerak sehingga waktu akses *latency* mendekati instan", "SSD menggunakan kaset magnetik berkecepatan putar ganda", "SSD tidak memerlukan listrik sama sekali", "SSD otomatis memperbaiki kode program yang rusak"], ans: 0 },

      // Topik 4: Jaringan Komputer dan Internet (7 Soal)
      { id: 25, topic: "Jaringan Komputer", solo: "Unistructural", q: "Protokol standar transfer dokumen web yang dilengkapi dengan lapisan enkripsi keamanan adalah...", opt: ["HTTP", "HTTPS", "FTP", "SMTP"], ans: 1 },
      { id: 26, topic: "Jaringan Komputer", solo: "Unistructural", q: "Format penulisan IPv4 terdiri dari sekumpulan angka biner sepanjang...", opt: ["16 bit", "32 bit", "64 bit", "128 bit"], ans: 1 },
      { id: 27, topic: "Jaringan Komputer", solo: "Multistructural", q: "Topologi jaringan di mana semua workstation terhubung ke satu perangkat pusat (Switch/Hub) disebut...", opt: ["Topologi Bus", "Topologi Cincin (Ring)", "Topologi Bintang (Star)", "Topologi Mesh"], ans: 2 },
      { id: 28, topic: "Jaringan Komputer", solo: "Multistructural", q: "Perangkat jaringan yang berfungsi menentukan rute terbaik dalam melewatkan paket data antar segmen jaringan yang berbeda adalah...", opt: ["Modem", "Switch", "Router", "Repeater"], ans: 2 },
      { id: 29, topic: "Jaringan Komputer", solo: "Relational", q: "Perbedaan mendasar antara koneksi kabel serat optik (Fiber Optic) dengan kabel tempa (UTP) adalah...", opt: ["Fiber Optic mentransmisikan data dalam bentuk pulsa cahaya dan kebal interferensi elektromagnetik", "UTP memiliki jarak transmisi ratusan kilometer tanpa repeater", "Fiber Optic menggunakan frekuensi radio bebas", "UTP memiliki transmisi kecepatan cahaya"], ans: 0 },
      { id: 30, topic: "Jaringan Komputer", solo: "Relational", q: "Jika Anda terhubung ke Wi-Fi publik tanpa password lalu mengakses akun bank tanpa protokol HTTPS, ancaman keamanan yang paling mungkin terjadi adalah...", opt: ["Serangan Man-in-the-Middle (Sniffing paket data sensitif)", "Komputer langsung terbakar", "Hard disk terkena bad sector fisik", "Layar smartphone berganti resolusi"], ans: 0 },
      { id: 31, topic: "Jaringan Komputer", solo: "Extended Abstract", q: "Mengapa transisi global dari IPv4 ke IPv6 menjadi keharusan mutlak dalam era Internet of Things (IoT)?", opt: ["IPv4 hanya bisa digunakan di komputer desktop lama", "Ruang alamat IPv4 (sekitar 4,3 miliar) telah habis untuk mengakomodasi miliaran gawai baru di dunia", "IPv6 membuat kecepatan kabel menjadi dua kali lipat secara fisik", "IPv4 tidak mendukung transfer gambar"], ans: 1 },

      // Topik 5: Google Colab & Python Sederhana (9 Soal)
      { id: 32, topic: "Google Colab & Python", solo: "Unistructural", q: "Google Colaboratory (Colab) adalah lingkungan komputasi interaktif berbasis cloud yang mengeksekusi format file...", opt: [".docx", ".ipynb (Jupyter Notebook)", ".apk", ".xlsx"], ans: 1 },
      { id: 33, topic: "Google Colab & Python", solo: "Unistructural", q: "Fungsi bawaan Python yang digunakan untuk menampilkan data/teks ke layar konsol adalah...", opt: ["echo()", "read()", "print()", "input()"], ans: 2 },
      { id: 34, topic: "Google Colab & Python", solo: "Unistructural", q: "Karakter yang digunakan untuk menulis baris komentar (tidak dieksekusi mesin) pada kode Python adalah...", opt: ["//", "/*", "#", "--"], ans: 2 },
      { id: 35, topic: "Google Colab & Python", solo: "Multistructural", q: "Perhatikan baris kode: `x = 7.5`. Tipe data dari variabel `x` tersebut adalah...", opt: ["Integer", "String", "Float", "Boolean"], ans: 2 },
      { id: 36, topic: "Google Colab & Python", solo: "Relational", q: "Perhatikan potongan kode Python berikut:<br><pre>nilai = 75\nif nilai >= 80:\n    print('A')\nelif nilai >= 70:\n    print('B')\nelse:\n    print('C')</pre>Output yang dihasilkan adalah...", opt: ["A", "B", "C", "Error"], ans: 1 },
      { id: 37, topic: "Google Colab & Python", solo: "Relational", q: "Perhatikan perulangan Python berikut:<br><pre>total = 0\nfor i in range(1, 4):\n    total += i\nprint(total)</pre>Nilai `total` di akhir eksekusi adalah...", opt: ["3", "6", "10", "4"], ans: 1 },
      { id: 38, topic: "Google Colab & Python", solo: "Relational", q: "Perhatikan kode list Python berikut:<br><pre>buah = ['Apel', 'Jeruk', 'Mangga']\nbuah.append('Pisang')\nprint(buah[2])</pre>Output yang dicetak adalah...", opt: ["Jeruk", "Mangga", "Pisang", "Apel"], ans: 1 },
      { id: 39, topic: "Google Colab & Python", solo: "Extended Abstract", q: "Kelebihan utama Google Colab bagi siswa yang belajar pemrograman pengolahan data atau AI dibanding instalasi Python lokal di HP adalah...", opt: ["Google Colab tidak membutuhkan koneksi internet sama sekali", "Penyediaan komputasi gratis GPU/TPU di cloud tanpa membebani hardware perangkat pengguna", "Colab dapat berjalan tanpa akun Google", "Colab tidak memerlukan browser"], ans: 1 },
      { id: 40, topic: "Google Colab & Python", solo: "Extended Abstract", q: "Diberikan algoritma Python:<br><pre>def hitung(n):\n    if n == 1:\n        return 1\n    return n + hitung(n-1)\nprint(hitung(4))</pre>Prinsip apa yang diterapkan dalam fungsi tersebut dan berapa hasil cetaknya?", opt: ["Iterasi for-loop biasa, hasil: 4", "Rekursi (pemanggilan fungsi diri sendiri), hasil: 10", "Pencarian biner, hasil: 7", "Stack overflow otomatis, hasil: 0"], ans: 1 }
    ];

    // RENDER SEMUA SOAL
    function renderSoal() {
      const container = document.getElementById("soalContainer");
      container.innerHTML = bankSoal.map((s, idx) => `
        <div class="card">
          <div class="soal-meta">
            <span>No. ${idx + 1} &bull; ${s.topic}</span>
            <span>SOLO: ${s.solo}</span>
          </div>
          <div class="soal-text">${s.q}</div>
          <div>
            ${s.opt.map((pilihan, optIdx) => `
              <label class="option-label">
                <input type="radio" name="soal_${s.id}" value="${optIdx}" required>
                <span><b>${String.fromCharCode(65 + optIdx)}.</b> ${pilihan}</span>
              </label>
            `).join('')}
          </div>
        </div>
      `).join('');
    }

    // MULAI UJIAN & FULLSCREEN
    function mulaiUjian() {
      const nama = document.getElementById("siswaNama").value.trim();
      const kelas = document.getElementById("siswaKelas").value.trim();
      const nis = document.getElementById("siswaNIS").value.trim();

      if (!nama || !kelas || !nis) {
        alert("Harap lengkapi semua data identitas terlebih dahulu!");
        return;
      }

      // Masuk Fullscreen (Android/Mobile Browser)
      const elem = document.documentElement;
      if (elem.requestFullscreen) {
        elem.requestFullscreen().catch(err => console.log(err));
      } else if (elem.webkitRequestFullscreen) {
        elem.webkitRequestFullscreen();
      }

      document.getElementById("loginBox").style.display = "none";
      document.getElementById("examBox").style.display = "block";
      
      renderSoal();
      mulaiTimer();
      aktifkanDeteksiKecurangan();
    }

    // DETEKSI PINDAH APLIKASI / SPLIT SCREEN / HOME BUTTON
    function aktifkanDeteksiKecurangan() {
      document.addEventListener("visibilitychange", function() {
        if (document.hidden && !ujianSelesai) {
          catatPelanggaran("Berpindah aplikasi atau membuka tab baru");
        }
      });

      window.addEventListener("blur", function() {
        if (!ujianSelesai) {
          catatPelanggaran("Meninggalkan fokus aplikasi layar ujian");
        }
      });
    }

    function catatPelanggaran(alasan) {
      pelanggaran++;
      document.getElementById("badgePelanggaran").innerText = `Pelanggaran: ${pelanggaran}/${MAKS_PELANGGARAN}`;
      
      const sisa = MAKS_PELANGGARAN - pelanggaran;
      document.getElementById("sisaPelanggaran").innerText = sisa > 0 ? sisa : 0;
      document.getElementById("cheatAlert").style.display = "flex";

      if (pelanggaran >= MAKS_PELANGGARAN) {
        alert("Batas toleransi kecurangan habis! Jawaban Anda otomatis terkirim sekarang.");
        tutupPeringatan();
        kirimJawabanOtomatis();
      }
    }

    function tutupPeringatan() {
      document.getElementById("cheatAlert").style.display = "none";
      // Paksa kembali fullscreen
      const elem = document.documentElement;
      if (elem.requestFullscreen) {
        elem.requestFullscreen().catch(err => console.log(err));
      }
    }

    // PENGATUR TIMER
    function mulaiTimer() {
      const display = document.getElementById("timerDisplay");
      timerInterval = setInterval(() => {
        waktuUjianDetik--;
        let menit = Math.floor(waktuUjianDetik / 60);
        let detik = waktuUjianDetik % 60;
        display.innerText = `${menit < 10 ? '0' : ''}${menit}:${detik < 10 ? '0' : ''}${detik}`;

        // Peringatan sisa 5 menit
        if (waktuUjianDetik === 300) {
          alert("PERHATIAN: Waktu ujian tersisa 5 menit lagi!");
          display.style.background = "#ea4335";
          display.style.color = "#fff";
        }

        if (waktuUjianDetik <= 0) {
          clearInterval(timerInterval);
          alert("WAKTU HABIS! Ujian otomatis dikumpulkan.");
          kirimJawabanOtomatis();
        }
      }, 1000);
    }

    // PROSES SUBMISI DATA KE GOOGLE DRIVE / APPS SCRIPT
    function konfirmasiKirim() {
      const konfirmasi = confirm("Apakah Anda yakin ingin menyelesaikan dan mengirim ujian?");
      if (konfirmasi) {
        kirimJawabanOtomatis();
      }
    }

    function hitungNilai() {
      let skorBenar = 0;
      bankSoal.forEach(s => {
        const selected = document.querySelector(`input[name="soal_${s.id}"]:checked`);
        if (selected && parseInt(selected.value) === s.ans) {
          skorBenar++;
        }
      });
      const nilaiAkhir = Math.round((skorBenar / bankSoal.length) * 100);
      return { skorBenar, nilaiAkhir };
    }

    function kirimJawabanOtomatis() {
      ujianSelesai = true;
      clearInterval(timerInterval);

      const hasil = hitungNilai();
      const payload = {
        nama: document.getElementById("siswaNama").value,
        kelas: document.getElementById("siswaKelas").value,
        nis: document.getElementById("siswaNIS").value,
        benar: hasil.skorBenar,
        totalSoal: bankSoal.length,
        skor: hasil.nilaiAkhir,
        pelanggaran: pelanggaran
      };

      // Tampilan status pengiriman
      document.body.innerHTML = `
        <div class="container" style="text-align: center; margin-top: 50px;">
          <div class="card">
            <h2>Mengirim data ke Google Drive...</h2>
            <p>Mohon jangan menutup browser sebelum selesai.</p>
          </div>
        </div>
      `;

      // Kirim via fetch API ke Google Apps Script
      fetch(GOOGLE_SCRIPT_URL, {
        method: "POST",
        mode: "no-cors",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      }).then(() => {
        document.body.innerHTML = `
          <div class="container" style="text-align: center; margin-top: 50px;">
            <div class="card">
              <h2 style="color: #1e8e3e;">UJIAN SELESAI</h2>
              <p>Terima kasih, data ujian Anda telah berhasil dicatat ke Google Drive Pengawas.</p>
              <hr>
              <p><b>Nama:</b> ${payload.nama}</p>
              <p><b>Skor Akhir:</b> ${payload.skor} / 100</p>
              <p><b>Benar:</b> ${payload.benar} dari ${payload.totalSoal} Soal</p>
              <p><b>Pelanggaran:</b> ${payload.pelanggaran} kali</p>
            </div>
          </div>
        `;
      }).catch(err => {
        alert("Gagal mengirim data otomatis: " + err);
      });
    }
  </script>
</body>
</html>
