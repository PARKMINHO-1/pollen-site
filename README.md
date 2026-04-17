<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>주식 뉴스 대시보드</title>

<style>
body {
  font-family: Arial;
  background:#0f172a;
  color:white;
  padding:20px;
}

h1 { text-align:center; }

.card {
  background:#1e293b;
  padding:20px;
  margin:15px auto;
  max-width:600px;
  border-radius:15px;
}

.bar {
  height: 12px;
  border-radius: 10px;
  margin: 10px 0;
}

.goodBar { background:#22c55e; }
.badBar { background:#ef4444; }

.score {
  font-size:20px;
  font-weight:bold;
}

a {
  display:block;
  margin:5px 0;
  color:#60a5fa;
  font-size:14px;
}
</style>

</head>
<body>

<h1>📊 오늘 시장 분석</h1>
<div id="result"></div>

<script>

const stocks = ["삼성전자","SK하이닉스","현대차","NAVER"];

const goodKeywords = ["수주","성장","흑자","AI","증가","확대","상승","개발","투자","계약"];
const badKeywords = ["적자","하락","소송","리콜","손실","위기","중단","논란","규제","폭락"];

async function getNews(stock) {
  const url = `https://news.google.com/rss/search?q=${encodeURIComponent(stock)}&hl=ko&gl=KR&ceid=KR:ko`;

  const res = await fetch("https://api.allorigins.win/get?url=" + encodeURIComponent(url));
  const data = await res.json();

  const parser = new DOMParser();
  const xml = parser.parseFromString(data.contents, "text/xml");

  return [...xml.querySelectorAll("item")].map(item => ({
    title: item.querySelector("title").textContent,
    link: item.querySelector("link").textContent
  }));
}

function analyze(news) {
  let good = 0;
  let bad = 0;

  news.forEach(n => {
    if (goodKeywords.some(k => n.title.includes(k))) good++;
    if (badKeywords.some(k => n.title.includes(k))) bad++;
  });

  return {good, bad};
}

async function run() {
  const container = document.getElementById("result");

  for (let stock of stocks) {
    const news = await getNews(stock);

    if (news.length < 10) continue;

    const {good, bad} = analyze(news);
    const score = good - bad;

    const barWidth = Math.min(Math.abs(score) * 10, 100);

    let barClass = score >= 0 ? "goodBar" : "badBar";
    let sign = score >= 0 ? "🟢 +" : "🔴 ";

    let html = `
    <div class="card">
      <h2>${stock}</h2>
      <div class="score">${sign}${score}</div>
      <div class="bar ${barClass}" style="width:${barWidth}%"></div>
      <p>뉴스 ${news.length}개 | 호재 ${good} / 악재 ${bad}</p>
    `;

    news.slice(0,3).forEach(n => {
      html += `<a href="${n.link}" target="_blank">${n.title}</a>`;
    });

    html += `</div>`;

    container.innerHTML += html;
  }
}

run();

</script>

</body>
</html>
