<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <!-- 아이폰 사파리 뷰포트 및 노치 영역 최적화 -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <!-- 홈 화면 추가(PWA) 모드 설정 -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="OTT Tracker">
  <title>OTT 시청 절제 캘린더</title>
  
  <style>
    :root {
      --bg-color: #0f172a;
      --card-bg: #1e293b;
      --accent-color: #f43f5e; /* OTT 시청 날짜 (빨간색) */
      --text-main: #f8fafc;
      --text-muted: #94a3b8;
    }

    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
      user-select: none;
    }

    body {
      margin: 0;
      padding: env(safe-area-inset-top) 16px env(safe-area-inset-bottom) 16px;
      background-color: var(--bg-color);
      color: var(--text-main);
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      justify-content: center;
    }

    .container {
      max-width: 400px;
      margin: 0 auto;
      width: 100%;
    }

    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
    }

    .header h2 {
      margin: 0;
      font-size: 1.25rem;
      font-weight: 700;
    }

    .nav-btn {
      background: none;
      border: none;
      color: var(--text-main);
      font-size: 1.2rem;
      padding: 8px 12px;
      cursor: pointer;
    }

    /* 캘린더 그리드 */
    .calendar {
      background-color: var(--card-bg);
      border-radius: 20px;
      padding: 16px;
      box-shadow: 0 10px 25px rgba(0,0,0,0.3);
    }

    .weekdays {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      text-align: center;
      font-weight: 600;
      color: var(--text-muted);
      font-size: 0.85rem;
      margin-bottom: 12px;
    }

    .days {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 6px;
    }

    .day {
      aspect-ratio: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      border-radius: 50%;
      font-size: 0.95rem;
      font-weight: 500;
      cursor: pointer;
      position: relative;
      transition: background-color 0.2s;
    }

    .day.empty {
      cursor: default;
    }

    .day.today {
      border: 1.5px solid var(--text-muted);
    }

    /* OTT 시청한 날 스티커/배지 */
    .day.watched {
      background-color: var(--accent-color);
      color: #fff;
      font-weight: 700;
    }

    .info-box {
      margin-top: 20px;
      background: var(--card-bg);
      border-radius: 16px;
      padding: 16px;
      text-align: center;
    }

    .info-box p {
      margin: 4px 0;
      font-size: 0.9rem;
      color: var(--text-muted);
    }

    .info-box span {
      color: var(--accent-color);
      font-weight: bold;
    }
  </style>
</head>
<body>

<div class="container">
  <div class="header">
    <button class="nav-btn" id="prevMonth">&#10094;</button>
    <h2 id="monthTitle">2026년 8월</h2>
    <button class="nav-btn" id="nextMonth">&#10095;</button>
  </div>

  <div class="calendar">
    <div class="weekdays">
      <div>일</div><div>월</div><div>화</div><div>수</div><div>목</div><div>금</div><div>토</div>
    </div>
    <div class="days" id="calendarDays"></div>
  </div>

  <div class="info-box">
    <p>이번 달 OTT 시청 횟수: <span id="monthCount">0</span>회</p>
    <p style="font-size: 0.8rem; margin-top: 6px;">💡 날짜를 터치하여 시청 여부를 토글하세요.</p>
  </div>
</div>

<script>
  let currentDate = new Date();
  // 로컬 스토리지에서 시청 기록 불러오기 (형식: "YYYY-MM-DD")
  let watchedDates = JSON.parse(localStorage.getItem('ott_watched_dates')) || [];

  const monthTitle = document.getElementById('monthTitle');
  const calendarDays = document.getElementById('calendarDays');
  const monthCount = document.getElementById('monthCount');

  function renderCalendar() {
    const year = currentDate.getFullYear();
    const month = currentDate.getMonth();

    // 헤더 월 표시
    monthTitle.innerText = `${year}년 ${month + 1}월`;

    // 이번 달 첫날 및 마지막 날 정보
    const firstDayIndex = new Date(year, month, 1).getDay();
    const lastDate = new Date(year, month + 1, 0).getDate();

    calendarDays.innerHTML = '';

    // 시작 빈 칸 추가
    for (let i = 0; i < firstDayIndex; i++) {
      const emptyDiv = document.createElement('div');
      emptyDiv.classList.add('day', 'empty');
      calendarDays.appendChild(emptyDiv);
    }

    // 날짜 렌더링
    let currentMonthWatchedCount = 0;
    const today = new Date();

    for (let day = 1; day <= lastDate; day++) {
      const dayDiv = document.createElement('div');
      dayDiv.classList.add('day');
      dayDiv.innerText = day;

      // 날짜 문자열 생성 (YYYY-MM-DD)
      const formattedMonth = String(month + 1).padStart(2, '0');
      const formattedDay = String(day).padStart(2, '0');
      const dateString = `${year}-${formattedMonth}-${formattedDay}`;

      // 오늘 날짜 표시
      if (year === today.getFullYear() && month === today.getMonth() && day === today.getDate()) {
        dayDiv.classList.add('today');
      }

      // 시청했는지 확인
      if (watchedDates.includes(dateString)) {
        dayDiv.classList.add('watched');
        currentMonthWatchedCount++;
      }

      // 날짜 클릭/터치 이벤트 (시청 기록 토글)
      dayDiv.addEventListener('click', () => {
        if (watchedDates.includes(dateString)) {
          watchedDates = watchedDates.filter(d => d !== dateString);
        } else {
          watchedDates.push(dateString);
        }
        localStorage.setItem('ott_watched_dates', JSON.stringify(watchedDates));
        renderCalendar();
      });

      calendarDays.appendChild(dayDiv);
    }

    monthCount.innerText = currentMonthWatchedCount;
  }

  // 월 이동 버튼 이벤트
  document.getElementById('prevMonth').addEventListener('click', () => {
    currentDate.setMonth(currentDate.getMonth() - 1);
    renderCalendar();
  });

  document.getElementById('nextMonth').addEventListener('click', () => {
    currentDate.setMonth(currentDate.getMonth() + 1);
    renderCalendar();
  });

  // 초기 실행
  renderCalendar();
</script>

</body>
</html>
