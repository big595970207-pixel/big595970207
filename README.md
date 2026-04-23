<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>대학부 종합 관제 대시보드 - 4월 3주차</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --bg: #0d1117; --card: #161b22; --border: #30363d; --neon: #00ff00; --blue: #4285F4; --yellow: #fbbc05; --red: #ea4335; }
        body { background-color: var(--bg); color: white; font-family: 'Pretendard', 'Arial', sans-serif; margin: 0; padding: 20px; }
        
        header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid var(--neon); padding-bottom: 10px; margin-bottom: 20px; }
        .live-tag { background: var(--neon); color: black; padding: 2px 8px; border-radius: 4px; font-weight: bold; font-size: 12px; }

        .summary-container { display: grid; grid-template-columns: repeat(4, 1fr); gap: 15px; margin-bottom: 20px; }
        .kpi-card { background: var(--card); border: 1px solid var(--border); padding: 15px; border-radius: 8px; text-align: center; }
        .kpi-value { font-size: 28px; font-weight: bold; color: var(--neon); margin-top: 5px; }

        .main-container { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .chart-box { background: var(--card); border: 1px solid var(--border); padding: 20px; border-radius: 12px; position: relative; }
        h2 { font-size: 16px; margin-top: 0; color: #8b949e; border-bottom: 1px solid var(--border); padding-bottom: 10px; }
    </style>
</head>
<body>

<header>
    <div><strong>UNIVERSITY MINISTRY</strong> | 통합 모니터링 시스템</div>
    <div><span class="live-tag">LIVE</span> 2026-04-19 갱신 (4월 3주차)</div>
</header>

<div class="summary-container">
    <div class="kpi-card"><div>전체 재적</div><div class="kpi-value">157명</div></div>
    <div class="kpi-card"><div>주간 심방 (출결재적 138명 기준)</div><div class="kpi-value" id="kpi-week-visit">62건</div></div>
    <div class="kpi-card"><div>토요 전도단 (미실시)</div><div class="kpi-value" id="kpi-evan" style="color:var(--yellow)">0명</div></div>
    <div class="kpi-card"><div>구역예배 평균 참석률</div><div class="kpi-value" id="kpi-cell" style="color:var(--blue)">38.4%</div></div>
</div>

<div class="main-container">
    <div class="chart-box">
        <h2>⛪ 구역예배 참석 현황 (4월 3주차)</h2>
        <canvas id="cellWorshipChart"></canvas>
    </div>

    <div class="chart-box">
        <h2>📞 구역별 심방 현황 (이미지 데이터 반영)</h2>
        <canvas id="visitCombinedChart"></canvas>
    </div>

    <div class="chart-box">
        <h2>📈 주차별 누적 심방 추이 (출결재적 138명 기준)</h2>
        <canvas id="weeklyVisitChart"></canvas>
    </div>

    <div class="chart-box">
        <h2>🚩 토요 전도단 출석 현황 (4월 3주차 미실시)</h2>
        <canvas id="evangelismChart"></canvas>
    </div>
</div>

<script>
    // ==========================================
    // 🛠️ 데이터 설정 영역 (4월 3주차 업데이트 완료)
    // ==========================================

    // [데이터 1] 구역예배 현황 (참석 53명 / 출결재적 138명)
    const cellLabels = ['1구역', '2구역', '3구역', '4구역', '5구역', '6구역', '7구역', '8구역', '9구역', '10구역', '11구역', '12구역', '13구역', '14구역'];
    const cellTotal = [7, 11, 12, 9, 12, 11, 11, 7, 11, 10, 8, 12, 11, 6]; 
    const cellAttend = [1, 3, 4, 5, 3, 3, 3, 0, 5, 8, 3, 4, 6, 4];
    const cellRates = cellAttend.map((attend, i) => ((attend / cellTotal[i]) * 100).toFixed(1));

    // [데이터 2] 구역별 심방 현황 (이미지 추출 데이터)
    const visitCounts = [0, 4, 6, 6, 5, 6, 4, 1, 7, 5, 4, 5, 7, 2]; 
    const visitBase = [6, 11, 12, 9, 12, 11, 11, 7, 11, 10, 8, 12, 11, 6];
    const visitRates = visitCounts.map((count, i) => ((count / visitBase[i]) * 100).toFixed(1)); 

    // [데이터 3] 주간 누적 심방 추이
    const weekLabels = ['3월 4주차', '4월 1주차', '4월 2주차', '4월 3주차'];
    const weekVisitCounts = [58, 50, 55, 62]; // 최신 62건 반영
    const baseTarget = 138; 
    const weekVisitRates = weekVisitCounts.map(count => ((count / baseTarget) * 100).toFixed(1));

    // [데이터 4] 토요 전도단 (4월 3주차 미실시로 0 처리)
    const evanLabels = ['3월 4주차', '4월 1주차', '4월 2주차', '4월 3주차'];
    const evanOnTime = [47, 40, 43, 0];
    const evanLate = [0, 3, 2, 0];


    // ==========================================
    // 🎨 차트 그리기 로직 (Chart.js)
    // ==========================================
    const commonOptions = {
        responsive: true,
        plugins: { legend: { labels: { color: '#fff' } } },
        scales: {
            y: { beginAtZero: true, grid: { color: '#333' }, ticks: { color: '#ccc' } }
        }
    };

    new Chart(document.getElementById('cellWorshipChart'), {
        type: 'bar',
        data: {
            labels: cellLabels,
            datasets: [
                { label: '참석 인원 (명)', data: cellAttend, backgroundColor: 'rgba(0, 255, 0, 0.7)', order: 2 },
                { label: '참석률 (%)', data: cellRates, type: 'line', borderColor: '#4285F4', borderWidth: 3, yAxisID: 'y1', order: 1 }
            ]
        },
        options: {
            scales: {
                y: { grid: { color: '#333' }, ticks: { color: '#ccc' }, title: { display: true, text: '명', color: '#00ff00' } },
                y1: { position: 'right', grid: { display: false }, ticks: { color: '#ccc' }, title: { display: true, text: '%', color: '#4285F4' } }
            },
            plugins: { legend: { labels: { color: '#fff' } } }
        }
    });

    new Chart(document.getElementById('visitCombinedChart'), {
        type: 'bar',
        data: {
            labels: cellLabels,
            datasets: [
                { label: '심방 완료 건수', data: visitCounts, backgroundColor: 'rgba(66, 133, 244, 0.5)', order: 2 },
                { label: '심방률 (%)', data: visitRates, type: 'line', borderColor: '#ea4335', borderWidth: 3, yAxisID: 'y1', order: 1 }
            ]
        },
        options: {
            scales: {
                y: { grid: { color: '#333' }, ticks: { color: '#ccc' } },
                y1: { position: 'right', grid: { display: false }, ticks: { color: '#ccc' } }
            },
            plugins: { legend: { labels: { color: '#fff' } } }
        }
    });

    new Chart(document.getElementById('weeklyVisitChart'), {
        type: 'line',
        data: {
            labels: weekLabels,
            datasets: [
                { label: '주차별 심방 건수', data: weekVisitCounts, borderColor: '#fbbc05', backgroundColor: '#fbbc05', tension: 0.3 },
                { label: '심방률 (%)', data: weekVisitRates, borderColor: '#00ff00', borderDash: [5, 5], tension: 0.3, yAxisID: 'y1' }
            ]
        },
        options: {
            scales: {
                y: { grid: { color: '#333' }, ticks: { color: '#ccc' } },
                y1: { position: 'right', grid: { display: false }, ticks: { color: '#ccc' } }
            },
            plugins: { legend: { labels: { color: '#fff' } } }
        }
    });

    new Chart(document.getElementById('evangelismChart'), {
        type: 'bar',
        data: {
            labels: evanLabels,
            datasets: [
                { label: '정시 도착', data: evanOnTime, backgroundColor: 'rgba(251, 188, 5, 0.8)' },
                { label: '지각/기타', data: evanLate, backgroundColor: 'rgba(234, 67, 53, 0.8)' }
            ]
        },
        options: commonOptions
    });
</script>
</body>
</html>
