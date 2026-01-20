<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { useAppearance } from '@/composables/useAppearance';
import { Sun, Moon, Laptop2 } from 'lucide-vue-next';
import { Toaster } from 'vue-sonner';
import 'vue-sonner/style.css'; // vue-sonner v2 requires this import
import { toast } from "vue-sonner";

const { appearance, updateAppearance } = useAppearance();

function toggleAppearance() {
  const current = appearance.value;
  const next = current === 'system' ? 'light' : current === 'light' ? 'dark' : 'system';
  updateAppearance(next);
}

// TESTIMONIAL DATA
const testimonials = [
  { text: 'Badan terasa lebih segar setiap pagi.', author: 'Andi' },
  { text: 'Pencernaan jauh lebih lancar.', author: 'Sinta' },
  { text: 'Tekanan darah lebih stabil.', author: 'Budi' },
  { text: 'Cocok untuk multivitamin harian.', author: 'Rina' },
  { text: 'Membantu mengurangi rasa mudah lelah.', author: 'Dewi' },
  { text: 'Tidur lebih nyenyak setelah rutin minum.', author: 'Agus' },
  { text: 'Nafsu makan lebih stabil.', author: 'Herlina' },
  { text: 'Nyeri sendi berkurang perlahan.', author: 'Putra' },
  { text: 'Tidak gampang sakit lagi.', author: 'Joko' },
  { text: 'Kualitas hidup terasa meningkat.', author: 'Mila' },
  { text: 'Tubuh terasa lebih ringan dan bertenaga.', author: 'Syahrul' },
  { text: 'Sangat membantu aktivitas harian saya.', author: 'Lestari' }
];

// CAROUSEL LOGIC
const carouselTrack = ref<HTMLElement | null>(null);
const currentIndex = ref(0);
let interval: any = null;

function updatePosition() {
  if (!carouselTrack.value) return;
  const card = carouselTrack.value.children[0] as HTMLElement;
  if (!card) return;
  const cardWidth = card.offsetWidth + 16;
  carouselTrack.value.scrollLeft = currentIndex.value * cardWidth;
}

function nextSlide() {
  currentIndex.value = (currentIndex.value + 1) % testimonials.length;
  updatePosition();
}

function autoplay() {
  interval = setInterval(() => {
    nextSlide();
  }, 3000);
}

// DRAGGING
let isDragging = false;
let startX = 0;
let scrollLeft = 0;

function startDrag(e: any) {
  isDragging = true;
  startX = e.pageX || e.touches[0].pageX;
  scrollLeft = carouselTrack.value?.scrollLeft || 0;
  clearInterval(interval);
}

function onDrag(e: any) {
  if (!isDragging || !carouselTrack.value) return;
  const x = e.pageX || e.touches[0].pageX;
  const walk = (x - startX) * 1.2;
  carouselTrack.value.scrollLeft = scrollLeft - walk;
}

function endDrag() {
  isDragging = false;
  autoplay();
}

onMounted(() => {
  autoplay();
});

onUnmounted(() => clearInterval(interval));

// ===== FORM ORDER =====

// Produk list (radio tiles)
const products = ref([
  { id: 1, name: 'Habbamax Garlic 200 (Dus)', price: 1750000, image: '/images/200dus.png', qty: 0 },
  { id: 2, name: 'Habbamax Garlic 100 (Dus)', price: 910000, image: '/images/100dus.png', qty: 0 },
  { id: 3, name: 'Habbamax Garlic 200 (Botol)', price: 150000, image: '/images/200botol.png', qty: 0 },
  { id: 4, name: 'Habbamax Garlic 100 (Botol)', price: 80000, image: '/images/100botol.png', qty: 0 },
]);

function increment(p) {
  p.qty++;
}

function decrement(p) {
  if (p.qty > 0) p.qty--;
}

const cartItems = computed(() => products.value.filter(p => p.qty > 0));

const subTotal = computed(() =>
  cartItems.value.reduce((sum, item) => sum + item.price * item.qty, 0)
);

const alamat = ref("");

// SELLER LIST
const sellerSearch = ref("");
const sellers = [
  { id: 1, name: "Rizal", city: "Bandung", phone: "6285950540055", avatar: "/images/s1.png" },
  { id: 2, name: "Sinta", city: "Jakarta", phone: "6285950540055", avatar: "/images/s2.png" },
  { id: 3, name: "Budi", city: "Surabaya", phone: "6285950540055", avatar: "/images/s3.png" },
];

const selectedSeller = ref(null);

const showSellerDropdown = ref(false);

function toggleSellerDropdown() {
  showSellerDropdown.value = !showSellerDropdown.value;
}

function selectSeller(s) {
  selectedSeller.value = s;
  showSellerDropdown.value = false;
  sellerSearch.value = s.name; // tampilkan nama seller di input
}

function clearSeller() {
  sellerSearch.value = "";
  selectedSeller.value = null;
  showSellerDropdown.value = true;
}

// COMPUTED FILTERED SELLER (SEARCHABLE)
const filteredSellers = computed(() => {
  if (!sellerSearch.value) return sellers;
  return sellers.filter(s =>
    s.name.toLowerCase().includes(sellerSearch.value.toLowerCase()) ||
    s.city.toLowerCase().includes(sellerSearch.value.toLowerCase())
  );
});


// SUBMIT KE WHATSAPP
function submitOrder() {
  
  if (cartItems.value.length === 0) {
    toast.error("Keranjang kosong!", {
      description: "Tambahkan produk sebelum checkout.",
    });
    return;
  }

  if (!alamat.value) {
    toast.error("Alamat kosong!", {
      description: "Perjelas alamat agar pengiriman lancar.",
    });
    return;
  }
  
  if (!selectedSeller.value) {
    toast.error("Seller belum diketahui?", {
      description: "Pilih seller terlebih dahulu.",
    });    
    return;
  }

  // FORMAT ITEM LIST (rapi & ter-encode newline)
  let items = cartItems.value.map((item, index) =>
    `${index + 1}. *${item.name}*%0A` +
    `   Rp ${item.price.toLocaleString()} x ${item.qty} = Rp ${(item.price * item.qty).toLocaleString()}`
  ).join("%0A%0A");

  // FORMAT PESAN UTAMA
  let message =
    `*🛒 Pesanan Baru*%0A%0A` +

    `*Seller :* ${selectedSeller.value.name}%0A` +
    `-------------------------%0A` +
    `${items}%0A` +
    `-------------------------%0A%0A` +

    `*Subtotal :* Rp ${subTotal.value.toLocaleString()}%0A%0A` +

    `*Alamat Penerima:* ${alamat.value}`
  ;

  // WHATSAPP URL
  const wa = `https://wa.me/${selectedSeller.value.phone}?text=${message}`;
  window.open(wa, "_blank");
}
</script>

<template>
  <Toaster position="top-right" richColors />
  <!-- HEADER -->
  <header class="fixed top-0 left-0 w-full flex justify-between items-center px-6 py-2 shadow-md 
                 bg-yellow-200 text-black 
                 dark:bg-yellow-600 dark:text-white z-50">
    <h1 class="text-xl font-bold">Herbal Juara</h1>

    <button @click="toggleAppearance" 
            class="p-2 rounded-lg bg-white text-black shadow 
                   dark:bg-black dark:text-yellow-300">
      <Sun v-if="appearance === 'light'" class="size-4" />
      <Moon v-else-if="appearance === 'dark'" class="size-4" />
      <Laptop2 v-else class="size-4" />
    </button>
  </header>

  <main class="pt-12 pb-10 bg-white text-black dark:bg-black dark:text-white">
    <!-- HERO & CTA -->
    <section class="text-center px-6 py-16 bg-yellow-100 dark:bg-yellow-700">
      <h2 class="text-3xl font-bold mb-4">Habbamax – Solusi Herbal untuk Kesehatan Optimal</h2>
      <p class="max-w-xl mx-auto mb-6">
        Formula HABBAMAX dirancang untuk meningkatkan imunitas tubuh, merawat kesehatan jantung dan memperbaiki metabolisme tubuh secara alami.
      </p>
      <a href="#ordernow" class="px-6 py-3 bg-yellow-400 dark:bg-black dark:text-yellow-300 font-semibold rounded-md">
        Beli Sekarang
      </a>
    </section>

    <!-- TENTANG -->
    <section class="px-6 py-16 max-w-4xl mx-auto">
      <h3 class="text-2xl font-bold mb-4">Tentang Habbamax</h3>
      <p class="leading-relaxed">
        Habbamax adalah suplemen herbal premium dari Herbal Juara yang diformulasikan dari bahan-bahan terbaik : 
        habbatussauda, propolis, garlic oil, dan zaitun extra virgin.
      </p>
    </section>

    <!-- KEUNGGULAN PRODUK -->
    <section class="px-6 py-16 max-w-4xl mx-auto bg-yellow-100 dark:bg-yellow-800 rounded-xl">
      <h3 class="text-2xl font-bold mb-6">Keunggulan Produk</h3>
      <ul class="space-y-3 list-decimal pl-6">
          <li>Menyehatkan Jantung</li>
          <li>Melancarkan Peredaran Darah </li>
          <li>Meningkatkan Kualitas Tidur</li>
          <li>Memperbaiki Pencernaan </li>
          <li>Meredakan Maag & Wasir</li>
          <li>Menaikkan Imunitas Tubuh</li>
          <li>Mengurangi Nyeri Haid</li>
          <li>Mengurangi Peradangan </li>
          <li>Terapi Stroke & Diabetes</li>
          <li>Terapi Darah Tinggi, Migrain, Kolesterol & Asam Urat</li>
      </ul>
    </section>

    <!-- TESTIMONI CAROUSEL -->
<section class="py-16 mx-auto">
  <h3 class="text-2xl font-bold mb-6 text-center">Testimoni</h3>

  <div class="relative overflow-hidden">
    <!-- TRACK -->
    <div
      class="flex gap-4 overflow-x-auto scroll-smooth snap-x snap-mandatory pb-4"
      ref="carouselTrack"
      @mousedown="startDrag"
      @mousemove="onDrag"
      @mouseup="endDrag"
      @mouseleave="endDrag"
      @touchstart="startDrag"
      @touchmove="onDrag"
      @touchend="endDrag"
    >
      <div
        v-for="(t, i) in testimonials"
        :key="i"
        class="min-w-[80%] sm:min-w-[48%] md:min-w-[35%] lg:min-w-[28%] xl:min-w-[25%]
               bg-yellow-50 dark:bg-gray-900 shadow rounded-lg p-4 snap-center"
      >
        <p class="italic mb-2">“{{ t.text }}”</p>
        <strong>- {{ t.author }}</strong>
      </div>
    </div>
  </div>
</section>

<!-- CARA PEMESANAN -->
    <section class="px-6 py-16 max-w-4xl mx-auto bg-yellow-100 dark:bg-yellow-700 rounded-xl">
      <h3 class="text-2xl font-bold mb-6">Cara Pemesanan</h3>
      <ol class="space-y-3 list-disc pl-6">
        <li>Kunjungi website resmi di <strong>herbaljuara.com</strong></li>
        <li>Tekan tombol Beli Sekarang</li>
        <li>Pilih produk Habbamax</li>
        <li>Isikan alamat Penerima</li>
        <li>Pilih Seller kepercayaan anda</li>
        <li>Lakukan checkout</li>
        <li>Selesaikan pembayaran dan kirim bukti transfer ke Whatsapp</li>
      </ol>
    </section>

<!-- FORM ORDER -->
<section id="ordernow" class="px-6 py-16 max-w-4xl mx-auto bg-gray-100 dark:bg-gray-800 rounded-xl mt-10">
  <h3 class="text-2xl font-bold mb-6 text-center">Form Order</h3>

<!-- LIST PRODUK DENGAN QTY -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 mb-8">
  <div
    v-for="p in products"
    :key="p.id"
    class="cursor-pointer border rounded-xl p-4 flex flex-col items-center relative
           bg-white dark:bg-gray-900 shadow transition-all"
  >

    <img :src="p.image" class="w-24 h-24 object-contain mb-3" />

    <p class="text-center font-semibold">{{ p.name }}</p>
    <p class="text-green-700 dark:text-green-300 mt-1">
      Rp {{ p.price.toLocaleString() }}
    </p>

    <!-- INCREMENT DECREMENT -->
    <div class="flex items-center gap-3 mt-4">
      <button
        @click="decrement(p)"
        class="w-8 h-8 flex items-center justify-center bg-red-500 text-white rounded-md"
      >−</button>

      <input
        type="number"
        min="0"
        v-model.number="p.qty"
        class="w-14 text-center px-2 py-1 border rounded-md bg-white dark:bg-gray-800
                 appearance-none 
         [&::-webkit-outer-spin-button]:appearance-none 
         [&::-webkit-inner-spin-button]:appearance-none 
         [-moz-appearance:textfield]
        "
      />

      <button
        @click="increment(p)"
        class="w-8 h-8 flex items-center justify-center bg-green-500 text-white rounded-md"
      >+</button>
    </div>

  </div>
</div>


<!-- CART SUMMARY -->
<div v-if="cartItems.length" class="bg-white dark:bg-gray-900 p-4 rounded-xl shadow mb-8">
  <h4 class="text-xl font-bold mb-4">Ringkasan Pesanan</h4>

  <div class="space-y-3">
    <div
      v-for="item in cartItems"
      :key="item.id"
      class="flex justify-between border-b pb-2"
    >
      <div>
        <p class="font-semibold">{{ item.name }}</p>
        <p class="text-sm text-gray-500">Rp{{ item.price.toLocaleString() }}</p>
      </div>

      <div class="text-right">
        <p>Qty: {{ item.qty }}</p>
        <p class="font-bold">Rp{{ (item.qty * item.price).toLocaleString() }}</p>
      </div>
    </div>
  </div>

  <!-- TOTAL -->
  <div class="flex justify-between mt-4 text-lg font-bold">
    <span>Total</span>
    <span>Rp {{ subTotal.toLocaleString() }}</span>
  </div>
</div>


  <!-- ALAMAT -->
  <div class="mb-6">
    <label class="font-semibold">Alamat Tujuan</label>
    <textarea v-model="alamat" rows="4" placeholder="Tulis dengan lengkap! Provinsi, Kabupaten / Kota, Kecamatan, Desa, Jalan, Gang, RT RW, Kode Pos."
              class="w-full mt-2 px-4 py-2 rounded-lg bg-white dark:bg-gray-900 border"></textarea>
  </div>

<!-- SELLER SELECT -->
<div class="mb-6 relative">
  <label class="font-semibold">Pilih Seller</label>

  <!-- SEARCH INPUT WRAPPER -->
  <div class="relative mt-2">

    <!-- INPUT -->
    <input
      v-model="sellerSearch"
      @click="toggleSellerDropdown"
      @input="showSellerDropdown = true"
      placeholder="Cari seller..."
      class="w-full px-4 py-2 rounded-lg bg-white dark:bg-gray-900 border pr-10"
    />

    <!-- CLEAR BUTTON (X) -->
    <button
      v-if="sellerSearch"
      @click="clearSeller"
      class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-500 hover:text-black dark:hover:text-white"
    >
      ✕
    </button>
  </div>

  <!-- DROPDOWN -->
  <div
    v-if="showSellerDropdown"
    class="absolute left-0 w-full bg-white dark:bg-gray-900 border rounded-lg mt-1 max-h-60 overflow-y-auto z-50 shadow-lg"
  >
    <div
      v-for="s in filteredSellers"
      :key="s.id"
      @click="selectSeller(s)"
      class="p-3 flex items-center gap-3 cursor-pointer hover:bg-gray-200 dark:hover:bg-gray-700"
    >
      <img :src="s.avatar" class="w-10 h-10 rounded-full" />
      <div>
        <p class="font-semibold">{{ s.name }}</p>
        <p class="text-sm text-gray-500 dark:text-gray-400">{{ s.city }}</p>
      </div>

      <span
        v-if="selectedSeller && selectedSeller.id === s.id"
        class="ml-auto text-green-500 font-bold"
      >
        ✔
      </span>
    </div>

    <!-- Jika tidak ada hasil -->
    <p
      v-if="filteredSellers.length === 0"
      class="p-3 text-center text-gray-500 dark:text-gray-400"
    >
      Tidak ditemukan
    </p>
  </div>
</div>

  <!-- SUBMIT -->
  <button
    @click="submitOrder"
    class="w-full py-3 bg-green-600 hover:bg-green-700 text-white font-bold rounded-lg"
  >
    Pesan via WhatsApp
  </button>
</section>
  </main>

  <!-- FOOTER -->
  <footer class="text-center py-6 bg-yellow-200 text-black dark:bg-yellow-600 dark:text-white">
    <p>© {{ new Date().getFullYear() }} Herbal Juara – herbaljuara.com</p>
  </footer>
</template>
