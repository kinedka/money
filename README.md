<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>班表小幫手</title>
<style>
body {
  font-family: Arial, sans-serif;
  background: #f0f4f9;
  padding: 5px;
  margin: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* 日曆容器 */
.calendar {
  width: 95%;
  max-width: 700px;
  margin: auto;
  background: #fff;
  border-radius: 10px;
  padding: 5px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
}

/* 標題列 */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 5px;
}
.header button {
  background: #4a90e2;
  border: none;
  color: #fff;
  padding: 6px 12px;
  border-radius: 5px;
  font-size: clamp(12px, 2.5vw, 16px);
}
.header h2 {
  margin: 0;
  font-size: clamp(14px, 3vw, 18px);
}

/* 表格 */
table {
  width: 100%;
  border-collapse: collapse;
  table-layout: fixed; /* 自動等分 */
}
th {
  background: #d9e6f2;
  padding: 6px;
  font-size: clamp(12px, 2.5vw, 16px);
}
td {
  height: 80px;
  border: 1px solid #ccc;
  vertical-align: top;
  position: relative;
  text-align: center;
  overflow: hidden;
}
td span {
  display: block;
  font-weight: bold;
  font-size: clamp(12px, 2.5vw, 16px);
  line-height: 1.1;
}
td .status {
  margin-top: 2px;
  font-size: clamp(10px, 2.5vw, 14px);
  line-height: 1.1;
  white-space: pre;
  overflow: hidden;
  text-overflow: ellipsis;
}
select {
  font-size: clamp(10px, 2.5vw, 14px);
  width: 90%;
  margin-top: 2px;
}

/* 狀態顏色 */
.work {background:#d0f0f0;}
.leave{background:#ffd6d6;}
.comp {background:#fff6c4;}
.stay {background:#d6e7ff;}
.stop {background:#e6d6ff;}

/* 統計 */
.stats {
  width: 95%;
  max-width: 700px;
  margin: 10px auto;
  background: #fff;
  border-radius: 10px;
  padding: 10px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
  font-size: clamp(14px, 3vw, 18px);
}
.stats h3 {
  margin: 0 0 5px 0;
  font-size: clamp(16px, 3.5vw, 20px);
}
</style>
</head>
<body>

<div class="calendar">
  <div class="header">
    <button onclick="prevMonth()">←</button>
    <h2 id="monthYear"></h2>
    <button onclick="nextMonth()">→</button>
  </div>
  <table>
    <thead>
      <tr><th>日</th><th>一</th><th>二</th><th>三</th><th>四</th><th>五</th><th>六</th></tr>
    </thead>
    <tbody id="calendarBody"></tbody>
  </table>
</div>

<div class="stats">
  <h3>統計</h3>
  <p id="workDays">上班:0天</p>
  <p id="leaveDays">休假:0天</p>
  <p id="compDays">補休:0天</p>
  <p id="stayDays">外宿:0天</p>
  <p id="stopHours">停休:0小時</p>
</div>

<script>
const statusOptions=["上班","補休","休假","外宿","停休"];
let currentDate=new Date();

function renderCalendar(date){
  const y=date.getFullYear(), m=date.getMonth();
  document.getElementById("monthYear").innerText=`${y}年 ${m+1}月`;
  const firstDay=new Date(y,m,1).getDay(), days=new Date(y,m+1,0).getDate();
  const body=document.getElementById("calendarBody"); body.innerHTML="";
  let row=document.createElement("tr");
  for(let i=0;i<firstDay;i++) row.appendChild(document.createElement("td"));
  for(let d=1;d<=days;d++){
    if(row.children.length==7){ body.appendChild(row); row=document.createElement("tr"); }
    const cell=document.createElement("td");
    const daySpan=document.createElement("span"); daySpan.innerText=d;
    const status=document.createElement("div"); status.className="status"; status.innerText="上班"; cell.className="work";
    const select=document.createElement("select");
    function fillOpts(sel){
      sel.innerHTML="";
      statusOptions.forEach(o=>{let opt=document.createElement("option"); opt.value=o; opt.text=o; if(status.innerText.includes(o)) opt.selected=true; sel.appendChild(opt);});
    }
    fillOpts(select);

    select.onchange=function(){
      if(select.value==="停休"){
        select.innerHTML=""; select.appendChild(new Option("取消","取消"));
        for(let i=1;i<=10;i++) select.appendChild(new Option(i,i)); select.selectedIndex=-1;
      } else if(parseInt(select.value)>=1 && parseInt(select.value)<=10){
        status.innerText=`停休\n${select.value}小時`;
        cell.className="stop";
      } else if(select.value==="取消"){ fillOpts(select); status.innerText="上班"; cell.className="work"; }
      else{ status.innerText=select.value; cell.className=""; if(select.value==="上班") cell.classList.add("work"); else if(select.value==="休假") cell.classList.add("leave"); else if(select.value==="補休") cell.classList.add("comp"); else if(select.value==="外宿") cell.classList.add("stay"); }
      updateStats();
    };
    cell.appendChild(daySpan); cell.appendChild(status); cell.appendChild(select); row.appendChild(cell);
  }
  while(row.children.length<7) row.appendChild(document.createElement("td")); body.appendChild(row);
  updateStats();
}

function updateStats(){
  let w=0,l=0,c=0,s=0,st=0;
  document.querySelectorAll("#calendarBody td").forEach(cell=>{
    const sDiv=cell.querySelector(".status"); if(!sDiv) return;
    if(sDiv.innerText.includes("上班")) w++;
    if(sDiv.innerText.includes("休假")&&!sDiv.innerText.includes("停休")) l++;
    if(sDiv.innerText.includes("補休")) c++;
    if(sDiv.innerText.includes("外宿")) s++;
    if(sDiv.innerText.includes("停休")) st+=parseInt(sDiv.innerText.replace(/[^0-9]/g,""))||0;
  });
  document.getElementById("workDays").innerText=`上班:${w}天`;
  document.getElementById("leaveDays").innerText=`休假:${l}天`;
  document.getElementById("compDays").innerText=`補休:${c}天`;
  document.getElementById("stayDays").innerText=`外宿:${s}天`;
  document.getElementById("stopHours").innerText=`停休:${st}小時`;
}

function prevMonth(){ currentDate.setMonth(currentDate.getMonth()-1); renderCalendar(currentDate);}
function nextMonth(){ currentDate.setMonth(currentDate.getMonth()+1); renderCalendar(currentDate);}
renderCalendar(currentDate);
</script>

</body>
</html>
