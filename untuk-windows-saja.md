Menggunakan Bun untuk mengembangkan perangkat lunak desktop yang dapat dieksekusi mandiri untuk Windows

Pertama, mari kita buat direktori proyek baru. Saya sudah menggunakannya gui, dan Anda dapat menggunakannya sesuai keinginan.

Kemudian buka terminal cd ke direktori proyek bun inituntuk menginisialisasi proyek


PS C:\Users\...\Desktop\Bun\gui> bun init

bun init helps you get started with a minimal project and tries to guess sensible defaults. Press ^C anytime to quit

package name (gui):
entry point (index.ts): main.js

Done! A package.json file was saved in the current directory.
 + main.js
 + .gitignore
 + jsconfig.json (for editor auto-complete)
 + README.md

To get started, run:

  bun run main.js

PS C:\Users\...\Desktop\Bun\gui>

Saya biasa menggunakan pustaka webview dan mengompilasinya menjadi dll agar bun dapat dipanggil. Tanpa diduga, saya menemukan bahwa seorang teman asing dari webview-bun telah mengemas pustaka tersebut, sehingga saya tidak perlu membangun webview sendiri.

Setelah inisialisasi selesai, kami menginstalbun i webview-bun

Mari membuat yang baru ui.htmluntuk menulis antarmuka pengguna perangkat lunak kita


<div>
  <h1>Halo, monster kecil</h1>
</div>
<div>
  <button id="increment">+</button>
  <button id="decrement">−</button>
  <span>Counter: <span id="counterResult">0</span></span>
</div>
<div>
  <button id="compute">Compute</button>
  <span>结果: <span id="computeResult">(not started)</span></span>
</div>
<script type="module">
  const ui = {};
  document.querySelectorAll('[id]').forEach((e) => (ui[e.id] = e));

  ui.increment.addEventListener('click', async () => {
    ui.counterResult.textContent = await window.count(1);
  });
  ui.decrement.addEventListener('click', async () => {
    ui.counterResult.textContent = await window.count(-1);
  });
  ui.compute.addEventListener('click', async () => {
    ui.compute.disabled = true;
    ui.computeResult.textContent = '(pending)';
    ui.computeResult.textContent = await window.compute(6, 7);
    ui.compute.disabled = false;
  });
  // Nonaktifkan menu klik kanan
  document.addEventListener('contextmenu', (e) => e.preventDefault());
</script>

Kita masukkan kode pengikatan backend di main.js


import { Webview, SizeHint } from "webview-bun";
// Sematkan html ke dalam file yang dapat dieksekusi terpisah, dan kompilasi tanpa ui.html
import html from "./ui.html" with { type: "text" };

// Parameter pertama adalah menonaktifkan alat pengembang SizeHint.FIXED adalah menonaktifkan maksimalisasi jendela
const webview = new Webview(false, { height: 480, width: 640, hint: SizeHint.FIXED });

let counter = 0;
// Ikat acara tombol front-end (bertambah, berkurang)
webview.bind("count", (a) => counter += a);

// Ikat acara tombol front-end (komputasi)
webview.bind("compute", (a, b) => a + b);

// Title Aplikasi
webview.title = "Bun App";
webview.setHTML(html);
webview.run();


Sekarang kita bisa bun run main.jsmelihat efeknya

Apakah kecepatannya sangat cepat? Dimulai dalam hitungan detik (bahkan lebih cepat setelah kompilasi dan pengemasan)

Akhirnya, kami mengkompilasi dan mengemas

--minify : Volume minimal

--compile -:Mengkompilasi satu executable

-windows-hide-console : Sembunyikan konsol Windows (jika tidak disembunyikan, akan ada jendela hitam besar di belakang perangkat lunak, yang jelek)

TERMINAL

bun build ./main.js --compile --minify --windows-hide-console --outfile myapp
Setelah dikompilasi, direktori Anda akan memiliki tambahanmyapp.exe

Tentu saja, Anda juga dapat menambahkan ikon ke file yang dapat dieksekusi

Cukup tambahkan ke perintah kompilasi--windows-icon=./icon.ico

bun build ./main.js --compile --minify --windows-hide-console --windows-icon=./icon.ico --outfile myapp
