<!doctype html>
};
state.push(entry); saveState(); entryForm.reset(); render();
}


function render(){
const month = monthInput.value; // YYYY-MM
entriesTable.innerHTML='';
const monthEntries = state.filter(e=> e.date.slice(0,7) === month);
monthEntries.sort((a,b)=> new Date(b.date)-new Date(a.date));
monthEntries.forEach(e=>{
const tr=document.createElement('tr');
tr.innerHTML = `<td>${e.amount.toFixed(2)} €</td><td>${e.category}</td><td>${e.date}</td><td>${escapeHtml(e.desc)}</td><td class="actions"><button data-id="${e.id}" class="edit">Bearb.</button><button data-id="${e.id}" class="del">Löschen</button></td>`;
entriesTable.appendChild(tr);
});
document.querySelectorAll('.edit').forEach(b=>b.addEventListener('click',e=>startEdit(e.target.dataset.id)));
document.querySelectorAll('.del').forEach(b=>b.addEventListener('click',e=>{ if(confirm('Eintrag löschen?')) deleteEntry(e.target.dataset.id);}));


const sum = monthEntries.reduce((s,x)=> s + Number(x.amount), 0);
sumEl.textContent = sum.toFixed(2) + ' €';
countEl.textContent = monthEntries.length;
avgEl.textContent = (monthEntries.length? (sum/monthEntries.length).toFixed(2) : '0.00') + ' €';
}


function startEdit(id){
const e = state.find(x=>String(x.id)===String(id));
if(!e) return;
editId = e.id;
amountInput.value = e.amount; categoryInput.value = e.category; dateInput.value = e.date; descInput.value = e.desc;
updateBtn.style.display='inline-block';
}


function saveEdit(){
const idx = state.findIndex(x=>x.id===editId);
if(idx===-1) return;
state[idx].amount = Math.round(parseFloat(amountInput.value)*100)/100;
state[idx].category = categoryInput.value;
state[idx].date = dateInput.value;
state[idx].desc = descInput.value;
editId = null; updateBtn.style.display='none'; entryForm.reset(); saveState(); render();
}


function deleteEntry(id){ state = state.filter(x=>String(x.id)!==String(id)); saveState(); render(); }


function exportCSV(){
const month = monthInput.value;
const rows = state.filter(e=> e.date.slice(0,7)===month);
if(rows.length===0) return alert('Keine Einträge für den gewählten Monat');
let csv = 'id,amount,category,date,desc\n';
rows.forEach(r=>{ csv += `${r.id},${r.amount},"${r.category}",${r.date},"${(r.desc||'').replace(/\"/g,'""')}"\n`; });
const blob = new Blob([csv], {type:'text/csv'});
const url = URL.createObjectURL(blob);
const a = document.createElement('a'); a.href=url; a.download = `kosten_${month}.csv`; a.click(); URL.revokeObjectURL(url);
}


function handleFile(e){
const f = e.target.files[0]; if(!f) return; const reader = new FileReader();
reader.onload = ()=>{
const text = reader.result; parseCSVImport(text);
fileInput.value='';
};
reader.readAsText(f,'utf-8');
}


function parseCSVImport(text){
// sehr einfacher CSV-Parser (erwartet id,amount,category,date,desc)
const lines = text.split(/\r?\n/).map(l=>l.trim()).filter(l=>l);
if(lines.length<2) return alert('Keine Daten gefunden');
const header = lines.shift().split(',').map(h=>h.toLowerCase());
const need = ['id','amount','category','date','desc'];
if(!need.every(n=> header.includes(n))) return alert('Unbekanntes CSV-Format');
lines.forEach(line=>{
// simple split by comma, handles quotes roughly
const parts = csvSplit(line);
const obj = { id: Number(parts[0]) || Date.now()+Math.random(), amount: parseFloat(parts[1]) || 0, category: parts[2] || 'Sonstiges', date: parts[3] || new Date().toISOString().slice(0,10), desc: parts[4] || '' };
// avoid duplicates by id
if(!state.some(s=>String(s.id)===String(obj.id))) state.push(obj);
});
saveState(); render(); alert('Import fertig');
}


function csvSplit(line){
const out=[]; let cur=''; let inQ=false;
for(let i=0;i<line.length;i++){
const ch=line[i];
if(ch==='"'){ if(inQ && line[i+1]==='"'){ cur+='"'; i++; } else inQ=!inQ; continue; }
if(ch===',' && !inQ){ out.push(cur); cur=''; continue; }
cur+=ch;
}
out.push(cur); return out.map(s=>s.replace(/^\s+|\s+$/g,''));
}


function escapeHtml(s){ return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
</script>
</body>
</html>
