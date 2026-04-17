<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>주식 뉴스 키워드 분석</title>
<style>
  body { font-family: Arial; background:#f5f6fa; padding:20px;}
  h1 { text-align:center; }
  .card {
    background:white;
    padding:20px;
    margin:15px auto;
    max-width:600px;
    border-radius:10px;
    box-shadow:0 5px 15px rgba(0,0,0,0.1);
  }
  .good { color:green; }
  .bad { color:red; }
  a { display:block; margin:5px 0; }
</style>
</head>
<body>

<h1>📊 주식 뉴스 분석</h1>
<div id="result"></div>

<script>

// 📌 대형주 리스트
const stocks = ["삼성전자", "SK하이닉스", "LG에너지솔루션", "현대차", "NAVER"];

// 📌 키워드 (20개씩)
const goodKeywords = ["수주","성장","호황","흑자","AI","증가","확대","상승","개발","진출",
"혁신","신사업","호재","투자","개선","성공","계약","파트너십","돌파","최대"];

const badKeywords = ["적자","하락","감소","리콜","소송","악재","위기","중단","폐기","손실",
"축소","철수","논란","지연","파업","규제","충격","불안","폭락","문제"];

// 📌 RSS 가져오기 (네이버 뉴스)
async function getNews(stock) {
  const url = `https://news.google.com/rss/search?q=${encodeURIComponent(stock)}&hl=ko&gl=KR&ceid=KR:ko`;

  const res = await fetch("https://api.allorigins.win/get?url=" + encodeURIComponent(url));
  const data = await res.json();

  const parser = new DOMParser();
  const xml = parser.parseFromString(data.contents, "text/xml");

  const items = xml.querySelectorAll("item");

  let newsList = [];

  items.forEach(item => {
    const title = item.querySelector("title").textContent;
    const link = item.querySelector("link").textContent;
    newsList.push({title, link});
  });

  return newsList;
}

// 📌 키워드 분석
function analyze(newsList) {
  let good = 0;
  let bad = 0;

  newsList.forEach(n => {
    goodKeywords.forEach(k => {
      if (n.title.includes(k)) good++;
    });
    badKeywords.forEach(k => {
      if (n.title.includes(k)) bad++;
    });
  });

  return {good, bad};
}

// 📌 실행
async function run() {
  const container = document.getElementById("result");

  for (let stock of stocks) {
    const news = await getNews(stock);

    if (news.length < 10) continue;

    const result = analyze(news);

    let html = `
      <div class="card">
        <h2>${stock}</h2>
        <p>총 뉴스: ${news.length}</p>
        <p class="good">호재: ${result.good}</p>
        <p class="bad">악재: ${result.bad}</p>
    `;

    news.slice(0,10).forEach(n => {
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
