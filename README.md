# Grade12 practices 
Flash cards website

<!DOCTYPE html>
<html lang="my">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Grade 12 Flashcards</title>
<style>
body{font-family:Arial,sans-serif;background:#f1f4ff;margin:0;color:#202b48}
header{background:#2848ad;color:white;padding:24px;text-align:center}
main{max-width:700px;margin:20px auto;padding:12px}
.box{background:white;padding:18px;border-radius:16px;margin-bottom:16px}
label{display:block;margin:12px 0 6px;font-weight:bold}
select,input,textarea{width:100%;box-sizing:border-box;padding:12px;border:1px solid #ccd5ed;border-radius:9px;font-size:16px}
button{padding:11px 14px;border:0;border-radius:9px;background:#2848ad;color:white;margin:4px;cursor:pointer}
.tabs{display:flex;flex-wrap:wrap;gap:5px}
.card{background:#edf1ff;border-radius:14px;padding:22px;text-align:center;min-height:150px}
#answer{display:none;color:#147b50;white-space:pre-wrap}
.hidden{display:none}
.option{display:block;width:100%;text-align:left;background:#edf1ff;color:#202b48}
.small{color:#68738b;font-size:13px}
</style>
</head>
<body>
<header>
<h1>📚 Grade 12 Flashcards</h1>
<p>Choose Subject • Choose Unit • Study • Quiz</p>
</header>
<main>

<div class="box">
<div class="tabs">
<button onclick="tab('study')">Flashcards</button>
<button onclick="tab('quiz')">Quiz</button>
<button onclick="tab('add')">Add Questions</button>
</div>

<label>Subject / ဘာသာရပ်</label>
<select id="subject" onchange="subjectChanged()">
<option>Myanmar</option>
<option>English</option>
<option>Chemistry</option>
<option>Biology</option>
</select>

<label id="chapterLabel">Category / အခန်း</label>
<select id="chapter" onchange="render()"></select>
</div>

<section id="study" class="box">
<p id="count"></p>
<div class="card" onclick="reveal()">
<h3 id="question">Question</h3>
<p id="answer">Answer</p>
<p class="small">ကတ်ကိုနှိပ်ပြီး အဖြေကြည့်ပါ</p>
</div>
<button onclick="move(-1)">← Previous</button>
<button onclick="reveal()">Show Answer</button>
<button onclick="move(1)">Next →</button>
</section>

<section id="quiz" class="box hidden">
<h2>Quiz</h2>
<p id="quizQuestion">Quiz စတင်ရန် နှိပ်ပါ။</p>
<div id="options"></div>
<p id="result"></p>
<button onclick="startQuiz()">Start Quiz</button>
<button id="next" class="hidden" onclick="nextQuiz()">Next</button>
</section>

<section id="add" class="box hidden">
<h2>မေးခွန်းအသစ်ထည့်ရန်</h2>

<label>Subject</label>
<select id="newSubject" onchange="updateNewChapters()">
<option>Myanmar</option>
<option>English</option>
<option>Chemistry</option>
<option>Biology</option>
</select>

<label id="newChapterLabel">Category</label>
<select id="newChapter"></select>

<label id="titleLabel">ခေါင်းစဉ်အမည် (လိုအပ်ပါက)</label>
<input id="title" placeholder="ဥပမာ - ကဗျာခေါင်းစဉ်">

<label>Question / မေးခွန်း</label>
<textarea id="newQ" placeholder="မေးခွန်းရေးပါ"></textarea>

<label>Answer / အဖြေ</label>
<textarea id="newA" placeholder="အဖြေရေးပါ"></textarea>

<button onclick="addCard()">Save Question</button>
<p id="message"></p>
</section>

<p class="small">Your personal Grade 12 study website</p>
</main>

<script>
const config={
 Myanmar:{
  label:"Category / အမျိုးအစား",
  items:["ကဗျာ","ဝတ္ထု","စာစီစာကုံး","သဒ္ဒါ","ဖတ်စာ","အခြားခေါင်းစဉ်"]
 },
 English:{
  label:"Unit",
  items:Array.from({length:12},(_,i)=>"Unit "+(i+1))
 },
 Chemistry:{
  label:"Chapter",
  items:Array.from({length:8},(_,i)=>"Chapter "+(i+1))
 },
 Biology:{
  label:"Chapter",
  items:Array.from({length:6},(_,i)=>"Chapter "+(i+1))
 }
};

const sample=[
 {subject:"English",chapter:"Unit 1",title:"Vocabulary",
  q:"What is a synonym?",a:"A word with the same or nearly the same meaning."},
 {subject:"Chemistry",chapter:"Chapter 1",title:"Chemistry",
  q:"What is the formula of baking soda?",a:"NaHCO₃"},
 {subject:"Biology",chapter:"Chapter 1",title:"Cell",
  q:"Which organelle produces most ATP?",a:"Mitochondria"}
];

let cards;
try{
 cards=JSON.parse(localStorage.getItem("grade12_v2"))||sample.slice();
}catch(e){cards=sample.slice()}

let index=0,quizCards=[],qi=0,score=0,answered=false;

const $=id=>document.getElementById(id);

function save(){
 try{localStorage.setItem("grade12_v2",JSON.stringify(cards))}
 catch(e){alert("Browser ထဲမှာ သိမ်းမရပါ။")}
}

function fillChapter(subject,selectId){
 const s=$(selectId);
 s.innerHTML="";
 config[subject].items.forEach(item=>{
  const o=document.createElement("option");
  o.value=item;o.textContent=item;s.appendChild(o);
 });
}

function subjectChanged(){
 fillChapter($("subject").value,"chapter");
 index=0;render();resetQuiz();
}

function updateNewChapters(){
 const s=$("newSubject").value;
 fillChapter(s,"newChapter");
 $("newChapterLabel").textContent=config[s].label;
 $("titleLabel").textContent=s==="Myanmar"
  ?"ခေါင်းစဉ်အမည်":"ခေါင်းစဉ်အမည် (လိုအပ်ပါက)";
}

function filtered(){
 const s=$("subject").value;
 const c=$("chapter").value;
 return cards.filter(x=>x.subject===s&&x.chapter===c);
}

function render(){
 const list=filtered();
 if(index>=list.length)index=0;
 $("count").textContent=list.length
  ?`မေးခွန်း ${index+1} / ${list.length}`:"ဒီအခန်းမှာ မေးခွန်းမရှိသေးပါ";
 $("question").textContent=list[index]
  ?((list[index].title?list[index].title+" — ":"")+list[index].q)
  :"မေးခွန်းအသစ် ထည့်ပါ";
 $("answer").textContent=list[index]?.a||"";
 $("answer").style.display="none";
}

function reveal(){
 const a=$("answer");
 a.style.display=a.style.display==="block"?"none":"block";
}

function move(n){
 const list=filtered();
 if(!list.length)return;
 index=(index+n+list.length)%list.length;
 render();
}

function tab(id){
 ["study","quiz","add"].forEach(x=>
  $(x).classList.toggle("hidden",x!==id));
}

function addCard(){
 const subject=$("newSubject").value;
 const chapter=$("newChapter").value;
 const title=$("title").value.trim();
 const q=$("newQ").value.trim();
 const a=$("newA").value.trim();

 if(!q||!a){
  $("message").textContent="မေးခွန်းနဲ့ အဖြေ ဖြည့်ပါ။";
  return;
 }

 cards.push({subject,chapter,title,q,a});
 save();
 $("message").textContent="မေးခွန်း သိမ်းပြီးပါပြီ။";
 $("newQ").value="";
 $("newA").value="";
 $("title").value="";

 $("subject").value=subject;
 subjectChanged();
 $("chapter").value=chapter;
 index=filtered().length-1;
 render();
}

function resetQuiz(){
 quizCards=[];qi=0;score=0;answered=false;
 $("quizQuestion").textContent="Quiz စတင်ရန် နှိပ်ပါ။";
 $("options").innerHTML="";
 $("result").textContent="";
 $("next").classList.add("hidden");
}

function startQuiz(){
 const list=filtered();
 if(!list.length){
  $("quizQuestion").textContent="ဒီအခန်းမှာ မေးခွန်းမရှိသေးပါ။";
  return;
 }
 quizCards=list.slice().sort(()=>Math.random()-.5);
 qi=0;score=0;showQuestion();
}

function showQuestion(){
 answered=false;
 const c=quizCards[qi];
 $("quizQuestion").textContent=
  `မေးခွန်း ${qi+1}/${quizCards.length}: ${c.q}`;
 $("options").innerHTML="";
 $("result").textContent="";
 $("next").classList.add("hidden");

 const wrong=cards.filter(x=>x.a!==c.a)
  .sort(()=>Math.random()-.5).slice(0,3).map(x=>x.a);
 const choices=[...new Set([c.a,...wrong])]
  .sort(()=>Math.random()-.5);

 choices.forEach(answer=>{
  const b=document.createElement("button");
  b.className="option";
  b.textContent=answer;
  b.onclick=()=>{
   if(answered)return;
   answered=true;
   if(answer===c.a)score++;
   $("result").textContent=
    (answer===c.a?"မှန်ပါတယ်!":"အဖြေမှန်: "+c.a)+
    ` | အမှတ် ${score}`;
   $("next").classList.remove("hidden");
   $("next").textContent=
    qi===quizCards.length-1?"ရလဒ်ကြည့်ရန်":"နောက်မေးခွန်း";
   [...$("options").children].forEach(x=>x.disabled=true);
  };
  $("options").appendChild(b);
 });
}

function nextQuiz(){
 if(!answered)return;
 if(qi<quizCards.length-1){qi++;showQuestion();}
 else{
  $("quizQuestion").textContent=
   `Quiz ပြီးပါပြီ။ အမှတ် ${score}/${quizCards.length}`;
  $("options").innerHTML="";
  $("next").classList.add("hidden");
 }
}

fillChapter("Myanmar","chapter");
fillChapter("Myanmar","newChapter");
updateNewChapters();
render();
</script>
</body>
</html>
