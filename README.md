# G50-script-tracker
<!-- 创建 index.html 文件 -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>G50客户端脚本部署记录系统</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { 
            font-family: 'Microsoft YaHei', sans-serif; 
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            border-radius: 15px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            overflow: hidden;
        }
        .header {
            background: linear-gradient(90deg, #4b6cb7 0%, #182848 100%);
            color: white;
            padding: 30px;
            text-align: center;
        }
        .header h1 {
            font-size: 28px;
            margin-bottom: 10px;
        }
        .header p {
            opacity: 0.9;
        }
        .controls {
            padding: 20px;
            background: #f8f9fa;
            border-bottom: 1px solid #dee2e6;
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }
        .btn {
            padding: 10px 20px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s;
        }
        .btn-primary {
            background: #4b6cb7;
            color: white;
        }
        .btn-primary:hover {
            background: #3a5ca9;
            transform: translateY(-2px);
        }
        .btn-success {
            background: #28a745;
            color: white;
        }
        .btn-danger {
            background: #dc3545;
            color: white;
        }
        .search-box {
            flex: 1;
            min-width: 200px;
            padding: 10px;
            border: 2px solid #4b6cb7;
            border-radius: 8px;
            font-size: 14px;
        }
        .table-container {
            overflow-x: auto;
            padding: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            min-width: 800px;
        }
        th {
            background: #4b6cb7;
            color: white;
            padding: 15px;
            text-align: left;
            position: sticky;
            top: 0;
        }
        td {
            padding: 12px 15px;
            border-bottom: 1px solid #dee2e6;
        }
        tr:hover {
            background: #f8f9fa;
        }
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.5);
            z-index: 1000;
            align-items: center;
            justify-content: center;
        }
        .modal-content {
            background: white;
            padding: 30px;
            border-radius: 15px;
            width: 90%;
            max-width: 500px;
        }
        .form-group {
            margin-bottom: 20px;
        }
        .form-group label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
            color: #333;
        }
        .form-control {
            width: 100%;
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 8px;
            font-size: 14px;
        }
        .form-control:focus {
            border-color: #4b6cb7;
            outline: none;
        }
        .stats {
            display: flex;
            gap: 20px;
            padding: 15px 20px;
            background: #e9ecef;
            border-top: 1px solid #dee2e6;
        }
        .stat-item {
            text-align: center;
        }
        .stat-value {
            font-size: 24px;
            font-weight: bold;
            color: #4b6cb7;
        }
        .stat-label {
            font-size: 12px;
            color: #666;
            margin-top: 5px;
        }
        @media (max-width: 768px) {
            .controls { flex-direction: column; }
            .search-box { width: 100%; }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>📋 G50客户端非常规脚本部署记录系统</h1>
            <p>记录替换时间、替换人、脚本版本和脚本变更点</p>
        </div>
        
        <div class="controls">
            <button class="btn btn-primary" onclick="openAddModal()">➕ 新增记录</button>
            <input type="text" class="search-box" placeholder="🔍 搜索脚本版本、替换人或变更点..." onkeyup="searchTable()">
            <button class="btn btn-success" onclick="exportToExcel()">📥 导出Excel</button>
            <button class="btn btn-danger" onclick="clearAll()">🗑️ 清空数据</button>
        </div>
        
        <div class="table-container">
            <table id="dataTable">
                <thead>
                    <tr>
                        <th>序号</th>
                        <th>替换时间</th>
                        <th>替换人</th>
                        <th>脚本版本</th>
                        <th>脚本变更点</th>
                        <th>部署状态</th>
                        <th>操作</th>
                    </tr>
                </thead>
                <tbody id="tableBody">
                    <!-- 数据动态加载 -->
                </tbody>
            </table>
        </div>
        
        <div class="stats">
            <div class="stat-item">
                <div class="stat-value" id="totalCount">0</div>
                <div class="stat-label">总记录数</div>
            </div>
            <div class="stat-item">
                <div class="stat-value" id="todayCount">0</div>
                <div class="stat-label">今日部署</div>
            </div>
            <div class="stat-item">
                <div class="stat-value" id="latestVersion">v1.0</div>
                <div class="stat-label">最新版本</div>
            </div>
        </div>
    </div>
    
    <!-- 新增/编辑模态框 -->
    <div class="modal" id="recordModal">
        <div class="modal-content">
            <h2 style="margin-bottom: 20px;" id="modalTitle">新增部署记录</h2>
            <form id="recordForm">
                <div class="form-group">
                    <label>替换时间 *</label>
                    <input type="datetime-local" class="form-control" id="replaceTime" required>
                </div>
                <div class="form-group">
                    <label>替换人 *</label>
                    <input type="text" class="form-control" id="replacer" placeholder="请输入姓名" required>
                </div>
                <div class="form-group">
                    <label>脚本版本 *</label>
                    <input type="text" class="form-control" id="scriptVersion" placeholder="格式：v1.0.0" required>
                </div>
                <div class="form-group">
                    <label>脚本变更点 *</label>
                    <textarea class="form-control" id="changePoints" rows="3" placeholder="详细描述脚本变更内容..." required></textarea>
                </div>
                <div class="form-group">
                    <label>部署状态</label>
                    <select class="form-control" id="deployStatus">
                        <option value="已部署">✅ 已部署</option>
                        <option value="待部署">⏳ 待部署</option>
                        <option value="已回滚">↩️ 已回滚</option>
                        <option value="测试中">🧪 测试中</option>
                    </select>
                </div>
                <div class="form-group">
                    <label>备注</label>
                    <textarea class="form-control" id="remarks" rows="2" placeholder="其他说明..."></textarea>
                </div>
                <div style="display: flex; gap: 10px; margin-top: 30px;">
                    <button type="submit" class="btn btn-primary" style="flex: 1;">保存记录</button>
                    <button type="button" class="btn btn-danger" onclick="closeModal()" style="flex: 1;">取消</button>
                </div>
            </form>
        </div>
    </div>
    
    <script>
        let records = JSON.parse(localStorage.getItem('g50_script_records')) || [];
        let editingIndex = -1;
        
        // 初始化表格
        function initTable() {
            const tbody = document.getElementById('tableBody');
            tbody.innerHTML = '';
            
            records.forEach((record, index) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${index + 1}</td>
                    <td>${formatDateTime(record.replaceTime)}</td>
                    <td>${record.replacer}</td>
                    <td><span class="version-badge">${record.scriptVersion}</span></td>
                    <td>${record.changePoints}</td>
                    <td><span class="status-badge ${getStatusClass(record.deployStatus)}">${record.deployStatus}</span></td>
                    <td>
                        <button onclick="editRecord(${index})" style="background:#ffc107;color:white;border:none;padding:5px 10px;border-radius:4px;cursor:pointer;margin-right:5px;">编辑</button>
                        <button onclick="deleteRecord(${index})" style="background:#dc3545;color:white;border:none;padding:5px 10px;border-radius:4px;cursor:pointer;">删除</button>
                    </td>
                `;
                tbody.appendChild(row);
            });
            
            updateStats();
        }
        
        // 打开新增模态框
        function openAddModal() {
            editingIndex = -1;
            document.getElementById('modalTitle').textContent = '新增部署记录';
            document.getElementById('recordForm').reset();
            document.getElementById('replaceTime').value = new Date().toISOString().slice(0, 16);
            document.getElementById('recordModal').style.display = 'flex';
        }
        
        // 编辑记录
        function editRecord(index) {
            editingIndex = index;
            const record = records[index];
            
            document.getElementById('modalTitle').textContent = '编辑部署记录';
            document.getElementById('replaceTime').value = record.replaceTime.replace(' ', 'T');
            document.getElementById('replacer').value = record.replacer;
            document.getElementById('scriptVersion').value = record.scriptVersion;
            document.getElementById('changePoints').value = record.changePoints;
            document.getElementById('deployStatus').value = record.deployStatus;
            document.getElementById('remarks').value = record.remarks || '';
            
            document.getElementById('recordModal').style.display = 'flex';
        }
        
        // 保存记录
        document.getElementById('recordForm').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const record = {
                replaceTime: document.getElementById('replaceTime').value.replace('T', ' '),
                replacer: document.getElementById('replacer').value,
                scriptVersion: document.getElementById('scriptVersion').value,
                changePoints: document.getElementById('changePoints').value,
                deployStatus: document.getElementById('deployStatus').value,
                remarks: document.getElementById('remarks').value,
                createdAt: new Date().toISOString()
            };
            
            if (editingIndex === -1) {
                records.unshift(record); // 新增放在最前面
            } else {
                records[editingIndex] = record;
            }
            
            localStorage.setItem('g50_script_records', JSON.stringify(records));
            closeModal();
            initTable();
            alert('保存成功！');
        });
        
        // 删除记录
        function deleteRecord(index) {
            if (confirm('确定要删除这条记录吗？')) {
                records.splice(index, 1);
                localStorage.setItem('g50_script_records', JSON.stringify(records));
                initTable();
            }
        }
        
        // 搜索功能
        function searchTable() {
            const searchTerm = document.querySelector('.search-box').value.toLowerCase();
            const rows = document.querySelectorAll('#tableBody tr');
            
            rows.forEach(row => {
                const text = row.textContent.toLowerCase();
                row.style.display = text.includes(searchTerm) ? '' : 'none';
            });
        }
        
        // 导出Excel
        function exportToExcel() {
            if (records.length === 0) {
                alert('没有数据可导出！');
                return;
            }
            
            let csv = '替换时间,替换人,脚本版本,脚本变更点,部署状态,备注\n';
            
            records.forEach(record => {
                csv += `"${record.replaceTime}","${record.replacer}","${record.scriptVersion}","${record.changePoints}","${record.deployStatus}","${record.remarks || ''}"\n`;
            });
            
            const blob = new Blob(['\ufeff' + csv], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = `G50脚本部署记录_${new Date().toISOString().slice(0,10)}.csv`;
            link.click();
        }
        
        // 清空数据
        function clearAll() {
            if (confirm('⚠️ 确定要清空所有数据吗？此操作不可恢复！')) {
                records = [];
                localStorage.removeItem('g50_script_records');
                initTable();
            }
        }
        
        // 关闭模态框
        function closeModal() {
            document.getElementById('recordModal').style.display = 'none';
        }
        
        // 更新统计信息
        function updateStats() {
            document.getElementById('totalCount').textContent = records.length;
            
            const today = new Date().toISOString().slice(0, 10);
            const todayCount = records.filter(r => r.replaceTime.startsWith(today)).length;
            document.getElementById('todayCount').textContent = todayCount;
            
            if (records.length > 0) {
                const latest = records[0].scriptVersion;
                document.getElementById('latestVersion').textContent = latest;
            }
        }
        
        // 格式化日期时间
        function formatDateTime(datetimeStr) {
            return datetimeStr.replace('T', ' ');
        }
        
        // 获取状态样式类
        function getStatusClass(status) {
            const classes = {
                '已部署': 'status-deployed',
                '待部署': 'status-pending',
                '已回滚': 'status-rolledback',
                '测试中': 'status-testing'
            };
            return classes[status] || '';
        }
        
        // 添加样式
        const style = document.createElement('style');
        style.textContent = `
            .version-badge {
                background: #e3f2fd;
                color: #1976d2;
                padding: 3px 8px;
                border-radius: 12px;
                font-size: 12px;
                font-weight: bold;
            }
            .status-badge {
                padding: 4px 10px;
                border-radius: 15px;
                font-size: 12px;
                font-weight: bold;
            }
            .status-deployed { background: #d4edda; color: #155724; }
            .status-pending { background: #fff3cd; color: #856404; }
            .status-rolledback { background: #f8d7da; color: #721c24; }
            .status-testing { background: #d1ecf1; color: #0c5460; }
        `;
        document.head.appendChild(style);
        
        // 页面加载时初始化
        window.onload = initTable;
        
        // 点击模态框外部关闭
        window.onclick = function(event) {
            const modal = document.getElementById('recordModal');
            if (event.target === modal) {
                closeModal();
            }
        };
    </script>
</body>
</html>
