Dokumentasi Sistem Notifikasi Toast SIDESA
Dokumentasi ini menjelaskan cara menggunakan sistem notifikasi toast terpusat di dalam aplikasi SIDESA. Sistem ini dirancang agar konsisten, mudah digunakan, dan bisa menangani dua skenario utama: notifikasi dari aksi yang me-redirect halaman dan notifikasi dari aksi di halaman yang sama (AJAX).

Teknologi yang Digunakan:

Backend: Livewire Events & Laravel Session Flash.

Frontend: Komponen Blade & Alpine.js.

1. Prasyarat & Setup (Sudah Selesai)
Sistem ini bisa bekerja karena ada dua komponen utama yang sudah terpasang di layout app.blade.php:

Komponen Penampil (<x-ui.alert />): Sebuah komponen Alpine.js yang bertugas menampilkan notifikasi saat menerima event flash-message-display.

"Jembatan" Session-ke-Event: Sebuah blok kode Blade/Alpine yang bertugas memeriksa session() setelah redirect dan mengubahnya menjadi event flash-message-display.

<!-- Contoh di app.blade.php -->

<main>
    {{-- Jembatan dari Session ke Alpine Event --}}
    @php
        $flashMessage = session('success') ?? session('error');
        $flashType = session()->has('success') ? 'success' : (session()->has('error') ? 'error' : null);
    @endphp
    @if($flashMessage && $flashType)
        <div x-data x-init="$dispatch('flash-message-display', [{ message: @js($flashMessage), type: @js($flashType) }])"></div>
    @endif

    {{ $slot }}
</main>

2. Cara Penggunaan
Ada dua cara untuk memicu notifikasi, tergantung pada jenis aksi yang Anda lakukan.

Kasus 1: Notifikasi untuk Aksi TANPA Redirect (AJAX)
Gunakan metode ini untuk aksi yang terjadi di halaman yang sama, seperti menghapus data dari tabel, mengubah status, dll.

Metode: Gunakan $this->dispatch() dari dalam komponen Livewire Anda.

Contoh (dari Users/Index.php):

<?php

namespace App\Livewire\Users;

// ...

class Index extends Component
{
    public function delete(User $user)
    {
        if ($user->id === Auth::id()) {
            // Kirim event dengan tipe 'error'
            $this->dispatch('flash-message-display', [
                'message' => 'Anda tidak dapat menghapus akun Anda sendiri.', 
                'type' => 'error'
            ]);
            return;
        }

        $user->delete();
        
        // Kirim event dengan tipe 'success'
        $this->dispatch('flash-message-display', [
            'message' => 'Pengguna berhasil dihapus.', 
            'type' => 'success'
        ]);
    }
}

Kasus 2: Notifikasi untuk Aksi DENGAN Redirect
Gunakan metode ini untuk aksi yang akan mengalihkan pengguna ke halaman lain, seperti menyimpan form Tambah atau Edit data.

Metode: Gunakan session()->flash() standar dari Laravel.

Contoh (dari Users/Form.php):

<?php

namespace App\Livewire\Users;

// ...

class Form extends Component
{
    public function save()
    {
        // ... logika validasi dan simpan data ...

        if ($this->isEditMode) {
            $this->user->update($validated);
            // Simpan pesan sukses ke session
            session()->flash('success', 'Pengguna berhasil diperbarui.');
        } else {
            User::create($validated);
            // Simpan pesan sukses ke session
            session()->flash('success', 'Pengguna berhasil ditambahkan.');
        }

        return $this->redirect(route('users.index'), navigate: true);
    }
}

3. Menambah Tipe Notifikasi Baru (Contoh: 'Warning')
Jika di masa depan Anda butuh tipe notifikasi baru (misalnya 'warning' dengan warna kuning), Anda hanya perlu melakukan dua hal:

Langkah 1: Kirim Tipe yang Benar
Saat memanggil notifikasi, gunakan type: 'warning'.

// Contoh untuk dispatch
$this->dispatch('flash-message-display', ['message' => 'Ini adalah peringatan.', 'type' => 'warning']);

// Contoh untuk session
session()->flash('warning', 'Ini adalah peringatan.'); // Anda perlu update "Jembatan" di app.blade.php

Langkah 2: Update Komponen <x-ui.alert />
Buka resources/views/components/ui/alert.blade.php untuk menambahkan styling dan ikon baru.

Contoh Perubahan:

<!-- 1. Tambahkan class binding baru di div utama -->
:class="{ 
    'bg-green-500': type === 'success', 
    'bg-red-500': type === 'error',
    'bg-amber-500': type === 'warning'  // <-- Tambahkan ini
}"

<!-- ... -->

<!-- 2. Tambahkan blok ikon baru di dalam div utama -->
<!-- Ikon Warning (Heroicons) -->
<div x-show="type === 'warning'">
    <!-- Ganti dengan SVG ikon warning yang sesuai -->
    <svg class="w-6 h-6 text-white" xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v3.75m-9.303 3.376c-.866 1.5.217 3.374 1.948 3.374h14.71c1.73 0 2.813-1.874 1.948-3.374L13.949 3.378c-.866-1.5-3.032-1.5-3.898 0L2.697 16.126ZM12 15.75h.007v.008H12v-.008Z" />
    </svg>
</div>
