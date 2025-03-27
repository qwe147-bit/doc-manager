<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文书分类管理系统</title>
    <style>
        body {
            font-family: 'Microsoft YaHei', Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
        }
        .search-box {
            margin-bottom: 20px;
            display: flex;
        }
        #searchInput {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px 0 0 4px;
            font-size: 16px;
        }
        #searchButton {
            padding: 10px 15px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 0 4px 4px 0;
            cursor: pointer;
        }
        .categories {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-bottom: 20px;
        }
        .category-item {
            display: flex;
            align-items: center;
        }
        .category-btn {
            padding: 8px 15px;
            background-color: #e7f3fe;
            border: 1px solid #b8daff;
            border-radius: 4px;
            cursor: pointer;
            color: #004085;
        }
        .category-btn.active {
            background-color: #007bff;
            color: white;
            border-color: #007bff;
        }
        .delete-category-btn {
            margin-left: 8px;
            color: #dc3545;
            background: none;
            border: none;
            cursor: pointer;
            font-size: 14px;
        }
        .content-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        .content-table th, .content-table td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: left;
        }
        .content-table th {
            background-color: #f2f2f2;
        }
        .content-table tr:hover {
            background-color: #f1f1f1;
        }
        .form-section {
            margin-top: 30px;
            padding: 20px;
            background-color: #f8f9fa;
            border-radius: 4px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        .form-group label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        .form-group input, .form-group textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        .form-actions {
            text-align: right;
        }
        .batch-section {
            margin-top: 30px;
            padding: 20px;
            background-color: #f1f8ff;
            border-radius: 4px;
        }
        .optional-field {
            color: #6c757d;
            font-style: italic;
        }
        .required-field {
            color: #dc3545;
        }
        .highlight {
            background-color: yellow;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>文书分类管理系统</h1>
        
        <div class="search-box">
            <input type="text" id="searchInput" placeholder="输入关键词搜索...">
            <button id="searchButton">搜索</button>
        </div>
        
        <div class="categories" id="categoriesContainer">
            <!-- 分类按钮将通过JavaScript动态生成 -->
        </div>
        
        <table class="content-table">
            <thead>
                <tr>
                    <th width="20%">分类名</th>
                    <th width="20%">标题</th>
                    <th width="60%">内容</th>
                </tr>
            </thead>
            <tbody id="contentBody">
                <!-- 内容将通过JavaScript动态生成 -->
            </tbody>
        </table>
        
        <div class="form-section">
            <h3>添加/编辑内容</h3>
            <div class="form-group">
                <label for="editCategory">分类名 <span class="required-field">*</span></label>
                <input type="text" id="editCategory" placeholder="输入分类名" required>
            </div>
            <div class="form-group">
                <label for="editTitle">标题 <span class="optional-field">(可选)</span></label>
                <input type="text" id="editTitle" placeholder="输入标题（可不填）">
            </div>
            <div class="form-group">
                <label for="editContent">内容 <span class="required-field">*</span></label>
                <textarea id="editContent" placeholder="输入内容" required></textarea>
            </div>
            <div class="form-actions">
                <button id="cancelEdit">取消</button>
                <button id="saveContent">保存</button>
            </div>
        </div>
        
        <div class="batch-section">
            <h3>批量操作</h3>
            <div class="form-group">
                <label for="importFile">导入数据</label>
                <input type="file" id="importFile" accept=".json,.csv,.txt">
            </div>
            <div class="form-actions">
                <button id="exportJSON">导出为JSON</button>
                <button id="exportCSV">导出为CSV</button>
                <button id="resetData">重置数据</button>
            </div>
        </div>
    </div>

    <script>
        // 初始数据
        let documents = [
            { category: "常用回复", title: "问候语", content: "您好，在的，有什么可以帮助您的吗？" },
            { category: "常用回复", content: "感谢您的咨询，祝您生活愉快！" },
            { category: "技术支持", title: "密码重置", content: "您可以点击登录页面的'忘记密码'链接，按照提示操作重置密码。" }
        ];
        
        // 当前选中的分类
        let currentCategory = null;
        
        // DOM元素
        const categoriesEl = document.getElementById('categoriesContainer');
        const contentBodyEl = document.getElementById('contentBody');
        const searchInputEl = document.getElementById('searchInput');
        const searchButtonEl = document.getElementById('searchButton');
        const editCategoryEl = document.getElementById('editCategory');
        const editTitleEl = document.getElementById('editTitle');
        const editContentEl = document.getElementById('editContent');
        const saveContentEl = document.getElementById('saveContent');
        const cancelEditEl = document.getElementById('cancelEdit');
        const exportJSONEl = document.getElementById('exportJSON');
        const exportCSVEl = document.getElementById('exportCSV');
        const resetDataEl = document.getElementById('resetData');
        const importFileEl = document.getElementById('importFile');
        
        // 初始化页面
        function initPage() {
            loadData();
            renderCategories();
            renderContent();
            setupEventListeners();
        }
        
        // 从localStorage加载数据
        function loadData() {
            const savedData = localStorage.getItem('documentData');
            if (savedData) {
                try {
                    documents = JSON.parse(savedData);
                } catch (e) {
                    console.error('加载保存的数据失败:', e);
                }
            }
        }
        
        // 保存数据到localStorage
        function saveData() {
            localStorage.setItem('documentData', JSON.stringify(documents));
        }
        
        // 渲染分类按钮
        function renderCategories() {
            const categories = [...new Set(documents.map(doc => doc.category))];
            
            categoriesEl.innerHTML = '';
            
            // 全部按钮
            const allBtn = document.createElement('button');
            allBtn.className = 'category-btn' + (currentCategory === null ? ' active' : '');
            allBtn.textContent = '全部';
            allBtn.addEventListener('click', () => {
                currentCategory = null;
                renderCategories();
                renderContent();
            });
            categoriesEl.appendChild(allBtn);
            
            // 各分类按钮
            categories.forEach(category => {
                const categoryItem = document.createElement('div');
                categoryItem.className = 'category-item';
                
                const btn = document.createElement('button');
                btn.className = 'category-btn' + (currentCategory === category ? ' active' : '');
                btn.textContent = category;
                btn.addEventListener('click', () => {
                    currentCategory = category;
                    renderCategories();
                    renderContent();
                });
                
                const deleteBtn = document.createElement('button');
                deleteBtn.className = 'delete-category-btn';
                deleteBtn.innerHTML = '&times;';
                deleteBtn.title = '删除此分类';
                deleteBtn.addEventListener('click', (e) => {
                    e.stopPropagation();
                    deleteCategory(category);
                });
                
                categoryItem.appendChild(btn);
                categoryItem.appendChild(deleteBtn);
                categoriesEl.appendChild(categoryItem);
            });
            
            // 添加分类按钮
            const addBtn = document.createElement('button');
            addBtn.className = 'category-btn';
            addBtn.textContent = '+ 添加分类';
            addBtn.addEventListener('click', () => {
                editCategoryEl.value = '';
                editTitleEl.value = '';
                editContentEl.value = '';
                delete saveContentEl.dataset.index;
                editCategoryEl.focus();
            });
            categoriesEl.appendChild(addBtn);
        }
        
        // 删除分类
        function deleteCategory(category) {
            if (confirm(`确定要删除分类"${category}"吗？该分类下的所有内容也将被删除！`)) {
                documents = documents.filter(doc => doc.category !== category);
                if (currentCategory === category) {
                    currentCategory = null;
                }
                renderCategories();
                renderContent();
                saveData();
            }
        }
        
        // 渲染内容表格
        function renderContent() {
            contentBodyEl.innerHTML = '';
            
            let filteredDocs = documents;
            if (currentCategory) {
                filteredDocs = documents.filter(doc => doc.category === currentCategory);
            }
            
            if (filteredDocs.length === 0) {
                const row = document.createElement('tr');
                row.innerHTML = `<td colspan="3">没有找到相关内容</td>`;
                contentBodyEl.appendChild(row);
                return;
            }
            
            // 创建文档索引映射
            const indexMap = [];
            let globalIndex = 0;
            
            filteredDocs.forEach(doc => {
                const originalIndex = documents.findIndex(d => 
                    d.category === poc.category && 
                    d.title === poc.title && 
                    d.content === poc.content
                );
                
                if (originalIndex !== -1) {
                    indexMap.push(originalIndex);
                }
            });
            
            filteredDocs.forEach((doc, filteredIndex) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${doc.category}</td>
                    <td>${doc.title || '<span class="optional-field">无标题</span>'}</td>
                    <td>${doc.content}</td>
                `;
                
                // 使用映射的原始索引
                const originalIndex = indexMap[filteredIndex];
                row.addEventListener('click', () => editDocument(originalIndex));
                
                contentBodyEl.appendChild(row);
            });
        }
        
        // 编辑文档
        function editDocument(index) {
            const doc = documents[index];
            editCategoryEl.value = doc.category;
            editTitleEl.value = doc.title || '';
            editContentEl.value = doc.content;
            saveContentEl.dataset.index = index;
        }
        
        // 保存文档
        function saveDocument() {
            const category = editCategoryEl.value.trim();
            const title = editTitleEl.value.trim();
            const content = editContentEl.value.trim();
            
            if (!category || !content) {
                alert('请至少填写分类名和内容');
                return;
            }
            
            const doc = { category, content };
            if (title) doc.title = title;
            
            const index = saveContentEl.dataset.index;
            if (index !== undefined) {
                // 更新现有文档
                documents[index] = doc;
            } else {
                // 添加新文档
                documents.push(doc);
            }
            
            // 重置表单
            editCategoryEl.value = '';
            editTitleEl.value = '';
            editContentEl.value = '';
            delete saveContentEl.dataset.index;
            
            // 重新渲染
            renderCategories();
            renderContent();
            saveData();
        }
        
        // 搜索内容
        function searchContent() {
            const keyword = searchInputEl.value.trim().toLowerCase();
            
            if (!keyword) {
                renderContent();
                return;
            }
            
            const filteredDocs = documents.filter(doc => 
                doc.category.toLowerCase().includes(keyword) || 
                (doc.title && doc.title.toLowerCase().includes(keyword)) || 
                doc.content.toLowerCase().includes(keyword)
            );
            
            contentBodyEl.innerHTML = '';
            
            if (filteredDocs.length === 0) {
                const row = document.createElement('tr');
                row.innerHTML = `<td colspan="3">没有找到包含"${keyword}"的内容</td>`;
                contentBodyEl.appendChild(row);
                return;
            }
            
            filteredDocs.forEach((doc, index) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${highlightText(doc.category, keyword)}</td>
                    <td>${doc.title ? highlightText(doc.title, keyword) : '<span class="optional-field">无标题</span>'}</td>
                    <td>${highlightText(doc.content, keyword)}</td>
                `;
                row.addEventListener('click', () => editDocument(index));
                contentBodyEl.appendChild(row);
            });
        }
        
        // 高亮文本
        function highlightText(text, keyword) {
            if (!keyword) return text;
            return text.replace(new RegExp(keyword, 'gi'), match => `<span class="highlight">${match}</span>`);
        }
        
        // 导出为JSON
        function exportToJSON() {
            const dataStr = JSON.stringify(documents, null, 2);
            const blob = new Blob([dataStr], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            
            const a = document.createElement('a');
            a.href = url;
            a.download = `文书数据_${new Date().toISOString().slice(0,10)}.json`;
            a.click();
            
            URL.revokeObjectURL(url);
        }
        
        // 导出为CSV
        function exportToCSV() {
            let csvContent = "分类,标题,内容\n";
            
            documents.forEach(doc => {
                const row = [
                    `"${(doc.category || '').replace(/"/g, '""')}"`,
                    `"${(doc.title || '').replace(/"/g, '""')}"`,
                    `"${(doc.content || '').replace(/"/g, '""')}"`
                ].join(',');
                csvContent += row + '\n';
            });
            
            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            
            const a = document.createElement('a');
            a.href = url;
            a.download = `文书数据_${new Date().toISOString().slice(0,10)}.csv`;
            a.click();
            
            URL.revokeObjectURL(url);
        }
        
        // 导入数据
        function importData(file) {
            const reader = new FileReader();
            
            reader.onload = function(e) {
                try {
                    let importedData = [];
                    const fileContent = e.target.result;
                    
                    // 根据文件类型选择解析方式
                    if (file.name.endsWith('.json')) {
                        importedData = JSON.parse(fileContent);
                    } 
                    else if (file.name.endsWith('.csv')) {
                        importedData = parseCSV(fileContent);
                    }
                    else if (file.name.endsWith('.txt')) {
                        importedData = parseTXT(fileContent);
                    }
                    
                    // 验证并标准化数据
                    const validatedData = importedData.map(item => {
                        // 处理不同格式的字段名
                        const category = item.category || item.分类 || item['分类名'] || '未分类';
                        const title = item.title || item.标题 || '';
                        const content = item.content || item.内容 || item.text || '';
                        
                        return { category, title, content };
                    }).filter(item => item.category && item.content);
                    
                    if (validatedData.length === 0) {
                        throw new Error('未找到有效数据');
                    }
                    
                    if (confirm(`找到 ${validatedData.length} 条有效数据，是否导入？`)) {
                        documents = validatedData;
                        currentCategory = null;
                        renderCategories();
                        renderContent();
                        saveData();
                        alert('导入成功！');
                    }
                } catch (error) {
                    alert(`导入失败: ${error.message}`);
                }
            };
            
            reader.readAsText(file);
        }
        
        // 解析CSV
        function parseCSV(csvContent) {
            const lines = csvContent.split('\n');
            const headers = lines[0].replace(/[\r"]/g, '').split(',');
            const result = [];
            
            for (let i = 1; i < lines.length; i++) {
                const line = lines[i].trim();
                if (!line) continue;
                
                const values = line.split(/,(?=(?:[^"]*"[^"]*")*[^"]*$)/)
                    .map(v => v.replace(/^"|"$/g, '').trim());
                
                const obj = {};
                headers.forEach((header, j) => {
                    obj[header] = values[j] || '';
                });
                
                result.push(obj);
            }
            
            return result;
        }
        
        // 解析TXT
        function parseTXT(txtContent) {
            return txtContent.split('\n')
                .filter(line => line.trim())
                .map(line => {
                    const parts = line.split('|').map(part => part.trim());
                    return {
                        category: parts[0] || '未分类',
                        title: parts[1] || '',
                        content: parts[2] || ''
                    };
                });
        }
        
        // 重置数据
        function resetData() {
            if (confirm('确定要重置为默认数据吗？所有修改将丢失。')) {
                documents = [
                    { category: "常用回复", title: "问候语", content: "您好，在的，有什么可以帮助您的吗？" },
                    { category: "常用回复", content: "感谢您的咨询，祝您生活愉快！" },
                    { category: "技术支持", title: "密码重置", content: "您可以点击登录页面的'忘记密码'链接，按照提示操作重置密码。" }
                ];
                currentCategory = null;
                renderCategories();
                renderContent();
                saveData();
            }
        }
        
        // 设置事件监听
        function setupEventListeners() {
            saveContentEl.addEventListener('click', saveDocument);
            cancelEditEl.addEventListener('click', () => {
                editCategoryEl.value = '';
                editTitleEl.value = '';
                editContentEl.value = '';
                delete saveContentEl.dataset.index;
            });
            
            searchButtonEl.addEventListener('click', searchContent);
            searchInputEl.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') searchContent();
            });
            
            exportJSONEl.addEventListener('click', exportToJSON);
            exportCSVEl.addEventListener('click', exportToCSV);
            resetDataEl.addEventListener('click', resetData);
            
            importFileEl.addEventListener('change', (e) => {
                const file = e.target.files[0];
                if (file) importData(file);
                e.target.value = ''; // 重置文件输入
            });
        }
        
        // 启动应用
        initPage();
    </script>
</body>
</html>
