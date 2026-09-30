<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OSA 개선 대책 유효성 점검 대시보드</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen">

    <!-- Header -->
    <header class="bg-indigo-900 text-white shadow-md">
        <div class="max-w-7xl mx-auto px-4 py-6 flex flex-col md:flex-row justify-between items-center">
            <div>
                <h1 class="text-2xl font-bold">OSA 개선 대책 유효성 점검 대시보드</h1>
                <p class="text-indigo-200 text-sm mt-1">주기별 자주 검증 진척도 및 현장 품질 관리 실시간 모니터링</p>
            </div>
            <div class="mt-4 md:mt-0 flex items-center space-x-2">
                <span id="sync-status" class="inline-flex items-center px-3 py-1 rounded-full text-xs font-medium bg-emerald-100 text-emerald-800">
                    <span class="w-2 h-2 mr-1.5 bg-emerald-500 rounded-full animate-pulse"></span> 자동 실시간 연동 중
                </span>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="max-w-7xl mx-auto px-4 py-8 space-y-8">

        <!-- KPI Cards Grid -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-sm font-medium text-slate-500">전체 관리 항목</p>
                    <h3 id="kpi-total" class="text-3xl font-bold text-slate-900 mt-1">0 건</h3>
                </div>
                <div class="p-3 bg-indigo-50 text-indigo-600 rounded-xl text-xl font-bold">📋</div>
            </div>
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-sm font-medium text-slate-500">전체 점검 완료율</p>
                    <h3 id="kpi-rate" class="text-3xl font-bold text-emerald-600 mt-1">0%</h3>
                </div>
                <div class="p-3 bg-emerald-50 text-emerald-600 rounded-xl text-xl font-bold">📈</div>
            </div>
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-sm font-medium text-slate-500">기한 초과 (지연)</p>
                    <h3 id="kpi-delayed" class="text-3xl font-bold text-rose-600 mt-1">0 건</h3>
                </div>
                <div class="p-3 bg-rose-50 text-rose-600 rounded-xl text-xl font-bold">⚠️</div>
            </div>
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-sm font-medium text-slate-500">참여 협력사 수</p>
                    <h3 id="kpi-partners" class="text-3xl font-bold text-blue-600 mt-1">0 개사</h3>
                </div>
                <div class="p-3 bg-blue-50 text-blue-600 rounded-xl text-xl font-bold">🏢</div>
            </div>
        </div>

        <!-- Charts Section -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <!-- 주차별 완료 현황 바 차트 -->
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 lg:col-span-2">
                <h3 class="text-lg font-bold text-slate-900 mb-4">유효성 점검 주기별 완료 현황</h3>
                <div class="relative h-72">
                    <canvas id="weeklyChart"></canvas>
                </div>
            </div>
            <!-- OSA 협력사별 비중 도넛 차트 -->
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
                <h3 class="text-lg font-bold text-slate-900 mb-4">OSA 협력사별 등록 비중</h3>
                <div class="relative h-72 flex justify-center items-center">
                    <canvas id="partnerChart"></canvas>
                </div>
            </div>
        </div>

        <!-- Detailed Table Section -->
        <div class="bg-white rounded-2xl shadow-sm border border-slate-100 overflow-hidden">
            <div class="p-6 border-b border-slate-100 flex justify-between items-center">
                <h3 class="text-lg font-bold text-slate-900">개선 대책 상세 진행 목록</h3>
                <span class="text-xs text-slate-400">* 실시간 스프레드시트 연동 데이터 기준 (행 자동 확장)</span>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-50 text-slate-500 text-xs uppercase font-semibold">
                            <th class="p-4">관리번호</th>
                            <th class="p-4">발생일</th>
                            <th class="p-4">OSA</th>
                            <th class="p-4">고객사</th>
                            <th class="p-4">품명</th>
                            <th class="p-4">불량내용</th>
                            <th class="p-4 text-center">진행 상태</th>
                        </tr>
                    </thead>
                    <tbody id="table-body" class="divide-y divide-slate-100 text-sm">
                        <tr>
                            <td colspan="7" class="p-6 text-center text-slate-400">데이터를 불러오는 중입니다...</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

    </main>

    <!-- JavaScript 로직 -->
    <script>
        const SHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vRCGmQTOQd1DK4cmMzKU618FjIvvcwcSCgX3PBwtNF0i7_Q6aK3Hux-W56QCwAwNY7O2bff6zZ01RZm/pub?gid=359651245&single=true&output=csv";

        let weeklyChartInstance = null;
        let partnerChartInstance = null;

        // 페이지 로드 시 자동 실행 및 60초마다 자동 갱신
        window.addEventListener('DOMContentLoaded', () => {
            fetchAndRenderData();
            setInterval(fetchAndRenderData, 60000);
        });

        async function fetchAndRenderData() {
            const targetUrl = SHEET_CSV_URL + (SHEET_CSV_URL.includes('?') ? '&' : '?') + 't=' + Date.now();

            try {
                const response = await fetch(targetUrl, { cache: 'no-store' });
                if (!response.ok) throw new Error('네트워크 응답 오류 발생');
                
                const csvText = await response.text();
                parseCSVAndRender(csvText);

                document.getElementById('sync-status').innerHTML = '<span class="w-2 h-2 mr-1.5 bg-emerald-500 rounded-full"></span> 연동 완료 (' + new Date().toLocaleTimeString() + ')';
            } catch (error) {
                console.error(error);
                document.getElementById('sync-status').innerHTML = '<span class="w-2 h-2 mr-1.5 bg-rose-500 rounded-full"></span> 연동 실패';
            }
        }

        // CSV 파싱 및 동적 확장 처리 함수
        function parseCSVAndRender(csvText) {
            const rows = parseCSVToArray(csvText);
            if (rows.length < 2) return;

            // 헤더 찾기 (관리번호가 포함된 행 탐색)
            let headerIndex = -1;
            for (let i = 0; i < rows.length; i++) {
                if (rows[i].includes("관리번호")) {
                    headerIndex = i;
                    break;
                }
            }

            if (headerIndex === -1) return;

            const dataRows = rows.slice(headerIndex + 1);

            let totalCount = 0;
            let partnersSet = new Set();
            let tableHtml = '';
            
            let weeklyCompleted = [0, 0, 0, 0, 0, 0, 0, 0];
            let partnerCounts = {};

            // 스마트 동적 스캐닝 루프 (행이 추가되어도 누락 없이 완벽 탐색)
            let i = 0;
            while (i < dataRows.length) {
                const row = dataRows[i];
                
                // 유효한 관리번호 행 탐색 (비어있지 않고 키워드가 아닌 경우)
                if (row && row[0] && row[0].trim() !== '' && !row[0].includes('점검') && !row[0].includes('기준일')) {
                    const id = row[0] || '';
                    const date = row[1] || '';
                    const osa = row[2] || '';
                    const client = row[3] || '';
                    const productName = row[4] || '';
                    const defect = row[5] || '';

                    totalCount++;
                    if (osa) partnersSet.add(osa);
                    partnerCounts[osa] = (partnerCounts[osa] || 0) + 1;

                    // 다음 행들 중에서 점검 실시일 데이터를 담고 있는 행을 동적으로 탐색
                    let row2 = [];
                    for (let k = 1; k <= 3; k++) {
                        if (i + k < dataRows.length) {
                            let candidate = dataRows[i + k];
                            // 만약 다음 행이 새로운 관리번호라면 실시일 행이 없는 것임
                            if (candidate[0] && candidate[0].trim() !== '' && !candidate[0].includes('점검')) {
                                break;
                            }
                            // 실시일 데이터가 포함된 행을 찾으면 확정
                            if (candidate.some(cell => cell.includes('점검 실시일') || cell.match(/\d{4}-\d{2}-\d{2}/))) {
                                row2 = candidate;
                                break;
                            }
                        }
                    }

                    // 1주차 ~ 6개월 완료 체크 (인덱스 10 ~ 17)
                    let completedChecks = 0;
                    for (let col = 10; col <= 17; col++) {
                        if (row2[col] && row2[col].trim().length > 5) {
                            completedChecks++;
                            let idx = col - 10;
                            if (idx >= 0 && idx < 8) weeklyCompleted[idx]++;
                        }
                    }

                    let statusBadge = `<span class="px-2.5 py-1 text-xs font-semibold rounded-full bg-emerald-100 text-emerald-800">진행중 (${completedChecks}/8)</span>`;
                    if (completedChecks === 8) {
                        statusBadge = `<span class="px-2.5 py-1 text-xs font-semibold rounded-full bg-blue-100 text-blue-800">완료 (8/8)</span>`;
                    }

                    tableHtml += `
                        <tr class="hover:bg-slate-50/80 transition-colors">
                            <td class="p-4 font-medium text-slate-900">${id}</td>
                            <td class="p-4 text-slate-600">${date}</td>
                            <td class="p-4 font-semibold text-indigo-900">${osa}</td>
                            <td class="p-4 text-slate-600">${client}</td>
                            <td class="p-4 text-slate-600">${productName}</td>
                            <td class="p-4 text-slate-600">${defect}</td>
                            <td class="p-4 text-center">${statusBadge}</td>
                        </tr>
                    `;
                }
                i++;
            }

            // KPI 업데이트
            document.getElementById('kpi-total').innerText = totalCount + " 건";
            document.getElementById('kpi-partners').innerText = partnersSet.size + " 개사";
            
            let totalPossible = totalCount * 8;
            let totalDone = weeklyCompleted.reduce((a, b) => a + b, 0);
            let completionRate = totalPossible > 0 ? Math.round((totalDone / totalPossible) * 100) : 0;
            document.getElementById('kpi-rate').innerText = completionRate + "%";
            document.getElementById('kpi-delayed').innerText = "0 건";

            document.getElementById('table-body').innerHTML = tableHtml || `<tr><td colspan="7" class="p-6 text-center text-slate-400">유효한 데이터가 없습니다.</td></tr>`;

            // 차트 갱신
            renderCharts(weeklyCompleted, partnerCounts);
        }

        // 안전한 CSV 파서
        function parseCSVToArray(str) {
            let arr = [];
            let row = [];
            let inQuotes = false;
            let c = '';
            let val = '';

            for (let i = 0; i < str.length; i++) {
                c = str[i];
                let nextC = str[i + 1];

                if (c === '"') {
                    if (inQuotes && nextC === '"') {
                        val += '"';
                        i++;
                    } else {
                        inQuotes = !inQuotes;
                    }
                } else if (c === ',' && !inQuotes) {
                    row.push(val.trim());
                    val = '';
                } else if ((c === '\r' || c === '\n') && !inQuotes) {
                    if (c === '\r' && nextC === '\n') { i++; }
                    row.push(val.trim());
                    arr.push(row);
                    row = [];
                    val = '';
                } else {
                    val += c;
                }
            }
            if (val !== '' || row.length > 0) {
                row.push(val.trim());
                arr.push(row);
            }
            return arr;
        }

        // 차트 렌더링 함수
        function renderCharts(weeklyData, partnerData) {
            const ctxWeekly = document.getElementById('weeklyChart').getContext('2d');
            if (weeklyChartInstance) weeklyChartInstance.destroy();

            weeklyChartInstance = new Chart(ctxWeekly, {
                type: 'bar',
                data: {
                    labels: ['1주차', '2주차', '3주차', '4주차', '2개월', '3개월', '4개월', '6개월'],
                    datasets: [{
                        label: '완료 건수',
                        data: weeklyData,
                        backgroundColor: 'rgba(99, 102, 241, 0.8)',
                        borderRadius: 8,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: {
                        y: { beginAtZero: true, grid: { borderDash: [4, 4] } },
                        x: { grid: { display: false } }
                    }
                }
            });

            const ctxPartner = document.getElementById('partnerChart').getContext('2d');
            if (partnerChartInstance) partnerChartInstance.destroy();

            const partnerLabels = Object.keys(partnerData);
            const partnerValues = Object.values(partnerData);

            partnerChartInstance = new Chart(ctxPartner, {
                type: 'doughnut',
                data: {
                    labels: partnerLabels.length > 0 ? partnerLabels : ['데이터 없음'],
                    datasets: [{
                        data: partnerValues.length > 0 ? partnerValues : [1],
                        backgroundColor: ['#4f46e5', '#06b6d4', '#10b981', '#f59e0b', '#ef4444'],
                        borderWidth: 0
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { position: 'bottom', labels: { boxWidth: 12, font: { size: 11 } } }
                    }
                }
            });
        }
    </script>
</body>
</html>
