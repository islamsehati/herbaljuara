<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { useAppearance } from '@/composables/useAppearance';
import { Sun, Moon, SpellCheck } from 'lucide-vue-next';
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

const ingredients = [
  { icon: '🌱', name: 'Habbatussauda', sub: 'Jintan hitam murni' },
  { icon: '🍯', name: 'Propolis', sub: 'Lebah pilihan' },
  { icon: '🧄', name: 'Garlic Oil', sub: 'Bawang putih cold-pressed' },
  { icon: '🫒', name: 'Zaitun Extra Virgin', sub: 'First press premium' },
];
 
const benefits = [
  'Menyehatkan Jantung',
  'Melancarkan Peredaran Darah',
  'Meningkatkan Kualitas Tidur',
  'Memperbaiki Pencernaan',
  'Meredakan Maag & Wasir',
  'Menaikkan Imunitas Tubuh',
  'Mengurangi Nyeri Haid',
  'Mengurangi Peradangan',
  'Terapi Stroke & Diabetes',
  'Terapi Darah Tinggi, Migrain, Kolesterol & Asam Urat',
];
 
const steps = [
  { title: 'Kunjungi website resmi', desc: 'Buka herbaljuara.com di browser Anda' },
  { title: 'Tekan tombol Beli Sekarang', desc: 'Gulir ke bagian Form Order di halaman ini' },
  { title: 'Pilih produk Habbamax', desc: 'Tentukan varian dan jumlah pesanan Anda' },
  { title: 'Isikan alamat Penerima', desc: 'Lengkapi dengan Provinsi, Kota, Kecamatan, dan kode pos' },
  { title: 'Pilih Seller kepercayaan Anda', desc: 'Cari seller terdekat atau yang sudah Anda kenal' },
  { title: 'Lakukan checkout', desc: 'Klik Pesan via WhatsApp untuk menghubungi seller' },
  { title: 'Selesaikan pembayaran', desc: 'Kirim bukti transfer ke WhatsApp seller Anda' },
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
  { id: 1, name: 'Habbamax Garlic 200 (Botol)', price: 200000, image: '/Foto_Produk_001.png', qty: 0 },
  // { id: 2, name: 'Habbamax Garlic 100 (Botol)', price: 150000, image: '/Foto_Produk_001.png', qty: 0 },
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
  { id: 1, name: "Agus Imaduddin", city: "Jawa Tengah", phone: "6281228008464", avatar: "/avatar.png" },
  // { id: 2, name: "Baruno", city: "Jakarta", phone: "6282134780459", avatar: "/avatar.png" },
  // { id: 3, name: "Yuda", city: "Bali", phone: "6285950540055", avatar: "/avatar.png" },
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
  <header class="fixed top-0 left-0 w-full z-50 flex justify-between items-center px-8 h-14
                 bg-[#3D2C1E] text-[#FAF0DC]">
    <span class="font-serif text-xl tracking-wide" style="font-family: 'Playfair Display', Georgia, serif;">
      Herbal Juara
    </span>
    <button @click="toggleAppearance"
            class="w-9 h-9 rounded-full flex items-center justify-center
                   border border-white/20 bg-white/10 hover:bg-white/20 transition-colors">
      <Sun v-if="appearance === 'light'" class="size-4 text-yellow-200" />
      <Moon v-else-if="appearance === 'dark'" class="size-4 text-yellow-300" />
      <SpellCheck v-else class="size-4 text-yellow-200" />
    </button>
  </header>
 
  <main class="pt-14 bg-[#FAF6EF] text-[#2A1F14] dark:bg-[#1A1208] dark:text-[#F0E8D8]"
        style="font-family: 'DM Sans', 'Segoe UI', sans-serif;">
 
    <!-- ── HERO ─────────────────────────────────────────── -->
    <section class="relative overflow-hidden bg-[#3D2C1E] dark:bg-[#1A1208]
                    px-6 pt-20 pb-28 text-center">
 
      <!-- Subtle glow -->
      <div class="absolute inset-0 pointer-events-none"
           style="background: radial-gradient(ellipse 70% 50% at 50% 100%, rgba(200,149,42,.18) 0%, transparent 70%);">
      </div>
 
      <!-- Eyebrow -->
      <span class="inline-block text-[11px] font-medium tracking-[0.16em] uppercase
                   text-[#C8952A] border border-[#7A5A10] rounded-full px-4 py-1 mb-6">
        Suplemen Herbal Premium
      </span>
 
      <h2 class="text-[#F5E6C8] font-bold leading-[1.2] mb-5
                 text-3xl sm:text-4xl lg:text-5xl max-w-2xl mx-auto"
          style="font-family: 'Playfair Display', Georgia, serif;">
        Habbamax – Solusi Herbal untuk
        <em class="text-[#C8952A] not-italic"> Kesehatan Optimal</em>
      </h2>
 
      <p class="text-[#C4B49A] text-[15px] leading-relaxed max-w-md mx-auto mb-9">
        Formula HABBAMAX dirancang untuk meningkatkan imunitas tubuh,
        merawat kesehatan jantung dan memperbaiki metabolisme tubuh secara alami.
      </p>
 
      <a href="#ordernow"
         class="inline-flex items-center gap-2.5
                bg-[#C8952A] hover:bg-[#DCA82E] text-[#3D2C1E]
                font-medium text-[15px] px-8 py-3.5 rounded-[4px]
                transition-all duration-200 hover:-translate-y-px">
        Beli Sekarang
        <span class="text-[18px] leading-none transition-transform duration-200 group-hover:translate-x-1">→</span>
      </a>
    </section>
 
    <!-- ── TENTANG ─────────────────────────────────────── -->
    <section class="bg-[#FAF6EF] dark:bg-[#1A1208]">
      <div class="max-w-5xl mx-auto px-6 py-20
                  grid grid-cols-1 md:grid-cols-2 gap-12 items-start">
 
        <!-- Text -->
        <div>
          <p class="text-[11px] font-medium tracking-[0.16em] uppercase
                     text-[#7A5A10] dark:text-[#C8952A] mb-3">
            Tentang Produk
          </p>
          <h3 class="text-[32px] leading-[1.25] font-bold text-[#3D2C1E] dark:text-[#F5E6C8] mb-4"
              style="font-family: 'Playfair Display', Georgia, serif;">
            Habbamax oleh Herbal Juara
          </h3>
          <p class="text-[15px] text-[#7A6A55] dark:text-[#B09880] leading-relaxed">
            Habbamax adalah suplemen herbal premium yang diformulasikan dari
            bahan-bahan terbaik pilihan alam. Diproduksi dengan standar kualitas
            tinggi untuk menjaga kesehatan Anda secara menyeluruh.
          </p>
        </div>
 
        <!-- Ingredient cards -->
        <div class="grid grid-cols-2 gap-3">
          <div v-for="ing in ingredients" :key="ing.name"
               class="bg-white dark:bg-[#2A1E10] border border-[#E8DCCC] dark:border-[#4A3520]
                      rounded-xl p-4 text-center
                      hover:border-[#C8952A] hover:-translate-y-0.5
                      transition-all duration-200">
            <span class="text-3xl mb-2 block">{{ ing.icon }}</span>
            <div class="text-[13px] font-medium text-[#3D2C1E] dark:text-[#F5E6C8]">{{ ing.name }}</div>
            <div class="text-[11px] text-[#7A6A55] dark:text-[#9A8A6A] mt-0.5">{{ ing.sub }}</div>
          </div>
        </div>
      </div>
    </section>
 
    <!-- ── KEUNGGULAN ─────────────────────────────────── -->
    <section class="bg-[#4A6741] dark:bg-[#2A3C25] relative overflow-hidden py-20 px-6">
      <!-- Decorative circle -->
      <div class="absolute -top-10 -right-10 w-72 h-72 rounded-full
                  bg-white/[0.03] pointer-events-none"></div>
 
      <div class="max-w-5xl mx-auto relative">
        <p class="text-[11px] font-medium tracking-[0.16em] uppercase text-[#A8D89A] mb-3">
          Manfaat
        </p>
        <h3 class="text-[32px] font-bold text-[#F0F8EE] mb-10"
            style="font-family: 'Playfair Display', Georgia, serif;">
          Keunggulan Produk
        </h3>
 
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3">
          <div v-for="(b, i) in benefits" :key="i"
               class="flex items-start gap-3
                      bg-white/[0.07] hover:bg-white/[0.12] border border-white/[0.12]
                      rounded-xl px-4 py-4 transition-colors duration-200">
            <span class="w-7 h-7 flex-shrink-0 rounded-full
                         bg-[#C8952A] text-[#3D2C1E]
                         text-[12px] font-bold flex items-center justify-center">
              {{ i + 1 }}
            </span>
            <span class="text-[13px] text-[#D8EDD4] leading-relaxed">{{ b }}</span>
          </div>
        </div>
      </div>
    </section>
 
    <!-- ── TESTIMONI ──────────────────────────────────── -->
    <section class="py-20 bg-[#FAF6EF] dark:bg-[#1A1208]">
      <div class="text-center px-6 mb-10">
        <p class="text-[11px] font-medium tracking-[0.16em] uppercase
                   text-[#7A5A10] dark:text-[#C8952A] mb-2">
          Dari Pelanggan Kami
        </p>
        <h3 class="text-[32px] font-bold text-[#3D2C1E] dark:text-[#F5E6C8]"
            style="font-family: 'Playfair Display', Georgia, serif;">
          Testimoni
        </h3>
        <p class="text-[14px] text-[#7A6A55] dark:text-[#9A8A6A] mt-2">
          Dipercaya ribuan pelanggan di seluruh Indonesia
        </p>
      </div>
 
      <!-- Carousel track -->
      <div class="flex gap-4 overflow-x-auto scroll-smooth snap-x snap-mandatory pb-4 px-6
                  [&::-webkit-scrollbar]:hidden [-ms-overflow-style:none] [scrollbar-width:none]"
           ref="carouselTrack"
           @mousedown="startDrag" @mousemove="onDrag" @mouseup="endDrag" @mouseleave="endDrag"
           @touchstart="startDrag" @touchmove="onDrag" @touchend="endDrag">
 
        <div v-for="(t, i) in testimonials" :key="i"
             class="min-w-[280px] max-w-[300px] flex-shrink-0 snap-start
                    bg-white dark:bg-[#2A1E10]
                    border border-[#E8DCCC] dark:border-[#4A3520]
                    rounded-xl p-6
                    hover:-translate-y-1 hover:shadow-lg hover:shadow-[#3D2C1E]/10
                    transition-all duration-200">
 
          <!-- Big decorative quote mark -->
          <div class="text-[42px] leading-none text-[#F5E6C8] dark:text-[#4A3520] mb-1 select-none"
               style="font-family: 'Playfair Display', Georgia, serif;">"</div>
 
          <p class="text-[14px] text-[#7A6A55] dark:text-[#B09880] leading-[1.7] italic">
            {{ t.text }}
          </p>
 
          <div class="flex items-center gap-2.5 mt-4">
            <div class="w-8 h-8 rounded-full bg-[#F5E6C8] dark:bg-[#4A3520]
                        flex items-center justify-center
                        text-[13px] font-semibold text-[#7A5A10] dark:text-[#C8952A]">
              {{ t.author[0] }}
            </div>
            <span class="text-[13px] font-medium text-[#3D2C1E] dark:text-[#F5E6C8]">
              {{ t.author }}
            </span>
          </div>
        </div>
      </div>
    </section>
 
    <!-- ── CARA PEMESANAN ─────────────────────────────── -->
    <section class="bg-[#3D2C1E] dark:bg-[#0F0A05] py-20 px-6">
      <div class="max-w-3xl mx-auto">
        <p class="text-[11px] font-medium tracking-[0.16em] uppercase text-[#C8952A] mb-3">
          Panduan
        </p>
        <h3 class="text-[32px] font-bold text-[#F5E6C8] mb-10"
            style="font-family: 'Playfair Display', Georgia, serif;">
          Cara Pemesanan
        </h3>
 
        <div class="flex flex-col">
          <div v-for="(step, i) in steps" :key="i"
               class="flex gap-5 items-start py-5
                      border-b border-white/[0.07] last:border-b-0">
            <span class="text-[40px] font-bold text-[#C8952A]/25 leading-none
                         min-w-[52px] select-none"
                  style="font-family: 'Playfair Display', Georgia, serif;">
              {{ String(i + 1).padStart(2, '0') }}
            </span>
            <div class="pt-1.5">
              <div class="text-[15px] font-medium text-[#F5E6C8] mb-0.5">{{ step.title }}</div>
              <div class="text-[13px] text-[#9A8C7A] leading-relaxed">{{ step.desc }}</div>
            </div>
          </div>
        </div>
      </div>
    </section>
 
    <!-- ── FORM ORDER (tidak diubah) ─────────────────── -->
    <section id="ordernow" class="px-6 py-16 max-w-4xl mx-auto bg-gray-100 dark:bg-gray-800 rounded-xl mt-10">
      <h3 class="text-2xl font-bold mb-6 text-center">Form Order</h3>
 
      <!-- LIST PRODUK DENGAN QTY -->
      <div class="grid grid-cols-2 lg:grid-cols-3 gap-4 mb-8">
        <div v-for="p in products" :key="p.id"
             class="cursor-pointer border rounded-xl overflow-hidden
                    flex flex-col relative
                    bg-white dark:bg-gray-900 shadow transition-all">
 
          <img :src="p.image" class="w-full h-40 object-cover rounded-t-xl" />
 
          <div class="p-4 grid grid-cols-1 items-center mb-8">
            <p class="font-semibold text-sm">{{ p.name }}</p>
            <p class="text-green-700 dark:text-green-300 mt-1 flex justify-left">
              Rp {{ p.price.toLocaleString() }}
            </p>
          </div>
 
          <!-- MINI COUNTER -->
          <div class="absolute bottom-3 right-1/2 transform translate-x-1/2
                      flex items-center bg-white dark:bg-gray-800
                      shadow-lg rounded-full overflow-hidden border">
            <button @click="decrement(p)"
                    class="w-8 h-8 flex items-center justify-center text-red-500 font-bold hover:bg-red-50 dark:hover:bg-gray-700">
              −
            </button>
            <input type="number" min="0" v-model.number="p.qty"
                   class="w-10 h-8 text-center text-sm border-x bg-transparent
                          focus:outline-none appearance-none
                          [&::-webkit-outer-spin-button]:appearance-none
                          [&::-webkit-inner-spin-button]:appearance-none
                          [-moz-appearance:textfield]" />
            <button @click="increment(p)"
                    class="w-8 h-8 flex items-center justify-center text-green-500 font-bold hover:bg-green-50 dark:hover:bg-gray-700">
              +
            </button>
          </div>
        </div>
      </div>
 
      <!-- CART SUMMARY -->
      <div v-if="cartItems.length" class="bg-white dark:bg-gray-900 p-4 rounded-xl shadow mb-8">
        <h4 class="text-xl font-bold mb-4">Ringkasan Pesanan</h4>
        <div class="space-y-3">
          <div v-for="item in cartItems" :key="item.id" class="flex justify-between border-b pb-2">
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
        <div class="flex justify-between mt-4 text-lg font-bold">
          <span>Total</span>
          <span>Rp {{ subTotal.toLocaleString() }}</span>
        </div>
      </div>
 
      <!-- ALAMAT -->
      <div class="mb-6">
        <label class="font-semibold">Alamat Tujuan</label>
        <textarea v-model="alamat" rows="4"
                  placeholder="Tulis dengan lengkap! Provinsi, Kabupaten / Kota, Kecamatan, Desa, Jalan, Gang, RT RW, Kode Pos."
                  class="w-full mt-2 px-4 py-2 rounded-lg bg-white dark:bg-gray-900 border">
        </textarea>
      </div>
 
      <!-- SELLER SELECT -->
      <div class="mb-6 relative">
        <label class="font-semibold">Pilih Seller</label>
        <div class="relative mt-2">
          <input v-model="sellerSearch"
                 @click="toggleSellerDropdown"
                 @input="showSellerDropdown = true"
                 placeholder="Cari seller..."
                 class="w-full px-4 py-2 rounded-lg bg-white dark:bg-gray-900 border pr-10" />
          <button v-if="sellerSearch" @click="clearSeller"
                  class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-500 hover:text-black dark:hover:text-white">
            ✕
          </button>
        </div>
 
        <div v-if="showSellerDropdown"
             class="absolute left-0 w-full bg-white dark:bg-gray-900 border rounded-lg mt-1 max-h-60 overflow-y-auto z-50 shadow-lg">
          <div v-for="s in filteredSellers" :key="s.id"
               @click="selectSeller(s)"
               class="p-3 flex items-center gap-3 cursor-pointer hover:bg-gray-200 dark:hover:bg-gray-700">
            <img :src="s.avatar" class="w-10 h-10 rounded-full" />
            <div>
              <p class="font-semibold">{{ s.name }}</p>
              <p class="text-sm text-gray-500 dark:text-gray-400">{{ s.city }}</p>
            </div>
            <span v-if="selectedSeller && selectedSeller.id === s.id"
                  class="ml-auto text-green-500 font-bold">✔</span>
          </div>
          <p v-if="filteredSellers.length === 0" class="p-3 text-center text-gray-500 dark:text-gray-400">
            Tidak ditemukan
          </p>
        </div>
      </div>
 
      <!-- SUBMIT -->
      <button @click="submitOrder"
              class="w-full py-3 bg-green-600 hover:bg-green-700 text-white font-bold rounded-lg">
        Pesan via WhatsApp
      </button>
    </section>
  </main>
 
  <!-- FOOTER -->
  <footer class="text-center py-7 bg-[#1E140A] text-[#7A6A55] text-[13px] tracking-wide">
    <p>© {{ new Date().getFullYear() }} <span class="text-[#C8952A]">Herbal Juara</span> — herbaljuara.com</p>
  </footer>
</template>
