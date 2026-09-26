---
layout: default
title: Heat Pump — Unit Lookup
---

<style>
  /* Only page-specific tweaks — fonts/colors/header come from the site theme */
  table { width: 100%; border-collapse: collapse; margin-top: 12px; }
  td { padding: 8px 4px; border-bottom: 1px solid #eee; vertical-align: top; }
  td:first-child { font-weight: 600; width: 45%; }
  .warn { background: #fff3cd; padding: 10px; border-radius: 6px; margin-top: 12px; }
  .roomtag { display: inline-block; background: #159957; color: #fff; padding: 4px 10px; border-radius: 6px; font-size: 0.9em; }
  .id-form { display: flex; gap: 8px; margin-top: 10px; }
  .id-form input { flex: 1; padding: 8px 10px; border: 1px solid #ccc; border-radius: 6px; font-size: 1em; }
  .id-form button { padding: 8px 14px; border: none; border-radius: 6px; background: #159957; color: #fff; font-size: 1em; cursor: pointer; }
  .id-form button:hover { background: #10794a; }
</style>

<div id="hp-content">Loading unit data…</div>

<script>
const params = new URLSearchParams(window.location.search);
const id = params.get('id');
const dataUrl = '../data/heat-pump.csv'; // relative to /hvac/heat-pump.html

function parseCSV(text) {
  const lines = text.trim().split('\n');
  const headers = lines[0].split(',');
  return lines.slice(1).map(line => {
    const cells = line.split(',');
    const row = {};
    headers.forEach((h, i) => row[h] = cells[i] || '');
    return row;
  });
}

function idFormHtml(prefillId) {
  const value = prefillId ? ` value="${prefillId.replace(/"/g, '&quot;')}"` : '';
  return `
    <form class="id-form" id="hp-id-form">
      <input type="text" id="hp-id-input" placeholder="Enter unit ID (e.g. room number)"${value} autofocus>
      <button type="submit">Go</button>
    </form>`;
}

function wireIdForm() {
  const form = document.getElementById('hp-id-form');
  if (!form) return;
  form.addEventListener('submit', (e) => {
    e.preventDefault();
    const newId = document.getElementById('hp-id-input').value.trim();
    if (newId) {
      const url = new URL(window.location.href);
      url.searchParams.set('id', newId);
      window.location.href = url.toString();
    }
  });
}

function render(row) {
  const labelMap = {
    room_number: 'Room',
    equipment_type: 'Type',
    manufacturer: 'Manufacturer',
    model: 'Model',
    serial: 'Serial',
    install_date: 'Installed',
    installed_by: 'Installed By',
    voltage: 'Electrical Service',
    refrigerant_type: 'Refrigerant',
    refrigerant_charge_oz: 'Refrigerant Charge (oz)',
    compressor_rla: 'Compressor RLA',
    compressor_lra: 'Compressor LRA',
    blower_fla: 'Blower FLA',
    design_pressure_high: 'Design Pressure High (PSIG)',
    design_pressure_low: 'Design Pressure Low (PSIG)',
    last_serviced: 'Last Serviced',
    next_service_due: 'Next Service Due',
    notes: 'Notes'
  };

  let html = `<span class="roomtag">Room ${row.room_number}</span>`;
  html += '<table>';
  for (const key in labelMap) {
    const value = row[key] && row[key].trim() !== '' ? row[key] : '—';
    html += `<tr><td>${labelMap[key]}</td><td>${value}</td></tr>`;
  }
  html += '</table>';
  html += idFormHtml(row.id || '');
  document.getElementById('hp-content').innerHTML = html;
  wireIdForm();
}

if (!id) {
  document.getElementById('hp-content').innerHTML =
    '<div class="warn">No unit ID provided in the link. Scan the QR code on the unit itself, or enter it below.</div>' +
    idFormHtml();
  wireIdForm();
} else {
  fetch(dataUrl)
    .then(res => res.text())
    .then(text => {
      const rows = parseCSV(text);
      const match = rows.find(r => r.id === id);
      if (match) {
        render(match);
      } else {
        document.getElementById('hp-content').innerHTML =
          `<div class="warn">No record found for unit ID "${id}". Check the equipment sheet or contact Engineering, or try a different ID below.</div>` +
          idFormHtml(id);
        wireIdForm();
      }
    })
    .catch(() => {
      document.getElementById('hp-content').innerHTML =
        '<div class="warn">Could not load equipment data. Check your connection and try again.</div>' +
        idFormHtml(id);
      wireIdForm();
    });
}
</script>


### Troubleshooting

- Turn off the heat pump fuse.
- Reset the thermostat. Make sure you return settings to normal after.
- Check that the filter is clean. If not, replace.
- Clean the coils with chemical coil cleaner.
- Drain the condensate pump.
- Turn on the heat pump fuse.

Some heat pumps will take up to 10 minutes to turn back on. Check temperature at the vent with the thermal gun. If the unit is still not engaging, call Hakkon with the information provided above.