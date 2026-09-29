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
    <!-- FontAwesome CDN -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>
<body class="bg-slate-50 text-slate-800 font-sans antialiased min-h-screen flex flex-col">

    <!-- Header -->
    <header class="bg-indigo-900 text-white shadow-md">
        <div class="max-w-7xl mx-auto px-4 py-4 flex flex-col md:flex-row justify-between items-center gap-4">
            <div class="flex items-center space-x-3">
                <div class="bg-indigo-600 p-2.5 rounded-lg text-xl shadow-inner">
                    <i class="fa-solid fa-chart-line"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold tracking-tight">OSA 개선 대책 유효성 점검 대시보드</h1>
                    <p class="text-xs text-indigo-200">주기별 자주 검증 진척도 및 현장 품질 관리 모니터링</p>
                </div>
            </div>
            
            <!-- Data Source Controls -->
            <div class="flex flex-wrap items-center gap-2 text-sm">
                <div class="flex items-center bg-indigo-800/80 px-3 py-1.5 rounded-lg border border-indigo-700">
                    <input type="text" id="csvUrlInput" placeholder="구글 스프레드시트 CSV 웹 게시 링크 입력" class="bg-transparent text-xs text-white placeholder-indigo-300 focus:outline-none w-64 md:w-80">
                    <button onclick="loadFromUrl()" class="ml-2 bg-indigo-500 hover:bg-indigo-400 px-3 py-1 rounded text-xs font-medium transition">연동</button>
                </div>
                <label class="bg-white/10 hover:bg-white/20 px-3 py-1.5 rounded-lg cursor-pointer transition border border-indigo-700 text-xs flex items-center gap-1.5">
                    <i class="fa-solid fa-file-arrow-up"></i> 파일 업로드(CSV)
                    <input type="file" id="csvFileInput" accept=".csv" class="hidden" onchange="loadFromFile(event)">
                </label>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="max-w-7xl mx-auto px-4 py-6 flex-grow w-full space-y-6">

        <!-- KPI Summary Cards -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
            <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold text-slate-500 uppercase tracking-wider">전체 관리 항목</p>
                    <h3 id="kpiTotal" class="text-2xl font-bold text-slate-800 mt-1">0 건</h3>
                </div>
                <div class="w-12 h-12 bg-blue-50 text-blue-600 rounded-xl flex items-center justify-center text-xl font-bold">
                    <i class="fa-solid fa-clipboard-list"></i>
                </div>
            </div>

            <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold text-slate-500 uppercase tracking-wider">전체 점검 완료율</p>
                    <h3 id="kpiRate" class="text-2xl font-bold text-emerald-600 mt-1">0%</h3>
                </div>
                <div class="w-12 h-12 bg-emerald-50 text-emerald-600 rounded-xl flex items-center justify-center text-xl font-bold">
                    <i class="fa-solid fa-circle-check"></i>
                </div>
            </div>

            <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold text-slate-500 uppercase tracking-wider">기한 초과 (지연)</p>
                    <h3 id="kpiDelayed" class="text-2xl font-bold text-rose-600 mt-1">0 건</h3>
                </div>
                <div class="w-12 h-12 bg-rose-50 text-rose-600 rounded-xl flex items-center justify-center text-xl font-bold">
                    <i class="fa-solid fa-triangle-exclamation"></i>
                </div>
            </div>

            <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold text-slate-500 uppercase tracking-wider">진행중 협력사</p>
                    <h3 id="kpiPartners" class="text-2xl font-bold text-indigo-600 mt-1">0 개사</h3>
                </div>
                <div class="w-12 h-12 bg-indigo-50 text-indigo-600 rounded-xl flex items-center justify-center text-xl font-bold">
                    <i class="fa-solid fa-building-user"></i>
                </div>
            </div>
        </div>

        <!-- Charts Section -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <!-- 주기별 완료율 바 차트 -->
            <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-200 lg:col-span-2 flex flex-col">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="font-bold text-slate-700 text-sm flex items-center gap-2">
                        <i class="fa-solid fa-chart-column text-indigo-600"></i> 유효성 점검 주기별 완료 현황
                    </h2>
                    <span class="text-xs text-slate-400">1주차 ~ 6개월 누적 검증률</span>
                </div>
                <div class="relative flex-grow h-72">
                    <canvas id="periodChart"></canvas>
                </div>
            </div>

            <!-- OSA 협력사별 점중 분포 도넛 차트 -->
            <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-200 flex flex-col">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="font-bold text-slate-700 text-sm flex items-center gap-2">
                        <i class="fa-solid fa-chart-pie text-indigo-600"></i> OSA 협력사별 비중
                    </h2>
                </div>
                <div class="relative flex-grow h-72 flex items-center justify-center">
                    <canvas id="partnerChart"></canvas>
                </div>
            </div>
        </div>

        <!-- Detailed Items Table -->
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
            <div class="p-4 border-b border-slate-100 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3">
                <h2 class="font-bold text-slate-700 text-sm flex items-center gap-2">
                    <i class="fa-solid fa-table-list text-indigo-600"></i> 관리번호별 상세 점검 현황표
                </h2>
                <div class="flex items-center gap-2 w-full sm:w-auto">
                    <input type="text" id="searchInput" placeholder="관리번호, 품명, 고객사 검색..." oninput="filterTable()" class="bg-slate-50 border border-slate-200 px-3 py-1.5 rounded-lg text-xs w-full sm:w-64 focus:outline-none focus:border-indigo-500">
                </div>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse text-xs">
                    <thead>
                        <tr class="bg-slate-100 text-slate-600 uppercase font-semibold">
                            <th class="p-3">관리번호</th>
                            <th class="p-3">발생일</th>
                            <th class="p-3">OSA 협력사</th>
                            <th class="p-3">고객사</th>
                            <th class="p-3">품명 / 불량내용</th>
                            <th class="p-3 text-center">1주차</th>
                            <th class="p-3 text-center">2주차</th>
                            <th class="p-3 text-center">3주차</th>
                            <th class="p-3 text-center">4주차</th>
                            <th class="p-3 text-center">2개월</th>
                            <th class="p-3 text-center">3개월</th>
                            <th class="p-3 text-center">4개월</th>
                            <th class="p-3 text-center">6개월</th>
                        </tr>
                    </thead>
                    <tbody id="dataTableBody" class="divide-y divide-slate-100">
                        <!-- Dynamic Rows -->
                    </tbody>
                </table>
            </div>
        </div>

    </main>

    <!-- Footer -->
    <footer class="bg-white border-t border-slate-200 py-4 mt-auto">
        <div class="max-w-7xl mx-auto px-4 text-center text-xs text-slate-400">
            &copy; 2026 OSA Quality Management System. Powered by HTML & Tailwind CSS.
        </div>
    </footer>

    <!-- JavaScript Application Logic -->
    <script>
        // 주기 컬럼 정의 (스프레드시트 구조 기준)
        const PERIOD_KEYS = ['1주차', '2주차', '3주차', '4주차', '2개월', '3개월', '4개월', '6개월'];
        let globalData = [];
        let periodChartInstance = null;
        let partnerChartInstance = null;

        // 샘플 데이터 (초기 로딩용 혹은 파일 없을 시 테스트용)
        const sampleCSV = `관리번호,발생일,OSA,고객사,품명,불량내용,담당자 이메일,1주차_기준,1주차_실시,2주차_기준,2주차_실시,3주차_기준,3주차_실시,4주차_기준,4주차_실시,2개월_기준,2개월_실시,3개월_기준,3개월_실시,4개월_기준,4개월_실시,6개월_기준,6개월_실시
L26_001,2026-02-04,한림테크,HITACHI,ZX130,Valve측 plug 오조립,oks@dy.co.kr,2026-02-11,2026-03-10,2026-02-18,2026-03-10,2026-02-25,2026-03-10,2026-03-04,2026-05-11,2026-04-04,2026-05-11,2026-05-04,2026-05-11,2026-06-04,2026-05-11,2026-08-04,2026-05-11
L26_002,2026-05-08,KT&S,현대,HX85,ROD부 누유,oks@dy.co.kr,2026-05-15,2026-03-10,2026-05-22,2026-03-11,2026-05-29,2026-03-18,2026-06-05,2026-05-11,2026-07-08,,2026-08-08,,2026-09-08,,2026-11-08,
L26_003,2026-05-07,HIMC,VOLVO,E50,도장 손상,oks@dy.co.kr,2026-05-14,2026-03-10,2026-05-21,2026-03-10,2026-05-28,2026-03-18,2026-06-04,2026-03-25,2026-07-07,2026-03-26,2026-08-07,2026-03-26,2026-09-07,2026-03-26,2026-11-07,`;

        window.addEventListener('DOMContentLoaded', () => {
            parseAndRender(sampleCSV);
        });

        function loadFromFile(event) {
            const file = event.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                parseAndRender(e.target.result);
            };
            reader.readAsText(file, 'UTF-8');
        }

        async function loadFromUrl() {
            const url = document.getElementById('csvUrlInput').value.trim();
            if (!url) {
                alert('구글 스프레드시트 CSV 웹 게시 링크를 입력해주세요.');
                return;
            }
            try {
                const response = await fetch(url);
                const data = await response.text();
                parseAndRender(data);
            } catch (error) {
                alert('데이터를 불러오는 중 오류가 발생했습니다. CORS 정책 또는 URL을 확인해주세요.');
                console.error(error);
            }
        }

        // 간단한 CSV 파서 (따옴표 포함 데이터 처리)
        function parseCSV(text) {
            let lines = text.split('\n');
            let result = [];
            let headers = lines[0].split(',').map(h => h.trim());

            for (let i = 1; i < lines.length; i++) {
                if (!lines[i].trim()) continue;
                let row = [];
                let inQuotes = false;
                let entry = '';
                for (let char of lines[i]) {
                    if (char === '"') {
                        inQuotes = !inQuotes;
                    } else if (char === ',' && !inQuotes) {
                        row.push(entry.trim());
                        entry = '';
                    } else {
                        entry += char;
                    }
                }
                row.push(entry.trim());
                
                // 객체 매핑 (구조에 맞춰 유연하게 처리)
                let obj = {
                    관리번호: row[0] || '',
                    발생일: row[1] || '',
                    OSA: row[2] || '',
                    고객사: row[3] || '',
                    품명: row[4] || '',
                    불량내용: row[5] || '',
                    이메일: row[6] || '',
                    // 1주차~6개월 (실시일 기준 데이터 매칭 인덱스 가정)
                    periods: [
                        { name: '1주차', 기준: row[9] || '', 실시: row[10] || '' },
                        { name: '2주차', 기준: row[11] || '', 실시: row[12] || '' },
                        { name: '3주차', 기준: row[13] || '', 실시: row[14] || '' },
                        { name: '4주차', 기준: row[15] || '', 실시: row[16] || '' },
                        { name: '2개월', 기준: row[17] || '', 실시: row[18] || '' },
                        { name: '3개월', 기준: row[19] || '', 실시: row[20] || '' },
                        { name: '4개월', 기준: row[21] || '', 실시: row[22] || '' },
                        { name: '6개월', 기준: row[23] || '', 실시: row[24] || '' }
                    ]
                };
                result.push(obj);
            }
            return result;
        }

        function parseAndRender(csvText) {
            globalData = parseCSV(csvText);
            updateKPIs(globalData);
            renderCharts(globalData);
            renderTable(globalData);
        }

        function updateKPIs(data) {
            const total = data.length;
            let totalChecks = 0;
            let completedChecks = 0;
            let delayedCount = 0;
            let partners = new Set();

            const today = new Date();

            data.forEach(item => {
                partners.add(item.OSA);
                item.periods.forEach(p => {
                    if (p.기준) {
                        totalChecks++;
                        if (p.실시) {
                            completedChecks++;
                        } else {
                            // 기준일이 오늘보다 이전인데 실시일이 없으면 지연
                            let기준일 = new Date(p.기준);
                            if (기준일 < today) {
                                delayedCount++;
                            }
                        }
                    }
                });
            });

            const completionRate = totalChecks > 0 ? ((completedChecks / totalChecks) * 100).toFixed(1) : 0;

            document.getElementById('kpiTotal').innerText = `${total} 건`;
            document.getElementById('kpiRate').innerText = `${completionRate}%`;
            document.getElementById('kpiDelayed').innerText = `${delayedCount} 건`;
            document.getElementById('kpiPartners').innerText = `${partners.size} 개사`;
        }

        function renderCharts(data) {
            // 1. 주기별 완료율 계산
            let periodStats = PERIOD_KEYS.map(key => ({ name: key, total: 0, completed: 0 }));
            
            data.forEach(item => {
                item.periods.forEach((p, idx) => {
                    if (p.기준) {
                        periodStats[idx].total++;
                        if (p.실시) {
                            periodStats[idx].completed++;
                        }
                    }
                });
            });

            let periodRates = periodStats.map(s => s.total > 0 ? Math.round((s.completed / s.total) * 100) : 0);

            // 주기별 차트 렌더링
            const ctxPeriod = document.getElementById('periodChart').getContext('2d');
            if (periodChartInstance) periodChartInstance.destroy();

            periodChartInstance = new Chart(ctxPeriod, {
                type: 'bar',
                data: {
                    labels: PERIOD_KEYS,
                    datasets: [{
                        label: '주기별 완료율 (%)',
                        data: periodRates,
                        backgroundColor: 'rgba(99, 102, 241, 0.8)',
                        borderRadius: 6
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: { beginAtZero: true, max: 100 }
                    }
                }
            });

            // 2. 협력사별 건수 계산
            let partnerCounts = {};
            data.forEach(item => {
                partnerCounts[item.OSA] = (partnerCounts[item.OSA] || 0) + 1;
            });

            const ctxPartner = document.getElementById('partnerChart').getContext('2d');
            if (partnerChartInstance) partnerChartInstance.destroy();

            partnerChartInstance = new Chart(ctxPartner, {
                type: 'doughnut',
                data: {
                    labels: Object.keys(partnerCounts),
                    datasets: [{
                        data: Object.values(partnerCounts),
                        backgroundColor: ['#6366f1', '#3b82f6', '#10b981', '#f59e0b', '#ef4444']
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { position: 'bottom' }
                    }
                }
            });
        }

        function renderTable(data) {
            const tbody = document.getElementById('dataTableBody');
            tbody.innerHTML = '';
            const today = new Date();

            data.forEach(item => {
                let tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 transition border-b border-slate-100";

                let rowHtml = `
                    <td class="p-3 font-semibold text-indigo-600">${item.관리번호}</td>
                    <td class="p-3 text-slate-500">${item.발생일}</td>
                    <td class="p-3 font-medium">${item.OSA}</td>
                    <td class="p-3">${item.고객사}</td>
                    <td class="p-3">
                        <div class="font-medium text-slate-800">${item.품명}</div>
                        <div class="text-slate-400 text-[11px]">${item.불량내용}</div>
                    </td>
                `;

                item.periods.forEach(p => {
                    let badgeClass = "bg-slate-100 text-slate-400";
                    let text = "-";
                    if (p.기준) {
                        if (p.실시) {
                            badgeClass = "bg-emerald-100 text-emerald-700 font-semibold";
                            text = "완료";
                        } else {
                            let 기준일 = new Date(p.기준);
                            if (기준일 < today) {
                                badgeClass = "bg-rose-100 text-rose-700 font-bold animate-pulse";
                                text = "지연";
                            } else {
                                badgeClass = "bg-amber-100 text-amber-700";
                                text = "진행중";
                            }
                        }
                    }
                    rowHtml += `<td class="p-3 text-center"><span class="px-2 py-1 rounded text-[10px] ${badgeClass}">${text}</span></td>`;
                });

                tr.innerHTML = rowHtml;
                tbody.appendChild(tr);
            });
        }

        function filterTable() {
            const keyword = document.getElementById('searchInput').value.toLowerCase();
            const filtered = globalData.filter(item => 
                item.관리번호.toLowerCase().includes(keyword) ||
                item.품명.toLowerCase().includes(keyword) ||
                item.고객사.toLowerCase().includes(keyword) ||
                item.OSA.toLowerCase().includes(keyword)
            );
            renderTable(filtered);
        }
    </script>
</body>
</html>
