<!DOCTYPE html>
<html lang="ms">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Stock Telephone PABX</title>
  <style>
    body { font-family: Arial, sans-serif; background:#f5f6fa; margin:0; padding:20px; }
    h1 { margin-bottom:10px; }
    .card { background:#fff; padding:16px; border-radius:10px; box-shadow:0 4px 10px rgba(0,0,0,0.05); margin-bottom:16px; }
    .grid { display:grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap:10px; }
    input, select { padding:8px; border-radius:6px; border:1px solid #ccc; width:100%; }
    button { padding:10px 14px; border:none; border-radius:8px; cursor:pointer; background:#2563eb; color:white; }
    button.danger { background:#dc2626; }
    table { width:100%; border-collapse:collapse; margin-top:10px; }
    th, td { border:1px solid #ddd; padding:8px; text-align:left; font-size:14px; }
    th { background:#f1f5f9; }
  </style>
</head>
<body>
  <h1>📞 Stock Telephone PABX</h1>

  <div class="card">
    <h3>Tambah Rekod</h3>
    <div class="grid">
      <select id="jenisTelefon">
        <option value="">-- Jenis Telefon --</option>
        <option value="Analog">Analog</option>
        <option value="Digital">Digital</option>
      </select>

      <select id="jenis">
        <option value="">-- Jenis --</option>
        <option value="Masuk">Telefon Masuk</option>
        <option value="Keluar">Telefon Keluar</option>
      </select>

      <input type="date" id="tarikh" />
      <input type="number" id="kuantiti" placeholder="Kuantiti" />
      <input type="text" id="catatan" placeholder="Catatan" />
    </div>
    <br />
    <button onclick="tambahRekod()">Simpan</button>
  </div>

  <div class="card">
    <h3>Senarai Rekod</h3>
    <table id="table">
      <thead>
        <tr>
          <th>Tarikh</th>
          <th>Jenis Telefon</th>
          <th>Masuk/Keluar</th>
          <th>Kuantiti</th>
          <th>Catatan</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <div class="card">
    <h3>Ringkasan Stok</h3>
    <button onclick="showSummary()">📊 Lihat Summary Report</button>
    <button onclick="exportExcel()">⬇️ Export Summary (Excel)</button>
    <table>
      <tr><th></th><th>Masuk</th><th>Keluar</th><th>Baki</th></tr>
      <tr><td><b>Analog</b></td><td id="masukAnalog">0</td><td id="keluarAnalog">0</td><td id="bakiAnalog">0</td></tr>
      <tr><td><b>Digital</b></td><td id="masukDigital">0</td><td id="keluarDigital">0</td><td id="bakiDigital">0</td></tr>
      <tr><td><b>Jumlah</b></td><td id="masukTotal">0</td><td id="keluarTotal">0</td><td id="bakiTotal">0</td></tr>
    </table>
  </div>

<script>
  let data = JSON.parse(localStorage.getItem('stokTelefon') || '[]');

  function render() {
    const tbody = document.querySelector('#table tbody');
    tbody.innerHTML = '';

    let masukAnalog = 0, keluarAnalog = 0;
    let masukDigital = 0, keluarDigital = 0;

    data.forEach((row, index) => {
      if (row.jenis === 'Masuk' && row.jenisTelefon === 'Analog') masukAnalog += Number(row.kuantiti || 0);
      if (row.jenis === 'Keluar' && row.jenisTelefon === 'Analog') keluarAnalog += Number(row.kuantiti || 0);
      if (row.jenis === 'Masuk' && row.jenisTelefon === 'Digital') masukDigital += Number(row.kuantiti || 0);
      if (row.jenis === 'Keluar' && row.jenisTelefon === 'Digital') keluarDigital += Number(row.kuantiti || 0);

      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${row.tarikh}</td>
        <td>${row.jenisTelefon}</td>
        <td>${row.jenis}</td>
        <td>${row.kuantiti}</td>
        <td>${row.catatan || ''}</td>
        <td><button class="danger" onclick="padam(${index})">Padam</button></td>
      `;
      tbody.appendChild(tr);
    });

    const masukTotal = masukAnalog + masukDigital;
    const keluarTotal = keluarAnalog + keluarDigital;

    document.getElementById('masukAnalog').innerText = masukAnalog;
    document.getElementById('keluarAnalog').innerText = keluarAnalog;
    document.getElementById('bakiAnalog').innerText = masukAnalog - keluarAnalog;

    document.getElementById('masukDigital').innerText = masukDigital;
    document.getElementById('keluarDigital').innerText = keluarDigital;
    document.getElementById('bakiDigital').innerText = masukDigital - keluarDigital;

    document.getElementById('masukTotal').innerText = masukTotal;
    document.getElementById('keluarTotal').innerText = keluarTotal;
    document.getElementById('bakiTotal').innerText = masukTotal - keluarTotal;
  }

  function tambahRekod() {
    const tarikh = document.getElementById('tarikh').value;
    const jenis = document.getElementById('jenis').value;
    const jenisTelefon = document.getElementById('jenisTelefon').value;
    const kuantiti = document.getElementById('kuantiti').value;
    const catatan = document.getElementById('catatan').value;

    if (!tarikh || !jenis || !jenisTelefon || !kuantiti) {
      alert('Sila lengkapkan: Tarikh, Jenis Telefon, Jenis (Masuk/Keluar) dan Kuantiti');
      return;
    }

    data.push({ tarikh, jenis, jenisTelefon, kuantiti, catatan });
    localStorage.setItem('stokTelefon', JSON.stringify(data));

    document.getElementById('tarikh').value = '';
    document.getElementById('jenis').value = '';
    document.getElementById('jenisTelefon').value = '';
    document.getElementById('kuantiti').value = '';
    document.getElementById('catatan').value = '';

    render();
  }

  function padam(index) {
    if (confirm('Padam rekod ini?')) {
      data.splice(index, 1);
      localStorage.setItem('stokTelefon', JSON.stringify(data));
      render();
    }
  }

  function showSummary() {
    let masukAnalog = 0, keluarAnalog = 0;
    let masukDigital = 0, keluarDigital = 0;

    data.forEach(row => {
      if (row.jenis === 'Masuk' && row.jenisTelefon === 'Analog') masukAnalog += Number(row.kuantiti || 0);
      if (row.jenis === 'Keluar' && row.jenisTelefon === 'Analog') keluarAnalog += Number(row.kuantiti || 0);
      if (row.jenis === 'Masuk' && row.jenisTelefon === 'Digital') masukDigital += Number(row.kuantiti || 0);
      if (row.jenis === 'Keluar' && row.jenisTelefon === 'Digital') keluarDigital += Number(row.kuantiti || 0);
    });

    alert(
      'SUMMARY REPORT STOCK TELEPHONE PABX\n\n' +
      'ANALOG:\n' +
      '  Masuk : ' + masukAnalog + '\n' +
      '  Keluar: ' + keluarAnalog + '\n' +
      '  Baki  : ' + (masukAnalog - keluarAnalog) + '\n\n' +
      'DIGITAL:\n' +
      '  Masuk : ' + masukDigital + '\n' +
      '  Keluar: ' + keluarDigital + '\n' +
      '  Baki  : ' + (masukDigital - keluarDigital) + '\n\n' +
      'JUMLAH:\n' +
      '  Masuk : ' + (masukAnalog + masukDigital) + '\n' +
      '  Keluar: ' + (keluarAnalog + keluarDigital) + '\n' +
      '  Baki  : ' + ((masukAnalog + masukDigital) - (keluarAnalog + keluarDigital))
    );
  }

  function exportExcel() {
    let masukAnalog = 0, keluarAnalog = 0;
    let masukDigital = 0, keluarDigital = 0;

    data.forEach(row => {
      if (row.jenis === 'Masuk' && row.jenisTelefon === 'Analog') masukAnalog += Number(row.kuantiti || 0);
      if (row.jenis === 'Keluar' && row.jenisTelefon === 'Analog') keluarAnalog += Number(row.kuantiti || 0);
      if (row.jenis === 'Masuk' && row.jenisTelefon === 'Digital') masukDigital += Number(row.kuantiti || 0);
      if (row.jenis === 'Keluar' && row.jenisTelefon === 'Digital') keluarDigital += Number(row.kuantiti || 0);
    });

    const bakiAnalog = masukAnalog - keluarAnalog;
    const bakiDigital = masukDigital - keluarDigital;
    const masukTotal = masukAnalog + masukDigital;
    const keluarTotal = keluarAnalog + keluarDigital;
    const bakiTotal = bakiAnalog + bakiDigital;

    // Create HTML table (Excel-friendly)
    let html = `
      <table border="1" style="border-collapse:collapse; font-family:Arial;">
        <tr style="background:#f1f5f9; font-weight:bold;">
          <th>Category</th><th>Masuk</th><th>Keluar</th><th>Baki</th>
        </tr>
        <tr><td>Analog</td><td>${masukAnalog}</td><td>${keluarAnalog}</td><td>${bakiAnalog}</td></tr>
        <tr><td>Digital</td><td>${masukDigital}</td><td>${keluarDigital}</td><td>${bakiDigital}</td></tr>
        <tr style="font-weight:bold;"><td>Jumlah</td><td>${masukTotal}</td><td>${keluarTotal}</td><td>${bakiTotal}</td></tr>
      </table>
    `;

    const blob = new Blob([html], { type: 'application/vnd.ms-excel;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = 'summary_stock_telephone_pabx_table.xls';
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }

  render();
</script>
</body>
</html>
