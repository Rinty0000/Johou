<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>高校情報ドリル</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: sans-serif; }
    body { background: #f4f4f4; color: #333; }
    .container { max-width: 600px; margin: 0 auto; padding: 20px; }
    .card { background: white; padding: 20px; border-radius: 8px; margin-bottom: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
    h1, h2, h3 { margin-bottom: 10px; }
    button { background: #4CAF50; color: white; border: none; padding: 10px 20px; border-radius: 5px; margin: 10px 0; cursor: pointer; }
    button:hover { background: #45a049; }
    .hidden { display: none; }
    .chapter-btn { display: block; background: #2196F3; margin: 10px 0; padding: 15px; color: white; border-radius: 6px; text-align: center; }
    .chapter-btn:hover { background: #1976D2; }
    .choice-btn { display: block; margin: 10px 0; padding: 10px; border: 1px solid #ccc; border-radius: 5px; cursor: pointer; background: #eee; }
    .choice-btn:hover { background: #ddd; }
    .progress-bar { width: 100%; background: #ccc; height: 10px; border-radius: 5px; margin-top: 10px; }
    .progress { height: 10px; background: #4CAF50; border-radius: 5px; }
    canvas { max-width: 100%; }
  </style>
</head>
<body>
  <div class="container">
    <div id="home" class="card">
      <h1>高校 情報ドリル</h1>
      <p>分野を選んでドリルを始めましょう！</p>
      <div id="chapter-list"></div>
    </div>

    <div id="quiz" class="card hidden">
      <h2 id="question-title"></h2>
      <div id="choices"></div>
      <div class="progress-bar"><div id="progress" class="progress"></div></div>
      <p id="explanation" class="hidden"></p>
      <button id="next-btn" class="hidden">次へ</button>
    </div>

    <div id="result" class="card hidden">
      <h2>結果発表</h2>
      <p id="score"></p>
      <canvas id="chart"></canvas>
      <button onclick="location.reload()">もう一度やる</button>
    </div>
  </div>

  <!-- 効果音 -->
  <audio id="correct-sound" src="https://sounds-mp3.com/mp3/0002036.mp3"></audio>
  <audio id="wrong-sound" src="https://sounds-mp3.com/mp3/0002032.mp3"></audio>

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script>
    const chapters = [
      { id: 1, title: "第1章：情報社会とモラル", color: "#f44336" },
      { id: 2, title: "第2章：コンピュータとプログラミングの基礎", color: "#ff9800" },
      { id: 3, title: "第3章：ネットワークとセキュリティ", color: "#9c27b0" },
      { id: 4, title: "第4章：データの活用と統計処理", color: "#3f51b5" },
      { id: 5, title: "第5章：アルゴリズムとフローチャート", color: "#009688" },
      { id: 6, title: "第6章：情報デザイン・メディアリテラシー", color: "#795548" },
      { id: 7, title: "第7章：データベースとAIの基礎", color: "#607d8b" },
    ];

    const questions = {
      1: [
        { q: "SNSでの個人情報の投稿はどのようなリスクを伴いますか？", a: ["犯罪に利用される可能性がある", "電波障害が起こる", "画面が暗くなる", "メモリが不足する"], c: 0, exp: "個人情報は第三者に悪用される危険があります。" },
        // ... 第1章の他の14問（後述で追加）
      ],
      2: [
        { q: "コンピュータの基本的な構成要素に含まれないのは？", a: ["CPU", "メモリ", "キーボード", "アルゴリズム"], c: 3, exp: "アルゴリズムは概念であり、ハードウェアではありません。" },
        // ... 第2章の他19問
      ],
      // ... 第3章～第7章の全問（合計110問）
    };

    let currentChapter = 1;
    let currentQuestion = 0;
    let score = 0;
    let correctCount = 0;
    let wrongCount = 0;
    let chapterStats = {};

    function init() {
      const chapterList = document.getElementById('chapter-list');
      chapters.forEach(ch => {
        const btn = document.createElement('div');
        btn.textContent = ch.title;
        btn.className = 'chapter-btn';
        btn.style.background = ch.color;
        btn.onclick = () => startQuiz(ch.id);
        chapterList.appendChild(btn);
      });
    }

    function startQuiz(chapterId) {
      document.getElementById('home').classList.add('hidden');
      document.getElementById('quiz').classList.remove('hidden');
      currentChapter = chapterId;
      currentQuestion = 0;
      score = 0;
      correctCount = 0;
      wrongCount = 0;
      chapterStats[chapterId] = { correct: 0, total: questions[chapterId].length };
      showQuestion();
    }

    function showQuestion() {
      const q = questions[currentChapter][currentQuestion];
      document.getElementById('question-title').textContent = q.q;
      const choicesDiv = document.getElementById('choices');
      choicesDiv.innerHTML = '';
      q.a.forEach((choice, i) => {
        const btn = document.createElement('button');
        btn.textContent = choice;
        btn.className = 'choice-btn';
        btn.onclick = () => checkAnswer(i);
        choicesDiv.appendChild(btn);
      });
      document.getElementById('explanation').classList.add('hidden');
      document.getElementById('next-btn').classList.add('hidden');
      updateProgress();
    }

    function checkAnswer(i) {
      const q = questions[currentChapter][currentQuestion];
      const correct = (i === q.c);
      if (correct) {
        score++;
        correctCount++;
        chapterStats[currentChapter].correct++;
        document.getElementById('correct-sound').play();
      } else {
        wrongCount++;
        document.getElementById('wrong-sound').play();
      }
      document.getElementById('explanation').textContent = q.exp;
      document.getElementById('explanation').classList.remove('hidden');
      document.getElementById('next-btn').classList.remove('hidden');
      Array.from(document.getElementsByClassName('choice-btn')).forEach(btn => btn.disabled = true);
    }

    function nextQuestion() {
      currentQuestion++;
      if (currentQuestion < questions[currentChapter].length) {
        showQuestion();
      } else {
        finishQuiz();
      }
    }

    function finishQuiz() {
      document.getElementById('quiz').classList.add('hidden');
      document.getElementById('result').classList.remove('hidden');
      const total = questions[currentChapter].length;
      document.getElementById('score').textContent = `正解数: ${score} / ${total}（${Math.round(score / total * 100)}%）`;
      localStorage.setItem(`chapter${currentChapter}_score`, score);
      drawChart();
    }

    function updateProgress() {
      const total = questions[currentChapter].length;
      const percent = currentQuestion / total * 100;
      document.getElementById('progress').style.width = percent + '%';
    }

    function drawChart() {
      const ctx = document.getElementById('chart').getContext('2d');
      const data = {
        labels: chapters.map(ch => ch.title),
        datasets: [{
          label: '正解数',
          data: chapters.map(ch => chapterStats[ch.id]?.correct || 0),
          backgroundColor: chapters.map(ch => ch.color)
        }]
      };
      new Chart(ctx, {
        type: 'bar',
        data: data,
      });
    }

    document.getElementById('next-btn').onclick = nextQuestion;
    window.onload = init;
  </script>
</body>
</html>
