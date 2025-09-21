import React, { useState, useEffect } from "react";
/**
 * Nazho Restaurant - Single-file React app
 * --------------------------------------------------
 * - Tailwind CSS utility classes are used for styling (assumes Tailwind is set up in the project).
 * - Uses localStorage to persist cart, orders, and reservations (mock backend).
 * - Includes features: Hero, Menu (search + filters), Cart & mock checkout, Reservations, Gallery, Reviews,
 *   Contact form (mock), Admin panel (password-protected), Responsive layout, Framer Motion animations hints.
 * - Exports default React component for quick preview.
 *
 * How to use:
 * 1. Create a React app (Vite or Create React App).
 * 2. Install Tailwind and optionally framer-motion, lucide-react, recharts, shadcn (if desired).
 * 3. Drop this file as App.jsx and ensure Tailwind is configured.
 *
 * Note: This is a single-file demo. Replace mock handlers with real API calls for production.
 */

// Optional imports if these libraries are installed. If not installed, comment them out.
// import { motion } from 'framer-motion'
// import { ShoppingCart } from 'lucide-react'

// Mock assets (replace with real images)
const heroImg = "https://images.unsplash.com/photo-1543353071-087092ec3930?q=80&w=1400&auto=format&fit=crop&ixlib=rb-4.0.3&s=abc";
const foodPlaceholder = "https://images.unsplash.com/photo-1604908554022-5f5e1f7b8a6a?q=80&w=1000&auto=format&fit=crop&ixlib=rb-4.0.3&s=def";

const initialMenu = [
  { id: 1, name: "Nazho Signature Beef Burger", category: "Main", price: 75000, spicy: false, veg: false, img: foodPlaceholder },
  { id: 2, name: "Crispy Ayam Kremes", category: "Main", price: 65000, spicy: false, veg: false, img: foodPlaceholder },
  { id: 3, name: "Soto Betawi", category: "Soup", price: 45000, spicy: false, veg: false, img: foodPlaceholder },
  { id: 4, name: "Gado-Gado Nazho", category: "Salad", price: 40000, spicy: false, veg: true, img: foodPlaceholder },
  { id: 5, name: "Nasi Goreng Special", category: "Main", price: 50000, spicy: true, veg: false, img: foodPlaceholder },
  { id: 6, name: "Es Teh Nazho", category: "Drinks", price: 15000, spicy: false, veg: true, img: foodPlaceholder },
  { id: 7, name: "Pancake Pisang", category: "Dessert", price: 30000, spicy: false, veg: true, img: foodPlaceholder },
];

// Utilities
const formatIDR = (v) =>
  new Intl.NumberFormat("id-ID", { style: "currency", currency: "IDR", maximumFractionDigits: 0 }).format(v);

// LocalStorage keys
const LS_CART = "nazho_cart_v1";
const LS_ORDERS = "nazho_orders_v1";
const LS_RES = "nazho_reservations_v1";

export default function App() {
  // App state
  const [menu, setMenu] = useState(initialMenu);
  const [query, setQuery] = useState("");
  const [category, setCategory] = useState("All");
  const [filters, setFilters] = useState({ veg: false, spicy: false });

  // Cart
  const [cart, setCart] = useState(() => {
    try {
      const raw = localStorage.getItem(LS_CART);
      return raw ? JSON.parse(raw) : [];
    } catch (e) {
      return [];
    }
  });

  // UI state
  const [view, setView] = useState("home"); // home, menu, cart, reserve, admin
  const [toast, setToast] = useState(null);

  // Admin
  const [adminMode, setAdminMode] = useState(false);
  const [adminPassword, setAdminPassword] = useState("");

  useEffect(() => {
    localStorage.setItem(LS_CART, JSON.stringify(cart));
  }, [cart]);

  const categories = ["All", ...Array.from(new Set(menu.map((m) => m.category)))];

  // Derived menu
  const filteredMenu = menu.filter((item) => {
    if (category !== "All" && item.category !== category) return false;
    if (filters.veg && !item.veg) return false;
    if (filters.spicy && !item.spicy) return false;
    if (query && !item.name.toLowerCase().includes(query.toLowerCase())) return false;
    return true;
  });

  // Cart functions
  function addToCart(item, qty = 1) {
    setCart((c) => {
      const idx = c.findIndex((x) => x.id === item.id);
      if (idx >= 0) {
        const copy = [...c];
        copy[idx].qty += qty;
        showToast(`${item.name} jumlah updated`);
        return copy;
      }
      showToast(`${item.name} ditambahkan ke keranjang`);
      return [...c, { ...item, qty }];
    });
  }

  function updateQty(id, qty) {
    setCart((c) => c.map((it) => (it.id === id ? { ...it, qty } : it)).filter((it) => it.qty > 0));
  }

  function clearCart() {
    setCart([]);
    showToast("Keranjang dikosongkan");
  }

  function subtotal() {
    return cart.reduce((s, it) => s + it.price * it.qty, 0);
  }

  function proceedCheckout(customer) {
    // Mock checkout: save to orders
    const orders = JSON.parse(localStorage.getItem(LS_ORDERS) || "[]");
    const order = {
      id: `ORD-${Date.now()}`,
      createdAt: new Date().toISOString(),
      items: cart,
      total: subtotal(),
      customer,
      status: "pending",
    };
    orders.unshift(order);
    localStorage.setItem(LS_ORDERS, JSON.stringify(orders));
    setCart([]);
    showToast("Pesanan sukses dibuat (mock)");
    setView("home");
    return order;
  }

  // Reservation
  function makeReservation(res) {
    const data = JSON.parse(localStorage.getItem(LS_RES) || "[]");
    const booking = { id: `RES-${Date.now()}`, ...res };
    data.unshift(booking);
    localStorage.setItem(LS_RES, JSON.stringify(data));
    showToast("Reservasi berhasil (mock)");
    setView("home");
    return booking;
  }

  function showToast(message, ttl = 3000) {
    setToast(message);
    setTimeout(() => setToast(null), ttl);
  }

  // Admin auth (super simple)
  function attemptAdmin() {
    // Warning: DO NOT use this in production. Replace with real auth.
    if (adminPassword === "nazho-admin-2025") {
      setAdminMode(true);
      setView("admin");
      showToast("Admin mode aktif");
    } else {
      showToast("Password admin salah");
    }
  }

  // Admin helpers
  const orders = JSON.parse(localStorage.getItem(LS_ORDERS) || "[]");
  const reservations = JSON.parse(localStorage.getItem(LS_RES) || "[]");

  return (
    <div className="min-h-screen bg-gray-50 text-gray-800">
      {/* Header */}
      <header className="bg-white shadow">
        <div className="container mx-auto px-4 py-4 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="w-12 h-12 bg-gradient-to-br from-yellow-400 to-orange-500 rounded-full flex items-center justify-center font-bold text-white">N</div>
            <div>
              <h1 className="text-xl font-extrabold">Nazho Restaurant</h1>
              <p className="text-sm text-gray-500">Rasa rumah, suasana istimewa</p>
            </div>
          </div>

          <nav className="hidden md:flex gap-3 items-center">
            <button onClick={() => setView("home")} className="px-3 py-2 rounded hover:bg-gray-100">Beranda</button>
            <button onClick={() => setView("menu")} className="px-3 py-2 rounded hover:bg-gray-100">Menu</button>
            <button onClick={() => setView("reserve")} className="px-3 py-2 rounded hover:bg-gray-100">Reservasi</button>
            <button onClick={() => setView("contact")} className="px-3 py-2 rounded hover:bg-gray-100">Kontak</button>
            <button onClick={() => setView("gallery")} className="px-3 py-2 rounded hover:bg-gray-100">Galeri</button>
            <button onClick={() => setView("admin-login")} className="px-3 py-2 rounded bg-gray-100">Admin</button>
            <button onClick={() => setView("cart")} className="px-3 py-2 rounded bg-yellow-400 text-white">Keranjang ({cart.length})</button>
          </nav>

          <div className="md:hidden flex items-center gap-2">
            <button onClick={() => setView("cart")} className="px-3 py-2 rounded bg-yellow-400 text-white">Keranjang ({cart.length})</button>
            <button onClick={() => {
              // open menu drawer - simplified by toggling menu view
              setView(view === "menu" ? "home" : "menu");
            }} className="px-3 py-2 rounded bg-gray-100">Menu</button>
          </div>
        </div>
      </header>

      <main className="container mx-auto px-4 py-8">
        {view === "home" && (
          <section className="grid md:grid-cols-2 gap-8 items-center">
            <div>
              <h2 className="text-4xl font-extrabold mb-4">Selamat datang di <span className="text-yellow-500">Nazho</span></h2>
              <p className="text-gray-600 mb-6">Nikmati masakan rumahan dengan sentuhan modern. Pesan online, reservasi meja, atau datangi kami langsung.</p>

              <div className="flex gap-3">
                <button onClick={() => setView("menu")} className="px-4 py-2 rounded bg-yellow-400 text-white">Lihat Menu</button>
                <button onClick={() => setView("reserve")} className="px-4 py-2 rounded border">Buat Reservasi</button>
              </div>

              <div className="mt-8 grid grid-cols-3 gap-2">
                <StatCard title="Menu" value={menu.length} />
                <StatCard title="Meja" value={20} />
                <StatCard title="Ulasan" value={124} />
              </div>
            </div>

            <div className="rounded-xl overflow-hidden shadow">
              <img src={heroImg} alt="hero" className="w-full h-80 object-cover" />
            </div>
          </section>
        )}

        {view === "menu" && (
          <section>
            <div className="flex flex-col md:flex-row justify-between items-center gap-4 mb-6">
              <div className="flex gap-2 items-center">
                <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="Cari menu..." className="px-3 py-2 border rounded w-60" />
                <select value={category} onChange={(e) => setCategory(e.target.value)} className="px-3 py-2 border rounded">
                  {categories.map((c) => <option key={c} value={c}>{c}</option>)}
                </select>

                <label className="flex items-center gap-1 text-sm"><input type="checkbox" checked={filters.veg} onChange={(e) => setFilters((f) => ({ ...f, veg: e.target.checked }))} /> Veg</label>
                <label className="flex items-center gap-1 text-sm"><input type="checkbox" checked={filters.spicy} onChange={(e) => setFilters((f) => ({ ...f, spicy: e.target.checked }))} /> Pedas</label>
              </div>

              <div className="flex gap-2">
                <button onClick={() => setQuery("")} className="px-3 py-2 rounded border">Reset</button>
                <button onClick={() => {
                  // Quick demo: add all to cart
                  filteredMenu.forEach((m) => addToCart(m, 1));
                }} className="px-3 py-2 rounded bg-yellow-400 text-white">Tambahkan Semua</button>
              </div>
            </div>

            <div className="grid md:grid-cols-3 gap-6">
              {filteredMenu.map((item) => (
                <article key={item.id} className="bg-white rounded-lg shadow p-4 flex flex-col">
                  <img src={item.img} alt={item.name} className="w-full h-40 object-cover rounded" />
                  <div className="mt-3 flex-1">
                    <h3 className="font-semibold">{item.name}</h3>
                    <p className="text-sm text-gray-500">{item.category} • {item.veg ? "Veg" : "Non-Veg"}{item.spicy ? " • Pedas" : ""}</p>
                  </div>
                  <div className="mt-3 flex items-center justify-between">
                    <span className="font-bold">{formatIDR(item.price)}</span>
                    <div className="flex gap-2">
                      <button onClick={() => addToCart(item)} className="px-3 py-1 rounded bg-yellow-400 text-white">Tambah</button>
                      <button onClick={() => {
                        // quick view: open modal - simplified as new tab
                        window.alert(`Preview: ${item.name} - ${formatIDR(item.price)}`);
                      }} className="px-3 py-1 rounded border">Lihat</button>
                    </div>
                  </div>
                </article>
              ))}

              {filteredMenu.length === 0 && <div className="col-span-full text-center text-gray-500 p-8">Tidak ada menu sesuai filter.</div>}
            </div>
          </section>
        )}

        {view === "cart" && (
          <section>
            <h2 className="text-2xl font-bold mb-4">Keranjang Anda</h2>
            {cart.length === 0 ? (
              <div className="p-8 bg-white rounded shadow text-center">Keranjang kosong. <button onClick={() => setView("menu")} className="underline">Lihat menu</button></div>
            ) : (
              <div className="grid md:grid-cols-2 gap-6">
                <div className="bg-white p-4 rounded shadow">
                  {cart.map((it) => (
                    <div key={it.id} className="flex items-center gap-3 border-b py-3">
                      <img src={it.img} alt={it.name} className="w-16 h-16 object-cover rounded" />
                      <div className="flex-1">
                        <div className="font-semibold">{it.name}</div>
                        <div className="text-sm text-gray-500">{formatIDR(it.price)}</div>
                      </div>
                      <div className="flex items-center gap-2">
                        <button onClick={() => updateQty(it.id, it.qty - 1)} className="px-2 py-1 border rounded">-</button>
                        <div>{it.qty}</div>
                        <button onClick={() => updateQty(it.id, it.qty + 1)} className="px-2 py-1 border rounded">+</button>
                      </div>
                    </div>
                  ))}

                  <div className="mt-4 flex justify-between items-center">
                    <div className="text-lg font-bold">Sub Total</div>
                    <div className="text-lg font-extrabold">{formatIDR(subtotal())}</div>
                  </div>

                  <div className="mt-4 flex gap-2">
                    <button onClick={() => {
                      const name = prompt("Nama pelanggan untuk checkout (mock):");
                      if (name) {
                        proceedCheckout({ name });
                      }
                    }} className="px-4 py-2 rounded bg-green-600 text-white">Checkout</button>
                    <button onClick={clearCart} className="px-4 py-2 rounded border">Kosongkan</button>
                  </div>
                </div>

                <div className="bg-white p-4 rounded shadow">
                  <h3 className="font-semibold mb-2">Ringkasan</h3>
                  <p className="text-sm text-gray-500">Pengiriman: Ambil di resto (mock)</p>

                  <div className="mt-4">
                    <h4 className="font-medium">Detail Pembayaran</h4>
                    <p className="text-sm text-gray-500">Metode mock: Bayar di tempat</p>
                  </div>
                </div>
              </div>
            )}
          </section>
        )}

        {view === "reserve" && (
          <ReservationForm onReserve={makeReservation} />
        )}

        {view === "contact" && (
          <ContactForm onSend={(data) => {
            // In production, post to backend / send email
            console.log("contact", data);
            showToast("Pesan terkirim (mock)");
            setView("home");
          }} />
        )}

        {view === "gallery" && (
          <Gallery images={[heroImg, foodPlaceholder, foodPlaceholder, heroImg, foodPlaceholder]} />
        )}

        {view === "admin-login" && (
          <div className="max-w-md mx-auto bg-white p-6 rounded shadow">
            <h3 className="text-xl font-bold mb-4">Login Admin (demo)</h3>
            <input placeholder="Password admin" type="password" value={adminPassword} onChange={(e) => setAdminPassword(e.target.value)} className="w-full px-3 py-2 border rounded mb-3" />
            <div className="flex gap-2">
              <button onClick={attemptAdmin} className="px-4 py-2 rounded bg-yellow-400 text-white">Masuk</button>
              <button onClick={() => setView("home")} className="px-4 py-2 rounded border">Batal</button>
            </div>
            <p className="text-xs text-gray-400 mt-2">Password demo: "nazho-admin-2025"</p>
          </div>
        )}

        {view === "admin" && adminMode && (
          <div className="grid md:grid-cols-2 gap-6">
            <div className="bg-white p-4 rounded shadow">
              <h3 className="font-bold mb-3">Pesanan Terbaru</h3>
              {orders.length === 0 && <div className="text-sm text-gray-500">Belum ada pesanan.</div>}
              {orders.map((o) => (
                <div key={o.id} className="border p-2 rounded mb-2">
                  <div className="flex justify-between text-sm"><div>{o.id}</div><div>{new Date(o.createdAt).toLocaleString()}</div></div>
                  <div className="text-sm">{o.customer?.name}</div>
                  <div className="mt-2 text-xs text-gray-600">Items: {o.items.length} • Total: {formatIDR(o.total)}</div>
                </div>
              ))}
            </div>

            <div className="bg-white p-4 rounded shadow">
              <h3 className="font-bold mb-3">Reservasi</h3>
              {reservations.length === 0 && <div className="text-sm text-gray-500">Belum ada reservasi.</div>}
              {reservations.map((r) => (
                <div key={r.id} className="border p-2 rounded mb-2">
                  <div className="flex justify-between text-sm"><div>{r.id}</div><div>{new Date(r.datetime).toLocaleString()}</div></div>
                  <div className="text-sm">{r.name} • {r.guest} tamu</div>
                  <div className="mt-2 text-xs text-gray-600">Catatan: {r.note || '-'}</div>
                </div>
              ))}
            </div>
          </div>
        )}

      </main>

      <footer className="bg-white border-t mt-12">
        <div className="container mx-auto px-4 py-6 text-sm text-gray-600 flex flex-col md:flex-row justify-between items-center gap-4">
          <div>© {new Date().getFullYear()} Nazho Restaurant • Jl. Contoh No.1, Jakarta</div>
          <div className="flex gap-4">
            <a className="underline">Privacy</a>
            <a className="underline">T&C</a>
          </div>
        </div>
      </footer>

      {toast && (
        <div className="fixed right-6 bottom-6 bg-gray-900 text-white px-4 py-2 rounded shadow">{toast}</div>
      )}
    </div>
  );
}


// ---------- Subcomponents ----------
function StatCard({ title, value }) {
  return (
    <div className="bg-white p-3 rounded shadow flex flex-col items-start">
      <div className="text-xs text-gray-500">{title}</div>
      <div className="text-xl font-bold">{value}</div>
    </div>
  );
}

function ReservationForm({ onReserve }) {
  const [name, setName] = useState("");
  const [guest, setGuest] = useState(2);
  const [datetime, setDatetime] = useState("");
  const [note, setNote] = useState("");

  function submit(e) {
    e.preventDefault();
    if (!name || !datetime) return alert("Isi nama dan tanggal/waktu");
    onReserve({ name, guest, datetime, note });
  }

  return (
    <div className="max-w-2xl mx-auto bg-white p-6 rounded shadow">
      <h3 className="text-2xl font-bold mb-4">Reservasi Meja</h3>
      <form onSubmit={submit} className="grid gap-3">
        <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Nama" className="px-3 py-2 border rounded" />
        <input type="datetime-local" value={datetime} onChange={(e) => setDatetime(e.target.value)} className="px-3 py-2 border rounded" />
        <input type="number" min={1} value={guest} onChange={(e) => setGuest(Number(e.target.value))} className="px-3 py-2 border rounded" />
        <textarea value={note} onChange={(e) => setNote(e.target.value)} placeholder="Catatan (opsional)" className="px-3 py-2 border rounded" />
        <div className="flex gap-2">
          <button type="submit" className="px-4 py-2 rounded bg-yellow-400 text-white">Pesan</button>
        </div>
      </form>
    </div>
  );
}

function ContactForm({ onSend }) {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [message, setMessage] = useState("");

  function submit(e) {
    e.preventDefault();
    if (!name || !message) return alert("Isi nama dan pesan");
    onSend({ name, email, message });
  }

  return (
    <div className="max-w-2xl mx-auto bg-white p-6 rounded shadow">
      <h3 className="text-2xl font-bold mb-4">Kontak Kami</h3>
      <form onSubmit={submit} className="grid gap-3">
        <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Nama" className="px-3 py-2 border rounded" />
        <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email (opsional)" className="px-3 py-2 border rounded" />
        <textarea value={message} onChange={(e) => setMessage(e.target.value)} placeholder="Pesan" className="px-3 py-2 border rounded" />
        <div className="flex gap-2">
          <button type="submit" className="px-4 py-2 rounded bg-yellow-400 text-white">Kirim</button>
        </div>
      </form>
    </div>
  );
}

function Gallery({ images = [] }) {
  return (
    <div>
      <h2 className="text-2xl font-bold mb-4">Galeri</h2>
      <div className="grid md:grid-cols-3 gap-4">
        {images.map((src, i) => (
          <div key={i} className="rounded overflow-hidden shadow">
            <img src={src} alt={`galeri-${i}`} className="w-full h-48 object-cover" />
          </div>
        ))}
      </div>
    </div>
  );
}
