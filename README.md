<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Süreç Yönetimi KPI Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js';
        import { getAuth, GoogleAuthProvider, signInWithPopup, onAuthStateChanged, signOut } from 'https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js';
        
        // Firebase yapılandırması
        const firebaseConfig = {
            apiKey: "AIzaSyAF8ZcI4lYPjnojma094lo_orSfX8I9Fh8",
            authDomain: "akca-pro-x-dashboard.firebaseapp.com",
            databaseURL: "https://operayonanaliz-default-rtdb.europe-west1.firebasedatabase.app/",
            projectId: "akca-pro-x-dashboard",
            storageBucket: "akca-pro-x-dashboard.appspot.com",
            messagingSenderId: "123456789012",
            appId: "1:123456789012:web:abcdef123456789012345678"
        };

        // Firebase'i başlat
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const provider = new GoogleAuthProvider();

        // Global değişkenler
        window.auth = auth;
        window.provider = provider;
        window.signInWithPopup = signInWithPopup;
        window.onAuthStateChanged = onAuthStateChanged;
        window.signOut = signOut;
    </script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .dashboard {
            max-width: 1400px;
            margin: 0 auto;
        }

        .header {
            text-align: center;
            color: white;
            margin-bottom: 30px;
        }

        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .header p {
            font-size: 1.1rem;
            opacity: 0.9;
        }

        .controls {
            background: white;
            padding: 20px;
            border-radius: 15px;
            margin-bottom: 30px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
        }

        .control-group {
            display: flex;
            gap: 20px;
            align-items: center;
            flex-wrap: wrap;
        }

        .control-item {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        .control-item label {
            font-weight: 600;
            color: #333;
            font-size: 0.9rem;
        }

        .control-item input, .control-item select {
            padding: 8px 12px;
            border: 2px solid #e1e5e9;
            border-radius: 8px;
            font-size: 0.9rem;
            transition: border-color 0.3s;
        }

        .control-item input:focus, .control-item select:focus {
            outline: none;
            border-color: #667eea;
        }

        .btn {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
            transition: transform 0.2s;
        }

        .btn:hover {
            transform: translateY(-2px);
        }

        .kpi-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .kpi-card {
            background: white;
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            transition: transform 0.3s, box-shadow 0.3s;
        }

        .kpi-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 15px 40px rgba(0,0,0,0.15);
        }

        .kpi-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .kpi-title {
            font-size: 1.1rem;
            font-weight: 700;
            color: #333;
        }

        .kpi-category {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 0.8rem;
            font-weight: 600;
        }

        .kpi-value {
            font-size: 2.5rem;
            font-weight: 800;
            margin-bottom: 10px;
            background: linear-gradient(135deg, #667eea, #764ba2);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .kpi-description {
            color: #666;
            font-size: 0.9rem;
            line-height: 1.4;
            margin-bottom: 15px;
        }

        .kpi-formula {
            background: #f8f9fa;
            padding: 10px;
            border-radius: 8px;
            font-family: 'Courier New', monospace;
            font-size: 0.85rem;
            color: #495057;
            border-left: 4px solid #667eea;
        }

        .status-indicator {
            display: inline-block;
            width: 12px;
            height: 12px;
            border-radius: 50%;
            margin-left: 10px;
        }

        .status-good { background-color: #28a745; }
        .status-warning { background-color: #ffc107; }
        .status-danger { background-color: #dc3545; }

        .charts-section {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .chart-card {
            background: white;
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
        }

        .chart-title {
            font-size: 1.2rem;
            font-weight: 700;
            color: #333;
            margin-bottom: 20px;
            text-align: center;
        }

        .ai-insights {
            background: linear-gradient(135deg, #28a745, #20c997);
            color: white;
            padding: 25px;
            border-radius: 15px;
            margin-top: 30px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
        }

        .ai-insights h3 {
            font-size: 1.3rem;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .ai-insights ul {
            list-style: none;
        }

        .ai-insights li {
            margin-bottom: 10px;
            padding-left: 20px;
            position: relative;
        }

        .ai-insights li:before {
            content: "🤖";
            position: absolute;
            left: 0;
        }

        .modal {
            display: none;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.5);
            backdrop-filter: blur(5px);
        }

        .modal-content {
            background-color: white;
            margin: 5% auto;
            padding: 30px;
            border-radius: 15px;
            width: 90%;
            max-width: 600px;
            max-height: 80vh;
            overflow-y: auto;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            animation: modalSlideIn 0.3s ease-out;
        }

        @keyframes modalSlideIn {
            from {
                transform: translateY(-50px);
                opacity: 0;
            }
            to {
                transform: translateY(0);
                opacity: 1;
            }
        }

        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 25px;
            padding-bottom: 15px;
            border-bottom: 2px solid #e1e5e9;
        }

        .modal-title {
            font-size: 1.4rem;
            font-weight: 700;
            color: #333;
        }

        .close {
            color: #aaa;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
            transition: color 0.3s;
        }

        .close:hover {
            color: #667eea;
        }

        .form-group {
            margin-bottom: 20px;
        }

        .form-group label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
            color: #333;
        }

        .form-group input {
            width: 100%;
            padding: 12px;
            border: 2px solid #e1e5e9;
            border-radius: 8px;
            font-size: 1rem;
            transition: border-color 0.3s;
        }

        .form-group input:focus {
            outline: none;
            border-color: #667eea;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }

        .form-row {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }

        .calculate-btn {
            background: linear-gradient(135deg, #28a745, #20c997);
            color: white;
            border: none;
            padding: 12px 25px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
            font-size: 1rem;
            width: 100%;
            margin-top: 20px;
            transition: transform 0.2s;
        }

        .calculate-btn:hover {
            transform: translateY(-2px);
        }

        .result-section {
            margin-top: 25px;
            padding: 20px;
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border-radius: 10px;
            display: none;
        }

        .result-value {
            font-size: 2rem;
            font-weight: 800;
            text-align: center;
            margin-bottom: 10px;
        }

        .result-interpretation {
            text-align: center;
            font-size: 1.1rem;
            opacity: 0.9;
        }

        .kpi-card {
            background: white;
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            transition: transform 0.3s, box-shadow 0.3s;
            cursor: pointer;
        }

        .kpi-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 15px 40px rgba(0,0,0,0.15);
        }

        .ai-report-content {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: #333;
        }

        .ai-report-content h3 {
            border-bottom: 2px solid #667eea;
            padding-bottom: 8px;
        }

        .ai-report-content h4 {
            border-left: 4px solid #667eea;
            padding-left: 12px;
        }

        .ai-report-content ul {
            background: #f8f9fa;
            border-radius: 8px;
            padding: 15px 20px;
        }

        .ai-report-content li {
            color: #555;
        }

        .btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none !important;
        }

        .click-hint {
            text-align: center;
            color: #667eea;
            font-size: 0.9rem;
            margin-top: 10px;
            font-weight: 600;
        }

        /* Giriş Ekranı Stilleri */
        .login-container {
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            padding: 20px;
        }

        .login-card {
            background: white;
            padding: 40px;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.2);
            text-align: center;
            max-width: 400px;
            width: 100%;
            animation: loginSlideIn 0.6s ease-out;
        }

        @keyframes loginSlideIn {
            from {
                transform: translateY(-30px);
                opacity: 0;
            }
            to {
                transform: translateY(0);
                opacity: 1;
            }
        }

        .login-logo {
            font-size: 4rem;
            margin-bottom: 20px;
        }

        .login-title {
            font-size: 2rem;
            font-weight: 700;
            color: #333;
            margin-bottom: 10px;
        }

        .login-subtitle {
            color: #666;
            margin-bottom: 30px;
            font-size: 1.1rem;
        }

        .google-login-btn {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 12px;
            background: white;
            border: 2px solid #e1e5e9;
            padding: 15px 25px;
            border-radius: 12px;
            cursor: pointer;
            font-size: 1rem;
            font-weight: 600;
            color: #333;
            transition: all 0.3s;
            width: 100%;
            margin-bottom: 20px;
        }

        .google-login-btn:hover {
            border-color: #667eea;
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.2);
            transform: translateY(-2px);
        }

        .google-icon {
            width: 20px;
            height: 20px;
        }

        .login-features {
            text-align: left;
            margin-top: 30px;
            padding-top: 20px;
            border-top: 1px solid #e1e5e9;
        }

        .login-features h4 {
            color: #333;
            margin-bottom: 15px;
            font-size: 1.1rem;
        }

        .login-features ul {
            list-style: none;
            padding: 0;
        }

        .login-features li {
            margin-bottom: 8px;
            color: #666;
            font-size: 0.9rem;
            padding-left: 20px;
            position: relative;
        }

        .login-features li:before {
            content: "✓";
            position: absolute;
            left: 0;
            color: #28a745;
            font-weight: bold;
        }

        .user-info {
            display: flex;
            align-items: center;
            gap: 15px;
            background: rgba(255,255,255,0.1);
            padding: 10px 15px;
            border-radius: 10px;
            margin-bottom: 20px;
        }

        .user-avatar {
            width: 40px;
            height: 40px;
            border-radius: 50%;
            border: 2px solid white;
        }

        .user-details {
            flex: 1;
            text-align: left;
        }

        .user-name {
            font-weight: 600;
            color: white;
            margin-bottom: 2px;
        }

        .user-email {
            font-size: 0.85rem;
            color: rgba(255,255,255,0.8);
        }

        .logout-btn {
            background: rgba(255,255,255,0.2);
            color: white;
            border: 1px solid rgba(255,255,255,0.3);
            padding: 6px 12px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 0.8rem;
            transition: all 0.3s;
        }

        .logout-btn:hover {
            background: rgba(255,255,255,0.3);
        }

        .dashboard-content {
            display: none;
        }

        .dashboard-content.show {
            display: block;
        }

        @media (max-width: 768px) {
            .control-group {
                flex-direction: column;
                align-items: stretch;
            }
            
            .kpi-grid {
                grid-template-columns: 1fr;
            }
            
            .charts-section {
                grid-template-columns: 1fr;
            }

            .login-card {
                padding: 30px 20px;
            }

            .login-title {
                font-size: 1.6rem;
            }
        }
    </style>
</head>
<body>
    <!-- Giriş Ekranı -->
    <div id="loginContainer" class="login-container">
        <div class="login-card">
            <div class="login-logo">🏭</div>
            <h1 class="login-title">Akça Pro X Dashboard</h1>
            <p class="login-subtitle">Yapay Zeka Destekli İş Süreçleri Analiz Paneli</p>
            
            <button id="googleLoginBtn" class="google-login-btn">
                <svg class="google-icon" viewBox="0 0 24 24">
                    <path fill="#4285F4" d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/>
                    <path fill="#34A853" d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/>
                    <path fill="#FBBC05" d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/>
                    <path fill="#EA4335" d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/>
                </svg>
                Google ile Giriş Yap
            </button>
            
            <div class="login-features">
                <h4>🚀 Platform Özellikleri</h4>
                <ul>
                    <li>Gerçek zamanlı KPI takibi</li>
                    <li>AI destekli analiz raporları</li>
                    <li>Interaktif veri girişi</li>
                    <li>Otomatik trend analizi</li>
                    <li>Güvenli bulut depolama</li>
                </ul>
            </div>
        </div>
    </div>

    <!-- Dashboard İçeriği -->
    <div id="dashboardContent" class="dashboard-content">
        <div class="dashboard">
            <div class="header">
                <div class="user-info" id="userInfo" style="display: none;">
                    <img id="userAvatar" class="user-avatar" src="" alt="Kullanıcı">
                    <div class="user-details">
                        <div class="user-name" id="userName"></div>
                        <div class="user-email" id="userEmail"></div>
                    </div>
                    <button class="logout-btn" onclick="handleLogout()">Çıkış</button>
                </div>
                <h1>🏭 Süreç Yönetimi KPI Dashboard</h1>
                <p>Yapay Zeka Destekli İş Süreçleri Analiz Paneli</p>
            </div>

        <div class="controls">
            <div class="control-group">
                <div class="control-item">
                    <label>Dönem Seçimi</label>
                    <select id="periodSelect">
                        <option value="monthly">Aylık</option>
                        <option value="quarterly">Çeyreklik</option>
                        <option value="yearly">Yıllık</option>
                    </select>
                </div>
                <div class="control-item">
                    <label>Kategori Filtresi</label>
                    <select id="categoryFilter">
                        <option value="all">Tüm KPI'lar</option>
                        <option value="planning">Planlama</option>
                        <option value="production">Üretim</option>
                        <option value="sales">Satış</option>
                        <option value="hr">İnsan Kaynakları</option>
                        <option value="inventory">Stok Yönetimi</option>
                        <option value="logistics">Lojistik</option>
                    </select>
                </div>
                <div class="control-item">
                    <label>Hedef Değer</label>
                    <input type="number" id="targetValue" placeholder="Hedef %" step="0.1">
                </div>
                <button class="btn" onclick="updateDashboard()">Güncelle</button>
                <button class="btn" onclick="generateReport()">Rapor Oluştur</button>
            </div>
        </div>

        <div class="kpi-grid" id="kpiGrid">
            <!-- KPI kartları buraya dinamik olarak eklenecek -->
        </div>

        <div class="charts-section">
            <div class="chart-card">
                <div class="chart-title">KPI Trend Analizi</div>
                <canvas id="trendChart"></canvas>
            </div>
            <div class="chart-card">
                <div class="chart-title">Kategori Bazında Performans</div>
                <canvas id="categoryChart"></canvas>
            </div>
            <div class="chart-card">
                <div class="chart-title">Hedef vs Gerçekleşen</div>
                <canvas id="targetChart"></canvas>
            </div>
            <div class="chart-card">
                <div class="chart-title">Risk Analizi</div>
                <canvas id="riskChart"></canvas>
            </div>
        </div>

        <div class="ai-insights">
            <h3>🤖 Akça Pro X Dashboard AI Önerileri</h3>
            <div id="aiInsightsContainer">
                <div style="text-align: center; color: rgba(255,255,255,0.8); font-style: italic; padding: 20px;">
                    📊 Dashboard verilerinizi analiz etmek için "Rapor Oluştur" butonuna tıklayın.<br>
                    Akça Pro X Dashboard AI size özel öneriler sunacak!
                </div>
            </div>
            </div>
        </div>
    </div>

    <!-- KPI Hesaplama Modal -->
    <div id="kpiModal" class="modal">
        <div class="modal-content">
            <div class="modal-header">
                <h2 class="modal-title" id="modalTitle">KPI Hesaplama</h2>
                <span class="close" onclick="closeModal()">&times;</span>
            </div>
            
            <div id="modalBody">
                <!-- Form içeriği dinamik olarak yüklenecek -->
            </div>
        </div>
    </div>

    <script>
        // KPI veri yapısı
        const kpiData = {
            planning: [
                {
                    name: "Talep Tahmin Doğruluğu",
                    value: 87.5,
                    unit: "%",
                    formula: "1 - ((Gerçekleşen Talep - Tahmin Edilen Talep) / Gerçekleşen Talep)",
                    description: "Gerçekleşen taleple yapılan tahmin arasındaki farkı ölçer. 1'e yaklaştıkça tahmin doğruluğu artar.",
                    target: 90,
                    trend: [82, 85, 87.5, 89, 87.5]
                },
                {
                    name: "Üretim Planına Uygunluk",
                    value: 92.3,
                    unit: "%",
                    formula: "Gerçekleşen Üretim Adedi / Planlanan Üretim Adedi",
                    description: "Planlanan üretim hacminin ne kadarının gerçekten üretildiğini gösterir.",
                    target: 95,
                    trend: [88, 90, 92.3, 94, 92.3]
                }
            ],
            production: [
                {
                    name: "OEE (Toplam Ekipman Etkinliği)",
                    value: 78.2,
                    unit: "%",
                    formula: "OEE = Kullanılabilirlik x Performans x Kalite",
                    description: "Üretim ekipmanlarının ne kadar etkili kullanıldığını gösterir. %100 mükemmel verimliliği ifade eder.",
                    target: 85,
                    trend: [75, 77, 78.2, 80, 78.2]
                },
                {
                    name: "Birim Başına Üretim Maliyeti",
                    value: 45.67,
                    unit: "₺",
                    formula: "Toplam Üretim Maliyeti / Üretilen Ürün Adedi",
                    description: "Bir adet ürün üretmek için yapılan toplam harcamayı gösterir.",
                    target: 42,
                    trend: [48, 47, 45.67, 44, 45.67]
                }
            ],
            sales: [
                {
                    name: "Müşteri Edinme Maliyeti (CAC)",
                    value: 125.50,
                    unit: "₺",
                    formula: "Toplam Pazarlama ve Satış Giderleri / Kazanılan Müşteri Sayısı",
                    description: "Bir müşteriyi kazanmak için yapılan ortalama pazarlama ve satış harcamasıdır.",
                    target: 100,
                    trend: [140, 130, 125.50, 120, 125.50]
                },
                {
                    name: "Müşteri Kayıp Oranı",
                    value: 8.3,
                    unit: "%",
                    formula: "Dönem Sonunda Kaybedilen Müşteri Sayısı / Dönem Başındaki Toplam Müşteri Sayısı",
                    description: "Belirli bir dönemde kaybedilen müşteri sayısının toplam müşteri sayısına oranıdır.",
                    target: 5,
                    trend: [10, 9, 8.3, 7, 8.3]
                }
            ],
            hr: [
                {
                    name: "Çalışan Devir Hızı",
                    value: 12.5,
                    unit: "%",
                    formula: "(İşten Ayrılan Çalışan Sayısı / Ortalama Çalışan Sayısı) * 100",
                    description: "Belirli bir dönemde işten ayrılan çalışan sayısının ortalama çalışan sayısına oranıdır.",
                    target: 10,
                    trend: [15, 14, 12.5, 11, 12.5]
                },
                {
                    name: "Eğitim ROI",
                    value: 2.8,
                    unit: "x",
                    formula: "(Eğitimden Sağlanan Fayda - Eğitim Maliyeti) / Eğitim Maliyeti",
                    description: "Eğitim programlarına yapılan harcamanın şirkete sağladığı değeri ölçer.",
                    target: 3,
                    trend: [2.2, 2.5, 2.8, 3.1, 2.8]
                }
            ],
            inventory: [
                {
                    name: "Stok Devir Hızı",
                    value: 6.2,
                    unit: "kez/yıl",
                    formula: "Dönemdeki Satılan Malın Maliyeti / Ortalama Stok Değeri",
                    description: "Belirli bir dönemde stoğun kaç kez satıldığını veya kullanıldığını ölçer.",
                    target: 8,
                    trend: [5.5, 6.0, 6.2, 6.8, 6.2]
                },
                {
                    name: "Stok Doğruluk Oranı",
                    value: 94.7,
                    unit: "%",
                    formula: "1 - ((Sistem Stoğu - Fiziksel Stok) / Sistem Stoğu)",
                    description: "Depodaki fiziksel stok adedi ile sistemdeki kayıtlı stok adedi arasındaki tutarlılığı ölçer.",
                    target: 98,
                    trend: [92, 93, 94.7, 96, 94.7]
                }
            ],
            logistics: [
                {
                    name: "Sipariş Karşılama Süresi",
                    value: 3.2,
                    unit: "gün",
                    formula: "Ortalama Karşılama Süresi = Toplam (Teslim Tarihi - Sipariş Tarihi) / Sipariş Sayısı",
                    description: "Bir siparişin verildiği andan müşteriye teslim edildiği ana kadar geçen toplam süreyi ölçer.",
                    target: 2.5,
                    trend: [4.0, 3.5, 3.2, 2.8, 3.2]
                },
                {
                    name: "Sevkiyat Doğruluk Oranı",
                    value: 96.8,
                    unit: "%",
                    formula: "Doğru Sevkiyat Adedi / Toplam Sevkiyat Adedi",
                    description: "Gönderilen sevkiyatların, siparişle birebir eşleşme oranını ölçer.",
                    target: 99,
                    trend: [94, 95, 96.8, 98, 96.8]
                }
            ]
        };

        // Grafik nesneleri
        let trendChart, categoryChart, targetChart, riskChart;

        // Firebase Auth durumunu kontrol et
        let currentUser = null;

        // Sayfa yüklendiğinde auth durumunu kontrol et
        document.addEventListener('DOMContentLoaded', function() {
            // Auth durumu değişikliklerini dinle
            window.onAuthStateChanged(window.auth, (user) => {
                if (user) {
                    // Kullanıcı giriş yapmış
                    currentUser = user;
                    showDashboard(user);
                } else {
                    // Kullanıcı giriş yapmamış
                    currentUser = null;
                    showLogin();
                }
            });

            // Google giriş butonuna event listener ekle
            document.getElementById('googleLoginBtn').addEventListener('click', handleGoogleLogin);
        });

        // Google ile giriş fonksiyonu
        async function handleGoogleLogin() {
            const loginBtn = document.getElementById('googleLoginBtn');
            const originalText = loginBtn.innerHTML;
            
            try {
                // Butonu devre dışı bırak
                loginBtn.disabled = true;
                loginBtn.innerHTML = `
                    <div style="display: flex; align-items: center; gap: 10px;">
                        <div style="width: 20px; height: 20px; border: 2px solid #667eea; border-top: 2px solid transparent; border-radius: 50%; animation: spin 1s linear infinite;"></div>
                        Giriş yapılıyor...
                    </div>
                `;
                
                // CSS animasyonu ekle
                if (!document.getElementById('spinAnimation')) {
                    const style = document.createElement('style');
                    style.id = 'spinAnimation';
                    style.textContent = `
                        @keyframes spin {
                            0% { transform: rotate(0deg); }
                            100% { transform: rotate(360deg); }
                        }
                    `;
                    document.head.appendChild(style);
                }

                // Google ile giriş yap
                const result = await window.signInWithPopup(window.auth, window.provider);
                const user = result.user;
                
                console.log('Giriş başarılı:', user.displayName);
                
            } catch (error) {
                console.error('Giriş hatası:', error);
                
                let errorMessage = 'Giriş yapılırken bir hata oluştu.';
                
                switch (error.code) {
                    case 'auth/popup-closed-by-user':
                        errorMessage = 'Giriş penceresi kapatıldı. Lütfen tekrar deneyin.';
                        break;
                    case 'auth/popup-blocked':
                        errorMessage = 'Pop-up engellendi. Lütfen tarayıcı ayarlarınızı kontrol edin.';
                        break;
                    case 'auth/cancelled-popup-request':
                        errorMessage = 'Giriş işlemi iptal edildi.';
                        break;
                    default:
                        errorMessage = `Hata: ${error.message}`;
                }
                
                alert('❌ ' + errorMessage);
                
                // Butonu tekrar aktif et
                loginBtn.disabled = false;
                loginBtn.innerHTML = originalText;
            }
        }

        // Dashboard'u göster
        function showDashboard(user) {
            document.getElementById('loginContainer').style.display = 'none';
            document.getElementById('dashboardContent').classList.add('show');
            
            // Kullanıcı bilgilerini göster
            const userInfo = document.getElementById('userInfo');
            const userAvatar = document.getElementById('userAvatar');
            const userName = document.getElementById('userName');
            const userEmail = document.getElementById('userEmail');
            
            userAvatar.src = user.photoURL || 'https://via.placeholder.com/40';
            userName.textContent = user.displayName || 'Kullanıcı';
            userEmail.textContent = user.email || '';
            userInfo.style.display = 'flex';
            
            // Dashboard'u başlat
            updateDashboard();
            initializeCharts();
            
            // Hoş geldin mesajı
            setTimeout(() => {
                alert(`🎉 Hoş geldiniz ${user.displayName}!\n\nAkça Pro X Dashboard'a başarıyla giriş yaptınız.`);
            }, 500);
        }

        // Giriş ekranını göster
        function showLogin() {
            document.getElementById('loginContainer').style.display = 'flex';
            document.getElementById('dashboardContent').classList.remove('show');
            document.getElementById('userInfo').style.display = 'none';
        }

        // Çıkış fonksiyonu
        async function handleLogout() {
            if (confirm('Çıkış yapmak istediğinizden emin misiniz?')) {
                try {
                    await window.signOut(window.auth);
                    alert('👋 Başarıyla çıkış yaptınız!');
                } catch (error) {
                    console.error('Çıkış hatası:', error);
                    alert('❌ Çıkış yapılırken bir hata oluştu.');
                }
            }
        }

        function updateDashboard() {
            const category = document.getElementById('categoryFilter').value;
            const kpiGrid = document.getElementById('kpiGrid');
            
            kpiGrid.innerHTML = '';
            
            const categoriesToShow = category === 'all' ? Object.keys(kpiData) : [category];
            
            categoriesToShow.forEach(cat => {
                if (kpiData[cat]) {
                    kpiData[cat].forEach(kpi => {
                        const kpiCard = createKPICard(kpi, cat);
                        kpiGrid.appendChild(kpiCard);
                    });
                }
            });
            
            updateCharts();
            updateAIInsights();
        }

        function createKPICard(kpi, category) {
            const card = document.createElement('div');
            card.className = 'kpi-card';
            
            const status = getKPIStatus(kpi.value, kpi.target, kpi.unit);
            const categoryNames = {
                planning: 'Planlama',
                production: 'Üretim',
                sales: 'Satış',
                hr: 'İK',
                inventory: 'Stok',
                logistics: 'Lojistik'
            };
            
            card.innerHTML = `
                <div class="kpi-header">
                    <div class="kpi-title">${kpi.name}</div>
                    <div class="kpi-category">${categoryNames[category]}</div>
                </div>
                <div class="kpi-value">
                    ${kpi.value}${kpi.unit}
                    <span class="status-indicator ${status}"></span>
                </div>
                <div class="kpi-description">${kpi.description}</div>
                <div class="kpi-formula">${kpi.formula}</div>
                <div class="click-hint">📊 Veri girmek için tıklayın</div>
            `;
            
            // Karta tıklama olayı ekle
            card.addEventListener('click', () => openKPIModal(kpi, category));
            
            return card;
        }

        function getKPIStatus(value, target, unit) {
            if (unit === '%' || unit === 'x') {
                if (value >= target * 0.9) return 'status-good';
                if (value >= target * 0.7) return 'status-warning';
                return 'status-danger';
            } else {
                // Maliyet ve süre KPI'ları için ters mantık
                if (value <= target * 1.1) return 'status-good';
                if (value <= target * 1.3) return 'status-warning';
                return 'status-danger';
            }
        }

        function initializeCharts() {
            // Trend Chart
            const trendCtx = document.getElementById('trendChart').getContext('2d');
            trendChart = new Chart(trendCtx, {
                type: 'line',
                data: {
                    labels: ['Ocak', 'Şubat', 'Mart', 'Nisan', 'Mayıs'],
                    datasets: []
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'top',
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            });

            // Category Chart
            const categoryCtx = document.getElementById('categoryChart').getContext('2d');
            categoryChart = new Chart(categoryCtx, {
                type: 'radar',
                data: {
                    labels: ['Planlama', 'Üretim', 'Satış', 'İK', 'Stok', 'Lojistik'],
                    datasets: [{
                        label: 'Performans Skoru',
                        data: [85, 78, 72, 80, 88, 92],
                        backgroundColor: 'rgba(102, 126, 234, 0.2)',
                        borderColor: 'rgba(102, 126, 234, 1)',
                        pointBackgroundColor: 'rgba(102, 126, 234, 1)',
                        pointBorderColor: '#fff',
                        pointHoverBackgroundColor: '#fff',
                        pointHoverBorderColor: 'rgba(102, 126, 234, 1)'
                    }]
                },
                options: {
                    responsive: true,
                    scales: {
                        r: {
                            beginAtZero: true,
                            max: 100
                        }
                    }
                }
            });

            // Target vs Actual Chart
            const targetCtx = document.getElementById('targetChart').getContext('2d');
            targetChart = new Chart(targetCtx, {
                type: 'bar',
                data: {
                    labels: ['Talep Tahmin', 'OEE', 'CAC', 'Devir Hızı', 'Stok Devir', 'Karşılama Süresi'],
                    datasets: [{
                        label: 'Hedef',
                        data: [90, 85, 100, 10, 8, 2.5],
                        backgroundColor: 'rgba(40, 167, 69, 0.8)'
                    }, {
                        label: 'Gerçekleşen',
                        data: [87.5, 78.2, 125.5, 12.5, 6.2, 3.2],
                        backgroundColor: 'rgba(102, 126, 234, 0.8)'
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'top',
                        }
                    }
                }
            });

            // Risk Chart
            const riskCtx = document.getElementById('riskChart').getContext('2d');
            riskChart = new Chart(riskCtx, {
                type: 'doughnut',
                data: {
                    labels: ['Düşük Risk', 'Orta Risk', 'Yüksek Risk'],
                    datasets: [{
                        data: [60, 25, 15],
                        backgroundColor: [
                            'rgba(40, 167, 69, 0.8)',
                            'rgba(255, 193, 7, 0.8)',
                            'rgba(220, 53, 69, 0.8)'
                        ]
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'bottom',
                        }
                    }
                }
            });
        }

        function updateCharts() {
            // Grafikleri güncelle
            if (trendChart) {
                const category = document.getElementById('categoryFilter').value;
                const datasets = [];
                
                if (category === 'all') {
                    // Tüm kategoriler için örnek veri
                    datasets.push({
                        label: 'Genel Performans',
                        data: [75, 78, 82, 85, 83],
                        borderColor: 'rgba(102, 126, 234, 1)',
                        backgroundColor: 'rgba(102, 126, 234, 0.1)',
                        tension: 0.4
                    });
                } else if (kpiData[category]) {
                    kpiData[category].forEach((kpi, index) => {
                        datasets.push({
                            label: kpi.name,
                            data: kpi.trend,
                            borderColor: `hsl(${index * 60}, 70%, 50%)`,
                            backgroundColor: `hsla(${index * 60}, 70%, 50%, 0.1)`,
                            tension: 0.4
                        });
                    });
                }
                
                trendChart.data.datasets = datasets;
                trendChart.update();
            }
        }

        function updateAIInsights() {
            // AI önerileri artık sadece rapor oluşturulduğunda gösterilecek
            // Bu fonksiyon boş bırakıldı
        }

        async function generateReport() {
            const reportBtn = document.querySelector('button[onclick="generateReport()"]');
            const originalText = reportBtn.textContent;
            
            try {
                // Butonu devre dışı bırak ve yükleme göster
                reportBtn.disabled = true;
                reportBtn.textContent = '🤖 Akça Pro X AI Analiz Ediyor...';
                
                // AI öneriler bölümünü güncelle
                updateAIInsightsSection('🤖 Akça Pro X Dashboard AI verilerinizi analiz ediyor...');
                
                // Mevcut KPI verilerini topla
                const reportData = collectKPIData();
                
                // Google AI API'ye öneriler için ayrı istek gönder
                const insights = await generateAIInsights(reportData);
                
                // AI öneriler bölümünü güncelle
                displayAIInsights(insights);
                
                // Google AI API'ye rapor için istek gönder
                const report = await generateAIReport(reportData);
                
                // Raporu modal olarak göster
                showReportModal(report);
                
            } catch (error) {
                console.error('Rapor oluşturma hatası:', error);
                updateAIInsightsSection('❌ Akça Pro X Dashboard AI analizi sırasında hata oluştu. Lütfen tekrar deneyin.');
                alert('❌ Rapor oluşturulurken bir hata oluştu. Lütfen tekrar deneyin.\n\nHata: ' + error.message);
            } finally {
                // Butonu tekrar aktif et
                reportBtn.disabled = false;
                reportBtn.textContent = originalText;
            }
        }

        function collectKPIData() {
            const category = document.getElementById('categoryFilter').value;
            const period = document.getElementById('periodSelect').value;
            
            let allKPIs = [];
            const categoriesToAnalyze = category === 'all' ? Object.keys(kpiData) : [category];
            
            categoriesToAnalyze.forEach(cat => {
                if (kpiData[cat]) {
                    kpiData[cat].forEach(kpi => {
                        allKPIs.push({
                            category: cat,
                            name: kpi.name,
                            value: kpi.value,
                            unit: kpi.unit,
                            target: kpi.target,
                            trend: kpi.trend,
                            performance: ((kpi.value / kpi.target) * 100).toFixed(1)
                        });
                    });
                }
            });
            
            return {
                period: period,
                category: category,
                kpis: allKPIs,
                totalKPIs: allKPIs.length,
                averagePerformance: (allKPIs.reduce((sum, kpi) => sum + parseFloat(kpi.performance), 0) / allKPIs.length).toFixed(1)
            };
        }

        async function generateAIInsights(data) {
            const API_KEY = 'AIzaSyD0lUqjjKw6-vMhh3_scWlPjG7D9Y8dD9s';
            const API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent';
            
            const prompt = `
Dashboard'daki KPI verilerine göre kısa ve öz öneriler ver. Sadece 4-5 madde halinde, her biri maksimum 15 kelime olsun:

KPI DURUMU:
${data.kpis.map(kpi => `• ${kpi.name}: ${kpi.value}${kpi.unit} (Hedef: ${kpi.target}${kpi.unit})`).join('\n')}

Lütfen sadece madde işaretli liste halinde, kısa ve uygulanabilir öneriler ver. Her öneri bir satır olsun.
            `;

            const response = await fetch(`${API_URL}?key=${API_KEY}`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{
                            text: prompt
                        }]
                    }]
                })
            });

            if (!response.ok) {
                throw new Error(`API Hatası: ${response.status} - ${response.statusText}`);
            }

            const result = await response.json();
            
            if (result.candidates && result.candidates[0] && result.candidates[0].content) {
                return result.candidates[0].content.parts[0].text;
            } else {
                throw new Error('AI\'dan geçerli bir yanıt alınamadı');
            }
        }

        function updateAIInsightsSection(message) {
            const container = document.getElementById('aiInsightsContainer');
            container.innerHTML = `
                <div style="text-align: center; color: rgba(255,255,255,0.9); padding: 20px;">
                    ${message}
                </div>
            `;
        }

        function displayAIInsights(insights) {
            const container = document.getElementById('aiInsightsContainer');
            
            // AI yanıtını madde işaretli listeye çevir
            const lines = insights.split('\n').filter(line => line.trim());
            const listItems = lines
                .filter(line => line.includes('•') || line.includes('-') || line.includes('*'))
                .map(line => line.replace(/^[•\-*]\s*/, '').trim())
                .slice(0, 5); // Maksimum 5 öneri

            if (listItems.length > 0) {
                container.innerHTML = `
                    <ul style="list-style: none; margin: 0; padding: 0;">
                        ${listItems.map(item => `
                            <li style="margin-bottom: 10px; padding-left: 20px; position: relative;">
                                <span style="position: absolute; left: 0;">🤖</span>
                                ${item}
                            </li>
                        `).join('')}
                    </ul>
                `;
            } else {
                container.innerHTML = `
                    <div style="text-align: center; color: rgba(255,255,255,0.8); padding: 20px;">
                        ✅ Akça Pro X Dashboard AI analizi tamamlandı! Detaylı rapor için modal pencereyi kontrol edin.
                    </div>
                `;
            }
        }

        async function generateAIReport(data) {
            const API_KEY = 'AIzaSyD0lUqjjKw6-vMhh3_scWlPjG7D9Y8dD9s';
            const API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent';
            
            const prompt = `
Türkçe olarak profesyonel bir KPI analiz raporu oluştur. Aşağıdaki veriler bir şirketin süreç yönetimi dashboard'undan geliyor:

GENEL BİLGİLER:
- Analiz Dönemi: ${data.period === 'monthly' ? 'Aylık' : data.period === 'quarterly' ? 'Çeyreklik' : 'Yıllık'}
- Kategori: ${data.category === 'all' ? 'Tüm KPI\'lar' : data.category}
- Toplam KPI Sayısı: ${data.totalKPIs}
- Ortalama Performans: %${data.averagePerformance}

KPI DETAYLARI:
${data.kpis.map(kpi => `
• ${kpi.name} (${kpi.category})
  - Mevcut Değer: ${kpi.value}${kpi.unit}
  - Hedef: ${kpi.target}${kpi.unit}
  - Performans: %${kpi.performance}
  - Trend: ${kpi.trend.join(' → ')}
`).join('')}

Lütfen şu başlıklar altında detaylı bir rapor hazırla:

1. YÖNETİCİ ÖZETİ
2. PERFORMANS ANALİZİ
3. GÜÇLÜ YÖNLER
4. İYİLEŞTİRME ALANLARI
5. DETAYLI ÖNERİLER
6. AKSİYON PLANI
7. RİSK ANALİZİ

Rapor profesyonel, anlaşılır ve uygulanabilir öneriler içersin. Her bölüm için somut veriler ve örnekler kullan.
            `;

            const response = await fetch(`${API_URL}?key=${API_KEY}`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{
                            text: prompt
                        }]
                    }]
                })
            });

            if (!response.ok) {
                throw new Error(`API Hatası: ${response.status} - ${response.statusText}`);
            }

            const result = await response.json();
            
            if (result.candidates && result.candidates[0] && result.candidates[0].content) {
                return result.candidates[0].content.parts[0].text;
            } else {
                throw new Error('AI\'dan geçerli bir yanıt alınamadı');
            }
        }

        function showReportModal(reportContent) {
            // Rapor modal'ını oluştur
            const modal = document.createElement('div');
            modal.className = 'modal';
            modal.style.display = 'block';
            modal.innerHTML = `
                <div class="modal-content" style="max-width: 900px; max-height: 90vh;">
                    <div class="modal-header">
                        <h2 class="modal-title">🤖 Akça Pro X Dashboard AI Analiz Raporu</h2>
                        <span class="close" onclick="this.closest('.modal').remove()">&times;</span>
                    </div>
                    <div style="max-height: 70vh; overflow-y: auto; padding: 20px; line-height: 1.6;">
                        <div class="ai-report-content">
                            ${formatReportContent(reportContent)}
                        </div>
                    </div>
                    <div style="padding: 20px; border-top: 2px solid #e1e5e9; display: flex; gap: 10px; justify-content: flex-end;">
                        <button class="btn" onclick="downloadReport('${reportContent.replace(/'/g, "\\'")}')">📥 PDF İndir</button>
                        <button class="btn" onclick="shareReport('${reportContent.replace(/'/g, "\\'")}')">📤 Paylaş</button>
                        <button class="btn" onclick="this.closest('.modal').remove()">Kapat</button>
                    </div>
                </div>
            `;
            
            document.body.appendChild(modal);
            
            // Modal dışına tıklandığında kapat
            modal.onclick = function(event) {
                if (event.target === modal) {
                    modal.remove();
                }
            }
        }

        function formatReportContent(content) {
            // Markdown benzeri formatlamayı HTML'e çevir
            return content
                .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
                .replace(/\*(.*?)\*/g, '<em>$1</em>')
                .replace(/^(\d+\.\s+[A-ZÜĞŞÇÖI][^:]+):?$/gm, '<h3 style="color: #667eea; margin-top: 25px; margin-bottom: 15px; font-size: 1.2rem;">$1</h3>')
                .replace(/^([A-ZÜĞŞÇÖI][^:]+):$/gm, '<h4 style="color: #333; margin-top: 20px; margin-bottom: 10px; font-size: 1.1rem;">$1:</h4>')
                .replace(/^-\s+(.+)$/gm, '<li style="margin-bottom: 8px;">$1</li>')
                .replace(/(<li.*<\/li>)/s, '<ul style="margin: 10px 0; padding-left: 20px;">$1</ul>')
                .replace(/\n\n/g, '</p><p style="margin-bottom: 15px;">')
                .replace(/^(?!<[h3|h4|ul|li])(.+)$/gm, '<p style="margin-bottom: 15px;">$1</p>')
                .replace(/^<p[^>]*><\/p>$/gm, '');
        }

        function downloadReport(content) {
            // Basit metin dosyası olarak indir
            const blob = new Blob([content], { type: 'text/plain;charset=utf-8' });
            const url = window.URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `KPI_Analiz_Raporu_${new Date().toISOString().split('T')[0]}.txt`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            window.URL.revokeObjectURL(url);
            
            alert('📥 Rapor başarıyla indirildi!');
        }

        function shareReport(content) {
            if (navigator.share) {
                navigator.share({
                    title: 'KPI Analiz Raporu',
                    text: content.substring(0, 200) + '...',
                    url: window.location.href
                });
            } else {
                // Clipboard'a kopyala
                navigator.clipboard.writeText(content).then(() => {
                    alert('📋 Rapor panoya kopyalandı! Artık istediğiniz yere yapıştırabilirsiniz.');
                }).catch(() => {
                    alert('📤 Paylaşım özelliği bu tarayıcıda desteklenmiyor. Raporu manuel olarak kopyalayabilirsiniz.');
                });
            }
        }

        // Modal fonksiyonları
        function openKPIModal(kpi, category) {
            const modal = document.getElementById('kpiModal');
            const modalTitle = document.getElementById('modalTitle');
            const modalBody = document.getElementById('modalBody');
            
            modalTitle.textContent = `${kpi.name} - Veri Girişi`;
            
            // KPI'ya göre form oluştur
            const formHTML = generateKPIForm(kpi, category);
            modalBody.innerHTML = formHTML;
            
            modal.style.display = 'block';
        }

        function closeModal() {
            document.getElementById('kpiModal').style.display = 'none';
        }

        function generateKPIForm(kpi, category) {
            const forms = {
                'Talep Tahmin Doğruluğu': `
                    <div class="form-group">
                        <label>Gerçekleşen Talep (Adet)</label>
                        <input type="number" id="actualDemand" placeholder="Örn: 1000" step="1">
                    </div>
                    <div class="form-group">
                        <label>Tahmin Edilen Talep (Adet)</label>
                        <input type="number" id="forecastDemand" placeholder="Örn: 950" step="1">
                    </div>
                    <button class="calculate-btn" onclick="calculateDemandAccuracy()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Üretim Planına Uygunluk': `
                    <div class="form-group">
                        <label>Gerçekleşen Üretim (Adet)</label>
                        <input type="number" id="actualProduction" placeholder="Örn: 920" step="1">
                    </div>
                    <div class="form-group">
                        <label>Planlanan Üretim (Adet)</label>
                        <input type="number" id="plannedProduction" placeholder="Örn: 1000" step="1">
                    </div>
                    <button class="calculate-btn" onclick="calculateProductionCompliance()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'OEE (Toplam Ekipman Etkinliği)': `
                    <div class="form-group">
                        <label>Kullanılabilirlik (%)</label>
                        <input type="number" id="availability" placeholder="Örn: 85" step="0.1" max="100">
                    </div>
                    <div class="form-group">
                        <label>Performans (%)</label>
                        <input type="number" id="performance" placeholder="Örn: 90" step="0.1" max="100">
                    </div>
                    <div class="form-group">
                        <label>Kalite (%)</label>
                        <input type="number" id="quality" placeholder="Örn: 95" step="0.1" max="100">
                    </div>
                    <button class="calculate-btn" onclick="calculateOEE()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Birim Başına Üretim Maliyeti': `
                    <div class="form-group">
                        <label>Toplam Üretim Maliyeti (₺)</label>
                        <input type="number" id="totalCost" placeholder="Örn: 45670" step="0.01">
                    </div>
                    <div class="form-group">
                        <label>Üretilen Ürün Adedi</label>
                        <input type="number" id="producedUnits" placeholder="Örn: 1000" step="1">
                    </div>
                    <button class="calculate-btn" onclick="calculateUnitCost()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Müşteri Edinme Maliyeti (CAC)': `
                    <div class="form-group">
                        <label>Toplam Pazarlama Giderleri (₺)</label>
                        <input type="number" id="marketingCost" placeholder="Örn: 50000" step="0.01">
                    </div>
                    <div class="form-group">
                        <label>Toplam Satış Giderleri (₺)</label>
                        <input type="number" id="salesCost" placeholder="Örn: 30000" step="0.01">
                    </div>
                    <div class="form-group">
                        <label>Kazanılan Müşteri Sayısı</label>
                        <input type="number" id="newCustomers" placeholder="Örn: 640" step="1">
                    </div>
                    <button class="calculate-btn" onclick="calculateCAC()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Müşteri Kayıp Oranı': `
                    <div class="form-group">
                        <label>Dönem Başındaki Toplam Müşteri Sayısı</label>
                        <input type="number" id="startCustomers" placeholder="Örn: 1200" step="1">
                    </div>
                    <div class="form-group">
                        <label>Dönem Sonunda Kaybedilen Müşteri Sayısı</label>
                        <input type="number" id="lostCustomers" placeholder="Örn: 100" step="1">
                    </div>
                    <button class="calculate-btn" onclick="calculateChurnRate()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Çalışan Devir Hızı': `
                    <div class="form-group">
                        <label>İşten Ayrılan Çalışan Sayısı</label>
                        <input type="number" id="leftEmployees" placeholder="Örn: 15" step="1">
                    </div>
                    <div class="form-group">
                        <label>Ortalama Çalışan Sayısı</label>
                        <input type="number" id="avgEmployees" placeholder="Örn: 120" step="1">
                    </div>
                    <button class="calculate-btn" onclick="calculateTurnoverRate()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Eğitim ROI': `
                    <div class="form-group">
                        <label>Eğitim Maliyeti (₺)</label>
                        <input type="number" id="trainingCost" placeholder="Örn: 25000" step="0.01">
                    </div>
                    <div class="form-group">
                        <label>Eğitimden Sağlanan Fayda (₺)</label>
                        <input type="number" id="trainingBenefit" placeholder="Örn: 95000" step="0.01">
                    </div>
                    <button class="calculate-btn" onclick="calculateTrainingROI()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Stok Devir Hızı': `
                    <div class="form-group">
                        <label>Dönemdeki Satılan Malın Maliyeti (₺)</label>
                        <input type="number" id="cogs" placeholder="Örn: 620000" step="0.01">
                    </div>
                    <div class="form-group">
                        <label>Ortalama Stok Değeri (₺)</label>
                        <input type="number" id="avgInventory" placeholder="Örn: 100000" step="0.01">
                    </div>
                    <button class="calculate-btn" onclick="calculateInventoryTurnover()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Stok Doğruluk Oranı': `
                    <div class="form-group">
                        <label>Sistem Stoğu (Adet)</label>
                        <input type="number" id="systemStock" placeholder="Örn: 1000" step="1">
                    </div>
                    <div class="form-group">
                        <label>Fiziksel Stok (Adet)</label>
                        <input type="number" id="physicalStock" placeholder="Örn: 947" step="1">
                    </div>
                    <button class="calculate-btn" onclick="calculateStockAccuracy()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Sipariş Karşılama Süresi': `
                    <div class="form-group">
                        <label>Toplam Sipariş Sayısı</label>
                        <input type="number" id="totalOrders" placeholder="Örn: 150" step="1">
                    </div>
                    <div class="form-group">
                        <label>Toplam Karşılama Süresi (Gün)</label>
                        <input type="number" id="totalFulfillmentTime" placeholder="Örn: 480" step="0.1">
                    </div>
                    <button class="calculate-btn" onclick="calculateFulfillmentTime()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `,
                'Sevkiyat Doğruluk Oranı': `
                    <div class="form-group">
                        <label>Toplam Sevkiyat Adedi</label>
                        <input type="number" id="totalShipments" placeholder="Örn: 320" step="1">
                    </div>
                    <div class="form-group">
                        <label>Doğru Sevkiyat Adedi</label>
                        <input type="number" id="correctShipments" placeholder="Örn: 310" step="1">
                    </div>
                    <button class="calculate-btn" onclick="calculateShipmentAccuracy()">Hesapla</button>
                    <div class="result-section" id="result">
                        <div class="result-value" id="resultValue"></div>
                        <div class="result-interpretation" id="resultText"></div>
                        <button class="calculate-btn" id="selectBtn" onclick="selectCalculatedValue()" style="display:none; background: linear-gradient(135deg, #28a745, #20c997); margin-top: 15px;">Bu Değeri Seç ve Dashboard'a Uygula</button>
                    </div>
                `
            };
            
            return forms[kpi.name] || '<p>Bu KPI için henüz form hazırlanmamış.</p>';
        }

        // Hesaplama fonksiyonları
        function calculateDemandAccuracy() {
            const actual = parseFloat(document.getElementById('actualDemand').value);
            const forecast = parseFloat(document.getElementById('forecastDemand').value);
            
            if (actual && forecast && actual > 0) {
                // Doğru formül: Mutlak hata yüzdesini 100'den çıkar
                const errorPercentage = Math.abs(actual - forecast) / actual * 100;
                const accuracy = Math.max(0, 100 - errorPercentage);
                showResult(accuracy.toFixed(1) + '%', getAccuracyInterpretation(accuracy));
                
                // Geçici olarak hesaplanan değeri sakla
                window.calculatedValue = accuracy.toFixed(1);
                window.calculatedKPI = 'Talep Tahmin Doğruluğu';
            }
        }

        function calculateProductionCompliance() {
            const actual = parseFloat(document.getElementById('actualProduction').value);
            const planned = parseFloat(document.getElementById('plannedProduction').value);
            
            if (actual && planned && planned > 0) {
                const compliance = (actual / planned) * 100;
                showResult(compliance.toFixed(1) + '%', getComplianceInterpretation(compliance));
                
                // Geçici olarak hesaplanan değeri sakla
                window.calculatedValue = compliance.toFixed(1);
                window.calculatedKPI = 'Üretim Planına Uygunluk';
            }
        }

        function calculateOEE() {
            const availability = parseFloat(document.getElementById('availability').value);
            const performance = parseFloat(document.getElementById('performance').value);
            const quality = parseFloat(document.getElementById('quality').value);
            
            if (availability && performance && quality) {
                const oee = (availability * performance * quality) / 10000;
                showResult(oee.toFixed(1) + '%', getOEEInterpretation(oee));
                
                window.calculatedValue = oee.toFixed(1);
                window.calculatedKPI = 'OEE (Toplam Ekipman Etkinliği)';
            }
        }

        function calculateUnitCost() {
            const totalCost = parseFloat(document.getElementById('totalCost').value);
            const units = parseFloat(document.getElementById('producedUnits').value);
            
            if (totalCost && units && units > 0) {
                const unitCost = totalCost / units;
                showResult(unitCost.toFixed(2) + ' ₺', getCostInterpretation(unitCost));
                
                window.calculatedValue = unitCost.toFixed(2);
                window.calculatedKPI = 'Birim Başına Üretim Maliyeti';
            }
        }

        function calculateCAC() {
            const marketing = parseFloat(document.getElementById('marketingCost').value);
            const sales = parseFloat(document.getElementById('salesCost').value);
            const customers = parseFloat(document.getElementById('newCustomers').value);
            
            if (marketing && sales && customers && customers > 0) {
                const cac = (marketing + sales) / customers;
                showResult(cac.toFixed(2) + ' ₺', getCACInterpretation(cac));
                
                window.calculatedValue = cac.toFixed(2);
                window.calculatedKPI = 'Müşteri Edinme Maliyeti (CAC)';
            }
        }

        function calculateChurnRate() {
            const start = parseFloat(document.getElementById('startCustomers').value);
            const lost = parseFloat(document.getElementById('lostCustomers').value);
            
            if (start && lost !== undefined && start > 0) {
                const churn = (lost / start) * 100;
                showResult(churn.toFixed(1) + '%', getChurnInterpretation(churn));
                
                window.calculatedValue = churn.toFixed(1);
                window.calculatedKPI = 'Müşteri Kayıp Oranı';
            }
        }

        function calculateTurnoverRate() {
            const left = parseFloat(document.getElementById('leftEmployees').value);
            const avg = parseFloat(document.getElementById('avgEmployees').value);
            
            if (left !== undefined && avg && avg > 0) {
                const turnover = (left / avg) * 100;
                showResult(turnover.toFixed(1) + '%', getTurnoverInterpretation(turnover));
                
                window.calculatedValue = turnover.toFixed(1);
                window.calculatedKPI = 'Çalışan Devir Hızı';
            }
        }

        function calculateTrainingROI() {
            const cost = parseFloat(document.getElementById('trainingCost').value);
            const benefit = parseFloat(document.getElementById('trainingBenefit').value);
            
            if (cost && benefit && cost > 0) {
                const roi = (benefit - cost) / cost;
                showResult(roi.toFixed(1) + 'x', getROIInterpretation(roi));
                
                window.calculatedValue = roi.toFixed(1);
                window.calculatedKPI = 'Eğitim ROI';
            }
        }

        function calculateInventoryTurnover() {
            const cogs = parseFloat(document.getElementById('cogs').value);
            const avgInventory = parseFloat(document.getElementById('avgInventory').value);
            
            if (cogs && avgInventory && avgInventory > 0) {
                const turnover = cogs / avgInventory;
                showResult(turnover.toFixed(1) + ' kez/yıl', getInventoryTurnoverInterpretation(turnover));
                
                window.calculatedValue = turnover.toFixed(1);
                window.calculatedKPI = 'Stok Devir Hızı';
            }
        }

        function calculateStockAccuracy() {
            const system = parseFloat(document.getElementById('systemStock').value);
            const physical = parseFloat(document.getElementById('physicalStock').value);
            
            if (system && physical !== undefined && system > 0) {
                const accuracy = Math.max(0, (1 - Math.abs(system - physical) / system) * 100);
                showResult(accuracy.toFixed(1) + '%', getStockAccuracyInterpretation(accuracy));
                
                window.calculatedValue = accuracy.toFixed(1);
                window.calculatedKPI = 'Stok Doğruluk Oranı';
            }
        }

        function calculateFulfillmentTime() {
            const orders = parseFloat(document.getElementById('totalOrders').value);
            const time = parseFloat(document.getElementById('totalFulfillmentTime').value);
            
            if (orders && time && orders > 0) {
                const avgTime = time / orders;
                showResult(avgTime.toFixed(1) + ' gün', getFulfillmentInterpretation(avgTime));
                
                window.calculatedValue = avgTime.toFixed(1);
                window.calculatedKPI = 'Sipariş Karşılama Süresi';
            }
        }

        function calculateShipmentAccuracy() {
            const total = parseFloat(document.getElementById('totalShipments').value);
            const correct = parseFloat(document.getElementById('correctShipments').value);
            
            if (total && correct !== undefined && total > 0) {
                const accuracy = (correct / total) * 100;
                showResult(accuracy.toFixed(1) + '%', getShipmentAccuracyInterpretation(accuracy));
                
                window.calculatedValue = accuracy.toFixed(1);
                window.calculatedKPI = 'Sevkiyat Doğruluk Oranı';
            }
        }

        function showResult(value, interpretation) {
            document.getElementById('result').style.display = 'block';
            document.getElementById('resultValue').textContent = value;
            document.getElementById('resultText').textContent = interpretation;
            document.getElementById('selectBtn').style.display = 'block';
        }

        function selectCalculatedValue() {
            if (window.calculatedValue && window.calculatedKPI) {
                updateKPIValue(window.calculatedKPI, window.calculatedValue);
                closeModal();
                
                // Başarı mesajı göster
                alert(`✅ ${window.calculatedKPI} değeri ${window.calculatedValue}% olarak dashboard'a uygulandı!`);
            }
        }

        function updateKPIValue(kpiName, newValue) {
            // KPI değerini güncelle ve dashboard'u yenile
            Object.keys(kpiData).forEach(category => {
                const kpi = kpiData[category].find(k => k.name === kpiName);
                if (kpi) {
                    kpi.value = parseFloat(newValue);
                }
            });
            updateDashboard();
        }

        // Yorum fonksiyonları
        function getAccuracyInterpretation(value) {
            if (value >= 90) return "Mükemmel! Tahmin doğruluğunuz çok yüksek.";
            if (value >= 80) return "İyi seviyede. Küçük iyileştirmeler yapılabilir.";
            if (value >= 70) return "Orta seviye. Tahmin modelinizi gözden geçirin.";
            return "Düşük seviye. Tahmin sürecinizi yeniden değerlendirin.";
        }

        function getComplianceInterpretation(value) {
            if (value >= 95) return "Mükemmel planlama uyumu!";
            if (value >= 85) return "İyi seviyede planlama başarısı.";
            if (value >= 75) return "Orta seviye. Planlama süreçlerini iyileştirin.";
            return "Düşük uyum. Planlama ve üretim süreçlerini gözden geçirin.";
        }

        function getOEEInterpretation(value) {
            if (value >= 85) return "Dünya standartlarında ekipman verimliliği!";
            if (value >= 75) return "İyi seviyede verimlilik.";
            if (value >= 65) return "Orta seviye. İyileştirme potansiyeli var.";
            return "Düşük verimlilik. Acil iyileştirme gerekli.";
        }

        function getCostInterpretation(value) {
            if (value <= 40) return "Mükemmel maliyet kontrolü!";
            if (value <= 50) return "İyi maliyet seviyesi.";
            if (value <= 60) return "Orta seviye. Maliyet optimizasyonu yapın.";
            return "Yüksek maliyet. Acil maliyet azaltma gerekli.";
        }

        function getCACInterpretation(value) {
            if (value <= 100) return "Mükemmel müşteri edinme verimliliği!";
            if (value <= 150) return "İyi seviyede CAC.";
            if (value <= 200) return "Orta seviye. Pazarlama stratejinizi optimize edin.";
            return "Yüksek CAC. Pazarlama kanallarınızı gözden geçirin.";
        }

        function getChurnInterpretation(value) {
            if (value <= 5) return "Mükemmel müşteri elde tutma!";
            if (value <= 10) return "İyi seviyede müşteri sadakati.";
            if (value <= 15) return "Orta seviye. Müşteri deneyimini iyileştirin.";
            return "Yüksek kayıp oranı. Acil müşteri memnuniyet analizi gerekli.";
        }

        function getTurnoverInterpretation(value) {
            if (value <= 10) return "Mükemmel çalışan elde tutma!";
            if (value <= 15) return "İyi seviyede devir hızı.";
            if (value <= 20) return "Orta seviye. İK politikalarını gözden geçirin.";
            return "Yüksek devir hızı. Çalışan memnuniyeti analizi gerekli.";
        }

        function getROIInterpretation(value) {
            if (value >= 3) return "Mükemmel eğitim yatırım getirisi!";
            if (value >= 2) return "İyi seviyede ROI.";
            if (value >= 1) return "Orta seviye. Eğitim programlarını optimize edin.";
            return "Düşük ROI. Eğitim stratejinizi yeniden değerlendirin.";
        }

        function getInventoryTurnoverInterpretation(value) {
            if (value >= 8) return "Mükemmel stok yönetimi!";
            if (value >= 6) return "İyi seviyede stok devir hızı.";
            if (value >= 4) return "Orta seviye. Stok optimizasyonu yapın.";
            return "Düşük devir hızı. Stok stratejinizi gözden geçirin.";
        }

        function getStockAccuracyInterpretation(value) {
            if (value >= 98) return "Mükemmel stok doğruluğu!";
            if (value >= 95) return "İyi seviyede doğruluk.";
            if (value >= 90) return "Orta seviye. Depo süreçlerini iyileştirin.";
            return "Düşük doğruluk. Stok takip sistemini gözden geçirin.";
        }

        function getFulfillmentInterpretation(value) {
            if (value <= 2) return "Mükemmel teslimat hızı!";
            if (value <= 3) return "İyi seviyede karşılama süresi.";
            if (value <= 5) return "Orta seviye. Lojistik süreçlerini optimize edin.";
            return "Yavaş teslimat. Acil süreç iyileştirmesi gerekli.";
        }

        function getShipmentAccuracyInterpretation(value) {
            if (value >= 99) return "Mükemmel sevkiyat doğruluğu!";
            if (value >= 97) return "İyi seviyede doğruluk.";
            if (value >= 95) return "Orta seviye. Kalite kontrol süreçlerini iyileştirin.";
            return "Düşük doğruluk. Sevkiyat süreçlerini gözden geçirin.";
        }

        // Modal dışına tıklandığında kapat
        window.onclick = function(event) {
            const modal = document.getElementById('kpiModal');
            if (event.target === modal) {
                closeModal();
            }
        }

        // Gerçek zamanlı veri simülasyonu
        setInterval(() => {
            // Her 30 saniyede bir verileri güncelle
            Object.keys(kpiData).forEach(category => {
                kpiData[category].forEach(kpi => {
                    // Küçük rastgele değişiklikler yap
                    const change = (Math.random() - 0.5) * 2;
                    kpi.value = Math.max(0, kpi.value + change);
                    kpi.value = Math.round(kpi.value * 10) / 10;
                });
            });
            
            // Dashboard'u güncelle
            updateDashboard();
        }, 30000);
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9841d840e50b93a0',t:'MTc1ODcxMjMyNS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
