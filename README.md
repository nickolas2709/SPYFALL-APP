<!DOCTYPE html>
<html lang="el">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Κατάσκοπος: Ζώα, Χώρες & Φαγητά</title>
<meta name="description" content="Spyfall-τύπου party game στα ελληνικά με ζώα, χώρες & φαγητά.">
<style>
:root {
  --bg:#f4f4f4;--fg:#111;--card-bg:#fff;--card-fg:#111;
  --accent:#0077ff;--danger:#d40000;
  --btn-bg:#fff;--btn-fg:#111;--btn-border:#ccc;
}
.dark{
  --bg:#1b1b1b;--fg:#eee;--card-bg:#2b2b2b;--card-fg:#eee;
  --accent:#4da3ff;--danger:#ff4d4d;
  --btn-bg:#333;--btn-fg:#eee;--btn-border:#555;
}
html,body{margin:0;padding:0;}
body{
  font-family:Arial,sans-serif;
  text-align:center;
  padding:20px;
  background:var(--bg);
  color:var(--fg);
  transition:background .2s,color .2s;
}
h1{font-size:clamp(1.8rem,4vw,2.5rem);margin-top:0}
label,p{font-size:clamp(1rem,3vw,1.2rem)}
input,select{font-size:1.1rem;padding:6px 8px;margin:5px}
button{
  font-size:1rem;padding:10px 18px;margin:8px;
  border-radius:8px;border:2px solid var(--btn-border);
  background:var(--btn-bg);color:var(--btn-fg);
  cursor:pointer;transition:.15s;
  max-width:90%;
}
button:hover{transform:scale(1.03);background:var(--accent);color:#fff}
button.danger:hover{background:var(--danger)}
.section{max-width:600px;margin:0 auto;display:none}
.section.active{display:block}
.card{
  display:none;font-size:1.6rem;margin:20px auto;padding:20px;
  border-radius:10px;background:var(--card-bg);color:var(--card-fg);
  max-width:400px;box-shadow:0 0 10px rgba(0,0,0,.25);word-wrap:break-word;
}
#timer{font-size:1.3rem;font-weight:bold;margin-top:10px}
@media(max-width:480px){
  button{font-size:.95rem;padding:10px 14px}
  .card{font-size:1.4rem;padding:16px}
}
</style>
</head>
<body>
<!-- LANDING -->
<section id="landing" class="section active">
  <h1>🎭 Κατάσκοπος</h1>
  <p>Διάλεξε ρύθμιση & ξεκίνα γύρο.</p>
  <div>
    <label for="category">Κατηγορία:</label>
    <select id="category">
      <option value="animals" selected>Ζώα</option>
      <option value="countries">Χώρες</option>
      <option value="foods">Φαγητά</option>
    </select><br>
    <label for="players">Παίκτες:</label>
    <input type="number" id="players" min="3" max="20" value="5"><br>
    <label for="spies">Κατάσκοποι:</label>
    <select id="spies">
      <option value="1" selected>1</option>
      <option value="2">2</option>
      <option value="3">3</option>
    </select><br><br>
    <button onclick="goToGame()">Έναρξη</button>
    <button onclick="toggleDarkMode()">Dark Mode</button>
  </div>
</section>

<!-- GAME -->
<section id="game" class="section">
  <h2 id="playerInfo"></h2>
  <button id="revealBtn" onclick="revealRole()">Δείξε Ρόλο</button>
  <div class="card" id="card"></div>
  <button id="hideBtn" style="display:none" onclick="hideRole()">Κρύψε & Δώσε</button>
  <div id="afterRoles" style="display:none;margin-top:20px">
    <button onclick="startTimer()">Ξεκινήστε Χρονόμετρο</button>
    <button onclick="showSecret()">Αποκάλυψη Λέξης</button>
    <button onclick="showSpies()">Αποκάλυψη Κατασκόπων</button>
    <p id="timer"></p>
    <p id="revealSecret"></p>
    <p id="revealSpies"></p>
    <br>
    <button class="danger" onclick="endRound()">Τέλος Γύρου</button>
  </div>
</section>

<!-- ROUND SUMMARY -->
<section id="summary" class="section">
  <h2>Αποτελέσματα Γύρου</h2>
  <p id="sumSecret"></p>
  <p id="sumSpies"></p>
  <button onclick="restartFromSummary()">Επιστροφή στην Αρχική</button>
  <button onclick="playAgainSameSettings()">Νέος Γύρος με ίδιες ρυθμίσεις</button>
  <button onclick="toggleDarkMode()">Dark Mode</button>
</section>

<script>
/* ================== ΔΕΔΟΜΕΝΑ ================== */
/* Μεγαλύτερη λίστα ζώων >50 (μπορείς να επεκτείνεις εύκολα) */
const animals = [
  // Θηλαστικά
  "Λιοντάρι","Τίγρη","Ελέφαντας","Πάντα","Καμηλοπάρδαλη","Καγκουρό","Λύκος","Αλεπού",
  "Αρκούδα","Ιπποπόταμος","Ρινόκερος","Πίθηκος","Γορίλας","Χιμπατζής","Λεοπάρδαλη","Τσιτάχ",
  "Ζέβρα","Καμήλα","Βίσονας","Αγριόχοιρος","Ελάφι","Άλκη","Λαγός","Σκίουρος","Νυφίτσα",
  "Σκαντζόχοιρος","Νυχτερίδα","Δελφίνι (θηλαστικό νερού)","Φάλαινα","Φώκια","Θαλάσσιος Λέοντας",
  // Πουλιά
  "Πιγκουίνος","Κουκουβάγια","Παπαγάλος","Αετός","Γεράκι","Φλαμίνγκο","Πελεκάνος","Γλάρος",
  "Παγώνι","Πελαργός","Σπουργίτι","Κοράκι","Κάρδιναλιος (κοκκινοπούλι)",
  // Θαλάσσια & Άλλα υδρόβια
  "Καρχαρίας","Χελώνα","Χταπόδι","Καβουράς","Αστακός","Μέδουσα","Ιππόκαμπος","Σφουγγάρι",
  "Σαλάχι","Ξιφίας",
  // Ερπετά & Αμφίβια
  "Κροκόδειλος","Βάτραχος","Χαμαιλέοντας","Σαύρα","Οχιά","Πύθωνας","Ιγκουάνα","Σαλαμάνδρα",
  // Έντομα & Αρθρόποδα
  "Μέλισσα","Πεταλούδα","Αράχνη","Μυρμήγκι","Σφήκα","Σκαθάρι","Τζιτζίκι","Ακρίδα","Κατσαρίδα","Τζίτζικας",
  // Extra
  "Σκύλος","Γάτα","Πρόβατο","Κατσίκα","Αγελάδα","Άλογο","Γουρούνι","Κουνέλι"
];

/* Όλες οι ευρωπαϊκές χώρες (πολύ-κοντά στα κοινά ελληνικά ονόματα) */
const europeanCountries = [
"Αλβανία","Ανδόρα","Αυστρία","Λευκορωσία","Βέλγιο","Βοσνία και Ερζεγοβίνη","Βουλγαρία","Κροατία",
"Κύπρος","Τσεχία","Δανία","Εσθονία","Φινλανδία","Γαλλία","Γερμανία","Ελλάδα","Ουγγαρία","Ισλανδία",
"Ιρλανδία","Ιταλία","Λετονία","Λιχτενστάιν","Λιθουανία","Λουξεμβούργο","Μάλτα","Μολδαβία","Μονακό",
"Μαυροβούνιο","Ολλανδία","Βόρεια Μακεδονία","Νορβηγία","Πολωνία","Πορτογαλία","Ρουμανία","Ρωσία",
"Άγιος Μαρίνος","Σερβία","Σλοβακία","Σλοβενία","Ισπανία","Σουηδία","Ελβετία","Ουκρανία","Ηνωμένο Βασίλειο",
"Βατικανό","Κόσοβο" /* ανεπίσημο ευρέως παικτικά */
];

/* 10 αντιπροσωπευτικές χώρες από κάθε άλλη ήπειρο (σύνολο 50): */
const africa10 = ["Αίγυπτος","Νιγηρία","Αιθιοπία","Νότια Αφρική","Κένυα","Μαρόκο","Γκάνα","Αλγερία","Τανζανία","Αγκόλα"];
const asia10   = ["Κίνα","Ινδία","Ιαπωνία","Νότια Κορέα","Ινδονησία","Ταϊλάνδη","Φιλιππίνες","Σαουδική Αραβία","Ιράν","Μαλαισία"];
const americas10 = ["Ηνωμένες Πολιτείες","Καναδάς","Βραζιλία","Μεξικό","Αργεντινή","Χιλή","Κολομβία","Περού","Κούβα","Τζαμάικα"];
const oceania10  = ["Αυστραλία","Νέα Ζηλανδία","Παπούα Νέα Γουινέα","Φίτζι","Σαμόα","Τόνγκα","Κιριμπάτι","Ναουρού","Βανουάτου","Νησιά Σολομώντος"];
const antarctica10 = [
  /* Ανταρκτική δεν έχει χώρες· βάζουμε ερευνητικούς σταθμούς ως fun words */
  "Σταθμός ΜακΜάρντο","Σταθμός Αμούνδσεν-Σκοτ","Σταθμός Παλάις","Σταθμός Ροθερά","Σταθμός Ντιμόν ντ'Υρβίλ",
  "Σταθμός Μάουσον","Σταθμός Σόβα","Σταθμός Νόβολαζαρέφσκαγια","Σταθμός Εσπεράνσα","Σταθμός Κονκόρντια"
];

/* Τελική λίστα χωρών/τοποθεσιών που θα παίξουν όταν επιλεγεί κατηγορία Χώρες */
const countries = [
  ...europeanCountries,
  ...africa10,
  ...asia10,
  ...americas10,
  ...oceania10,
  ...antarctica10
];

/* Φαγητά: ελληνικά + διεθνή */
const foods = [
"Πίτσα","Μακαρόνια","Σουβλάκι","Χωριάτικη Σαλάτα","Μουσακάς","Γύρος","Παστίτσιο","Τυρόπιτα",
"Σπανακόπιτα","Μπιφτέκι","Χάμπουργκερ","Σάντουιτς","Κρέπα","Ριζότο","Κεμπάπ","Σούσι","Τάκο",
"Φαλάφελ","Παέγια","Καρι","Λουκουμάδες","Μπακλαβάς","Ντολμαδάκια","Γαριδομακαρονάδα","Σαλάτα Καίσαρα",
"Τάρτα Φρούτων","Τζελάτο","Τσιζκέικ","Παγωτό","Χούμους"
];

/* ================== ΚΑΤΑΣΤΑΣΗ ================== */
let roles = [];
let current = 0;
let selectedSecret = "";
let currentCategory = "";
let spyIndicesGlobal = [];
let timerInterval;
let timeLeft = 360;
let darkMode = false;
/* αποθήκευση τελευταίων ρυθμίσεων για «Νέος Γύρος με ίδιες ρυθμίσεις» */
let lastSettings = null;

/* ================== ΠΛΟΗΓΗΣΗ ΣΕΛΙΔΩΝ ================== */
function showSection(id){
  document.querySelectorAll('.section').forEach(sec=>sec.classList.remove('active'));
  document.getElementById(id).classList.add('active');
}
function goToGame(){
  startGameFromLanding();
  showSection('game');
}

/* ================== ΕΝΑΡΞΗ ΠΑΙΧΝΙΔΙΟΥ ================== */
function startGameFromLanding(){
  const numPlayers = parseInt(document.getElementById('players').value,10);
  const numSpies   = parseInt(document.getElementById('spies').value,10);
  currentCategory  = document.getElementById('category').value;

  if(numPlayers<3){alert("Τουλάχιστον 3 παίκτες!");return;}
  if(numSpies>=numPlayers){alert("Οι κατάσκοποι πρέπει να είναι λιγότεροι από τους παίκτες!");return;}

  const list = pickList(currentCategory);
  selectedSecret = list[Math.floor(Math.random()*list.length)];

  roles = Array(numPlayers).fill(selectedSecret);
  spyIndicesGlobal = pickUniqueRandomIndices(numSpies,numPlayers);
  spyIndicesGlobal.forEach(i=>roles[i]="Κατάσκοπος");

  current=0;
  clearInterval(timerInterval);
  document.getElementById('timer').textContent="";
  document.getElementById('revealSecret').textContent="";
  document.getElementById('revealSpies').textContent="";
  document.getElementById('afterRoles').style.display="none";
  document.getElementById('card').style.display="none";
  document.getElementById('hideBtn').style.display="none";
  document.getElementById('revealBtn').style.display="inline-block";
  document.getElementById('playerInfo').textContent="Παίκτης 1: Δείξε Ρόλο";

  // Αποθήκευση ρυθμίσεων
  lastSettings = {category:currentCategory,players:numPlayers,spies:numSpies};
}

/* Όταν ξεκινάμε νέο γύρο από σύνοψη με ίδιες ρυθμίσεις */
function playAgainSameSettings(){
  if(!lastSettings){showSection('landing');return;}
  // Επαναφέρει πεδία landing ώστε να συμβαδίζουν
  document.getElementById('category').value = lastSettings.category;
  document.getElementById('players').value  = lastSettings.players;
  document.getElementById('spies').value    = lastSettings.spies;
  goToGame();
}

/* ================== ΔΕΙΞΕ / ΚΡΥΨΕ ΡΟΛΟ ================== */
function revealRole(){
  const c=document.getElementById('card');
  c.style.display="block";
  c.textContent = roles[current]==="Κατάσκοπος" ? "Είσαι ΚΑΤΑΣΚΟΠΟΣ!" : label()+roles[current];
  document.getElementById('revealBtn').style.display="none";
  document.getElementById('hideBtn').style.display="inline-block";
}
function hideRole(){
  document.getElementById('card').style.display="none";
  document.getElementById('hideBtn').style.display="none";
  current++;
  if(current<roles.length){
    document.getElementById('playerInfo').textContent=`Παίκτης ${current+1}: Δείξε Ρόλο`;
    document.getElementById('revealBtn').style.display="inline-block";
  }else{
    document.getElementById('playerInfo').textContent="Όλοι έτοιμοι! Ρωτήστε, απαντήστε, βρείτε τον κατάσκοπο!";
    document.getElementById('afterRoles').style.display="block";
  }
}

/* ================== ΧΡΟΝΟΜΕΤΡΟ ================== */
function startTimer(){
  timeLeft=360;
  updateTimer();
  clearInterval(timerInterval);
  timerInterval=setInterval(()=>{
    timeLeft--;
    updateTimer();
    if(timeLeft<=0){
      clearInterval(timerInterval);
      document.getElementById('timer').textContent="Τέλος χρόνου!";
      autoAnnounce(); // αυτόματη ανακοίνωση στο τέλος χρόνου
    }
  },1000);
}
function updateTimer(){
  const m=Math.floor(timeLeft/60),s=timeLeft%60;
  document.getElementById('timer').textContent=`${m}:${s<10?"0":""}${s}`;
}

/* ================== ΑΠΟΚΑΛΥΨΕΙΣ ================== */
function label(){
  if(currentCategory==="animals") return "Ζώο: ";
  if(currentCategory==="countries") return "Χώρα/Τοποθεσία: ";
  return "Φαγητό: ";
}
function pickList(cat){
  if(cat==="animals") return animals;
  if(cat==="countries") return countries;
  return foods;
}
function showSecret(){
  document.getElementById('revealSecret').textContent = label()+selectedSecret;
  showSpies(); // μόλις δείχνουμε τη λέξη, δείχνουμε και κατασκόπους
}
function showSpies(){
  const names = spyIndicesGlobal.map(i=>`Παίκτης ${i+1}`).join(", ");
  document.getElementById('revealSpies').textContent = "Κατάσκοποι: "+names;
}
/* Αυτόματη ανακοίνωση στο τέλος χρόνου */
function autoAnnounce(){
  // Αν δεν έχει ήδη εμφανιστεί, εμφάνισε
  if(!document.getElementById('revealSecret').textContent){
    document.getElementById('revealSecret').textContent="(Χρόνος έληξε) "+label()+selectedSecret;
  }
  showSpies();
}

/* ================== ΤΕΛΟΣ ΓΥΡΟΥ & ΣΥΝΟΨΗ ================== */
function endRound(){
  // Εμφάνιση σύνοψης
  document.getElementById('sumSecret').textContent = label()+selectedSecret;
  const names = spyIndicesGlobal.map(i=>`Παίκτης ${i+1}`).join(", ");
  document.getElementById('sumSpies').textContent = "Κατάσκοποι: "+names;
  showSection('summary');
}
function restartFromSummary(){
  // Πίσω στην αρχική για νέες ρυθμίσεις
  showSection('landing');
}

/* ================== ΝΕΟ ΠΑΙΧΝΙΔΙ ΑΠΟ ΟΠΟΙΑΔΗΠΟΤΕ ΣΕΛΙΔΑ ================== */
function restartGame(fromButton=false){
  if(fromButton){
    const ok=confirm("Νέο παιχνίδι; Θα χαθεί ο τρέχων γύρος.");
    if(!ok)return;
  }
  clearInterval(timerInterval);
  showSection('landing');
}

/* ================== DARK MODE ================== */
function toggleDarkMode(){
  darkMode=!darkMode;
  document.body.classList.toggle('dark',darkMode);
}

/* ================== ΒΟΗΘΗΤΙΚΗ ================== */
function pickUniqueRandomIndices(count,max){
  const arr=[];
  while(arr.length<count){
    const i=Math.floor(Math.random()*max);
    if(!arr.includes(i))arr.push(i);
  }
  return arr;
}
</script>
</body>
</html>
