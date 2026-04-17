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

.good { color:#22c55e; }
.bad { color:#ef4444; }

.status {
  font-size:18px;
  margin-top:5px;
}

a {
  display:block;
  margin:5px 0;
  color:#60a5fa;
  text-decoration:none;
}
</style>

</head>
<body>

<h1>📊 오늘 시장 뉴스 요약</h1>
<p id="date" style="text-align:center;"></p>

<div id="result"></div>

<script>

// 날짜
const today = new Date();
document.getElementById("date").textContent =
  today.toLocaleDateString();

// 종목
const stocks = ["삼성전자","SK하이닉스","현대차","NAVER"];

// 키워드
const goodKeywords = ["수주","성장","흑자","AI","증가","확대","상승","개발","투자","계약"];
const badKeywords = ["적자","하락","소송","리콜","손실","위기","중단","논란","규제","폭락"];

// 뉴스 가져오기
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

// 분석
function analyze(news) {
  let good = 0;
  let bad = 0;

  news.forEach(n => {
    if (goodKeywords.some(k => n.title.includes(k))) good++;
    if (badKeywords.some(k => n.title.includes(k))) bad++;
  });

  return {good, bad};
}

// 상태 판단
function getStatus(good, bad) {
  if (good > bad) return "🟢 긍정";
  if (bad > good) return "🔴 부정";
  return "🟡 혼재";
}

// 실행
async function run() {
  const container = document.getElementById("result");

  for (let stock of stocks) {
    const news = await getNews(stock);

    if (news.length < 10) continue;

    const {good, bad} = analyze(news);
    const status = getStatus(good, bad);

    let html = `
    <div class="card">
      <h2>${stock}</h2>
      <div class="status">${status}</div>
      <p>뉴스 ${news.length}개</p>
      <p class="good">🟢 호재 ${good}</p>
      <p class="bad">🔴 악재 ${bad}</p>
    `;

    news.slice(0,5).forEach(n => {
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
