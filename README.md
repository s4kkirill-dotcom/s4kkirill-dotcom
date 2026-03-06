<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Side Quest Leaderboard · SQE</title>

<style>

*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Segoe UI,Tahoma,Geneva,Verdana,sans-serif;
}

body{
background:#121212;
color:#eee;
padding:30px;
background-image: radial-gradient(circle at 20% 20%, #1a1a1a 0%, #0f0f0f 80%);
}

.container{
max-width:1100px;
margin:auto;
}

header{
text-align:center;
margin-bottom:40px;
border-bottom:2px solid #ff3c00;
padding-bottom:20px;
}

h1{
color:#ff3c00;
font-size:2.8rem;
text-transform:uppercase;
letter-spacing:3px;
text-shadow:0 0 10px rgba(255,60,0,.5);
}

.subtitle{
color:#aaa;
margin-top:5px;
}

.card{
background:#1f1f1f;
padding:25px;
border-radius:10px;
margin-bottom:25px;
box-shadow:0 10px 25px rgba(0,0,0,.5);
}

/* Admin cards are hidden by default */
.card.admin-card {
    display: none;
}

h2{
color:#ff3c00;
margin-bottom:15px;
}

input,select{
padding:10px;
margin:6px;
border-radius:6px;
border:1px solid #444;
background:#222;
color:#fff;
}

button{
padding:10px 16px;
background:#ff3c00;
color:white;
border:none;
border-radius:6px;
cursor:pointer;
font-weight:bold;
transition:.2s;
}

button:hover{
background:#ff5a2d;
transform:translateY(-2px);
}

button.danger{
background:#6c1e1e;
}
button.danger:hover{
background:#8a2a2a;
}

.participants{
margin-top:10px;
display:grid;
grid-template-columns:repeat(auto-fill,minmax(150px,1fr));
}

.participants label{
margin:4px 0;
font-size:.95rem;
}

table{
width:100%;
border-collapse:collapse;
margin-top:15px;
background:#181818;
}

thead{
background:#ff3c00;
}

th,td{
padding:12px;
text-align:center;
}

tbody tr{
border-bottom:1px solid #333;
transition:.2s;
}

tbody tr:hover{
background:rgba(255,60,0,.1);
}

.rank1{
color:gold;
font-weight:bold;
}

.rank2{
color:silver;
font-weight:bold;
}

.rank3{
color:#cd7f32;
font-weight:bold;
}

.historyTable td{
font-size:.9rem;
}

.empty{
text-align:center;
color:#777;
padding:20px;
}

.admin-unlock-area {
    margin-top: 30px;
    padding-top: 20px;
    border-top: 1px solid #333;
    display: flex;
    gap: 10px;
    align-items: center;
    flex-wrap: wrap;
}

.admin-unlock-area input {
    flex: 1 1 250px;
}

.admin-unlock-area button {
    background: #444;
}
.admin-unlock-area button:hover {
    background: #666;
}

.admin-unlock-area .admin-active {
    background: #28a745;
}
.admin-unlock-area .admin-active:hover {
    background: #34ce57;
}

/* Manual adjust row */
.adjust-row {
    display: flex;
    gap: 10px;
    align-items: center;
    flex-wrap: wrap;
    background: #2a2a2a;
    padding: 15px;
    border-radius: 8px;
    margin-top: 20px;
}
.adjust-row select, .adjust-row input {
    min-width: 180px;
    flex: 1 1 auto;
}
</style>
</head>
<body>

<div class="container">

<header>
<h1>Side Quest Leaderboard</h1>
<p class="subtitle">Complete quests. Earn SQE. Climb the ranks.</p>
</header>

<!-- LEADERBOARD - always visible -->
<div class="card">
<h2>Leaderboard</h2>
<table>
<thead>
<tr>
<th>Rank</th>
<th>Member</th>
<th>Total SQE</th>
<th>Quests</th>
</tr>
</thead>
<tbody id="leaderboardBody"></tbody>
</table>
</div>

<!-- QUEST HISTORY - now always visible (non‑admin too) -->
<div class="card" id="historyCard">
<h2>Quest History</h2>
<table class="historyTable">
<thead>
<tr>
<th>Date</th>
<th>Quest</th>
<th>Participants</th>
<th>SQE</th>
<th id="adminHistHeader" style="display:none;">Actions</th>
</tr>
</thead>
<tbody id="historyBody"></tbody>
</table>
</div>

<!-- ADMIN CARDS (hidden initially) -->
<div class="card admin-card" id="addMemberCard">
<h2>Add Member</h2>
<input id="memberName" placeholder="Member Name">
<button onclick="addMember()">Add Member</button>
</div>

<div class="card admin-card" id="questLogCard">
<h2>Log Quest</h2>
<input id="questName" placeholder="Quest Name">
<br>
<label>Health Risk</label>
<input type="number" id="healthRisk" min="1" max="32">
<label>Consequence Risk</label>
<input type="number" id="consequenceRisk" min="1" max="32">
<br>
<label>Gear Required</label>
<input type="number" id="gearRequired" min="1" max="32">
<label>Social Exposure</label>
<input type="number" id="socialExposure" min="1" max="32">
<h3 style="margin-top:15px">Participants</h3>
<div class="participants" id="participants"></div>
<button style="margin-top:10px" onclick="addQuest()">Add Quest (SQE)</button>
</div>

<!-- NEW ADMIN CARD: manual SQE adjustment & quest deletion -->
<div class="card admin-card" id="adminToolsCard">
<h2>🔧 Admin Tools</h2>
<!-- Manual adjust section -->
<div class="adjust-row">
    <select id="adjustMemberSelect">
        <option value="">-- select member --</option>
    </select>
    <input type="number" id="adjustSQE" value="0" placeholder="SQE change">
    <button onclick="manualAdjustSQE()">Apply SQE change</button>
</div>
<!-- Quest deletion dropdown (only appears if quests exist) -->
<div class="adjust-row" id="deleteQuestRow">
    <select id="deleteQuestSelect">
        <option value="">-- select quest to delete --</option>
    </select>
    <button class="danger" onclick="deleteSelectedQuest()">Delete quest & revert SQE</button>
</div>
</div>

<!-- ADMIN UNLOCK BUTTON -->
<div class="admin-unlock-area">
    <input type="password" id="adminCode" placeholder="operator code" value="">
    <button id="adminToggleBtn" onclick="toggleAdminAccess()">Admin Panel</button>
</div>

</div>

<script>
let members = [];
let quests = [];

const OPERATOR_CODE = "ADMIN123";

// Helper: show/hide admin cards + history action column
function setAdminVisibility(visible) {
    const cards = document.querySelectorAll('.admin-card');
    cards.forEach(card => {
        card.style.display = visible ? 'block' : 'none';
    });
    
    // Show or hide the "Actions" header in quest history
    const adminHistHeader = document.getElementById('adminHistHeader');
    if (adminHistHeader) {
        adminHistHeader.style.display = visible ? 'table-cell' : 'none';
    }
    
    // Re-draw history so that action buttons appear/disappear
    displayHistory();
    
    const btn = document.getElementById('adminToggleBtn');
    if (visible) {
        btn.textContent = 'Disable Admin Panel';
        btn.classList.add('admin-active');
    } else {
        btn.textContent = 'Admin Panel';
        btn.classList.remove('admin-active');
    }
}

// Toggle admin access
function toggleAdminAccess() {
    const codeInput = document.getElementById('adminCode').value.trim();
    const isAdminVisible = document.querySelector('.admin-card')?.style.display !== 'none';
    if (isAdminVisible) {
        setAdminVisibility(false);
        document.getElementById('adminCode').value = '';
    } else {
        if (codeInput === OPERATOR_CODE) {
            setAdminVisibility(true);
            document.getElementById('adminCode').value = '';
            // populate admin dropdowns
            populateMemberDropdowns();
            populateQuestDropdown();
        } else {
            alert('Invalid operator code');
        }
    }
}

// Calculate SQE (formerly XP)
function calculateSQE(h, c, g, s, players) {
    let sqe = h + c + g + s;
    if (players.length >= 2) sqe += 10;
    return sqe;
}

function addMember() {
    let name = document.getElementById("memberName").value.trim();
    if (!name) return alert("Enter member name");
    if (members.find(m => m.name.toLowerCase() === name.toLowerCase()))
        return alert("Member already exists");
    members.push({ name, xp: 0, quests: 0 });  // xp field holds SQE
    document.getElementById("memberName").value = "";
    updateParticipants();
    rankMembers();
    displayMembers();
    populateMemberDropdowns();
    save();
}

function addQuest() {
    let quest = document.getElementById("questName").value;
    let h = parseInt(document.getElementById("healthRisk").value) || 0;
    let c = parseInt(document.getElementById("consequenceRisk").value) || 0;
    let g = parseInt(document.getElementById("gearRequired").value) || 0;
    let s = parseInt(document.getElementById("socialExposure").value) || 0;
    let players = [...document.querySelectorAll("#participants input:checked")].map(x => x.value);
    if (!quest || players.length === 0)
        return alert("Add quest name and participants");
    let sqe = calculateSQE(h, c, g, s, players);
    quests.unshift({
        name: quest,
        players: players,
        xp: sqe,            // store as SQE
        date: new Date().toLocaleDateString()
    });
    players.forEach(p => {
        let m = members.find(x => x.name === p);
        if (m) {
            m.xp += sqe;
            m.quests++;
        }
    });
    rankMembers();
    displayMembers();
    displayHistory();
    populateMemberDropdowns();
    populateQuestDropdown();
    save();
    // clear inputs
    document.getElementById("questName").value = "";
    document.querySelectorAll("#participants input:checked").forEach(cb => cb.checked = false);
}

// --- ADMIN: manual SQE adjustment ---
function manualAdjustSQE() {
    const select = document.getElementById('adjustMemberSelect');
    const memberName = select.value;
    const delta = parseInt(document.getElementById('adjustSQE').value);
    if (!memberName) return alert('Choose a member');
    if (isNaN(delta) || delta === 0) return alert('Enter non‑zero SQE change');
    
    const member = members.find(m => m.name === memberName);
    if (!member) return;
    
    member.xp = Math.max(0, member.xp + delta);  // never negative
    // We do not log this as a quest; just adjust.
    rankMembers();
    displayMembers();
    populateMemberDropdowns();
    save();
    document.getElementById('adjustSQE').value = 0;
}

// --- ADMIN: delete quest and revert SQE ---
function deleteSelectedQuest() {
    const select = document.getElementById('deleteQuestSelect');
    const questIndex = select.value;
    if (questIndex === '') return alert('Select a quest');
    
    const quest = quests[questIndex];
    if (!quest) return;
    
    if (!confirm(`Delete "${quest.name}" and subtract ${quest.xp} SQE from each participant?`)) return;
    
    // revert SQE from each participant
    quest.players.forEach(playerName => {
        const member = members.find(m => m.name === playerName);
        if (member) {
            member.xp = Math.max(0, member.xp - quest.xp);
            member.quests = Math.max(0, member.quests - 1);
        }
    });
    
    // remove quest from array
    quests.splice(questIndex, 1);
    
    rankMembers();
    displayMembers();
    displayHistory();
    populateMemberDropdowns();
    populateQuestDropdown();
    save();
}

// populate member dropdown (for manual adjust)
function populateMemberDropdowns() {
    const select = document.getElementById('adjustMemberSelect');
    if (!select) return;
    select.innerHTML = '<option value="">-- select member --</option>';
    members.forEach(m => {
        const opt = document.createElement('option');
        opt.value = m.name;
        opt.textContent = `${m.name} (${m.xp} SQE)`;
        select.appendChild(opt);
    });
}

// populate quest dropdown (for deletion)
function populateQuestDropdown() {
    const select = document.getElementById('deleteQuestSelect');
    if (!select) return;
    select.innerHTML = '<option value="">-- select quest to delete --</option>';
    quests.forEach((q, idx) => {
        const opt = document.createElement('option');
        opt.value = idx;
        opt.textContent = `${q.date} | ${q.name} (${q.xp} SQE)`;
        select.appendChild(opt);
    });
}

function updateParticipants() {
    let box = document.getElementById("participants");
    box.innerHTML = "";
    members.forEach(m => {
        let label = document.createElement("label");
        label.innerHTML = `<input type="checkbox" value="${m.name}"> ${m.name}`;
        box.appendChild(label);
    });
}

function rankMembers() {
    members.sort((a,b) => b.xp - a.xp);
    members.forEach((m,i) => m.rank = i+1);
}

function displayMembers() {
    let body = document.getElementById("leaderboardBody");
    if (members.length === 0) {
        body.innerHTML = `<tr><td colspan="4" class="empty">No members yet</td></tr>`;
        return;
    }
    body.innerHTML = "";
    members.forEach(m => {
        let rankClass = "";
        if (m.rank === 1) rankClass = "rank1";
        if (m.rank === 2) rankClass = "rank2";
        if (m.rank === 3) rankClass = "rank3";
        let row = document.createElement("tr");
        row.innerHTML = `
        <td class="${rankClass}">#${m.rank}</td>
        <td>${m.name}</td>
        <td>${m.xp}</td>
        <td>${m.quests}</td>
        `;
        body.appendChild(row);
    });
}

// History now includes optional delete buttons (admin only)
function displayHistory() {
    let body = document.getElementById("historyBody");
    const isAdmin = document.querySelector('.admin-card')?.style.display === 'block';
    
    if (quests.length === 0) {
        body.innerHTML = `<tr><td colspan="5" class="empty">No quests yet</td></tr>`;
        return;
    }
    body.innerHTML = "";
    quests.forEach((q, idx) => {
        let row = document.createElement("tr");
        let actionsCell = '';
        if (isAdmin) {
            actionsCell = `<td><button class="danger" onclick="deleteSpecificQuest(${idx})" style="padding:4px 8px;">🗑️</button></td>`;
        }
        row.innerHTML = `
        <td>${q.date}</td>
        <td>${q.name}</td>
        <td>${q.players.join(", ")}</td>
        <td>${q.xp}</td>
        ${actionsCell}
        `;
        body.appendChild(row);
    });
}

// Direct delete from history button (reuses same logic)
function deleteSpecificQuest(index) {
    const quest = quests[index];
    if (!quest) return;
    if (!confirm(`Delete "${quest.name}"? SQE will be subtracted from participants.`)) return;
    
    quest.players.forEach(playerName => {
        const member = members.find(m => m.name === playerName);
        if (member) {
            member.xp = Math.max(0, member.xp - quest.xp);
            member.quests = Math.max(0, member.quests - 1);
        }
    });
    
    quests.splice(index, 1);
    rankMembers();
    displayMembers();
    displayHistory();
    populateMemberDropdowns();
    populateQuestDropdown();
    save();
}

function save() {
    localStorage.setItem("sq_members", JSON.stringify(members));
    localStorage.setItem("sq_quests", JSON.stringify(quests));
}

function load() {
    let m = localStorage.getItem("sq_members");
    let q = localStorage.getItem("sq_quests");
    if (m) members = JSON.parse(m);
    if (q) quests = JSON.parse(q);
}

function init() {
    load();
    updateParticipants();
    rankMembers();
    displayMembers();
    displayHistory();
    populateMemberDropdowns();
    populateQuestDropdown();
    // start with admin hidden
    setAdminVisibility(false);
    document.getElementById('adminCode').value = '';
}

// Expose functions globally
window.toggleAdminAccess = toggleAdminAccess;
window.addMember = addMember;
window.addQuest = addQuest;
window.manualAdjustSQE = manualAdjustSQE;
window.deleteSelectedQuest = deleteSelectedQuest;
window.deleteSpecificQuest = deleteSpecificQuest;

document.addEventListener("DOMContentLoaded", init);
</script>

</body>
</html>
