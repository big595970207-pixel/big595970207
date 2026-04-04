<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>대학부 심방 관제 대시보드</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --bg: #0d1117; --card: #161b22; --border: #30363d; --neon: #00ff00; --blue: #4285F4; }
        body { background-color: var(--bg); color: white; font-family: 'Pretendard', sans-serif; margin: 0; padding: 20px; }
        
        /* 헤더 스타일 */
        header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid var(--neon); padding-bottom: 10px; margin-bottom: 20px; }
        .live-tag { background: var(--neon); color: black; padding: 2px 8px; border-radius: 4px; font-weight: bold; font-size: 12px; }

        /* 상단 요약 카드 */
        .summary-container { display: grid; grid-template-columns: repeat(4, 1fr); gap: 15px; margin-bottom: 20px; }
        .kpi-card { background: var(--card); border: 1px solid var(--border); padding: 15px; border-radius: 8px; text-align: center; }
        .kpi-value { font-size: 28px; font-weight: bold; color: var(--neon); margin-top: 5px; }

        /* 메인 차트 영역 */
        .main-container { display: grid; grid-template-columns: 2fr 1fr; gap: 20px; }
        .chart-box { background: var(--card); border: 1px solid var(--border); padding: 20px; border-radius: 12px; position: relative; }
        h2 { font-size: 16px; margin-top: 0; color: #8b949e; }
    </style>
</head>
<body>

<header>
    <div><strong>UNIVERSITY MINISTRY</strong> | 심방 관리 시스템</div>
    <div><span class="live-tag">LIVE</span> 2026-03-30 갱신</div>
</header>

<div class="summary-container">
    <div class="kpi-card"><div>전체 재적</div><div class="kpi-value">162명</div></div>
    <div class="kpi-card"><div>평균 출석</div><div class="kpi-value">148명</div></div>
    <div class="kpi-card"><div>누적 심방</div><div class="kpi-value">46회</div></div>
    <div class="kpi-card"><div>평균 심방률</div><div class="kpi-value" style="color:var(--blue)">31.1%</div></div>
</div>

<div class="main-container">
    <div class="chart-box">
        <h2>구역별 심방 현황 (심방횟수 & 심방률)</h2>
        <canvas id="combinedChart"></canvas>
    </div>
    <div class="chart-box">
        <h2>구역별 출석 분포</h2>
        <canvas id="doughnutChart"></canvas>
    </div>
</div>

<script>
    const ctx = document.getElementById('combinedChart').getContext('2d');
    const labels = ['1구역', '2구역', '3구역', '4구역', '5구역', '6구역', '7구역', '8구역', '9구역', '10구역', '11구역', '12구역', '13구역', '14구역'];
    
    new Chart(ctx, {
        type: 'bar',
        data: {
            labels: labels,
            datasets: [
                {
                    label: '심방횟수 (막대)',
                    data: [0, 4, 4, 3, 4, 4, 2, 0, 6, 6, 4, 4, 2, 3],
                    backgroundColor: 'rgba(0, 255, 0, 0.6)',
                    order: 2
                },
                {
                    label: '심방률 % (선)',
                    data: [0, 36, 29, 38, 33, 36, 18, 0, 55, 60, 50, 33, 18, 50],
                    type: 'line',
                    borderColor: '#4285F4',
                    borderWidth: 3,
                    pointBackgroundColor: '#fff',
                    fill: false,
                    order: 1,
                    yAxisID: 'y1'
                }
            ]
        },
        options: {
            responsive: true,
            scales: {
                y: { beginAtZero: true, grid: { color: '#333' }, ticks: { color: '#ccc' }, title: { display: true, text: '횟수', color: '#00ff00' } },
                y1: { position: 'right', grid: { display: false }, ticks: { color: '#ccc' }, title: { display: true, text: '퍼센트(%)', color: '#4285F4' } }
            },
            plugins: { legend: { labels: { color: '#fff' } } }
        }
    });

    // 오른쪽 도넛 차트 (간단 예시)
    const ctx2 = document.getElementById('doughnutChart').getContext('2d');
    new Chart(ctx2, {
        type: 'doughnut',
        data: {
            labels: ['심방완료', '미실시'],
            datasets: [{ data: [46, 116], backgroundColor: ['#00ff00', '#30363d'] }]
        },
        options: { plugins: { legend: { position: 'bottom', labels: { color: '#fff' } } } }
    });
</script>
</body>
</html>
