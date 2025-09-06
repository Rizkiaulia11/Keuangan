<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        .tab-btn {
            padding: 0.5rem 1rem;
            border-radius: 0.375rem;
            background: #f3f4f6;
            transition: all 0.3s;
        }
        .active-tab {
            background: #2563eb;
            color: white;
        }
        .transaction-item {
            transition: transform 0.2s;
        }
        .transaction-item:hover {
            transform: translateX(5px);
        }
        #clearHistory {
            transition: all 0.3s;
        }
        .notification {
            position: fixed;
            top: 20px;
            right: 20px;
            padding: 15px 20px;
            border-radius: 8px;
            color: white;
            opacity: 0;
            transform: translateY(-20px);
            transition: all 0.3s ease;
            z-index: 1000;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        .notification.show {
            opacity: 1;
            transform: translateY(0);
        }
        .notification.success {
            background-color: #10B981;
        }
        .notification.error {
            background-color: #EF4444;
        }
        .loading {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid rgba(255,255,255,.3);
            border-radius: 50%;
            border-top-color: #fff;
            animation: spin 1s ease-in-out infinite;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen flex flex-col items-center py-8 px-4">
    <!-- Notifikasi -->
    <div id="notification" class="notification"></div>
    
    <div class="bg-white w-full max-w-2xl rounded-xl shadow-lg p-6 mb-8">
        <h1 class="text-3xl font-bold mb-6 text-center text-indigo-700">
            <i class="fas fa-bookkeeping mr-2"></i>Catatan Keuangan
        </h1>
        
        <!-- Koneksi Google Sheets -->
        <div id="connectionStatus" class="mb-4 p-3 bg-yellow-100 text-yellow-800 rounded-lg flex items-center hidden">
            <i class="fas fa-sync-alt mr-2"></i>
            <span>Menyambungkan ke Google Sheets...</span>
        </div>
        
        <!-- Saldo -->
        <div class="mb-8 text-center p-4 bg-gradient-to-r from-blue-100 to-indigo-100 rounded-lg border border-indigo-200">
            <h2 class="text-lg font-medium text-indigo-800">Saldo Saat Ini</h2>
            <p id="balance" class="text-3xl font-bold text-indigo-700 mt-2">Rp 0</p>
        </div>
        
        <!-- Form input transaksi -->
        <form id="transactionForm" class="space-y-4 mb-8 p-4 bg-gray-50 rounded-lg border border-gray-200">
            <h3 class="text-lg font-semibold text-gray-700 mb-2">
                <i class="fas fa-plus-circle mr-2 text-indigo-600"></i>Tambah Transaksi Baru
            </h3>
            <input type="date" id="date" class="w-full border rounded-lg px-4 py-3 focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400" required>
            <input type="text" id="description" placeholder="Deskripsi transaksi" class="w-full border rounded-lg px-4 py-3 focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400" required>
            <input type="number" id="amount" placeholder="Jumlah (Rp)" class="w-full border rounded-lg px-4 py-3 focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400" required>
            <select id="type" class="w-full border rounded-lg px-4 py-3 focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400" required>
                <option value="Pemasukan">Pemasukan</option>
                <option value="Pengeluaran">Pengeluaran</option>
            </select>
            <button type="submit" class="w-full bg-indigo-600 text-white py-3 rounded-lg hover:bg-indigo-700 transition duration-300 flex items-center justify-center font-medium">
                <i class="fas fa-save mr-2"></i> Simpan Transaksi
            </button>
        </form>
        
        <!-- Riwayat transaksi -->
        <div class="mb-6">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-semibold text-gray-700 flex items-center">
                    <i class="fas fa-history mr-2 text-indigo-600"></i>Riwayat Transaksi
                </h2>
                <button id="refreshBtn" class="text-indigo-600 hover:text-indigo-800 flex items-center">
                    <i class="fas fa-sync-alt mr-1"></i> Segarkan
                </button>
            </div>
            <div id="transactionsList" class="space-y-3"></div>
        </div>
        
        <!-- Tombol hapus -->
        <div class="mt-6">
            <button id="clearHistory" class="w-full bg-gradient-to-r from-red-500 to-orange-500 text-white py-3 rounded-lg hover:from-red-600 hover:to-orange-600 transition duration-300 flex items-center justify-center font-medium">
                <i class="fas fa-trash-alt mr-2"></i> Hapus Semua Riwayat
            </button>
        </div>
    </div>
    
    <!-- Tabs laporan -->
    <div class="bg-white w-full max-w-4xl rounded-xl shadow-lg p-6">
        <h2 class="text-xl font-semibold mb-6 text-center text-gray-700">
            <i class="fas fa-chart-line mr-2 text-indigo-600"></i>Laporan Keuangan
        </h2>
        <div class="flex justify-center space-x-4 mb-6">
            <button id="dailyTab" class="tab-btn active-tab">
                <i class="fas fa-calendar-day mr-2"></i>Harian
            </button>
            <button id="weeklyTab" class="tab-btn">
                <i class="fas fa-calendar-week mr-2"></i>Mingguan
            </button>
            <button id="monthlyTab" class="tab-btn">
                <i class="fas fa-calendar-alt mr-2"></i>Bulanan
            </button>
        </div>
        <div id="reportContent" class="text-center text-gray-500 p-4 bg-gray-50 rounded-lg">
            <i class="fas fa-info-circle text-2xl mb-2 text-indigo-400"></i>
            <p>Pilih tab untuk melihat laporan keuangan</p>
        </div>
    </div>

    <script>
        // URL Web App Google Apps Script (Ganti dengan URL Anda setelah deploy)
        const WEB_APP_URL = 'https://script.google.com/macros/s/AKfycbzdAlFyOsIFIdlp3apRRx_bqvj0XYu-ZkwYBOblaKr-QtRG2nHBFFV6sxlw4Vq0nwpf4g/exec';
        
        document.addEventListener('DOMContentLoaded', () => {
            // Mengambil elemen DOM
            const form = document.getElementById('transactionForm');
            const transactionsList = document.getElementById('transactionsList');
            const dailyTab = document.getElementById('dailyTab');
            const weeklyTab = document.getElementById('weeklyTab');
            const monthlyTab = document.getElementById('monthlyTab');
            const reportContent = document.getElementById('reportContent');
            const clearBtn = document.getElementById('clearHistory');
            const balanceEl = document.getElementById('balance');
            const notification = document.getElementById('notification');
            const refreshBtn = document.getElementById('refreshBtn');
            const connectionStatus = document.getElementById('connectionStatus');
            
            let transactions = [];

            // Fungsi untuk menampilkan notifikasi
            function showNotification(message, type = 'success') {
                notification.textContent = message;
                notification.className = `notification ${type} show`;
                
                setTimeout(() => {
                    notification.classList.remove('show');
                }, 3000);
            }

            // Fungsi untuk menampilkan status koneksi
            function showConnectionStatus(show, message = '') {
                if (show) {
                    connectionStatus.classList.remove('hidden');
                    if (message) {
                        connectionStatus.querySelector('span').textContent = message;
                    }
                } else {
                    connectionStatus.classList.add('hidden');
                }
            }

            // Fungsi untuk mengambil data dari Google Sheets
            async function fetchTransactions() {
                showConnectionStatus(true, 'Mengambil data dari Google Sheets...');
                
                try {
                    const response = await fetch(`${WEB_APP_URL}?action=getTransactions`);
                    if (!response.ok) {
                        throw new Error('Gagal mengambil data');
                    }
                    
                    const result = await response.json();
                    
                    if (result.success) {
                        transactions = result.data;
                        renderTransactions();
                        showConnectionStatus(false);
                        return true;
                    } else {
                        throw new Error(result.error || 'Gagal mengambil data');
                    }
                } catch (error) {
                    console.error('Error:', error);
                    showConnectionStatus(true, 'Gagal terhubung ke Google Sheets. Menggunakan mode offline...');
                    
                    // Fallback ke localStorage jika online tidak berhasil
                    const localData = localStorage.getItem('transactions');
                    if (localData) {
                        transactions = JSON.parse(localData);
                        renderTransactions();
                    }
                    
                    return false;
                }
            }

            // Fungsi untuk menyimpan transaksi ke Google Sheets
            async function saveTransactionToSheets(transaction) {
                showConnectionStatus(true, 'Menyimpan ke Google Sheets...');
                
                try {
                    const response = await fetch(WEB_APP_URL, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify({
                            action: 'addTransaction',
                            ...transaction
                        })
                    });
                    
                    if (!response.ok) {
                        throw new Error('Gagal menyimpan data');
                    }
                    
                    const result = await response.json();
                    
                    if (result.success) {
                        showConnectionStatus(false);
                        return true;
                    } else {
                        throw new Error(result.error || 'Gagal menyimpan data');
                    }
                } catch (error) {
                    console.error('Error:', error);
                    showConnectionStatus(true, 'Gagal menyimpan ke Google Sheets. Data disimpan secara lokal...');
                    
                    // Fallback ke localStorage
                    transactions.push(transaction);
                    localStorage.setItem('transactions', JSON.stringify(transactions));
                    renderTransactions();
                    
                    return false;
                }
            }

            // Fungsi untuk menghapus semua data
            async function clearAllTransactions() {
                showConnectionStatus(true, 'Menghapus data dari Google Sheets...');
                
                try {
                    const response = await fetch(WEB_APP_URL, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify({
                            action: 'clearTransactions'
                        })
                    });
                    
                    if (!response.ok) {
                        throw new Error('Gagal menghapus data');
                    }
                    
                    const result = await response.json();
                    
                    if (result.success) {
                        transactions = [];
                        renderTransactions();
                        reportContent.innerHTML = `
                            <div class="py-4">
                                <i class="fas fa-check-circle text-2xl mb-2 text-green-400"></i>
                                <p class="text-gray-500">Pilih tab untuk melihat laporan</p>
                            </div>
                        `;
                        showConnectionStatus(false);
                        showNotification('Semua data berhasil dihapus!');
                    } else {
                        throw new Error(result.error || 'Gagal menghapus data');
                    }
                } catch (error) {
                    console.error('Error:', error);
                    showConnectionStatus(true, 'Gagal menghapus dari Google Sheets...');
                    
                    // Fallback ke localStorage
                    const password = prompt("Masukkan sandi untuk menghapus riwayat:");
                    if (password === "Nmax122333") {
                        transactions = [];
                        localStorage.removeItem('transactions');
                        renderTransactions();
                        reportContent.innerHTML = `
                            <div class="py-4">
                                <i class="fas fa-check-circle text-2xl mb-2 text-green-400"></i>
                                <p class="text-gray-500">Pilih tab untuk melihat laporan</p>
                            </div>
                        `;
                        showNotification('Riwayat transaksi berhasil dihapus!');
                    } else {
                        showNotification('Sandi salah! Riwayat tidak dihapus.', 'error');
                    }
                }
            }

            // Fungsi untuk memperbarui tampilan saldo
            function updateBalance() {
                let income = 0, expense = 0;
                transactions.forEach(t => {
                    if (t[3] === "Pemasukan") income += Number(t[2]);
                    else if (t[3] === "Pengeluaran") expense += Number(t[2]);
                });
                
                const balance = income - expense;
                balanceEl.textContent = "Rp " + balance.toLocaleString('id-ID');
                balanceEl.className = "text-3xl font-bold mt-2 " + (balance >= 0 ? "text-green-600" : "text-red-600");
            }

            // Fungsi untuk merender daftar transaksi
            function renderTransactions() {
                transactionsList.innerHTML = transactions.length > 1 ? "" : '<p class="text-gray-500 text-center py-4"><i class="fas fa-receipt mr-2"></i>Tidak ada transaksi</p>';
                
                // Lewati header (baris pertama) dan urutkan berdasarkan tanggal (terbaru pertama)
                const transactionData = transactions.slice(1).sort((a, b) => new Date(b[0]) - new Date(a[0]));
                
                transactionData.forEach(t => {
                    const item = document.createElement('div');
                    item.className = `transaction-item flex justify-between items-center p-4 border rounded-lg ${t[3] === 'Pemasukan' ? 'bg-green-50 border-green-200' : 'bg-red-50 border-red-200'}`;
                    
                    const dateObj = new Date(t[0]);
                    const formattedDate = dateObj.toLocaleDateString('id-ID', {
                        weekday: 'short',
                        year: 'numeric',
                        month: 'short',
                        day: 'numeric'
                    });
                    
                    item.innerHTML = `
                        <div class="flex items-center">
                            <div class="mr-3 text-xl ${t[3] === 'Pemasukan' ? 'text-green-500' : 'text-red-500'}">
                                <i class="fas ${t[3] === 'Pemasukan' ? 'fa-arrow-circle-down' : 'fa-arrow-circle-up'}"></i>
                            </div>
                            <div>
                                <div class="font-medium text-gray-800">${t[1]}</div>
                                <div class="text-sm text-gray-500">${formattedDate}</div>
                            </div>
                        </div>
                        <div class="text-right">
                            <div class="font-semibold ${t[3] === 'Pemasukan' ? 'text-green-600' : 'text-red-600'}">
                                ${t[3] === 'Pemasukan' ? '+' : '-'} Rp ${Number(t[2]).toLocaleString('id-ID')}
                            </div>
                            <div class="text-xs ${t[3] === 'Pemasukan' ? 'text-green-500' : 'text-red-500'}">
                                ${t[3]}
                            </div>
                        </div>
                    `;
                    transactionsList.appendChild(item);
                });
                
                updateBalance();
            }

            // Fungsi untuk merender laporan
            function renderReport(type) {
                // Lewati header (baris pertama)
                const transactionData = transactions.slice(1);
                const groups = {};
                
                transactionData.forEach(t => {
                    let key;
                    const date = new Date(t[0]);
                    
                    if (type === "daily") {
                        key = new Date(t[0]).toLocaleDateString('id-ID', {
                            weekday: 'long',
                            year: 'numeric',
                            month: 'long',
                            day: 'numeric'
                        });
                    } else if (type === "weekly") {
                        const weekStart = new Date(date);
                        weekStart.setDate(date.getDate() - date.getDay());
                        key = `Minggu ${weekStart.toLocaleDateString('id-ID', { day: 'numeric', month: 'short' })}`;
                    } else if (type === "monthly") {
                        key = date.toLocaleDateString('id-ID', { year: 'numeric', month: 'long' });
                    }
                    
                    if (!groups[key]) groups[key] = { Pemasukan: 0, Pengeluaran: 0 };
                    groups[key][t[3]] += Number(t[2]);
                });
                
                if (Object.keys(groups).length === 0) {
                    reportContent.innerHTML = `
                        <div class="py-6">
                            <i class="fas fa-file-invoice-dollar text-3xl mb-3 text-indigo-400"></i>
                            <p class="text-gray-500">Tidak ada data laporan</p>
                        </div>
                    `;
                    return;
                }
                
                let html = '<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-4">';
                Object.keys(groups).sort().reverse().forEach(key => {
                    const balance = groups[key].Pemasukan - groups[key].Pengeluaran;
                    
                    html += `
                        <div class="bg-white p-5 rounded-xl border border-gray-200 shadow-sm">
                            <h3 class="font-semibold text-lg mb-3 text-gray-700">${key}</h3>
                            <div class="space-y-2">
                                <p class="text-green-600 flex justify-between items-center">
                                    <span class="flex items-center"><i class="fas fa-arrow-down mr-2"></i> Pemasukan:</span> 
                                    <span>Rp ${groups[key].Pemasukan.toLocaleString('id-ID')}</span>
                                </p>
                                <p class="text-red-600 flex justify-between items-center">
                                    <span class="flex items-center"><i class="fas fa-arrow-up mr-2"></i> Pengeluaran:</span> 
                                    <span>Rp ${groups[key].Pengeluaran.toLocaleString('id-ID')}</span>
                                </p>
                                <p class="font-semibold pt-3 border-t flex justify-between items-center">
                                    <span class="flex items-center"><i class="fas fa-balance-scale mr-2"></i> Saldo:</span> 
                                    <span class="${balance >= 0 ? 'text-green-600' : 'text-red-600'}">
                                        Rp ${balance.toLocaleString('id-ID')}
                                    </span>
                                </p>
                            </div>
                        </div>
                    `;
                });
                
                html += '</div>';
                reportContent.innerHTML = html;
            }

            // Fungsi untuk mengganti tab
            function switchTab(tab, type) {
                [dailyTab, weeklyTab, monthlyTab].forEach(t => t.classList.remove('active-tab'));
                tab.classList.add('active-tab');
                renderReport(type);
            }

            // Event listeners untuk tab
            dailyTab.addEventListener('click', () => switchTab(dailyTab, 'daily'));
            weeklyTab.addEventListener('click', () => switchTab(weeklyTab, 'weekly'));
            monthlyTab.addEventListener('click', () => switchTab(monthlyTab, 'monthly'));

            // Event listener untuk form
            form.addEventListener('submit', async (e) => {
                e.preventDefault();
                
                const date = document.getElementById('date').value;
                const description = document.getElementById('description').value;
                const amount = parseFloat(document.getElementById('amount').value);
                const type = document.getElementById('type').value;
                
                // Validasi input
                if (!date || !description || !amount || amount <= 0) {
                    showNotification('Harap isi semua field dengan benar!', 'error');
                    return;
                }
                
                // Tampilkan loading state pada tombol
                const submitBtn = form.querySelector('button[type="submit"]');
                const originalText = submitBtn.innerHTML;
                submitBtn.innerHTML = '<span class="loading"></span> Menyimpan...';
                submitBtn.disabled = true;
                
                // Simpan transaksi
                const success = await saveTransactionToSheets([date, description, amount, type]);
                
                // Kembalikan state tombol
                submitBtn.innerHTML = originalText;
                submitBtn.disabled = false;
                
                if (success) {
                    // Perbarui data dari server
                    await fetchTransactions();
                    
                    // Perbarui laporan jika tab aktif
                    if (dailyTab.classList.contains('active-tab')) renderReport('daily');
                    else if (weeklyTab.classList.contains('active-tab')) renderReport('weekly');
                    else if (monthlyTab.classList.contains('active-tab')) renderReport('monthly');
                    
                    // Reset form
                    form.reset();
                    
                    // Set tanggal ke hari ini
                    document.getElementById('date').valueAsDate = new Date();
                    
                    // Tampilkan notifikasi sukses
                    showNotification('Transaksi berhasil disimpan!');
                }
            });

            // Event listener untuk tombol refresh
            refreshBtn.addEventListener('click', async () => {
                refreshBtn.innerHTML = '<span class="loading"></span>';
                await fetchTransactions();
                refreshBtn.innerHTML = '<i class="fas fa-sync-alt mr-1"></i> Segarkan';
                
                // Perbarui laporan jika tab aktif
                if (dailyTab.classList.contains('active-tab')) renderReport('daily');
                else if (weeklyTab.classList.contains('active-tab')) renderReport('weekly');
                else if (monthlyTab.classList.contains('active-tab')) renderReport('monthly');
                
                showNotification('Data diperbarui!');
            });

            // Event listener untuk tombol hapus
            clearBtn.addEventListener('click', () => {
                clearAllTransactions();
            });

            // Inisialisasi aplikasi
            fetchTransactions();
            
            // Set tanggal hari ini sebagai nilai default
            const today = new Date().toISOString().split('T')[0];
            document.getElementById('date').value = today;
        });
    </script>
</body>
</html>
