<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>排班管理 - 增强版</title>
    <style>
        /* === 1. 基础变量与重置 === */
        :root {
            --primary-color: #1890ff;
            --success-color: #67c23a;
            --warning-color: #e6a23c;
            --danger-color: #f56c6c;
            --menu-bg: #304156;
            --main-bg: #f0f2f5;
            --border-color: #dfe6ec;
            --text-regular: #606266;
            --text-primary: #303133;
            --disabled-bg: #f5f7fa;
        }

        body { margin: 0; font-family: "Helvetica Neue", Arial, sans-serif; background-color: var(--main-bg); color: var(--text-regular); height: 100vh; display: flex; overflow: hidden; }
        * { box-sizing: border-box; }

        /* === 2. 布局样式 === */
        .sidebar-container { width: 220px; background-color: var(--menu-bg); height: 100%; display: flex; flex-direction: column; flex-shrink: 0; }
        .logo-area { height: 50px; line-height: 50px; padding-left: 20px; color: #fff; font-weight: 600; background-color: #2b2f3a; display: flex; align-items: center; }
        .menu-item { height: 50px; line-height: 50px; padding-left: 20px; color: #bfcbd9; cursor: pointer; font-size: 14px; transition: 0.3s; display: flex; align-items: center; }
        .menu-item:hover { background-color: #263445; }
        .menu-item.active { color: #409eff; background-color: #1f2d3d; border-left: 4px solid var(--primary-color); }

        .main-container { flex: 1; display: flex; flex-direction: column; min-width: 0; }
        .navbar { height: 50px; background: #fff; box-shadow: 0 1px 4px rgba(0,21,41,.08); display: flex; align-items: center; padding: 0 20px; justify-content: space-between; }
        .tags-view { height: 34px; background: #fff; border-bottom: 1px solid #d8dce5; display: flex; align-items: center; padding-left: 15px; }
        .tag-item { height: 26px; line-height: 26px; border: 1px solid #d8dce5; padding: 0 8px; font-size: 12px; margin-left: 5px; border-radius: 2px; }
        .tag-item.active { background-color: var(--primary-color); color: #fff; border-color: var(--primary-color); }

        .app-main { padding: 20px; flex: 1; overflow: hidden; display: flex; gap: 20px; }

        /* 左侧组织树 */
        .org-panel { width: 300px; background: #fff; border-radius: 4px; display: flex; flex-direction: column; box-shadow: 0 2px 12px 0 rgba(0,0,0,.1); }
        .panel-header { padding: 15px; border-bottom: 1px solid var(--border-color); }
        .search-input { width: 100%; height: 32px; border: 1px solid #dcdfe6; border-radius: 4px; padding: 0 10px; outline: none; }
        .org-tree-container { flex: 1; overflow-y: auto; }
        .tree-node { display: flex; align-items: center; padding: 10px; border-bottom: 1px solid #ebeef5; cursor: pointer; font-size: 14px; transition: 0.2s; }
        .tree-node:hover { background-color: #f5f7fa; }
        .tree-node.selected { background-color: #ecf5ff; }
        .tree-node.selected .node-name { color: var(--primary-color); font-weight: bold; }
        .type-tag { font-size: 12px; padding: 2px 6px; border-radius: 3px; margin-left: auto; transform: scale(0.9); }
        .tag-dept { background: #fdf6ec; color: #e6a23c; border: 1px solid #faecd8; }
        .tag-user { background: #ecf5ff; color: #409EFF; border: 1px solid #d9ecff; }

        /* 右侧排班区 */
        .schedule-panel { flex: 1; background: #fff; border-radius: 4px; display: flex; flex-direction: column; box-shadow: 0 2px 12px 0 rgba(0,0,0,.1); position: relative; }
        .schedule-header { padding: 15px 20px; border-bottom: 1px solid var(--border-color); display: flex; justify-content: space-between; align-items: center; }
        .target-info { display: flex; align-items: center; gap: 10px; font-size: 16px; font-weight: bold; color: var(--text-primary); }
        
        /* 模式切换区 */
        .config-mode-wrapper { display: flex; align-items: center; gap: 15px; font-size: 14px; font-weight: normal; margin-left: 20px; padding-left: 20px; border-left: 1px solid #eee; }
        .radio-group { display: flex; gap: 10px; }
        .radio-label { display: flex; align-items: center; cursor: pointer; user-select: none; }
        .radio-label input { margin-right: 5px; }

        .schedule-body { flex: 1; padding: 20px; overflow-y: auto; display: flex; flex-direction: column; }
        
        /* 工具栏与时间设置 */
        .toolbar { margin-bottom: 15px; display: flex; justify-content: space-between; align-items: center; background: #fafafa; padding: 10px; border-radius: 4px; border: 1px solid #ebeef5; }
        .time-setting { display: flex; align-items: center; gap: 8px; font-size: 13px; color: var(--text-regular); }
        .time-input { width: 100px; height: 28px; border: 1px solid #dcdfe6; border-radius: 3px; padding: 0 5px; }

        /* 日历样式 */
        .calendar-container { border: 1px solid #ebeef5; border-radius: 4px; display: flex; flex-direction: column; flex: 1; min-height: 0; }
        .cal-header-row { display: grid; grid-template-columns: repeat(7, 1fr); background: #f5f7fa; border-bottom: 1px solid #ebeef5; }
        .cal-header-cell { text-align: center; padding: 10px 0; font-size: 13px; color: #909399; font-weight: 600; }
        .cal-body-grid { display: grid; grid-template-columns: repeat(7, 1fr); flex: 1; overflow-y: auto; }
        
        .cal-cell { border-right: 1px solid #ebeef5; border-bottom: 1px solid #ebeef5; min-height: 80px; padding: 8px; position: relative; cursor: pointer; transition: all 0.2s; display: flex; flex-direction: column; }
        .cal-cell:nth-child(7n) { border-right: none; }
        .day-number { font-size: 16px; font-weight: bold; color: var(--text-primary); }
        .status-badge { margin-top: auto; align-self: flex-start; font-size: 12px; padding: 3px 8px; border-radius: 10px; }
        
        /* 状态颜色 */
        .cal-cell.work { background-color: #f0f9eb; }
        .cal-cell.work .status-badge { background: var(--success-color); color: white; }
        .cal-cell.work .day-number { color: var(--success-color); }
        
        .cal-cell.rest { background-color: #fff; }
        .cal-cell.rest .status-badge { background: #f4f4f5; color: #909399; }

        /* 禁用/只读状态 */
        .cal-cell.disabled { background-color: var(--disabled-bg); cursor: not-allowed; opacity: 0.7; }
        .cal-cell.disabled:hover { box-shadow: none; }
        .cal-cell.disabled .status-badge { display: none; }
        
        /* 遮罩层 (用于非自定义模式) */
        .mask-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255,255,255,0.6); z-index: 10; cursor: not-allowed; display: flex; justify-content: center; align-items: center; font-size: 20px; color: #909399; font-weight: bold; backdrop-filter: blur(1px); }

        /* 按钮与Toast */
        .btn { padding: 8px 15px; border: 1px solid #dcdfe6; background: #fff; cursor: pointer; font-size: 12px; border-radius: 3px; transition: 0.2s; outline: none; }
        .btn:hover { color: var(--primary-color); border-color: #c6e2ff; background-color: #ecf5ff; }
        .btn-primary { color: #fff; background-color: var(--primary-color); border-color: var(--primary-color); }
        .btn-primary:hover { background: #46a6ff; border-color: #46a6ff; }
        .btn:disabled { cursor: not-allowed; background: #f5f7fa; color: #c0c4cc; border-color: #ebeef5; }

        .schedule-footer { padding: 15px 20px; border-top: 1px solid var(--border-color); text-align: right; background: #fff; }

        /* Toast 组件样式 */
        .toast-container { position: fixed; top: 20px; left: 50%; transform: translateX(-50%); z-index: 9999; }
        .toast { min-width: 300px; padding: 10px 20px; margin-bottom: 10px; border-radius: 4px; box-shadow: 0 4px 12px rgba(0,0,0,0.15); font-size: 14px; display: flex; align-items: center; animation: slideDown 0.3s ease-out; }
        .toast-success { background: #f0f9eb; border: 1px solid #e1f3d8; color: #67c23a; }
        .toast-error { background: #fef0f0; border: 1px solid #fde2e2; color: #f56c6c; }
        .toast-warning { background: #fdf6ec; border: 1px solid #faecd8; color: #e6a23c; }
        .toast-icon { margin-right: 10px; font-size: 16px; }

        @keyframes slideDown { from { opacity: 0; transform: translateY(-20px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>

    <!-- Toast 挂载点 -->
    <div class="toast-container" id="toastContainer"></div>

    <div class="sidebar-container">
        <div class="logo-area">🇲🇽 墨西哥-后台管理</div>
        <div class="menu-item"><span style="margin-right:10px">🏢</span> 部门管理</div>
        <div class="menu-item active"><span style="margin-right:10px">📅</span> 排班管理</div>
        <div class="menu-item"><span style="margin-right:10px">⚙️</span> 系统设置</div>
    </div>

    <div class="main-container">
        <div class="navbar">
            <div style="font-size:14px; color:#97a8be;">首页 / 排班管理</div>
            <div style="font-size:12px; color:#606266;">超级管理员 - 邱婷婷</div>
        </div>
        <div class="tags-view">
            <div class="tag-item active">排班管理</div>
        </div>

        <div class="app-main">
            <!-- 左侧树 -->
            <div class="org-panel">
                <div class="panel-header">
                    <input type="text" class="search-input" placeholder="搜索部门或员工姓名...">
                </div>
                <div class="org-tree-container" id="orgTree"></div>
            </div>

            <!-- 右侧排班 -->
            <div class="schedule-panel">
                <!-- 头部信息 -->
                <div class="schedule-header">
                    <div class="target-info">
                        <span id="targetIcon">🏢</span>
                        <span id="targetName">未选择</span>
                        
                        <!-- 模式切换（仅员工可见） -->
                        <div class="config-mode-wrapper" id="modeSwitchArea" style="display: none;">
                            <span style="color:#909399; font-size:12px;">配置模式：</span>
                            <div class="radio-group">
                                <label class="radio-label">
                                    <input type="radio" name="mode" value="inherit" onchange="switchMode('inherit')"> 跟随部门
                                </label>
                                <label class="radio-label">
                                    <input type="radio" name="mode" value="custom" onchange="switchMode('custom')"> 自定义配置
                                </label>
                            </div>
                        </div>
                    </div>

                    <div style="display: flex; gap: 10px;">
                        <button class="btn" onclick="changeMonth(-1)">◀ 上月</button>
                        <button class="btn" style="font-weight:bold; min-width: 100px;" id="currentMonthDisplay">--</button>
                        <button class="btn" onclick="changeMonth(1)">下月 ▶</button>
                    </div>
                </div>

                <!-- 遮罩层：继承模式下显示 -->
                <div id="inheritMask" class="mask-overlay" style="display: none;">
                    当前正在跟随部门排班规则 (只读)
                </div>

                <div class="schedule-body">
                    <!-- 工具栏 -->
                    <div class="toolbar">
                        <div style="display:flex; gap:10px;">
                            <button class="btn" onclick="quickSet('workdays')">仅工作日 (一至六)</button>
                            <button class="btn" onclick="quickSet('all')">全月上班</button>
                            <button class="btn" onclick="quickSet('clear')">全月休息</button>
                        </div>
                        
                        <!-- 班次时间设置 -->
                        <div class="time-setting">
                            <span>班次时间:</span>
                            <input type="time" class="time-input" id="startTime" value="09:00" onchange="markDirty()">
                            <span>至</span>
                            <input type="time" class="time-input" id="endTime" value="18:00" onchange="markDirty()">
                        </div>
                    </div>

                    <!-- 日历 -->
                    <div class="calendar-container">
                        <div class="cal-header-row">
                            <div class="cal-header-cell">周一</div><div class="cal-header-cell">周二</div><div class="cal-header-cell">周三</div>
                            <div class="cal-header-cell">周四</div><div class="cal-header-cell">周五</div><div class="cal-header-cell">周六</div>
                            <div class="cal-header-cell">周日</div>
                        </div>
                        <div class="cal-body-grid" id="calGrid"></div>
                    </div>
                </div>

                <div class="schedule-footer">
                    <span style="float:left; font-size:12px; color:#909399; line-height:30px;">
                        提示：仅支持修改当月及下月排班
                    </span>
                    <button class="btn" onclick="resetData()">重置</button>
                    <button class="btn btn-primary" onclick="handleSave()">保存设置</button>
                </div>
            </div>
        </div>
    </div>

<script>
    // === Toast组件封装 ===
    const Toast = {
        show(message, type = 'success', duration = 3000) {
            const container = document.getElementById('toastContainer');
            const el = document.createElement('div');
            el.className = `toast toast-${type}`;
            
            let icon = '✅';
            if(type === 'error') icon = '❌';
            if(type === 'warning') icon = '⚠️';

            el.innerHTML = `<span class="toast-icon">${icon}</span>${message}`;
            container.appendChild(el);

            setTimeout(() => {
                el.style.opacity = '0';
                setTimeout(() => el.remove(), 300);
            }, duration);
        },
        success(msg) { this.show(msg, 'success'); },
        error(msg) { this.show(msg, 'error'); },
        warning(msg) { this.show(msg, 'warning'); }
    };

    // === 全局状态 ===
    const state = {
        now: new Date(),          // 当前真实时间
        viewYear: new Date().getFullYear(),
        viewMonth: new Date().getMonth(), // 0-11
        selectedNode: null,       // 当前选中的树节点
        isDirty: false,           // 是否有未保存的修改
        currentMode: 'custom',    // 'inherit' | 'custom'
        dataVersion: 1,           // 模拟乐观锁版本号
        calendarData: {}          // 模拟存储排班数据 key: nodeId-year-month
    };

    // 模拟组织数据
    const treeData = [
        { id: 1, name: "ZeroTech", type: "dept", children: [
            { id: 11, name: "催收一部", type: "dept", children: [
                { id: 101, name: "张三", type: "user" },
                { id: 102, name: "李四", type: "user" }
            ]}
        ]}
    ];

    // === 初始化 ===
    window.onload = () => {
        renderTree(treeData, document.getElementById('orgTree'));
        // 默认选中第一个
        selectNode(treeData[0]);
    };

    // === 逻辑控制：日期范围 ===
    // 返回 true 表示在允许编辑的范围内（当月或下月）
    function isEditableMonth(y, m) {
        const currY = state.now.getFullYear();
        const currM = state.now.getMonth();
        
        // 计算绝对月份差
        const diff = (y - currY) * 12 + (m - currM);
        return diff === 0 || diff === 1;
    }

    // === 渲染树 ===
    function renderTree(nodes, container, level = 0) {
        nodes.forEach(node => {
            const div = document.createElement('div');
            div.className = 'tree-node';
            div.style.paddingLeft = (level * 15 + 10) + 'px';
            const icon = node.type === 'dept' ? '📁' : '👤';
            const typeClass = node.type === 'dept' ? 'tag-dept' : 'tag-user';
            const typeText = node.type === 'dept' ? '部门' : '员工';
            
            div.innerHTML = `
                <span style="margin-right:5px">${icon}</span>
                <span class="node-name">${node.name}</span>
                <span class="type-tag ${typeClass}">${typeText}</span>
            `;
            
            div.onclick = () => handleNodeClick(node, div);
            container.appendChild(div);
            
            if(node.children) renderTree(node.children, container, level + 1);
        });
    }

    // === 交互：节点切换处理 ===
    async function handleNodeClick(node, el) {
        if(state.selectedNode && state.selectedNode.id === node.id) return;
        
        // 未保存检查
        if(state.isDirty) {
            const confirm = window.confirm("存在未保存的排班，是否放弃修改？");
            if(!confirm) return;
        }

        // 更新UI选中态
        document.querySelectorAll('.tree-node').forEach(d => d.classList.remove('selected'));
        el.classList.add('selected');

        selectNode(node);
    }

    function selectNode(node) {
        state.selectedNode = node;
        state.isDirty = false;
        state.dataVersion++; // 模拟重新获取版本号

        // 更新头部信息
        document.getElementById('targetName').innerText = node.name;
        document.getElementById('targetIcon').innerText = node.type === 'dept' ? '🏢' : '👤';

        // 模式切换控制
        const switchArea = document.getElementById('modeSwitchArea');
        if(node.type === 'user') {
            switchArea.style.display = 'flex';
            // 模拟：默认读取该用户配置，假设有的跟随有的自定义
            // 这里简单模拟：ID为偶数跟随，奇数自定义
            const mode = node.id % 2 === 0 ? 'inherit' : 'custom';
            setRadioValue('mode', mode);
            state.currentMode = mode;
        } else {
            switchArea.style.display = 'none';
            state.currentMode = 'custom'; // 部门强制自定义
        }

        renderCalendar();
    }

    // === 交互：模式切换 ===
    function switchMode(mode) {
        if(state.currentMode === mode) return;
        state.currentMode = mode;
        state.isDirty = true;
        renderCalendar();
    }

    function setRadioValue(name, val) {
        const radios = document.getElementsByName(name);
        for(let r of radios) {
            r.checked = (r.value === val);
        }
    }

    // === 交互：月份切换 ===
    function changeMonth(offset) {
        if(state.isDirty) {
            if(!confirm("存在未保存的排班，切换月份将丢失修改，是否继续？")) return;
        }

        let newM = state.viewMonth + offset;
        let newY = state.viewYear;
        if(newM > 11) { newM = 0; newY++; }
        if(newM < 0) { newM = 11; newY--; }

        // 检查是否在允许范围内（仅用于交互提示，不强制锁定视图，但锁定编辑）
        // 需求：越界操作提示，这里我们允许查看，但如果点击格子的逻辑会拦截
        state.viewMonth = newM;
        state.viewYear = newY;
        state.isDirty = false;
        renderCalendar();
    }

    // === 核心：渲染日历 ===
    function renderCalendar() {
        const grid = document.getElementById('calGrid');
        grid.innerHTML = '';
        
        document.getElementById('currentMonthDisplay').innerText = `${state.viewYear}年 ${state.viewMonth + 1}月`;
        
        // 判断是否只读
        const editable = isEditableMonth(state.viewYear, state.viewMonth);
        const isInherit = state.currentMode === 'inherit';
        
        // 遮罩层控制
        const mask = document.getElementById('inheritMask');
        if(isInherit) {
            mask.style.display = 'flex';
            mask.innerText = "当前正在跟随部门排班规则 (只读)";
        } else if (!editable) {
            // 如果不在当月或下月，也显示遮罩但提示不同
            // 或者通过 grid cell class 控制
            mask.style.display = 'none'; 
        } else {
            mask.style.display = 'none';
        }

        // 获取或初始化数据
        const dataKey = `${state.selectedNode.id}-${state.viewYear}-${state.viewMonth}`;
        if(!state.calendarData[dataKey]) {
            // 默认初始化：周一到周六上班，周日休息
            const initData = [];
            const daysInMonth = new Date(state.viewYear, state.viewMonth + 1, 0).getDate();
            for(let i=1; i<=daysInMonth; i++) {
                const day = new Date(state.viewYear, state.viewMonth, i).getDay();
                // 需求：周一到周六上班
                initData[i] = (day !== 0) ? 1 : 0; 
            }
            state.calendarData[dataKey] = initData;
        }
        const currentData = state.calendarData[dataKey];

        // 填充空白
        let firstDay = new Date(state.viewYear, state.viewMonth, 1).getDay();
        if(firstDay === 0) firstDay = 7;
        for(let i=1; i<firstDay; i++) {
            grid.appendChild(createCell('', 'empty'));
        }

        // 填充日期
        for(let d=1; d < currentData.length; d++) {
            const status = currentData[d]; // 1=Work, 0=Rest
            const cell = document.createElement('div');
            const isWork = status === 1;
            
            let classes = `cal-cell ${isWork ? 'work' : 'rest'}`;
            if(!editable) classes += ' disabled'; // 非编辑范围置灰

            cell.className = classes;
            cell.innerHTML = `
                <span class="day-number">${d}</span>
                <span class="status-badge">${isWork ? '上班' : '休息'}</span>
            `;

            // 点击事件
            cell.onclick = () => {
                if(isInherit) return; // 继承模式不可点
                if(!editable) {
                    Toast.warning("仅支持当月和下个月的排班调整");
                    return;
                }
                // 切换状态
                currentData[d] = isWork ? 0 : 1;
                markDirty();
                renderCalendar(); // 重绘
            };

            grid.appendChild(cell);
        }
    }

    function createCell(content, type) {
        const div = document.createElement('div');
        div.className = `cal-cell ${type}`;
        return div;
    }

    // === 工具：快捷设置 ===
    function quickSet(type) {
        if(!checkEditable()) return;

        const dataKey = `${state.selectedNode.id}-${state.viewYear}-${state.viewMonth}`;
        const currentData = state.calendarData[dataKey];
        const daysInMonth = currentData.length - 1;

        for(let i=1; i<=daysInMonth; i++) {
            const dayOfWeek = new Date(state.viewYear, state.viewMonth, i).getDay();
            if(type === 'all') currentData[i] = 1;
            else if(type === 'clear') currentData[i] = 0;
            else if(type === 'workdays') {
                // 仅周一至周六
                currentData[i] = (dayOfWeek !== 0) ? 1 : 0;
            }
        }
        markDirty();
        renderCalendar();
    }

    function checkEditable() {
        if(state.currentMode === 'inherit') {
            Toast.warning("跟随模式下不可编辑");
            return false;
        }
        if(!isEditableMonth(state.viewYear, state.viewMonth)) {
            Toast.warning("仅支持当月和下个月的排班调整");
            return false;
        }
        return true;
    }

    function markDirty() {
        state.isDirty = true;
    }

    function resetData() {
        if(!checkEditable()) return;
        state.calendarData[`${state.selectedNode.id}-${state.viewYear}-${state.viewMonth}`] = null;
        renderCalendar();
        state.isDirty = true;
        Toast.show("已重置为默认模板，请重新保存", "warning");
    }

    // === 核心：保存与校验 ===
    async function handleSave() {
        if(state.currentMode === 'inherit') {
            // 1. 继承模式校验：父级有效性
            if(Math.random() > 0.8) { // 模拟父级失效
                Toast.error("父级排班已失效，请切换为自定义配置");
                return;
            }
            // 2. 循环继承检查
            if(checkCircularInheritance()) {
                Toast.error("检测到循环继承，请调整继承关系");
                return;
            }
        } else {
            // 3. 自定义模式校验：至少一天上班
            const dataKey = `${state.selectedNode.id}-${state.viewYear}-${state.viewMonth}`;
            const data = state.calendarData[dataKey];
            const hasWorkDay = data && data.some(s => s === 1);
            if(!hasWorkDay) {
                Toast.error("请至少选择一天工作日");
                return;
            }

            // 4. 时间校验
            const start = document.getElementById('startTime').value;
            const end = document.getElementById('endTime').value;
            if(start >= end) {
                Toast.error("班次开始时间必须早于结束时间");
                return;
            }
        }

        // 5. 并发版本校验 (模拟)
        if(Math.random() > 0.9) {
            Toast.error("排班已被他人更新，请刷新后重试");
            return;
        }

        // 6. 网络重试逻辑 (模拟)
        let retryCount = 0;
        const trySubmit = async () => {
            Toast.show("正在保存...", "warning"); // Loading state
            
            // 模拟 API 延迟
            await new Promise(r => setTimeout(r, 500));
            
            // 模拟成功
            state.isDirty = false;
            state.dataVersion++;
            Toast.success("保存成功！");
        };

        trySubmit();
    }

    function checkCircularInheritance() {
        // 模拟检查逻辑：此处返回 false 代表无循环
        return false; 
    }

</script>
</body>
</html>
