<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Jens’ Tennis Log — Road to 3.0</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Playfair+Display:wght@600&display=swap" rel="stylesheet">
<style>
:root {
  --green:#347C4C;
  --gray:#F8F8F8;
}
body {
  font-family:'Inter',sans-serif;
  background:#fff;
  color:#1E2A3A;
  margin:0;
  padding:0;
}
header {
  text-align:center;
  font-family:'Playfair Display',serif;
  color:var(--green);
  padding:20px 0 10px;
  font-size:22px;
  border-bottom:1px solid #eee;
}
main {
  max-width:480px;
  margin:auto;
  padding:16px;
}
button {
  cursor:pointer;
  border:none;
  border-radius:8px;
  font-weight:500;
}
.card {
  background:#fff;
  border-radius:12px;
  box-shadow:0 1px 4px rgba(0,0,0,0.06);
  padding:16px;
  margin-bottom:16px;
}
h2 {
  color:var(--green);
  font-family:'Playfair Display',serif;
  font-size:18px;
  margin:0 0 12px;
}
input,select,textarea {
  width:100%;
  padding:8px 10px;
  border:1px solid #ddd;
  border-radius:8px;
  font-size:14px;
  margin-bottom:10px;
  font-family:'Inter',sans-serif;
}
#addSessionBtn {
  width:100%;
  padding:10px;
  background:var(--green);
  color:#fff;
  margin-bottom:12px;
}
#sessionForm {
  display:none;
}
#sessionForm.show {
  display:block;
  animation:fadeIn .3s ease;
}
@keyframes fadeIn {from{opacity:0;transform:translateY(-6px);}to{opacity:1;transform:translateY(0);}}
.stat-grid {
  display:grid;
  grid-template-columns:repeat(2,1fr);
  gap:8px;
}
.stat {
  background:var(--gray);
  border-radius:8px;
  text-align:center;
  padding:10px;
}
.stat strong {
  color:var(--green);
  display:block;
  font-size:16px;
}
.toast {
  visibility:hidden;
  min-width:200px;
  background-color:var(--green);
  color:#fff;
  text-align:center;
  border-radius:6px;
  padding:10px;
  position:fixed;
  z-index:10;
  left:50%;
  bottom:30px;
  transform:translateX(-50%);
  opacity:0;
  transition:opacity .3s,bottom .3s;
}
.toast.show {visibility:visible;opacity:1;bottom:50px;}
.session-card {
  border:1px solid #eee;
  border-radius:10px;
  padding:12px;
  margin-bottom:10px;
  font-size:14px;
}
.session-card strong {color:var(--green);}
.action-btns {float:right;}
.action-btns button {
  background:none;
  color:#777;
  font-size:16px;
  margin-left:8px;
}
.ellipsis {
  float:right;
  cursor:pointer;
  font-size:20px;
  color:var(--green);
}
.dropdown {
  display:none;
  position:absolute;
  right:16px;
  background:#fff;
  border:1px solid #ddd;
  border-radius:8px;
  box-shadow:0 2px 6px rgba(0,0,0,0.1);
}
.dropdown a {
  display:block;
  padding:8px 12px;
  color:#333;
  text-decoration:none;
  font-size:14px;
}
.dropdown a:hover {background:#f3f3f3;}
footer {
  text-align:center;
  font-size:12px;
  color:#999;
  padding:16px 0;
}
</style>
</head>
<body>
<header>🎾 Jens’ Tennis Log — Road to 3.0</header>
<main>
  <div class="card">
    <h2>Summary</h2>
    <div class="stat-grid" id="stats"></div>
  </div>

  <button id="addSessionBtn">+ New Session</button>
  <form id="sessionForm" class="card">
    <h2>Add Session</h2>
    <input type="date" id="date" required>
    <div style="display:flex;gap:6px;">
      <input type="time" id="start" required>
      <input type="time" id="end" required>
    </div>
    <input type="text" id="location" placeholder="Location">
    <input type="text" id="court" placeholder="Court Name">
    <input type="number" id="fee" placeholder="Court Fee (RM)">
    <select id="clockedBy"><option>Jens</option><option>Coach Leong</option></select>
    <textarea id="notes" placeholder="Notes / Remarks"></textarea>
    <button type="submit" style="background:var(--green);color:white;width:100%;padding:10px;">Save Session</button>
  </form>

  <div class="card" style="position:relative;">
    <h2>Session Log <span class="ellipsis" id="menuBtn">⋯</span></h2>
    <div class="dropdown" id="menuDropdown">
      <a href="#" id="exportPDF">Export as PDF</a>
      <a href="#" id="resetAll">Reset Data</a>
    </div>
    <div id="logs"></div>
  </div>
</main>

<div id="toast" class="toast">Saved ✅</div>
<footer>© 2025 Jens’ Tennis Log</footer>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script>
let sessions=JSON.parse(localStorage.getItem('sessions')||'[]');
const statsDiv=document.getElementById('stats'),logsDiv=document.getElementById('logs');
const toast=document.getElementById('toast');
function showToast(){toast.classList.add('show');setTimeout(()=>toast.classList.remove('show'),2000);}
function formatDate(d){if(!d)return'';return new Date(d).toLocaleDateString('en-GB',{day:'numeric',month:'short',year:'numeric'});}
function saveData(){localStorage.setItem('sessions',JSON.stringify(sessions));}
function totalLessons(){return 10;}
function totalFees(){return sessions.reduce((a,s)=>a+(parseFloat(s.fee)||0),0);}
function totalTime(){
 let total=0;sessions.forEach(s=>total+=parseInt(s.mins||0));
 const h=Math.floor(total/60),m=total%60;return(h?h+'h ':'')+(m?m+'m':'0m');
}
function updateStats(){
 const completed=sessions.length,remaining=Math.max(totalLessons()-completed,0);
 statsDiv.innerHTML=`
 <div class='stat'><strong>${totalLessons()}</strong>Total Lessons</div>
 <div class='stat'><strong>${completed}</strong>Completed</div>
 <div class='stat'><strong>${remaining}</strong>Remaining</div>
 <div class='stat'><strong>RM${totalFees().toFixed(2)}</strong>Total Fees</div>
 <div class='stat'><strong>${totalTime()}</strong>Total Time</div>`;
}
function renderLogs(){
 logsDiv.innerHTML=sessions.map((s,i)=>`
 <div class='session-card'>
 <div class='action-btns'>
 <button onclick='editSession(${i})'>✏️</button>
 <button onclick='deleteSession(${i})'>🗑️</button>
 </div>
 <strong>${formatDate(s.date)}</strong><br>
 ${s.start}–${s.end} (${s.duration})<br>
 ${s.court?'Court: '+s.court+'<br>':''}
 ${s.fee?'Fee: RM'+s.fee+'<br>':''}
 Logged by ${s.clockedBy}<br>
 ${s.notes||''}
 </div>`).join('')||"<p style='color:#999'>No sessions logged yet.</p>";
 updateStats();
}
document.getElementById('addSessionBtn').onclick=()=>{
 document.getElementById('sessionForm').classList.toggle('show');
}
document.getElementById('sessionForm').onsubmit=e=>{
 e.preventDefault();
 const f=e.target;
 const start=new Date('2000-01-01T'+f.start.value),end=new Date('2000-01-01T'+f.end.value);
 const mins=Math.round((end-start)/60000),h=Math.floor(mins/60),m=mins%60;
 const duration=(h?h+'h ':'')+(m?m+'m':'');
 sessions.push({date:f.date.value,start:f.start.value,end:f.end.value,mins:mins,court:f.court.value,fee:f.fee.value,notes:f.notes.value,clockedBy:f.clockedBy.value,duration});
 saveData();renderLogs();showToast();f.reset();
 document.getElementById('sessionForm').classList.remove('show');
};
function deleteSession(i){if(confirm("Delete this session?")){sessions.splice(i,1);saveData();renderLogs();}}
function editSession(i){
 const s=sessions[i];document.getElementById('sessionForm').classList.add('show');
 Object.entries(s).forEach(([k,v])=>{if(document.getElementById(k))document.getElementById(k).value=v;});
 sessions.splice(i,1);saveData();renderLogs();
}
const menuBtn=document.getElementById('menuBtn'),dropdown=document.getElementById('menuDropdown');
menuBtn.onclick=(e)=>{dropdown.style.display=dropdown.style.display==='block'?'none':'block';e.stopPropagation();}
window.onclick=(e)=>{if(e.target!=menuBtn)dropdown.style.display='none';}
document.getElementById('resetAll').onclick=()=>{if(confirm("Clear all data?")){sessions=[];saveData();renderLogs();}}
document.getElementById('exportPDF').onclick=()=>{
 const{jsPDF}=window.jspdf;const doc=new jsPDF();
 doc.setFont('helvetica','bold');doc.text("Jens’ Tennis Log — Road to 3.0",10,10);
 doc.setFont('helvetica','normal');let y=20;
 sessions.forEach((s,i)=>{doc.text(`${i+1}. ${formatDate(s.date)} ${s.start}–${s.end} (${s.duration})`,10,y);
 y+=6;if(s.court){doc.text(`Court: ${s.court}`,10,y);y+=6;}
 if(s.fee){doc.text(`Fee: RM${Number(s.fee).toFixed(2)}`,10,y);y+=6;}
 if(s.notes){doc.text(`Notes: ${s.notes}`,10,y);y+=6;}
 if(y>280){doc.addPage();y=10;}
 });
 doc.save("Jens_Tennis_Log.pdf");
};
updateStats();renderLogs();
</script>
</body>
</html>
