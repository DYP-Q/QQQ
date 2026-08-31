<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OSA 유효성 점검 실시간 상황판</title>
    <!-- CSV 파싱 라이브러리 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <style>
        :root {
            --primary: #2c3e50;
            --secondary: #34495e;
            --success-bg: #e8f5e9; --success-text: #2e7d32;
            --warning-bg: #fff3e0; --warning-text: #e65100;
            --danger-bg: #ffebee;  --danger-text: #c62828;
            --pending-bg: #f5f5f5; --pending-text: #757575;
            --border: #cfd8dc;
        }
        body { font-family: 'Malgun Gothic', '맑은 고딕', sans-serif; background-color: #f4f7f6; margin: 0; padding: 20px; color: #333; font-size: 13px; }
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
        h1 { margin: 0; color: var(--primary); font-size: 22px; }
        
        /* 상단 요약 카드 스타일 */
        .company-summary-container { display: flex; flex-wrap: wrap; gap: 15px; margin-bottom: 20px; }
        .company-card { 
            flex: 1; min-width: 180px; padding: 15px; border-radius: 8px; border: 1px solid var(--border); 
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); text-align: center; font-weight: bold; background: white;
        }
        .company-card .name { font-size: 15px; margin-bottom: 10px; color: #333; }
        .company-card .status { font-size: 22px; margin-bottom: 8px; }
        .company-card .details { font-size: 13px; color: #555; }
        .company-card .sub-details { font-size: 11px; color: #888; margin-top: 4px; font-weight: normal; }
        
        .card-good { border-top: 5px solid var(--success-text); }
        .card-good .status { color: var(--success-text); }
        .card-warn { border-top: 5px solid var(--warning-text); background-color: var(--warning-bg); }
        .card-warn .status { color: var(--warning-text); }
        .card-danger { border-top: 5px solid var(--danger-text); background-color: var(--danger-bg); }
        .card-danger .status { color: var(--danger-text); }

        /* 메인 테이블 스타일 */
        .table-container { background: white; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); overflow-x: auto; border: 1px solid var(--border); }
        table { width: 100%; border-collapse: collapse; text-align: center; white-space: nowrap; }
        th, td { padding: 10px; border: 1px solid var(--border); }
        th { background-color: var(--secondary); color: white; position: sticky; top: 0; }
        
        .stage-cell { font-size: 12px; line-height: 1.4; }
        .date-target { color: #555; }
        .date-actual { font-weight: bold; }
        .badge { font-weight: bold; border-radius: 4px; padding: 3px 6px; display: inline-block; margin-top: 5px; }
        .badge-success { background-color: var(--success-bg); color: var(--success-text); }
        .badge-warning { background-color: var(--warning-bg); color: var(--warning-text); }
        .badge-danger { background-color: var(--danger-bg); color: var(--danger-text); }
        .badge-pending { background-color: var(--pending-bg); color: var(--pending-text); font-weight: normal; }

        .message-box { font-size: 15px; font-weight: bold; text-align: center; padding: 40px; line-height: 1.6; }
        .error-msg { color: var(--danger-text); }
        .loading-msg { color: var(--primary); }
    </style>
</head>
<body>

    <div class="header">
        <h1>📑 OSA 유효성 점검 상황판 (실시간 연동)</h1>
        <div id="todayDate" style="font-weight:bold; color:#7f8c8d; font-size: 14px;"></div>
    </div>

    <!-- 업체별 현황 요약 (평균 데이터) -->
    <div class="company-summary-container" id="companySummary"></div>

    <!-- 메인 테이블 -->
    <div class="table-container">
        <table id="mainTable">
            <thead>
                <tr>
                    <th>관리번호</th>
                    <th>발생일</th>
                    <th>OSA</th>
                    <th>고객사</th>
                    <th>품명</th>
                    <th>1주차</th>
                    <th>2주차</th>
                    <th>3주차</th>
                    <th>4주차</th>
                    <th>2개월</th>
                    <th>3개월</th>
                    <th>4개월</th>
                    <th>6개월</th>
                    <th style="background-color: #2c3e50;">해당 건 지연합계</th>
                </tr>
            </thead>
            <tbody id="dataTableBody">
                <tr><td colspan="14" class="message-box loading-msg">데이터를 실시간으로 불러오는 중입니다...<br><span style="font-size:12px; font-weight:normal; color:#777;">(네트워크 상태에 따라 2~5초 정도 소요될 수 있습니다)</span></td></tr>
            </tbody>
        </table>
    </div>

    <script>
        document.getElementById('todayDate').innerText = `조회 일시: ${new Date().toLocaleString('ko-KR')}`;
        const csvUrl = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vRCGmQTOQd1DK4cmMzKU618FjIvvcwcSCgX3PBwtNF0i7_Q6aK3Hux-W56QCwAwNY7O2bff6zZ01RZm/pub?gid=359651245&single=true&output=csv';

        let parsedData = [];

        // 데이터 불러오기 (CORS 우회 로직 포함)
        async function fetchSpreadsheetData() {
            try {
                // 1차 시도: 직접 호출
                const response = await fetch(csvUrl);
                if (!response.ok) throw new Error('직접 연결 실패');
                const text = await response.text();
                parseCSV(text);
            } catch (error) {
                console.warn("직접 연결 실패. CORS 우회 프록시를 통해 재시도합니다...");
                try {
                    // 2차 시도: 프록시 서버를 경유하여 보안 우회 (로컬 실행 시 브라우저 차단 우회용)
                    const proxyUrl = 'https://api.allorigins.win/raw?url=' + encodeURIComponent(csvUrl);
                    const proxyResponse = await fetch(proxyUrl);
                    if (!proxyResponse.ok) throw new Error('프록시 연결 실패');
                    const text = await proxyResponse.text();
                    parseCSV(text);
                } catch (proxyError) {
                    showError("데이터 연결에 완전히 실패했습니다.<br>구글 시트가 웹에 올바르게 게시되었는지 확인하거나 인터넷 연결 상태를 점검해주세요.");
                }
            }
        }

        // CSV 텍스트 파싱
        function parseCSV(csvText) {
            Papa.parse(csvText, {
                header: false,
                skipEmptyLines: true,
                complete: function(results) {
                    processData(results.data);
                }
            });
        }

        function showError(msg) {
            document.getElementById('dataTableBody').innerHTML = `<tr><td colspan="14" class="message-box error-msg">${msg}</td></tr>`;
        }

        // 날짜 텍스트 정제
        function cleanDate(dateStr) {
            if (!dateStr || String(dateStr).trim() === "") return null;
            const match = String(dateStr).match(/(\d{4})[^\d]+(\d{1,2})[^\d]+(\d{1,2})/);
            if (match) {
                return `${match[1]}-${match[2].padStart(2, '0')}-${match[3].padStart(2, '0')}`;
            }
            return null;
        }

        // 데이터 처리 로직
        function processData(rows) {
            const resultList = [];
            
            for (let i = 1; i < rows.length; i++) {
                const currRow = rows[i];
                const id = String(currRow[0]).trim();
                
                if (id && id !== '관리번호') {
                    const prevRow = rows[i - 1] || [];
                    
                    let targetRow = prevRow;
                    let actualRow = currRow;
                    let dateColStart = 9;

                    const actualIdx = currRow.findIndex(c => String(c).replace(/\s/g, '').includes('점검실시일'));
                    if (actualIdx !== -1) {
                        dateColStart = actualIdx;
                    }

                    const item = {
                        id: id,
                        date: String(currRow[1]).trim(),
                        company: String(currRow[2]).trim(),
                        client: String(currRow[3]).trim(),
                        product: String(currRow[4]).trim(),
                        stages: []
                    };

                    for (let k = 1; k <= 8; k++) {
                        item.stages.push({
                            target: cleanDate(targetRow[dateColStart + k]),
                            actual: cleanDate(actualRow[dateColStart + k])
                        });
                    }
                    resultList.push(item);
                }
            }
            parsedData = resultList;
            renderDashboard();
        }

        // 지연 상태 계산
        function getStageStatus(targetRaw, actualRaw) {
            const targetStr = cleanDate(targetRaw);
            const actualStr = cleanDate(actualRaw);
            
            if (!targetStr) return { html: '-', delayDays: 0 };
            const targetFormat = targetStr.replace(/-/g, '. ');
            
            if (!actualStr) {
                return {
                    html: `<div class="stage-cell">
                             <div class="date-target">기준: ${targetFormat}</div>
                             <div class="date-actual" style="color:#aaa;">실시: -</div>
                             <div class="badge badge-pending">미진행</div>
                           </div>`,
                    delayDays: 0
                };
            }

            const t = new Date(targetStr);
            const a = new Date(actualStr);
            const actualFormat = actualStr.replace(/-/g, '. ');
            
            const diffDays = Math.floor((a - t) / (1000 * 60 * 60 * 24));
            
            if (diffDays <= 2) {
                return {
                    html: `<div class="stage-cell">
                             <div class="date-target">기준: ${targetFormat}</div>
                             <div class="date-actual">실시: ${actualFormat}</div>
                             <div class="badge badge-success">정상</div>
                           </div>`,
                    delayDays: 0 
                };
            } else if (diffDays <= 5) {
                return {
                    html: `<div class="stage-cell">
                             <div class="date-target">기준: ${targetFormat}</div>
                             <div class="date-actual" style="color:var(--warning-text);">실시: ${actualFormat}</div>
                             <div class="badge badge-warning">지연 (${diffDays}일)</div>
                           </div>`,
                    delayDays: diffDays
                };
            } else {
                return {
                    html: `<div class="stage-cell">
                             <div class="date-target">기준: ${targetFormat}</div>
                             <div class="date-actual" style="color:var(--danger-text);">실시: ${actualFormat}</div>
                             <div class="badge badge-danger">지연 (${diffDays}일)</div>
                           </div>`,
                    delayDays: diffDays
                };
            }
        }

        // 화면 렌더링
        function renderDashboard() {
            const tbody = document.getElementById('dataTableBody');
            const summaryContainer = document.getElementById('companySummary');
            
            if (parsedData.length === 0) {
                showError("데이터를 찾을 수 없습니다. 스프레드시트 구조를 확인해주세요.");
                return;
            }

            tbody.innerHTML = '';
            summaryContainer.innerHTML = '';
            
            // 업체별 통계 객체 (총 지연일, 등록된 아이템 수)
            const companyStats = {}; 

            // 메인 테이블 생성
            parsedData.forEach(item => {
                const tr = document.createElement('tr');
                let rowDelayTotal = 0;
                let stagesHtml = '';

                item.stages.forEach(stage => {
                    const status = getStageStatus(stage.target, stage.actual);
                    stagesHtml += `<td>${status.html}</td>`;
                    rowDelayTotal += status.delayDays;
                });

                // 통계 누적
                if (!companyStats[item.company]) {
                    companyStats[item.company] = { totalDelay: 0, itemCount: 0 };
                }
                companyStats[item.company].totalDelay += rowDelayTotal;
                companyStats[item.company].itemCount += 1;

                // 행별 지연일 색상 결정
                let totalColor = '#2e7d32'; // 초록
                if (rowDelayTotal > 5) totalColor = '#c62828'; // 5일 초과: 빨강
                else if (rowDelayTotal > 0) totalColor = '#e65100'; // 1~5일: 주황

                tr.innerHTML = `
                    <td><strong>${item.id}</strong></td>
                    <td>${item.date}</td>
                    <td>${item.company}</td>
                    <td>${item.client}</td>
                    <td>${item.product}</td>
                    ${stagesHtml}
                    <td style="font-size: 15px; font-weight: bold; color: ${totalColor};">
                        ${rowDelayTotal} 일
                    </td>
                `;
                tbody.appendChild(tr);
            });

            // 상단 업체별 요약 카드 생성 (평균 기반)
            Object.keys(companyStats).forEach(company => {
                const stats = companyStats[company];
                // 평균 지연일 계산 (소수점 1자리까지 표시)
                const avgDelay = (stats.totalDelay / stats.itemCount).toFixed(1);

                let cardClass = 'card-good';
                let statusText = '정상';

                // 평균 기준 판별
                if (avgDelay > 5) {
                    cardClass = 'card-danger';
                    statusText = '위험 (심각한 지연)';
                } else if (avgDelay > 0) {
                    cardClass = 'card-warn';
                    statusText = '지연 (주의 요망)';
                }

                const card = document.createElement('div');
                card.className = `company-card ${cardClass}`;
                card.innerHTML = `
                    <div class="name">${company}</div>
                    <div class="status">${statusText}</div>
                    <div class="details">평균 지연: <strong>${avgDelay}일</strong></div>
                    <div class="sub-details">(총 ${stats.itemCount}건 / 누적 ${stats.totalDelay}일 지연)</div>
                `;
                summaryContainer.appendChild(card);
            });
        }

        // 최초 실행
        fetchSpreadsheetData();
    </script>
</body>
</html>
