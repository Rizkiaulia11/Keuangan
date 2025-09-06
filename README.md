<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Catatan Keuangan</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet"/>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"/>
  <style>
    /* Styles bisa disalin dari kode awalmu */
    .tab-btn { padding: .5rem 1rem; border-radius: .375rem; background: #f3f4f6; transition: .3s; }
    .active-tab { background: #2563eb; color: white; }
    .transaction-item { transition: transform .2s; }
    .transaction-item:hover { transform: translateX(5px); }
    .notification { position: fixed; top: 20px; right: 20px; padding: 15px 20px; border-radius: 8px; color: white; opacity: 0; transform: translateY(-20px); transition: .3s; z-index: 1000; box-shadow: 0 4px 12px rgba(0,0,0,0.15); }
    .notification.show { opacity:1; transform: translateY(0); }
    .notification.success { background-color: #10B981; }
    .notification.error { background-color: #EF4444; }
  </style>
</head>
<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen flex flex-col items-center py-8 px-4">

  <div id="notification" class="notification"></div>

  <div class="bg-white w-full max-w-2xl rounded-xl shadow-lg p-6 mb-8">
    <h1 class="text-3xl font-bold mb-6 text-center text-indigo-700">
      <i class="fas fa-bookkeeping mr-2"></i>Catatan Keuangan
    </h1>

    <div class="mb-8 text-center p-4 bg-gradient-to-r from-blue-100 to-indigo-100 rounded-lg border border-indigo-200">
      <h2 class="text-lg font-medium text-indigo-800">Saldo Saat Ini</h2>
      <p id="balance" class="text-3xl font-bold text-indigo-700 mt-2">Rp 0</p>
    </div>

    <form id="transactionForm" class="space-y-4 mb-8 p-4 bg-gray-50 rounded-lg border border-gray-200">
      <h3 class="text-lg font-semibold text-gray-700 mb-2">
        <i class="fas fa-plus-circle mr-2 text-indigo-600"></i>Tambah Transaksi Baru
      </h3>
      <input type="date" id="date" class="w-full border rounded-lg px-4 py-3 focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400" required/>
      <input type="text" id="description" placeholder="Deskripsi transaksi" class="w-full border rounded-lg px-4 py-3 focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400" required/>
      <input type="number" id="amount" placeholder="Jumlah (Rp)" class="w-full border rounded-lg px-4 py-3 focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400" required/>
      <select id="type" class="w-full border rounded-lg px-4 py-3 focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400" required>
        <option value="income">Pemasukan</option>
        <option value="expense">Pengeluaran</option>
      </select>
      <button type="submit" class="w-full bg-indigo-600 text-white py-3 rounded-lg hover:bg-indigo-700 transition duration-300 flex items-center justify-center font-medium">
        <i class="fas fa-save mr-2"></i> Simpan Transaksi
      </button>
    </form>

    <div class="mb-6">
      <h2 class="text-xl font-semibold mb-4 text-gray-700 flex items-center">
        <i class="fas fa-history mr-2 text-indigo-600"></i>Riwayat Transaksi
      </h2>
      <div id="transactionsList" class="space-y-3"></div>
    </div>

    <div class="mt-6">
      <button id="clearHistory" class="w-full bg-gradient-to-r from-red-500 to-orange-500 text-white py-3 rounded-lg hover:from-red-600 hover:to-orange-600 transition duration-300 flex items-center justify-center font-medium">
        <i class="fas fa-trash-alt mr-2"></i> Hapus Semua Riwayat
      </button>
    </div>
  </div>

  <div class="bg-white w-full max-w-4xl rounded-xl shadow-lg p-6">
    <h2 class="text-xl font-semibold mb-6 text-center text-gray-700">
      <i class="fas fa-chart-line mr-2 text-indigo-600"></i>Laporan Keuangan
    </h2>
    <div class="flex justify-center space-x-4 mb-6">
      <button id="dailyTab" class="tab-btn active-tab"><i class="fas fa-calendar-day mr-2"></i>Harian</button>
      <button id="weeklyTab" class="tab-btn"><i class="fas fa-calendar-week mr-2"></i>Mingguan</button>
      <button id="monthlyTab" class="tab-btn"><i class="fas fa-calendar-alt mr-2"></i>Bulanan</button>
    </div>
    <div id="reportContent" class="text-center text-gray-500 p-4 bg-gray-50 rounded-lg">
      <i class="fas fa-info-circle text-2xl mb-2 text-indigo-400"></i>
      <p>Pilih tab untuk melihat laporan keuangan</p>
    </div>
  </div>

  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const scriptURL = "https://script.google.com/macros/s/AKfycbzdAlFyOsIFIdlp3apRRx_bqvj0XYu-ZkwYBOblaKr-QtRG2nHBFFV6sxlw4Vq0nwpf4g/exec";
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

      function showNotification(message, type = 'success') {
        notification.textContent = message;
        notification.className = `notification ${type} show`;
        setTimeout(() => notification.classList.remove('show'), 3000);
      }

      async function loadTransactions() {
        const res = await fetch(scriptURL);
        transactions = await res.json();
        renderTransactions();
        if (dailyTab.classList.contains('active-tab')) renderReport('daily');
        else if (weeklyTab.classList.contains('active-tab')) renderReport('weekly');
        else renderReport('monthly');
      }

      async function saveTransaction(t) {
        await fetch(scriptURL, { method: 'POST', body: JSON.stringify(t) });
        loadTransactions();
      }

      async function clearTransactions() {
        await fetch(scriptURL, { method: 'POST', body: JSON.stringify({ action: 'clear' }) });
        loadTransactions();
      }

      function updateBalance() {
        let income = 0, expense = 0;
        transactions.forEach(t => {
          if (t.type === 'income') income += Number(t.amount);
          else expense += Number(t.amount);
        });
        const bal = income - expense;
        balanceEl.textContent = `Rp ${bal.toLocaleString('id-ID')}`;
        balanceEl.className = bal >= 0 ? 'text-3xl font-bold mt-2 text-green-600' : 'text-3xl font-bold mt-2 text-red-600';
      }

      function renderTransactions() {
        transactionsList.innerHTML = transactions.length
          ? ''
          : '<p class="text-gray-500 text-center py-4"><i class="fas fa-receipt mr-2"></i>Tidak ada transaksi</p>';
        [...transactions].sort((a,b) => new Date(b.date) - new Date(a.date)).forEach(t => {
          const item = document.createElement('div');
          item.className = `transaction-item flex justify-between items-center p-4 border rounded-lg ${t.type === 'income' ? 'bg-green-50 border-green-200' : 'bg-red-50 border-red-200'}`;
          item.innerHTML = `
            <div class="flex items-center">
              <div class="mr-3 text-xl ${t.type==='income'?'text-green-500':'text-red-500'}">
                <i class="fas ${t.type==='income'?'fa-arrow-circle-down':'fa-arrow-circle-up'}"></i>
              </div>
              <div>
                <div class="font-medium text-gray-800">${t.description}</div>
                <div class="text-sm text-gray-500">${new Date(t.date).toLocaleDateString('id-ID', {weekday:'short', year:'numeric', month:'short', day:'numeric'})}</div>
              </div>
            </div>
            <div class="text-right">
              <div class="font-semibold ${t.type==='income'?'text-green-600':'text-red-600'}">${t.type==='income'?'+':'-'} Rp ${Number(t.amount).toLocaleString('id-ID')}</div>
              <div class="text-xs ${t.type==='income'?'text-green-500':'text-red-500'}">${t.type==='income'?'Pemasukan':'Pengeluaran'}</div>
            </div>`;
          transactionsList.appendChild(item);
        });
        updateBalance();
      }

      function renderReport(type) {
        const groups = {};
        transactions.forEach(t => {
          const date = new Date(t.date);
          let key;
          if (type === 'daily') key = date.toLocaleDateString('id-ID',{weekday:'long', year:'numeric', month:'long', day:'numeric'});
          else if (type === 'weekly') {
            const weekStart = new Date(date);
            weekStart.setDate(date.getDate() - date.getDay());
            key = `Minggu ${weekStart.toLocaleDateString('id-ID',{day:'numeric',month:'short'})}`;
          } else key = date.toLocaleDateString('id-ID',{year:'numeric',month:'long'});
          groups[key] = groups[key] || {income:0, expense:0};
          groups[key][t.type] += Number(t.amount);
        });

        if (!Object.keys(groups).length) {
          reportContent.innerHTML = `<div class="py-6"><i class="fas fa-file-invoice-dollar text-3xl mb-3 text-indigo-400"></i><p class="text-gray-500">Tidak ada data laporan</p></div>`;
          return;
        }

        let html = '<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-4">';
        Object.keys(groups).sort().reverse().forEach(key => {
          const bal = groups[key].income - groups[key].expense;
          html += `<div class="bg-white p-5 rounded-xl border border-gray-200 shadow-sm">
              <h3 class="font-semibold text-lg mb-3 text-gray-700">${key}</h3>
              <div class="space-y-2">
                <p class="text-green-600 justify-between flex"><span><i class="fas fa-arrow-down mr-2"></i>Pemasukan:</span><span>Rp ${groups[key].income.toLocaleString('id-ID')}</span></p>
                <p class="text-red-600 justify-between flex"><span><i class="fas fa-arrow-up mr-2"></i>Pengeluaran:</span><span>Rp ${groups[key].expense.toLocaleString('id-ID')}</span></p>
                <p class="font-semibold pt-3 border-t flex justify-between"><span><i class="fas fa-balance-scale mr-2"></i>Saldo:</span><span class="${bal>=0?'text-green-600':'text-red-600'}">Rp ${bal.toLocaleString('id-ID')}</span></p>
              </div>
            </div>`;
        });
        html += '</div>';
        reportContent.innerHTML = html;
      }

      form.addEventListener('submit', (e) => {
        e.preventDefault();
        const date = document.getElementById('date').value;
        const description = document.getElementById('description').value;
        const amount = document.getElementById('amount').value;
        const type = document.getElementById('type').value;
        if (!date || !description || !amount || amount <= 0) { showNotification('Isi semua field dengan benar!', 'error'); return; }
        saveTransaction({date, description, amount, type});
        form.reset();
        document.getElementById('date').valueAsDate = new Date();
        showNotification('Transaksi berhasil disimpan!');
      });

      clearBtn.addEventListener('click', () => {
        const pw = prompt('Masukkan sandi untuk menghapus riwayat:');
        if (pw === 'Nmax122333') { clearTransactions(); showNotification('Riwayat transaksi berhasil dihapus!'); }
        else showNotification('Sandi salah!', 'error');
      });

      document.getElementById('date').valueAsDate = new Date();
      loadTransactions();

      function switchTab(tab, type) {
        [dailyTab, weeklyTab, monthlyTab].forEach(t => t.classList.remove('active-tab'));
        tab.classList.add('active-tab'); renderReport(type);
      }
      dailyTab.addEventListener('click', () => switchTab(dailyTab,'daily'));
      weeklyTab.addEventListener('click', () => switchTab(weeklyTab,'weekly'));
      monthlyTab.addEventListener('click', () => switchTab(monthlyTab,'monthly'));
    });
  </script>
</body>
</html>
