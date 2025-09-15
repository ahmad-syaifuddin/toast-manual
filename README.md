# 🍞 Toast Notification SIDESA - Gen Z Edition

> **TL;DR**: Cara bikin notifikasi keren di SIDESA yang gampang banget dipake. No more alert() yang cringe! ✨

## 🚀 What's This About?

Jadi gini bestie, di SIDESA kita udah setup sistem notifikasi toast yang smooth banget. Basically ada dua scenario utama:

1. **Notifikasi tanpa redirect** (AJAX vibes) - buat aksi yang stay di halaman yang sama
2. **Notifikasi dengan redirect** - buat aksi yang pindah halaman (kayak save form terus redirect)

## 🛠️ Tech Stack (Yang Udah Ada)

- **Backend**: Livewire Events + Laravel Session Flash (solid combo fr)
- **Frontend**: Blade Components + Alpine.js (chef's kiss 👌)

## 📦 Setup (Already Done, No Worries)

Udah ada 2 komponen yang jalan di background:

1. **Komponen Penampil** (`<x-ui.alert />`) - yang nampilin notifikasi
2. **"Bridge" Session-to-Event** - yang convert session flash jadi event

```php
<!-- Udah dipasang di app.blade.php -->
<main>
    {{-- Bridge dari Session ke Alpine Event --}}
    @php
        $flashMessage = session('success') ?? session('error');
        $flashType = session()->has('success') ? 'success' : 
                    (session()->has('error') ? 'error' : null);
    @endphp
    @if($flashMessage && $flashType)
        <div x-data x-init="$dispatch('flash-message-display', 
             [{ message: @js($flashMessage), type: @js($flashType) }])">
        </div>
    @endif

    {{ $slot }}
</main>
```

## 💡 How To Use (The Real Deal)

### Scenario 1: AJAX Actions (No Redirect) 🔄

**Use Case**: Delete data, update status, basically any action yang stay di halaman yang sama.

**Method**: Pake `$this->dispatch()` dari Livewire component

```php
<?php
// Example dari Users/Index.php

class Index extends Component
{
    public function delete(User $user)
    {
        // Kalo user mau delete diri sendiri (big no no)
        if ($user->id === Auth::id()) {
            // Kirim error toast
            $this->dispatch('flash-message-display', [
                'message' => 'Gabisa delete akun sendiri dong bestie! 🤦‍♂️', 
                'type' => 'error'
            ]);
            return; // Stop execution
        }

        $user->delete();
        
        // Success toast
        $this->dispatch('flash-message-display', [
            'message' => 'User berhasil dihapus! Bye bye~ 👋', 
            'type' => 'success'
        ]);
    }
}
```

### Scenario 2: Redirect Actions 🔀

**Use Case**: Save form terus redirect, basically any action yang pindah halaman.

**Method**: Pake `session()->flash()` Laravel standard

```php
<?php
// Example dari Users/Form.php

class Form extends Component
{
    public function save()
    {
        // Validasi & save logic here...

        if ($this->isEditMode) {
            $this->user->update($validated);
            // Flash message buat redirect
            session()->flash('success', 'User berhasil diupdate! Fresh banget! ✨');
        } else {
            User::create($validated);
            session()->flash('success', 'User baru berhasil ditambahin! Welcome to the club! 🎉');
        }

        return $this->redirect(route('users.index'), navigate: true);
    }
}
```

## 🎨 Adding New Notification Types

Mau nambahin tipe baru kayak 'warning'? Ez pz lemon squeezy:

### Step 1: Send the Right Type

```php
// Buat dispatch (AJAX)
$this->dispatch('flash-message-display', [
    'message' => 'Hati-hati bestie, ini warning! ⚠️', 
    'type' => 'warning'
]);

// Buat session (Redirect)
session()->flash('warning', 'Ini warning message!');
// Note: Lu perlu update bridge di app.blade.php buat handle 'warning'
```

### Step 2: Update Alert Component

Buka `resources/views/components/ui/alert.blade.php` dan tambahin styling:

```html
<!-- Update class binding -->
:class="{ 
    'bg-green-500': type === 'success', 
    'bg-red-500': type === 'error',
    'bg-amber-500': type === 'warning'  // 👈 New kid on the block
}"

<!-- Tambahin icon warning -->
<div x-show="type === 'warning'">
    <!-- Warning icon SVG here -->
    <svg class="w-6 h-6 text-white" xmlns="http://www.w3.org/2000/svg" 
         fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" 
            d="M12 9v3.75m-9.303 3.376c-.866 1.5.217 3.374 1.948 3.374h14.71c1.73 0 2.813-1.874 1.948-3.374L13.949 3.378c-.866-1.5-3.032-1.5-3.898 0L2.697 16.126ZM12 15.75h.007v.008H12v-.008Z" />
    </svg>
</div>
```

## 📝 Quick Reference

| Scenario | Method | Example |
|----------|---------|---------|
| **AJAX (No Redirect)** | `$this->dispatch()` | Delete, Update Status |
| **Redirect** | `session()->flash()` | Save Form, Create Data |

## 💭 Pro Tips

- **Message Types**: `success`, `error` (available), `warning` (need setup)
- **Keep it short**: Toast messages should be concise tapi informative
- **Use emojis**: Bikin message lebih engaging (but don't overdo it)
- **Test both scenarios**: Make sure AJAX dan redirect works properly

## 🐛 Common Issues

1. **Toast gak muncul setelah redirect?** 
   - Check bridge di app.blade.php
   - Make sure session flash key match

2. **AJAX toast gak muncul?**
   - Check console buat errors
   - Make sure event name spelling correct

3. **Styling aneh?**
   - Check alert component styling
   - Make sure Alpine.js loaded properly

---

**Happy coding bestie!** 🚀✨

*P.S: Kalo ada yang bingung, just ask. We're all learning together! 💪*
