<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OSA 개선 대책 유효성 점검 대시보드</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen">

    <!-- Header (슬림하게 폭 조정) -->
    <header class="bg-indigo-900 text-white shadow-md">
        <div class="max-w-7xl mx-auto px-4 py-3 flex flex-col md:flex-row justify-between items-center">
            <div>
                <h1 class="text-xl font-bold">OSA 개선 대책 유효성 점검 대시보드</h1>
                <p class="text-indigo-200 text-xs mt-0.5">주기별 자주 검증 진척도 및 현장 품질 관리 실시간 모니터링</p>
            </div>
            <div class="mt-2 md:mt-0 flex items-center space-x-3">
                <a href="https://docs.google.com/spreadsheets/d/1CAAK11qrimDmHXvii7zRcUaWFTY6ENMT-LB7q5nrJz4/edit?gid=359651245#gid=359651245" 
                   target="_blank" 
                   class="inline-flex items-center px-3 py-1 rounded-lg text-xs font-semibold bg-indigo-800 hover:bg-indigo-700 text-indigo-100 transition-colors border border-indigo-700 shadow-sm">
                    📊 원본 스프레드시트 열기
                </a>
                <span id="sync-status" class="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-medium bg-emerald-100 text-emerald-800">
                    <span class="w-2 h-2 mr-1.5 bg-emerald-500 rounded-full animate-pulse"></span> 연동 중
                </span>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="max-w-7xl mx-auto px-4 py-6 space-y-6">

        <!-- KPI Cards Grid -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500">전체 관리 항목</p>
                    <h3 id="kpi-total" class="text-2xl font-bold text-slate-900 mt-1">0 건</h3>
                </div>
                <div class="p-3 bg-indigo-50 text-indigo-600 rounded-xl text-lg font-bold">📋</div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500">전체 점검 완료율</p>
                    <h3 id="kpi-rate" class="text-2xl font-bold text-emerald-600 mt-1">0%</h3>
                </div>
                <div class="p-3 bg-emerald-50 text-emerald-600 rounded-xl text-lg font-bold">📈</div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500">기한 초과 (지연)</p>
                    <h3 id="kpi-delayed" class="text-2xl font-bold text-amber-600 mt-1">0 건</h3>
                </div>
                <div class="p-3 bg-amber-50 text-amber-600 rounded-xl text-lg font-bold">⚠</div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500">참여 협력사 수</p>
                    <h3 id="kpi-partners" class="text-2xl font-bold text-blue-600 mt-1">0 개사</h3>
                </div>
                <div class="p-3 bg-blue-50 text-blue-600 rounded-xl text-lg font-bold">🏢</div>
            </div>
        </div>

        <!-- Detailed Table Section -->
        <div class="bg-white rounded-2xl shadow-sm border border-slate-100 overflow-hidden">
            <div class="p-5 border-b border-slate-100 flex flex-col sm:flex-row justify-between items-center gap-4">
                <div>
                    <h3 class="text-base font-bold text-slate-900">개선 대책 상세 진행 목록</h3>
                    <p class="text-xs text-slate-400 mt-0.5">* 관리번호 표기 형식: 업체명_년도_순번</p>
                </div>
                <!-- 협력사별 필터 셀렉트박스 -->
                <div class="flex items-center space-x-2">
                    <label for="partner-filter" class="text-xs font-semibold text-slate-600">OSA 필터:</label>
                    <select id="partner-filter" onchange="filterTable()" class="px-3 py-1.5 text-xs bg-slate-50 border border-slate-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500 text-slate-700 font-medium">
                        <option value="ALL">전체 협력사 보기</option>
                    </select>
                </div>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-50 text-slate-500 text-xs uppercase font-semibold">
                            <th class="p-3.5 w-32">관리번호</th>
                            <th class="p-3.5 w-28">발생일</th>
                            <th class="p-3.5 w-28">OSA</th>
                            <th class="p-3.5 w-32">고객사</th>
                            <th class="p-3.5 w-36">품명</th>
                            <th class="p-3.5">불량내용</th>
                        </tr>
                    </thead>
                    <tbody id="table-body" class="divide-y divide-slate-100 text-sm">
                        <tr>
                            <td colspan="6" class="p-6 text-center text-slate-400">데이터를 불러오는 중입니다...</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

    </main>

    <!-- JavaScript 로직 -->
    <script>
        const SHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vRCGmQTOQd1DK4cmMzKU618FjIvvcwcSCgX3PBwtNF0i7_Q6aK3Hux-W56QCwAwNY7O2bff6zZ01RZm/pub?gid=359651245&single=true&output=csv";

        let allParsedData = []; // 원본 파싱 데이터 보관용

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

        function parseDate(str) {
            if (!str) return null;
            let clean = str.replace(/\./g, '-').replace(/\s+/g, ' ').trim();
            let datePart = clean.split(' ')[0];
            let parts = datePart.split('-');
            if (parts.length >= 3) {
                let y = parseInt(parts[0]);
                let m = parseInt(parts[1]) - 1;
                let d = parseInt(parts[2]);
                return new Date(y, m, d);
            }
            return null;
        }

        function parseCSVAndRender(csvText) {
            const rows = parseCSVToArray(csvText);
            if (rows.length < 2) return;

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
            let delayedCount = 0;
            let partnersSet = new Set();
            let parsedItems = [];
            
            let weeklyCompleted = [0, 0, 0, 0, 0, 0, 0, 0];
            let partnerCounts = {};

            const periodLabels = ['1주', '2주', '3주', '4주', '2달', '3달', '4달', '6달'];

            let i = 0;
            while (i < dataRows.length) {
                const row1 = dataRows[i];
                
                if (row1 && row1[0] && row1[0].trim() !== '' && !row1[0].includes('점검') && !row1[0].includes('기준일')) {
                    const id = row1[0].trim();

                    // 숨겨진 행 완벽 제외[cite: 5]
                    if (['L26_001', 'L26_002', 'L26_003'].includes(id)) {
                        i++;
                        continue;
                    }

                    const date = row1[1] || '';
                    const osa = row1[2] || '';
                    const client = row1[3] || '';
                    const productName = row1[4] || '';
                    const defect = row1[5] || '';

                    totalCount++;
                    if (osa) partnersSet.add(osa);
                    partnerCounts[osa] = (partnerCounts[osa] || 0) + 1;

                    let rowStandard = [];
                    let rowExecution = [];
                    
                    for (let k = 1; k <= 3; k++) {
                        if (i + k < dataRows.length) {
                            let candidate = dataRows[i + k];
                            if (candidate[0] && candidate[0].trim() !== '' && !candidate[0].includes('점검')) {
                                break;
                            }
                            if (candidate.some(cell => cell.includes('점검 기준일'))) {
                                rowStandard = candidate;
                            }
                            if (candidate.some(cell => cell.includes('점검 실시일'))) {
                                rowExecution = candidate;
                            }
                        }
                    }

                    let gaugeHtml = '<div class="flex space-x-1.5 items-center">';
                    let itemDelayed = false;

                    for (let col = 10; col <= 17; col++) {
                        let idx = col - 10;
                        let execVal = rowExecution[col] ? rowExecution[col].trim() : '';
                        let stdVal = rowStandard[col] ? rowStandard[col].trim() : '';

                        if (execVal.length > 5) {
                            let dtExec = parseDate(execVal);
                            let dtStd = parseDate(stdVal);

                            let badgeColor = 'bg-emerald-500 text-white'; // 초록색 (2일 이내)
                            let statusText = '정상 완료';

                            if (dtExec && dtStd) {
                                let diffTime = dtExec - dtStd;
                                let diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24)); // 실시일 - 기준일

                                // 5일 초과: 빨간색, 3일~5일 이내: 주황색, 2일 이내: 초록색
                                if (diffDays > 5) {
                                    badgeColor = 'bg-rose-500 text-white'; // 빨간색
                                    statusText = `지연 (${diffDays}일 초과)`;
                                    delayedCount++;
                                    itemDelayed = true;
                                } else if (diffDays > 2) {
                                    badgeColor = 'bg-amber-500 text-white'; // 주황색
                                    statusText = `지연 (${diffDays}일 초과)`;
                                    delayedCount++;
                                    itemDelayed = true;
                                } else {
                                    weeklyCompleted[idx]++;
                                }
                            } else {
                                weeklyCompleted[idx]++;
                            }

                            let titleText = `${periodLabels[idx]} 완료 (${execVal}) - ${statusText}`;
                            gaugeHtml += `<span class="px-2 py-1 text-xs font-bold rounded ${badgeColor} shadow-sm" title="${titleText}">${periodLabels[idx]}</span>`;
                        } else {
                            gaugeHtml += `<span class="px-2 py-1 text-xs font-semibold rounded bg-slate-200 text-slate-500" title="${periodLabels[idx]} 미완료">${periodLabels[idx]}</span>`;
                        }
                    }
                    gaugeHtml += '</div>';

                    parsedItems.push({
                        id, date, osa, client, productName, defect, gaugeHtml
                    });
                }
                i++;
            }

            allParsedData = parsedItems;

            // KPI 업데이트
            document.getElementById('kpi-total').innerText = totalCount + " 건";
            document.getElementById('kpi-partners').innerText = partnersSet.size + " 개사";
            document.getElementById('kpi-delayed').innerText = delayedCount + " 건";
            
            let totalPossible = totalCount * 8;
            let totalDone = weeklyCompleted.reduce((a, b) => a + b, 0);
            let completionRate = totalPossible > 0 ? Math.round((totalDone / totalPossible) * 100) : 0;
            document.getElementById('kpi-rate').innerText = completionRate + "%";

            // 필터 옵션 업데이트
            updateFilterOptions(partnersSet);

            // 테이블 렌더링
            renderTable(allParsedData);
        }

        function updateFilterOptions(partnersSet) {
            const selectEl = document.getElementById('partner-filter');
            const currentVal = selectEl.value;
            let optionsHtml = '<option value="ALL">전체 협력사 보기</option>';
            partnersSet.forEach(partner => {
                optionsHtml += `<option value="${partner}">${partner}</option>`;
            });
            selectEl.innerHTML = optionsHtml;
            selectEl.value = currentVal;
        }

        function filterTable() {
            const selectedPartner = document.getElementById('partner-filter').value;
            if (selectedPartner === 'ALL') {
                renderTable(allParsedData);
            } else {
                const filtered = allParsedData.filter(item => item.osa === selectedPartner);
                renderTable(filtered);
            }
        }

        function renderTable(items) {
            let tableHtml = '';
            items.forEach(item => {
                tableHtml += `
                    <tr class="hover:bg-slate-50/50 transition-colors">
                        <td class="p-3 font-bold text-slate-900" rowspan="2" style="vertical-align: middle;">${item.id}</td>
                        <td class="p-3 text-slate-600 text-xs">${item.date}</td>
                        <td class="p-3 font-semibold text-indigo-900 text-xs">${item.osa}</td>
                        <td class="p-3 text-slate-600 text-xs">${item.client}</td>
                        <td class="p-3 text-slate-600 text-xs">${item.productName}</td>
                        <td class="p-3 text-slate-600 text-xs">${item.defect}</td>
                    </tr>
                    <tr class="hover:bg-slate-50/50 transition-colors bg-slate-50/60 border-b border-slate-200">
                        <td class="p-3" colspan="5">
                            <div class="flex items-center space-x-3">
                                <span class="text-xs font-bold text-slate-500">기간별 진행 상태:</span>
                                ${item.gaugeHtml}
                            </div>
                        </td>
                    </tr>
                `;
            });

            document.getElementById('table-body').innerHTML = tableHtml || `<tr><td colspan="6" class="p-6 text-center text-slate-400">조건에 해당하는 데이터가 없습니다.</td></tr>`;
        }

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
    </script>
</body>
</html>
