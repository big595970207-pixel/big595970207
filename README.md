<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>대학부 종합 대시보드</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --bg: #0d1117;
            --card: #161b22;
            --border: #30363d;
            --text: #c9d1d9;
            --neon: #58a6ff;
            --highlight: #2ea043;
            --warning: #d29922;
            --accent: #a371f7;
            --gray: #8b949e;
            --danger: #f85149;
        }
        body {
            background-color: var(--bg);
            color: var(--text);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0; padding: 20px;
        }
        h1, h2, h3 { color: #ffffff; text-align: center; }
        
        /* 상단 요약 박스 */
        .summary-container {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
            margin-bottom: 30px;
        }
        .kpi-card {
            background: var(--card);
            border: 1px solid var(--border);
            padding: 20px;
            border-radius: 8px;
            text-align: center;
        }
        .kpi-value {
            font-size: 32px;
            font-weight: bold;
            color: var(--neon);
            margin-top: 10px;
        }
        #total-score-val { color: var(--highlight); }

        /* 섹션 컨테이너 */
        .section-box {
            background: var(--card);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 30px;
        }
        .section-box h2 { border-bottom: 1px solid var(--border); padding-bottom: 10px; margin-top: 0; }

        /* 테이블 공통 스타일 */
        table {
            width: 100%;
            border-collapse: collapse;
            font-size: 14px;
            margin-top: 15px;
        }
        th, td {
            padding: 10px;
            text-align: center;
            border-bottom: 1px solid var(--border);
        }
        th { background-color: #21262d; color: var(--gray); }
        .total-row { font-weight: bold; color: var(--neon); background-color: #21262d; }
        
        /* 막대 그래프 그리드 배열 */
        .chart-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        .chart-box {
            background: var(--bg);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 15px;
            height: 330px; /* 높이 약간 조정 */
            position: relative;
        }
        .chart-box h3 { 
            margin-top: 0; 
            font-size: 16px; 
            color: var(--gray); 
            margin-bottom: 15px; 
            text-align: left;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .delta-badge {
            font-size: 13px;
            padding: 4px 8px;
            border-radius: 12px;
            background: rgba(255, 255, 255, 0.05);
        }

        /* 추이 차트 컨테이너 */
        .trend-chart-container {
            width: 100%;
            height: 300px;
            position: relative;
        }
    </style>
</head>
<body>

    <h1 id="dashboard-title">대학부 주간 종합 대시보드 (로딩중...)</h1>

    <div class="summary-container">
        <div class="kpi-card"><h3>총 재적</h3><div class="kpi-value">158명</div></div>
        <div class="kpi-card"><h3>출결 제외</h3><div class="kpi-value">22명</div></div>
        <div class="kpi-card"><h3>출결 재적</h3><div class="kpi-value">136명</div></div>
        <div class="kpi-card"><h3>이번주 전방부 총점</h3><div id="total-score-val" class="kpi-value">0점</div></div>
    </div>

    <div class="section-box">
        <h2>📈 주차별 전방부 종합점수 추이</h2>
        <div class="trend-chart-container">
            <canvas id="trendChart"></canvas>
        </div>
    </div>

    <div class="section-box">
        <h2>🙏 이번주 예배 및 모임 현황</h2>
        <table id="worship-table">
            <thead>
                <tr>
                    <th>구역</th>
                    <th>예배 참석</th>
                    <th>줌 예배</th>
                    <th>결석</th>
                    <th>심방률</th>
                    <th>구역예배</th>
                </tr>
            </thead>
            <tbody id="worship-tbody"></tbody>
        </table>
    </div>

    <div class="section-box">
        <h2>📊 평가 항목별 구역 세부 지표 (지난주 대비 증감)</h2>
        <div class="chart-grid" id="bar-charts-container">
            </div>
    </div>

    <script>
        const sheetUrl = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vSjhl42zs2Zd_xygVqvJ1Dwcls5YqU0YZU31sgqE2XeO_5LWodLkjjKkO-WmzLKeuja4Sriodynh59c/pub?output=csv';

        const categories = [
            { id: 'cat1', name: '센터등록', colIndex: 5, points: 50 },
            { id: 'cat2', name: '성공 사례발표', colIndex: 6, points: 10 },
            { id: 'cat3', name: '활동자', colIndex: 7, points: 1 },
            { id: 'cat4', name: '섬김이', colIndex: 8, points: 5 },
            { id: 'cat5', name: '신임사명자 양성', colIndex: 9, points: 20 },
            { id: 'cat6', name: '성공사례발표', colIndex: 10, points: 10 }
        ];

        Chart.defaults.color = '#8b949e';
        Chart.defaults.borderColor = '#30363d';

        async function fetchSheetData() {
            try {
                const response = await fetch(sheetUrl);
                const data = await response.text();
                
                const rows = data.split('\n').map(row => row.trim()).filter(row => row.length > 0);
                if (rows.length < 2) return;

                const weeklyData = {};
                const weekLabels = [];
                const weekScores = [];

                rows.slice(1).forEach(row => {
                    const cols = row.split(',');
                    const week = cols[0] ? cols[0].trim() : '';
                    if(!week) return;

                    if (!weeklyData[week]) {
                        weeklyData[week] = [];
                        weekLabels.push(week);
                    }
                    weeklyData[week].push(cols);
                });

                // 주차별 총점 계산
                weekLabels.forEach(week => {
                    let wScore = 0;
                    weeklyData[week].forEach(cols => {
                        categories.forEach(cat => {
                            const count = parseInt(cols[cat.colIndex]) || 0;
                            wScore += count * cat.points;
                        });
                    });
                    weekScores.push(wScore);
                });

                drawTrendChart(weekLabels, weekScores);

                // 최신 주차 및 지난주 데이터 가져오기
                const latestWeekIndex = weekLabels.length - 1;
                const latestWeek = weekLabels[latestWeekIndex];
                const prevWeek = latestWeekIndex > 0 ? weekLabels[latestWeekIndex - 1] : null;

                const latestRows = weeklyData[latestWeek];
                const prevRows = prevWeek ? weeklyData[prevWeek] : [];

                document.getElementById('dashboard-title').innerText = `대학부 종합 대시보드 (${latestWeek})`;

                // 지난주 데이터를 구역별로 쉽게 찾기 위해 맵핑
                const prevDataMap = {};
                prevRows.forEach(cols => {
                    const gName = cols[1] ? cols[1].trim() : '-';
                    prevDataMap[gName] = cols;
                });

                let grandTotalScore = 0;
                let worshipHtml = '';
                let sumWorship = { att: 0, zoom: 0, abs: 0 };
                
                const groupNames = []; 
                const categoryCounts = [[], [], [], [], [], []]; // 이번 주 데이터
                const prevCategoryCounts = [[], [], [], [], [], []]; // 지난주 데이터
                
                // 전체 합계 및 증감 계산용
                const catTotals = [0, 0, 0, 0, 0, 0];
                const prevCatTotals = [0, 0, 0, 0, 0, 0];

                latestRows.forEach(cols => {
                    const groupName = cols[1] ? cols[1].trim() : '-';
                    groupNames.push(groupName);

                    const prevCols = prevDataMap[groupName] || null;

                    let groupTotal = 0;
                    categories.forEach((cat, idx) => {
                        const count = parseInt(cols[cat.colIndex]) || 0;
                        const prevCount = prevCols ? (parseInt(prevCols[cat.colIndex]) || 0) : 0;

                        groupTotal += count * cat.points;
                        
                        categoryCounts[idx].push(count); 
                        prevCategoryCounts[idx].push(prevCount);

                        catTotals[idx] += count;
                        prevCatTotals[idx] += prevCount;
                    });
                    grandTotalScore += groupTotal;

                    const wAtt = parseInt(cols[2]) || 0;
                    const wZoom = parseInt(cols[3]) || 0;
                    const wAbs = parseInt(cols[4]) || 0;
                    const visitRate = cols[11] ? cols[11].trim() : '-';
                    const groupWorship = cols[12] ? cols[12].trim() : '-';
                    
                    sumWorship.att += wAtt;
                    sumWorship.zoom += wZoom;
                    sumWorship.abs += wAbs;

                    worshipHtml += `<tr>
                        <td>${groupName}</td>
                        <td style="color:var(--highlight);">${wAtt}</td>
                        <td style="color:var(--warning);">${wZoom}</td>
                        <td style="color:var(--danger);">${wAbs}</td>
                        <td style="color:var(--accent);">${visitRate}</td>
                        <td>${groupWorship}</td>
                    </tr>`;
                });

                worshipHtml += `<tr class="total-row">
                    <td>합계</td>
                    <td>${sumWorship.att}명</td>
                    <td>${sumWorship.zoom}명</td>
                    <td>${sumWorship.abs}명</td>
                    <td>-</td>
                    <td>-</td>
                </tr>`;
                document.getElementById('worship-tbody').innerHTML = worshipHtml;
                document.getElementById('total-score-val').innerText = grandTotalScore + '점';

                const chartContainer = document.getElementById('bar-charts-container');
                chartContainer.innerHTML = ''; 

                categories.forEach((cat, idx) => {
                    const diff = catTotals[idx] - prevCatTotals[idx];
                    let diffHtml = '';
                    if (diff > 0) diffHtml = `<span style="color:var(--highlight);">▲ ${diff}</span>`;
                    else if (diff < 0) diffHtml = `<span style="color:var(--danger);">▼ ${Math.abs(diff)}</span>`;
                    else diffHtml = `<span style="color:var(--gray);">-</span>`;

                    const chartDiv = document.createElement('div');
                    chartDiv.className = 'chart-box';
                    chartDiv.innerHTML = `
                        <h3>
                            <span>${cat.name} (${cat.points}점)</span>
                            <span class="delta-badge">총 ${catTotals[idx]}건 ${prevWeek ? `(${diffHtml})` : ''}</span>
                        </h3>
                        <canvas id="chart-${cat.id}"></canvas>
                    `;
                    chartContainer.appendChild(chartDiv);

                    const ctx = document.getElementById(`chart-${cat.id}`).getContext('2d');
                    new Chart(ctx, {
                        type: 'bar',
                        data: {
                            labels: groupNames, 
                            datasets: [
                                {
                                    label: '지난주',
                                    data: prevCategoryCounts[idx],
                                    backgroundColor: 'rgba(139, 148, 158, 0.3)', // 회색 (지난주)
                                    borderRadius: 4
                                },
                                {
                                    label: '이번주',
                                    data: categoryCounts[idx],
                                    backgroundColor: 'rgba(88, 166, 255, 0.8)', // 파란색 (이번주)
                                    borderRadius: 4
                                }
                            ]
                        },
                        options: {
                            responsive: true,
                            maintainAspectRatio: false,
                            plugins: { 
                                legend: { display: true, position: 'bottom', labels: { boxWidth: 12 } } 
                            },
                            scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } }
                        }
                    });
                });

            } catch (error) {
                console.error('데이터 오류:', error);
            }
        }

        function drawTrendChart(labels, data) {
            const ctx = document.getElementById('trendChart').getContext('2d');
            new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [{
                        label: '전방부 총점',
                        data: data,
                        borderColor: '#2ea043',
                        backgroundColor: 'rgba(46, 160, 67, 0.1)',
                        borderWidth: 2,
                        pointBackgroundColor: '#2ea043',
                        fill: true,
                        tension: 0.3
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } }
                }
            });
        }

        window.onload = fetchSheetData;
    </script>
</body>
</html>
