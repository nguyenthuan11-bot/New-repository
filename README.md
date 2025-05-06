<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Trang chủ - Azota Clone</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <h1>Chào mừng đến với Azota Clone</h1>
    <p>Bạn là:</p>
    <a href="teacher.html" class="btn">Giáo viên</a>
    <a href="quiz.html" class="btn">Học sinh</a>
  </div>
</body>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Giáo viên - Upload đề</title>
  <link rel="stylesheet" href="style.css">
  <script src="xlsx.full.min.js"></script>
</head>
<body>
  <div class="container">
    <h2>Upload đề bài (Excel .xlsx hoặc .csv)</h2>
    <input type="file" id="file-input" accept=".xlsx,.csv" />
    <div id="status"></div>
    <a href="quiz.html" class="btn">Chuyển sang giao diện học sinh</a>
  </div>

  <script>
    document.getElementById('file-input').addEventListener('change', function(e) {
      const file = e.target.files[0];
      const reader = new FileReader();

      reader.onload = function(event) {
        const data = new Uint8Array(event.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const sheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[sheetName];
        const jsonData = XLSX.utils.sheet_to_json(worksheet);

        localStorage.setItem('quiz', JSON.stringify(jsonData));
        document.getElementById('status').innerText = 'Tải đề thành công!';
      };

      reader.readAsArrayBuffer(file);
    });
  </script>
</body>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Làm bài kiểm tra</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <h2>Làm bài trắc nghiệm</h2>
    <div id="timer">Thời gian: 03:00</div>
    <form id="quiz-form"></form>
    <button onclick="submitQuiz()">Nộp bài</button>
  </div>

  <script>
    const quiz = JSON.parse(localStorage.getItem('quiz')) || [];
    const form = document.getElementById('quiz-form');

    quiz.forEach((q, i) => {
      const div = document.createElement('div');
      div.innerHTML = `
        <p><strong>${i + 1}. ${q.Question}</strong></p>
        <label><input type="radio" name="q${i}" value="A"> ${q.A}</label><br>
        <label><input type="radio" name="q${i}" value="B"> ${q.B}</label><br>
        <label><input type="radio" name="q${i}" value="C"> ${q.C}</label><br>
      `;
      form.appendChild(div);
    });

    function submitQuiz() {
      let score = 0;
      quiz.forEach((q, i) => {
        const ans = document.querySelector(`input[name="q${i}"]:checked`);
        if (ans && ans.value === q.Correct) score++;
      });
      localStorage.setItem('score', score);
      window.location.href = "result.html";
    }

    // Countdown
    let time = 180; // 3 phút
    const timerDiv = document.getElementById('timer');
    const interval = setInterval(() => {
      const min = Math.floor(time / 60);
      const sec = time % 60;
      timerDiv.innerText = `Thời gian: ${min}:${sec.toString().padStart(2, '0')}`;
      time--;
      if (time < 0) {
        clearInterval(interval);
        submitQuiz();
      }
    }, 1000);
  </script>
</body>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Kết quả bài làm</title>
  <link rel="stylesheet" href="style.css">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
</head>
<body>
  <div class="container" id="result-container">
    <h2>Kết quả</h2>
    <p id="score"></p>
    <button onclick="downloadPDF()">Tải kết quả PDF</button>
  </div>

  <script>
    const score = localStorage.getItem('score') || 0;
    const total = JSON.parse(localStorage.getItem('quiz')).length;
    document.getElementById('score').innerText = `Bạn đạt: ${score}/${total} điểm`;

    function downloadPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      doc.text(`KẾT QUẢ BÀI LÀM`, 10, 10);
      doc.text(`Số câu đúng: ${score}/${total}`, 10, 20);
      doc.save('ket-qua.pdf');
    }
  </script>
</body>
</html>
body {
  font-family: Arial;
  background-color: #f4f4f4;
  padding: 20px;
}
.container {
  max-width: 600px;
  margin: auto;
  background: #fff;
  padding: 25px;
  border-radius: 10px;
}
h1, h2 {
  text-align: center;
}
.btn {
  display: inline-block;
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  margin-top: 10px;
  text-decoration: none;
  border-radius: 5px;
}
button {
  padding: 10px;
  background: #2196F3;
  color: white;
  border: none;
  border-radius: 5px;
  margin-top: 10px;
  cursor: pointer;
}
