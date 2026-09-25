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
  document.getElementById('hp-content').innerHTML = html;
}

if (!id) {
  document.getElementById('hp-content').innerHTML =
    '<div class="warn">No unit ID provided in the link. Scan the QR code on the unit itself, or add <code>?id=</code> to the URL.</div>';
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
          `<div class="warn">No record found for unit ID "${id}". Check the equipment sheet or contact Engineering.</div>`;
      }
    })
    .catch(() => {
      document.getElementById('hp-content').innerHTML =
        '<div class="warn">Could not load equipment data. Check your connection and try again.</div>';
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