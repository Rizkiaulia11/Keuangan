<script>
document.addEventListener('DOMContentLoaded', () => {
    const scriptURL = "https://script.google.com/macros/s/AKfycbwrzO8Rpq_90W4oobd8WoSxCCO9z8NLsbNhFFBR-YjMfxbG0n-MAJl7Idt3VabEdcXTI6w/exec";

    const form = document.getElementById('transactionForm');
    const transactionsList = document.getElementById('transactionsList');
    const dailyTab = document.getElementById('dailyTab');
    const weeklyTab = document.getElementById('weeklyTab');
    const monthlyTab = document.getElementById('monthlyTab');
    const reportContent = document.getElementById('reportContent');
    const clearBtn = document.getElementById('clearHistory');
    const balanceEl = document.getElementById('balance');
    const notification = document.getElementById('notification');

    let transactions = [];

    // 🔹 Ambil data dari Google Sheets
    async function loadTransactions() {
        const res = await fetch(scriptURL);
        transactions = await res.json();
        renderTransactions();
        // refresh laporan sesuai tab aktif
        if (dailyTab.classList.contains('active-tab')) renderReport('daily');
        else if (weeklyTab.classList.contains('active-tab')) renderReport('weekly');
        else if (monthlyTab.classList.contains('active-tab')) renderReport('monthly');
    }

    // 🔹 Kirim data ke Google Sheets
    async function saveTransaction(t) {
        await fetch(scriptURL, {
            method: "POST",
            body: JSON.stringify(t)
        });
        loadTransactions();
    }

    // 🔹 Hapus semua data di Google Sheets
    async function clearTransactions() {
        await fetch(scriptURL, {
            method: "POST",
            body: JSON.stringify({ action: "clear" })
        });
        loadTransactions();
    }

    // Fungsi notifikasi
    function showNotification(message, type = 'success') {
        notification.textContent = message;
        notification.className = `notification ${type} show`;
        setTimeout(() => {
            notification.classList.remove('show');
        }, 3000);
    }

    // Hitung saldo
    function updateBalance() {
        let income = 0, expense = 0;
        transactions.forEach(t => {
            if (t.type === "income") income += Number(t.amount);
            else if (t.type === "expense") expense += Number(t.amount);
        });
        const balance = income - expense;
        balanceEl.textContent = "Rp " + balance.toLocaleString('id-ID');
        balanceEl.className = "text-3xl font-bold mt-2 " + (balance >= 0 ? "text-green-600" : "text-red-600");
    }

    // Render transaksi
    function renderTransactions() {
        transactionsList.innerHTML = transactions.length ? "" : '<p class="text-gray-500 text-center py-4"><i class="fas fa-receipt mr-2"></i>Tidak ada transaksi</p>';
        const sorted = [...transactions].sort((a, b) => new Date(b.date) - new Date(a.date));
        sorted.forEach(t => {
            const item = document.createElement('div');
            item.className = `transaction-item flex justify-between items-center p-4 border rounded-lg ${t.type === 'income' ? 'bg-green-50 border-green-200' : 'bg-red-50 border-red-200'}`;
            const formattedDate = new Date(t.date).toLocaleDateString('id-ID',{weekday:'short',year:'numeric',month:'short',day:'numeric'});
            item.innerHTML = `
                <div class="flex items-center">
                    <div class="mr-3 text-xl ${t.type === 'income' ? 'text-green-500' : 'text-red-500'}">
                        <i class="fas ${t.type === 'income' ? 'fa-arrow-circle-down' : 'fa-arrow-circle-up'}"></i>
                    </div>
                    <div>
                        <div class="font-medium text-gray-800">${t.description}</div>
                        <div class="text-sm text-gray-500">${formattedDate}</div>
                    </div>
                </div>
                <div class="text-right">
                    <div class="font-semibold ${t.type === 'income' ? 'text-green-600' : 'text-red-600'}">
                        ${t.type === 'income' ? '+' : '-'} Rp ${Number(t.amount).toLocaleString('id-ID')}
                    </div>
                    <div class="text-xs ${t.type === 'income' ? 'text-green-500' : 'text-red-500'}">
                        ${t.type === 'income' ? 'Pemasukan' : 'Pengeluaran'}
                    </div>
                </div>
            `;
            transactionsList.appendChild(item);
        });
        updateBalance();
    }

    // Render laporan
    function renderReport(type) {
        const groups = {};
        transactions.forEach(t => {
            const date = new Date(t.date);
            let key;
            if (type === "daily") {
                key = date.toLocaleDateString('id-ID',{weekday:'long',year:'numeric',month:'long',day:'numeric'});
            } else if (type === "weekly") {
                const weekStart = new Date(date);
                weekStart.setDate(date.getDate() - date.getDay());
                key = `Minggu ${weekStart.toLocaleDateString('id-ID',{day:'numeric',month:'short'})}`;
            } else if (type === "monthly") {
                key = date.toLocaleDateString('id-ID',{year:'numeric',month:'long'});
            }
            if (!groups[key]) groups[key] = { income: 0, expense: 0 };
            groups[key][t.type] += Number(t.amount);
        });

        if (Object.keys(groups).length === 0) {
            reportContent.innerHTML = `
                <div class="py-6">
                    <i class="fas fa-file-invoice-dollar text-3xl mb-3 text-indigo-400"></i>
                    <p class="text-gray-500">Tidak ada data laporan</p>
                </div>`;
            return;
        }

        let html = '<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-4">';
        Object.keys(groups).sort().reverse().forEach(key => {
            const balance = groups[key].income - groups[key].expense;
            html += `
                <div class="bg-white p-5 rounded-xl border border-gray-200 shadow-sm">
                    <h3 class="font-semibold text-lg mb-3 text-gray-700">${key}</h3>
                    <div class="space-y-2">
                        <p class="text-green-600 flex justify-between items-center">
                            <span><i class="fas fa-arrow-down mr-2"></i> Pemasukan:</span>
                            <span>Rp ${groups[key].income.toLocaleString('id-ID')}</span>
                        </p>
                        <p class="text-red-600 flex justify-between items-center">
                            <span><i class="fas fa-arrow-up mr-2"></i> Pengeluaran:</span>
                            <span>Rp ${groups[key].expense.toLocaleString('id-ID')}</span>
                        </p>
                        <p class="font-semibold pt-3 border-t flex justify-between items-center">
                            <span><i class="fas fa-balance-scale mr-2"></i> Saldo:</span>
                            <span class="${balance >= 0 ? 'text-green-600' : 'text-red-600'}">
                                Rp ${balance.toLocaleString('id-ID')}
                            </span>
                        </p>
                    </div>
                </div>`;
        });
        html += '</div>';
        reportContent.innerHTML = html;
    }

    // Event submit form
    form.addEventListener('submit', (e) => {
        e.preventDefault();
        const date = document.getElementById('date').value;
        const description = document.getElementById('description').value;
        const amount = parseFloat(document.getElementById('amount').value);
        const type = document.getElementById('type').value;
        if (!date || !description || !amount || amount <= 0) {
            showNotification('Harap isi semua field dengan benar!', 'error');
            return;
        }
        saveTransaction({ date, description, amount, type });
        form.reset();
        document.getElementById('date').valueAsDate = new Date();
        showNotification('Transaksi berhasil disimpan!');
    });

    // Tombol hapus
    clearBtn.addEventListener('click', () => {
        const password = prompt("Masukkan sandi untuk menghapus riwayat:");
        if (password === "Nmax122333") {
            clearTransactions();
            showNotification('Riwayat transaksi berhasil dihapus!');
        } else {
            showNotification('Sandi salah! Riwayat tidak dihapus.', 'error');
        }
    });

    // Inisialisasi
    document.getElementById('date').valueAsDate = new Date();
    loadTransactions();
    renderReport('daily');
    dailyTab.classList.add('active-tab');
    dailyTab.addEventListener('click', () => {switchTab(dailyTab,'daily')});
    weeklyTab.addEventListener('click', () => {switchTab(weeklyTab,'weekly')});
    monthlyTab.addEventListener('click', () => {switchTab(monthlyTab,'monthly')});

    function switchTab(tab, type) {
        [dailyTab, weeklyTab, monthlyTab].forEach(t => t.classList.remove('active-tab'));
        tab.classList.add('active-tab');
        renderReport(type);
    }
});
</script>
