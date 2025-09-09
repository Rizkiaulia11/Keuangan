// Event listener untuk form
            form.addEventListener('submit', (e) => {
                e.preventDefault();
                
                const date = document.getElementById('date').value;
                const description = document.getElementById('description').value;
                const amount = parseFloat(document.getElementById('amount').value);
                const type = document.getElementById('type').value;
                
                // Validasi input
                if (!date  !description  !amount  amount <= 0) {
                    showNotification('Harap isi semua field dengan benar!', 'error');
                    return;
                }
                
                transactions.push({ date, description, amount, type });
                saveTransactions();
                renderTransactions();
                
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
            });

            // Event listener untuk tombol hapus
            clearBtn.addEventListener('click', () => {
                const password = prompt("Masukkan sandi untuk menghapus riwayat:");
                if (password === "Nmax122333") {
                    transactions = [];
                    saveTransactions();
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
            });

            // Inisialisasi aplikasi
            renderTransactions();
            
            // Set tanggal hari ini sebagai nilai default
            const today = new Date().toISOString().split('T')[0];
            document.getElementById('date').value = today;
            
            // Tampilkan laporan harian secara default
            renderReport('daily');
        });
    </script>
</body>
</html>
