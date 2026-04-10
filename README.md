<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Tourist Risks Reflection</title>

<style>
body{
    font-family: Arial;
    text-align:center;
    background:#f4f4f4;
    padding:20px;
}

h2{
    margin-bottom:20px;
}

table{
    width:90%;
    margin:auto;
    border-collapse: collapse;
    background:white;
}

th, td{
    border:1px solid #ccc;
    padding:10px;
}

.name{
    text-align:left;
    font-weight:bold;
}

.grid{
    display:flex;
    justify-content:center;
    flex-wrap:wrap;
    gap:4px;
}

.box{
    width:25px;
    height:25px;
    border:1px solid #999;
    background:white;
    cursor:pointer;
}

.red{ background:#ff4d4d; }
.yellow{ background:#ffd633; }
.green{ background:#4dff88; }

.result{
    margin-top:15px;
    font-size:18px;
    font-weight:bold;
}

button{
    margin-top:20px;
    padding:10px 20px;
    cursor:pointer;
    background:#333;
    color:white;
    border:none;
    border-radius:8px;
}
</style>
</head>

<body>

<h2>🌍 The Main Risks for Tourists – Reflection</h2>

<table>
<tr>
<th>Student</th>
<th>Score (1–10)</th>
</tr>

<tbody id="body"></tbody>
</table>

<div class="result" id="result"></div>

<button onclick="resetAll()">🔄 Reset</button>

<script>

const students = [
"Ботагөз",
"Жуманова Медина",
"Жасұлан Ансар",
"Ануар Ансар",
"Ануар",
"Мөлдір",
"Е. Медина",
"Жансулу",
"Акбота",
"Айлана",
"Мансур",
"Асанали"
];

let scores = Array(students.length).fill(0);

// 🎵 SOUND
function playSound(){
    let audio = new Audio("https://www.soundjay.com/buttons/sounds/button-16.mp3");
    audio.play();
}

// 🎨 COLOR RULE
function colorClass(n){
    if(n>=1 && n<=5) return "red";
    if(n>=6 && n<=7) return "yellow";
    if(n>=8 && n<=10) return "green";
    return "";
}

// 📊 RENDER TABLE
function render(){
    let body = document.getElementById("body");
    body.innerHTML = "";

    students.forEach((name, i)=>{
        let row = document.createElement("tr");

        let tdName = document.createElement("td");
        tdName.className = "name";
        tdName.innerHTML = name;

        let tdScore = document.createElement("td");

        let grid = document.createElement("div");
        grid.className = "grid";

        for(let j=1; j<=10; j++){
            let box = document.createElement("div");
            box.className = "box " + (scores[i]===j ? colorClass(j) : "");

            box.onclick = function(){
                scores[i] = j;
                playSound();
                render();
                showResult();
            };

            grid.appendChild(box);
        }

        tdScore.appendChild(grid);
        row.appendChild(tdName);
        row.appendChild(tdScore);
        body.appendChild(row);
    });
}

// 📈 RESULT
function showResult(){
    let sum = scores.reduce((a,b)=>a+b,0);
    let avg = (sum / scores.length).toFixed(1);
    document.getElementById("result").innerHTML =
        "📊 Average score: " + avg;
}

// 🔄 RESET
function resetAll(){
    scores = scores.map(()=>0);
    render();
    showResult();
}

render();
showResult();

</script>

</body>
</html>