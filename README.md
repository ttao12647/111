<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>基金持仓管理系统（稳定版）</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.4.3/dist/echarts.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: "Microsoft YaHei", sans-serif;
        }
        body {
            background: #f5f7fa;
            padding: 15px;
            max-width: 1200px;
            margin: 0 auto;
        }
        .container {
            background: #fff;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.08);
            padding: 20px;
            margin-bottom: 15px;
        }
        h1 {
            font-size: 20px;
            color: #333;
            margin-bottom: 18px;
            text-align: center;
            font-weight: 600;
        }
        h2 {
            font-size: 16px;
            color: #444;
            margin: 12px 0 8px;
            border-left: 4px solid #1890ff;
            padding-left: 8px;
        }
        .add-form {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            align-items: center;
            margin-bottom: 18px;
        }
        .add-form input {
            flex: 1;
            min-width: 100px;
            padding: 8px 12px;
            border: 1px solid #e5e6eb;
            border-radius: 6px;
            outline: none;
            font-size: 14px;
        }
        .add-form input:focus {
            border-color: #1890ff;
        }
        .btn {
            padding: 8px 16px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 500;
            transition: all 0.2s;
        }
        .btn-primary {
            background: #1890ff;
            color: #fff;
        }
        .btn-primary:hover {
            background: #096dd9;
        }
        .btn-danger {
            background: #ff4d4f;
            color: #fff;
            padding: 6px 12px;
            font-size: 12px;
        }
        .btn-danger:hover {
            background: #d9363e;
        }
        .btn-refresh {
            background: #52c41a;
            color: #fff;
            padding: 6px 12px;
            font-size: 12px;
            margin-right: 4px;
        }
        .btn-refresh:hover {
            background: #389e0d;
        }
        .fund-table {
            width: 100%;
            border-collapse: collapse;
            margin: 8px 0;
            font-size: 12px;
        }
        .fund-table th, .fund-table td {
            padding: 8px 4px;
            text-align: center;
            border: 1px solid #e5e6eb;
        }
        .fund-table th {
            background: #f8f9fa;
            color: #666;
            font-weight: 600;
        }
        .fund-table tr:hover {
            background: #fafafa;
        }
        .profit {
            color: #ff4d4f;
            font-weight: 600;
        }
        .loss {
            color: #52c41a;
            font-weight: 600;
        }
        .total-info {
            font-size: 14px;
            font-weight: 600;
            margin: 12px 0;
            padding: 8px;
            background: #f0f9ff;
            border-radius: 6px;
            text-align: center;
        }
        .total-info span {
            margin: 0 6px;
        }
        #chart-container {
            width: 100%;
            height: 300px;
            margin-top: 15px;
        }
        .tip {
            font-size: 11px;
            color: #999;
            margin-top: 8px;
            text-align: center;
            line-height: 1.4;
        }
        .loading {
            text-align: center;
            padding: 10px;
            color: #1890ff;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>基金持仓管理系统（稳定版）</h1>
        <div class="add-form">
            <input type="text" id="fundCode" placeholder="基金代码（如000001）" required>
            <input type="number" id="fundShare" step="0.01" placeholder="持有份额" required>
            <input type="number" id="costPrice" step="0.001" placeholder="成本价" required>
            <button class="btn btn-primary" onclick="addFund()">添加基金</button>
        </div>

        <div class="total-info" id="totalInfo">
            持仓总本金：0.00元 | 总市值：0.00元 | 总收益：0.00元 | 总收益率：0.00%
        </div>

        <h2>我的持仓</h2>
        <table class="fund-table" id="fundTable">
            <thead>
                <tr>
                    <th>代码</th>
                    <th>名称</th>
                    <th>最新净值</th>
                    <th>当日涨幅</th>
                    <th>持仓成本</th>
                    <th>份额</th>
                    <th>市值</th>
                    <th>收益</th>
                    <th>收益率</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody id="fundList">
                <tr><td colspan="10" style="color: #999;">暂无持仓，添加后显示</td></tr>
            </tbody>
        </table>

        <h2>净值走势（近30天）</h2>
        <div id="chart-container"></div>
        <div class="tip">
            💡 提示：1. 优先用电脑Chrome/Edge浏览器打开更稳定；2. 数据保存在浏览器本地，清除缓存会丢失；3. 仅支持国内公募基金纯数字代码。
        </div>
    </div>

    <script>
        let myChart = echarts.init(document.getElementById('chart-container'));
        myChart.setOption({
            title: { text: '请选择基金查看走势', fontSize: 14 },
            xAxis: { type: 'category', data: [] },
            yAxis: { type: 'value' },
            series: [{ name: '净值', type: 'line', data: [], smooth: true }],
            tooltip: { trigger: 'axis' },
            grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true }
        });

        const FUND_KEY = 'myFundListStable';
        // 更换为更稳定的跨域代理
        const PROXY_URL = 'https://corsproxy.io/?';
        const FUND_INFO_API = (code) => PROXY_URL + encodeURIComponent(`https://fund.eastmoney.com/pingzhongdata/${code}.js`);
        const FUND_KLINE_API = (code) => PROXY_URL + encodeURIComponent(`https://api.fund.eastmoney.com/f10/lsjz?fundCode=${code}&page=1&size=30&_=${new Date().getTime()}`);

        $(function() {
            renderFundList();
        });

        function addFund() {
            const code = $('#fundCode').val().trim();
            const share = parseFloat($('#fundShare').val());
            const cost = parseFloat($('#costPrice').val());

            if (!code || isNaN(share) || share <= 0 || isNaN(cost) || cost <= 0) {
                alert('请输入有效的基金代码、份额（>0）、成本价（>0）');
                return;
            }

            $('.add-form .btn-primary').text('加载中...').prop('disabled', true);

            getFundInfo(code, (fundInfo) => {
                $('.add-form .btn-primary').text('添加基金').prop('disabled', false);
                if (!fundInfo) {
                    alert('❌ 基金代码无效或接口请求失败，请检查代码后重试');
                    return;
                }

                let fundList = JSON.parse(localStorage.getItem(FUND_KEY)) || [];
                if (fundList.some(item => item.code === code)) {
                    alert('⚠️ 该基金已在持仓中，请勿重复添加');
                    return;
                }

                const fundData = {
                    code: code,
                    name: fundInfo.name,
                    share: share,
                    costPrice: cost,
                    netValue: fundInfo.netValue,
                    riseRate: fundInfo.riseRate,
                    marketValue: (share * fundInfo.netValue).toFixed(2),
                    profit: ((fundInfo.netValue - cost) * share).toFixed(2),
                    profitRate: (((fundInfo.netValue - cost) / cost) * 100).toFixed(2)
                };

                fundList.push(fundData);
                localStorage.setItem(FUND_KEY, JSON.stringify(fundList));
                renderFundList();

                $('#fundCode').val('');
                $('#fundShare').val('');
                $('#costPrice').val('');
                alert('✅ 基金添加成功！');
            });
        }

        function getFundInfo(code, callback) {
            $.ajax({
                url: FUND_INFO_API(code),
                type: 'get',
                dataType: 'script',
                timeout: 10000,
                success: function() {
                    try {
                        const fundName = window.fundChangerInfo?.name || '未知名称';
                        const trend = window.Data_netWorthTrend || [];
                        if (trend.length < 2) {
                            callback(null);
                            return;
                        }
                        const netValue = parseFloat(trend[trend.length - 1].y);
                        const prevValue = parseFloat(trend[trend.length - 2].y);
                        const riseRate = prevValue > 0 ? (((netValue / prevValue) - 1) * 100).toFixed(2) + '%' : '0.00%';
                        callback({ name: fundName, netValue: netValue, riseRate: riseRate });
                    } catch (e) {
                        console.error('解析基金信息失败:', e);
                        callback(null);
                    }
                },
                error: function(xhr, status, error) {
                    console.error('请求基金信息失败:', status, error);
                    callback(null);
                }
            });
        }

        function getFundKline(code, name) {
            $.ajax({
                url: FUND_KLINE_API(code),
                type: 'get',
                dataType: 'json',
                timeout: 10000,
                success: function(res) {
                    if (res?.Data?.LSJZList) {
                        const klineData = res.Data.LSJZList.reverse();
                        const xData = klineData.map(item => item.FSRQ);
                        const yData = klineData.map(item => parseFloat(item.DWJZ));
                        myChart.setOption({
                            title: { text: `${name}（${code}）近30天净值` },
                            xAxis: { data: xData },
                            series: [{ data: yData }]
                        });
                    } else {
                        alert('获取走势数据失败');
                    }
                },
                error: function() {
                    alert('获取走势数据超时，请稍后重试');
                }
            });
        }

        function refreshFund(code, index) {
            $('.btn-refresh').text('刷新中...').prop('disabled', true);
            getFundInfo(code, (fundInfo) => {
                $('.btn-refresh').text('刷新').prop('disabled', false);
                if (!fundInfo) {
                    alert('❌ 刷新失败，请检查网络');
                    return;
                }
                let fundList = JSON.parse(localStorage.getItem(FUND_KEY)) || [];
                const fund = fundList[index];
                fund.netValue = fundInfo.netValue;
                fund.riseRate = fundInfo.riseRate;
                fund.marketValue = (fund.share * fundInfo.netValue).toFixed(2);
                fund.profit = ((fundInfo.netValue - fund.costPrice) * fund.share).toFixed(2);
                fund.profitRate = (((fundInfo.netValue - fund.costPrice) / fund.costPrice) * 100).toFixed(2);
                fundList[index] = fund;
                localStorage.setItem(FUND_KEY, JSON.stringify(fundList));
                renderFundList();
                alert('✅ 刷新成功！');
            });
        }

        function deleteFund(index) {
            if (confirm('确定要删除该持仓吗？')) {
                let fundList = JSON.parse(localStorage.getItem(FUND_KEY)) || [];
                fundList.splice(index, 1);
                localStorage.setItem(FUND_KEY, JSON.stringify(fundList));
                renderFundList();
                myChart.setOption({
                    title: { text: '请选择基金查看走势' },
                    xAxis: { data: [] },
                    series: [{ data: [] }]
                });
            }
        }

        function renderFundList() {
            const fundList = JSON.parse(localStorage.getItem(FUND_KEY)) || [];
            const $fundList = $('#fundList');
            $fundList.empty();

            let totalCost = 0, totalMarket = 0, totalProfit = 0;

            if (fundList.length === 0) {
                $fundList.html('<tr><td colspan="10" style="color: #999;">暂无持仓，添加后显示</td></tr>');
                $('#totalInfo').text('持仓总本金：0.00元 | 总市值：0.00元 | 总收益：0.00元 | 总收益率：0.00%');
                return;
            }

            fundList.forEach((fund, index) => {
                const cost = (fund.share * fund.costPrice).toFixed(2);
                totalCost += parseFloat(cost);
                totalMarket += parseFloat(fund.marketValue);
                totalProfit += parseFloat(fund.profit);

                const profitClass = parseFloat(fund.profit) >= 0 ? 'profit' : 'loss';
                const rateClass = parseFloat(fund.profitRate) >= 0 ? 'profit' : 'loss';
                const riseClass = fund.riseRate.includes('-') ? 'loss' : 'profit';

                const tr = `
                    <tr onclick="getFundKline('${fund.code}', '${fund.name}')">
                        <td>${fund.code}</td>
                        <td title="点击查看走势">${fund.name}</td>
                        <td>${fund.netValue}</td>
                        <td class="${riseClass}">${fund.riseRate}</td>
                        <td>${cost}</td>
                        <td>${fund.share}</td>
                        <td>${fund.marketValue}</td>
                        <td class="${profitClass}">${fund.profit}</td>
                        <td class="${rateClass}">${fund.profitRate}%</td>
                        <td>
                            <button class="btn btn-refresh" onclick="event.stopPropagation();refreshFund('${fund.code}', ${index})">刷新</button>
                            <button class="btn btn-danger" onclick="event.stopPropagation();deleteFund(${index})">删除</button>
                        </td>
                    </tr>
                `;
                $fundList.append(tr);
            });

            const totalProfitRate = totalCost === 0 ? 0 : ((totalProfit / totalCost) * 100).toFixed(2);
            $('#totalInfo').html(
                `持仓总本金：${totalCost.toFixed(2)}元 | 总市值：${totalMarket.toFixed(2)}元 | 
                总收益：<span class="${totalProfit >= 0 ? 'profit' : 'loss'}">${totalProfit.toFixed(2)}</span>元 | 
                总收益率：<span class="${totalProfitRate >= 0 ? 'profit' : 'loss'}">${totalProfitRate}%</span>`
            );
        }

        window.onresize = function() {
            myChart.resize();
        };
    </script>
</body>
</html>
