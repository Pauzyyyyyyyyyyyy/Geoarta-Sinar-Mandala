<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web GIS - Pengalaman Kerja PT Geoarta Sinar Mandala</title>
    
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            display: flex;
            flex-direction: column;
            height: 100vh;
        }
        header {
            background: linear-gradient(135deg, #1a365d 0%, #2b6cb0 100%);
            color: white;
            padding: 15px 25px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.3);
            z-index: 1000;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
        }
        header .title h1 {
            font-size: 1.3rem;
            margin-bottom: 2px;
        }
        header .title p {
            font-size: 0.8rem;
            color: #cbd5e0;
        }
        header .stats {
            background: rgba(255,255,255,0.15);
            padding: 8px 15px;
            border-radius: 20px;
            font-size: 0.8rem;
            backdrop-filter: blur(5px);
        }
        #map {
            flex: 1;
            width: 100%;
            z-index: 1;
        }
        
        /* Popup Styling */
        .leaflet-popup-content-wrapper {
            border-radius: 12px;
            padding: 0;
            box-shadow: 0 4px 20px rgba(0,0,0,0.2);
            overflow: hidden;
        }
        .leaflet-popup-content {
            margin: 0;
            padding: 0;
        }
        .popup-card {
            width: 300px;
            max-width: 300px;
        }
        .popup-header {
            background: linear-gradient(135deg, #1a365d, #2b6cb0);
            color: white;
            padding: 12px 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .popup-header .badge-year {
            background: rgba(255,255,255,0.2);
            padding: 2px 12px;
            border-radius: 12px;
            font-size: 0.7rem;
            font-weight: bold;
        }
        .popup-body {
            padding: 12px 15px 15px;
        }
        .popup-body h3 {
            color: #2b6cb0;
            font-size: 1rem;
            margin-bottom: 6px;
        }
        .popup-body .popup-info {
            font-size: 0.82rem;
            line-height: 1.6;
            color: #4a5568;
        }
        .popup-body .popup-info strong {
            color: #2d3748;
        }
        .popup-img-container {
            width: 100%;
            height: 170px;
            background: #edf2f7;
            overflow: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
            border-top: 1px solid #e2e8f0;
            position: relative;
        }
        .popup-img-container img {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transition: transform 0.3s ease;
        }
        .popup-img-container img:hover {
            transform: scale(1.05);
        }
        .popup-img-container .img-placeholder {
            font-size: 0.75rem;
            color: #718096;
            text-align: center;
            padding: 15px;
        }
        .popup-img-container .img-placeholder .icon {
            font-size: 2rem;
            display: block;
            margin-bottom: 5px;
        }
        .popup-img-container .loading-spinner {
            width: 40px;
            height: 40px;
            border: 4px solid #e2e8f0;
            border-top: 4px solid #2b6cb0;
            border-radius: 50%;
            animation: spin 0.8s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Custom SVG Pin Marker */
        .custom-pin-icon {
            background: transparent;
            border: none;
        }
        .pin-svg {
            filter: drop-shadow(0px 3px 6px rgba(0,0,0,0.4));
            transition: transform 0.2s ease, filter 0.2s ease;
            cursor: pointer;
        }
        .pin-svg:hover {
            transform: scale(1.2);
            filter: drop-shadow(0px 5px 10px rgba(0,0,0,0.5));
        }

        /* Legend */
        .legend {
            background: rgba(255,255,255,0.95);
            padding: 14px 18px;
            line-height: 22px;
            color: #333;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.15);
            font-size: 0.8rem;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255,255,255,0.2);
            min-width: 180px;
        }
        .legend h4 {
            margin: 0 0 10px 0;
            font-size: 0.85rem;
            color: #1a365d;
            border-bottom: 2px solid #e2e8f0;
            padding-bottom: 6px;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .legend-item {
            display: flex;
            align-items: center;
            margin-bottom: 5px;
            padding: 2px 4px;
            border-radius: 4px;
            transition: background 0.2s;
        }
        .legend-item:hover {
            background: #f7fafc;
        }
        .legend-item:last-child {
            margin-bottom: 0;
        }
        .legend-pin-icon {
            width: 18px;
            height: 24px;
            margin-right: 10px;
            flex-shrink: 0;
        }
        .legend-color-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            margin-right: 8px;
            border: 1px solid rgba(0,0,0,0.1);
        }

        /* Responsive */
        @media (max-width: 600px) {
            header .title h1 { font-size: 1rem; }
            header .stats { font-size: 0.7rem; padding: 4px 10px; }
            .popup-card { width: 250px; }
            .popup-img-container { height: 140px; }
            .legend { padding: 10px 12px; font-size: 0.7rem; min-width: 140px; }
        }
    </style>
</head>
<body>

    <header>
        <div class="title">
            <h1>🗺️ PT Geoarta Sinar Mandala</h1>
            <p>Peta Persebaran Pengalaman Pekerjaan (2020 - 2025)</p>
        </div>
        <div class="stats" id="statsDisplay">
            📍 <span id="totalPoints">0</span> titik pengalaman
        </div>
    </header>

    <div id="map"></div>

    <!-- Leaflet JS -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

    <script>
        // ============================================================
        // 1. KONFIGURASI GOOGLE DRIVE
        // ============================================================
        const FOLDER_ID = '1LNFFkxzlmtvb52aM7-VAWbtIwhigg0Qa';
        const FOLDER_URL = `https://drive.google.com/drive/folders/${FOLDER_ID}`;

        // ============================================================
        // 2. MAPPING FILE ID GOOGLE DRIVE
        //    📌 CARA MENDAPATKAN FILE ID:
        //    Buka file di Google Drive → Klik kanan → "Dapatkan link"
        //    URL: https://drive.google.com/file/d/[FILE_ID]/view
        //    Salin [FILE_ID] dan isi di bawah ini.
        // ============================================================
        const googleDriveFiles = {
            // ===== TAHUN 2025 =====
            "pemotretan udara menggunakan UAV di Kabupaten Pesisir Barat, Lampung 2025.png": {
                fileId: "", // Isi dengan File ID
                tahun: "2025"
            },
            "pengukuran dan pemetaan di kawasan Sungai Progo, Opak, dan Serang (POS) 2025.png": {
                fileId: "", // Isi dengan File ID
                tahun: "2025"
            },
            "pengukuran pekerjaan detail desain Daerah Irigasi Kalibawang 2025.png": {
                fileId: "", // Isi dengan File ID
                tahun: "2025"
            },
            "pengukuran stakeout tapak batas di Daerah Irigasi Tingal 2025.png": {
                fileId: "", // Isi dengan File ID
                tahun: "2025"
            },
            "Foto Tegak di Kabupaten Rembang 2025.png": {
                fileId: "", // Isi dengan File ID
                tahun: "2025"
            },
            "pembuatan Peta Foto Tegak di Kabupaten Malang 2025.png": {
                fileId: "", // Isi dengan File ID
                tahun: "2025"
            }
        };

        // ============================================================
        // 3. FUNGSI UTILITY
        // ============================================================
        function getGoogleDriveImageUrl(fileId) {
            if (!fileId) return null;
            return `https://drive.google.com/uc?export=view&id=${fileId}`;
        }

        function getImageUrl(namaFile) {
            if (!namaFile || namaFile === "") return null;
            
            // Cek mapping
            const fileData = googleDriveFiles[namaFile];
            if (fileData && fileData.fileId) {
                return getGoogleDriveImageUrl(fileData.fileId);
            }
            
            // Fallback: cari berdasarkan kemiripan nama
            const searchKey = namaFile.toLowerCase().replace(/\s+/g, ' ');
            for (const [key, value] of Object.entries(googleDriveFiles)) {
                const keyClean = key.toLowerCase().replace(/\s+/g, ' ');
                if (keyClean.includes(searchKey) || searchKey.includes(keyClean)) {
                    if (value.fileId) {
                        return getGoogleDriveImageUrl(value.fileId);
                    }
                }
            }
            
            return null;
        }

        function checkImageExists(url) {
            return new Promise((resolve) => {
                if (!url) { resolve(false); return; }
                const img = new Image();
                img.onload = () => resolve(true);
                img.onerror = () => resolve(false);
                img.src = url;
            });
        }

        // ============================================================
        // 4. KATEGORI & WARNA
        // ============================================================
        const colorMap = {
            "Pengukuran Topografi": "#D2B48C",
            "Pengukuran Fotogrametri": "#38BDF8",
            "Pengukuran Batimetri": "#1E3A8A",
            "Pengukuran BM / Kontrol Geodetik": "#8B4513",
            "Pengolahan Data Spasial": "#4ADE80"
        };

        function getCategory(pekerjaan) {
            const p = pekerjaan.toLowerCase();
            if (p.includes('topografi') || p.includes('detail desain')) return "Pengukuran Topografi";
            if (p.includes('foto udara') || p.includes('lidar') || p.includes('video') || 
                p.includes('pemotretan') || p.includes('peta foto') || p.includes('foto tegak') ||
                p.includes('uav')) return "Pengukuran Fotogrametri";
            if (p.includes('hidrografi') || p.includes('batimetri') || p.includes('usv')) return "Pengukuran Batimetri";
            if (p.includes('stake out') || p.includes('stakeout') || p.includes('soil test')) return "Pengukuran BM / Kontrol Geodetik";
            if (p.includes('lod 1') || p.includes('dem') || p.includes('pos')) return "Pengolahan Data Spasial";
            return "Pengukuran Topografi";
        }

        // ============================================================
        // 5. BUAT PIN MARKER
        // ============================================================
        function createPinIcon(color) {
            const svgHtml = `
                <svg class="pin-svg" width="28" height="40" viewBox="0 0 24 36" xmlns="http://www.w3.org/2000/svg">
                    <defs>
                        <filter id="shadow_${color.replace('#', '')}" x="-20%" y="-20%" width="140%" height="140%">
                            <feDropShadow dx="0" dy="3" stdDeviation="3" flood-opacity="0.4"/>
                        </filter>
                    </defs>
                    <path fill="${color}" stroke="#FFFFFF" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" 
                          d="M12 0C5.37 0 0 5.37 0 12c0 9 12 24 12 24s12-15 12-24c0-6.63-5.37-12-12-12z"
                          filter="url(#shadow_${color.replace('#', '')})"/>
                    <circle cx="12" cy="11" r="4" fill="#FFFFFF" stroke="#${color.replace('#', '')}" stroke-width="1.5"/>
                </svg>
            `;
            return L.divIcon({
                className: 'custom-pin-icon',
                html: svgHtml,
                iconSize: [28, 40],
                iconAnchor: [14, 40],
                popupAnchor: [0, -36]
            });
        }

        // ============================================================
        // 6. DATA PENGALAMAN KERJA (LENGKAP 2020-2025)
        // ============================================================
        const dataPengalaman = [
            // --- 2020 ---
            { tahun: 2020, pekerjaan: "Topografi", lokasi: "Wates, Kulon Progo", instansi: "Rekan", lat: -7.861, lng: 110.158, foto: "" },
            { tahun: 2020, pekerjaan: "Topografi", lokasi: "Godean, Sleman", instansi: "Rekan", lat: -7.768, lng: 110.294, foto: "" },
            { tahun: 2020, pekerjaan: "Topografi", lokasi: "Bengawan Solo, Sukoharjo", instansi: "Tekling UPN", lat: -7.683, lng: 110.830, foto: "" },
            { tahun: 2020, pekerjaan: "Foto Udara", lokasi: "Banyu Urip, Purworejo", instansi: "Rekan", lat: -7.794, lng: 109.972, foto: "" },
            { tahun: 2020, pekerjaan: "Video", lokasi: "Nusa Tenggara Barat", instansi: "Dosen Bandung", lat: -8.652, lng: 117.361, foto: "" },

            // --- 2021 ---
            { tahun: 2021, pekerjaan: "Foto Udara", lokasi: "Tembalang, Semarang", instansi: "BEE HAVE Drone", lat: -7.051, lng: 110.439, foto: "" },
            { tahun: 2021, pekerjaan: "Foto Udara", lokasi: "Muara Teweh, Kalteng", instansi: "Greenline", lat: -0.957, lng: 114.896, foto: "" },
            { tahun: 2021, pekerjaan: "Foto Udara", lokasi: "Buhud, Kalteng", instansi: "Rekan", lat: -1.020, lng: 114.600, foto: "" },
            { tahun: 2021, pekerjaan: "Build USV", lokasi: "Balai Pantai, Bali", instansi: "PUPR Balai Pantai", lat: -8.683, lng: 115.215, foto: "" },
            { tahun: 2021, pekerjaan: "Foto Udara", lokasi: "Pongkor, Bogor", instansi: "ANTAM", lat: -6.662, lng: 106.568, foto: "" },
            { tahun: 2021, pekerjaan: "Topografi", lokasi: "Sungai Ciliwung, Jakarta", instansi: "Rekan", lat: -6.229, lng: 106.840, foto: "" },

            // --- 2022 ---
            { tahun: 2022, pekerjaan: "Foto Udara", lokasi: "Candi Borobudur", instansi: "UGM", lat: -7.607, lng: 110.203, foto: "" },
            { tahun: 2022, pekerjaan: "Foto Udara", lokasi: "Barong, Tongkok, Kutai Barat", instansi: "PT. ASG", lat: -0.226, lng: 115.700, foto: "" },
            { tahun: 2022, pekerjaan: "Foto Udara", lokasi: "Muara Lawa, Kutai Barat", instansi: "PT. ASG", lat: -0.428, lng: 115.824, foto: "" },
            { tahun: 2022, pekerjaan: "LiDAR", lokasi: "Muara Enim, Sumatera Selatan", instansi: "Rekan", lat: -3.655, lng: 103.774, foto: "" },
            { tahun: 2022, pekerjaan: "LiDAR", lokasi: "Lawe-lawe, Balikpapan", instansi: "PERTAMINA", lat: -1.312, lng: 116.790, foto: "" },
            { tahun: 2022, pekerjaan: "Foto Udara", lokasi: "Sungai Gajah Wong, Yogyakarta", instansi: "UCY", lat: -7.808, lng: 110.395, foto: "" },
            { tahun: 2022, pekerjaan: "LiDAR", lokasi: "Samarinda", instansi: "PT. Hexa Internasional", lat: -0.502, lng: 117.153, foto: "" },
            { tahun: 2022, pekerjaan: "Foto Udara", lokasi: "Pacitan, Jawa Timur", instansi: "ITENAS", lat: -8.205, lng: 111.092, foto: "" },
            { tahun: 2022, pekerjaan: "LiDAR", lokasi: "Pacitan, Jawa Timur", instansi: "ITENAS", lat: -8.190, lng: 111.100, foto: "" },
            { tahun: 2022, pekerjaan: "Foto Udara", lokasi: "Pulau Bintan, Tanjung Pinang", instansi: "SANDBOX", lat: 0.917, lng: 104.466, foto: "" },
            { tahun: 2022, pekerjaan: "LiDAR", lokasi: "Muara Enim, Sumatera Selatan", instansi: "Rekan", lat: -3.630, lng: 103.780, foto: "" },
            { tahun: 2022, pekerjaan: "Topografi", lokasi: "Kupang, Nusa Tenggara Timur", instansi: "ANUGERAH GLOBAL SUPERINTENDING", lat: -10.177, lng: 123.607, foto: "" },
            { tahun: 2022, pekerjaan: "Topografi", lokasi: "Kupang, Nusa Tenggara Timur", instansi: "PLTU Kupang", lat: -10.160, lng: 123.590, foto: "" },

            // --- 2023 ---
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Gorontalo Utara", instansi: "SANDBOX", lat: 0.888, lng: 122.880, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Buru, Maluku", instansi: "BPN Buru", lat: -3.328, lng: 127.100, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Maluku", instansi: "BPN Maluku", lat: -3.200, lng: 130.000, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Gunung Mas, Kalimantan Tengah", instansi: "BPN Gunung Mas", lat: -1.050, lng: 113.840, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Jambi", instansi: "PT KBB", lat: -1.610, lng: 103.613, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Pamekasan, Jawa Timur", instansi: "BPN Pamekasan", lat: -7.161, lng: 113.482, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Minggir, Sleman", instansi: "BPN Sleman", lat: -7.731, lng: 110.252, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Cilacap, Jawa Tengah", instansi: "BPN Cilacap", lat: -7.718, lng: 109.015, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Gunung Kidul, Yogyakarta", instansi: "BPN Gunung Kidul", lat: -7.962, lng: 110.601, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Lampung Barat", instansi: "BPN Lampung Barat", lat: -5.150, lng: 104.190, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Gorontalo", instansi: "BPN Gorontalo", lat: 0.540, lng: 123.060, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Pringsewu, Lampung", instansi: "BPN Lampung", lat: -5.358, lng: 104.974, foto: "" },
            { tahun: 2023, pekerjaan: "Foto Udara", lokasi: "Gunung Mas, Kalimantan Tengah", instansi: "BPN Gunung Mas", lat: -1.070, lng: 113.850, foto: "" },

            // --- 2024 ---
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "Lampung Timur, Lampung", instansi: "BPN Lampung Timur", lat: -5.110, lng: 105.680, foto: "" },
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "Pesisir Barat, Lampung", instansi: "BPN Pesisir Barat", lat: -5.192, lng: 103.858, foto: "" },
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "Godean, Sleman", instansi: "BPN Sleman", lat: -7.770, lng: 110.290, foto: "" },
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "Gunung Kidul, Yogyakarta", instansi: "KJSB Shinta Aprilia Indarwati", lat: -7.970, lng: 110.610, foto: "" },
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "Sleman, Yogyakarta", instansi: "BPN Sleman", lat: -7.715, lng: 110.355, foto: "" },
            { tahun: 2024, pekerjaan: "Stake Out", lokasi: "DI Tingal, Temanggung", instansi: "BBWS-SO", lat: -7.310, lng: 110.170, foto: "" },
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "DI Tingal, Temanggung", instansi: "BBWS-SO", lat: -7.315, lng: 110.175, foto: "" },
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "Way Kanan, Lampung", instansi: "BPN Way Kanan", lat: -4.500, lng: 104.520, foto: "" },
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "Gunung Mas, Kalimantan Tengah", instansi: "BPN Kalimantan Tengah", lat: -1.030, lng: 113.830, foto: "" },
            { tahun: 2024, pekerjaan: "Foto Udara", lokasi: "Sempor, Kebumen", instansi: "BBWSSO", lat: -7.568, lng: 109.581, foto: "" },
            { tahun: 2024, pekerjaan: "Hidrografi", lokasi: "Karawang", instansi: "Karta Bhumi Nusantara", lat: -6.305, lng: 107.301, foto: "" },
            { tahun: 2024, pekerjaan: "Topografi", lokasi: "Gumuk Pasir, Parangtritis", instansi: "Rekan", lat: -8.020, lng: 110.323, foto: "" },
            { tahun: 2024, pekerjaan: "LOD 1", lokasi: "Karanganyar, Jawa Tengah", instansi: "BPN Karanganyar", lat: -7.597, lng: 110.951, foto: "" },

            // --- 2025 ---
            { tahun: 2025, pekerjaan: "Hidrografi", lokasi: "Banjarmasin", instansi: "Obsluzivo Intiloc Indonesia", lat: -3.319, lng: 114.590, foto: "" },
            { tahun: 2025, pekerjaan: "Soil Test", lokasi: "Yogyakarta", instansi: "Rekan", lat: -7.795, lng: 110.369, foto: "" },
            { tahun: 2025, pekerjaan: "Foto Udara", lokasi: "Batam, Kepri", instansi: "PT Geoservices", lat: 1.130, lng: 104.053, foto: "" },
            { tahun: 2025, pekerjaan: "Foto Udara", lokasi: "Padang", instansi: "PT Geoservices", lat: -0.947, lng: 100.417, foto: "" },
            { tahun: 2025, pekerjaan: "Foto Udara", lokasi: "Jambi", instansi: "PT Geoservices", lat: -1.600, lng: 103.600, foto: "" },
            { tahun: 2025, pekerjaan: "Batimetri", lokasi: "Batam", instansi: "Rekan", lat: 1.110, lng: 104.030, foto: "" },
            { tahun: 2025, pekerjaan: "Pemotretan Udara UAV", lokasi: "Pesisir Barat, Lampung", instansi: "KJSB Shinta Aprilia Indarwati", lat: -5.210, lng: 103.850, foto: "pemotretan udara menggunakan UAV di Kabupaten Pesisir Barat, Lampung 2025.png" },
            { tahun: 2025, pekerjaan: "Foto Udara", lokasi: "Lampung Barat", instansi: "KJSB Shinta Aprilia Indarwati", lat: -5.140, lng: 104.180, foto: "" },
            { tahun: 2025, pekerjaan: "Batimetri", lokasi: "Kapuas", instansi: "PT PETA", lat: -2.766, lng: 114.385, foto: "" },
            { tahun: 2025, pekerjaan: "DEM / Pemetaan POS", lokasi: "Sungai Progo, Opak, dan Serang (Bantul, Sleman, Klaten)", instansi: "BBWS-SO", lat: -7.800, lng: 110.400, foto: "pengukuran dan pemetaan di kawasan Sungai Progo, Opak, dan Serang (POS) 2025.png" },
            { tahun: 2025, pekerjaan: "Topografi / Detail Desain", lokasi: "DI Kalibawang, Kulon Progo, DIY", instansi: "BBWS-SO", lat: -7.718, lng: 110.220, foto: "pengukuran pekerjaan detail desain Daerah Irigasi Kalibawang 2025.png" },
            { tahun: 2025, pekerjaan: "Stakeout Tapak Batas", lokasi: "DI Tingal, Temanggung, Jawa Tengah", instansi: "BBWS-SO", lat: -7.318, lng: 110.168, foto: "pengukuran stakeout tapak batas di Daerah Irigasi Tingal 2025.png" },
            { tahun: 2025, pekerjaan: "Foto Tegak", lokasi: "Kabupaten Rembang", instansi: "Rekan", lat: -6.708, lng: 111.341, foto: "Foto Tegak di Kabupaten Rembang 2025.png" },
            { tahun: 2025, pekerjaan: "Pembuatan Peta Foto Tegak", lokasi: "Kabupaten Malang", instansi: "KJSB Agus Purwanto", lat: -8.131, lng: 112.571, foto: "pembuatan Peta Foto Tegak di Kabupaten Malang 2025.png" }
        ];

        // ============================================================
        // 7. INISIALISASI PETA
        // ============================================================
        const map = L.map('map', {
            center: [-2.548926, 118.014863],
            zoom: 5,
            zoomControl: true,
            fadeAnimation: true,
            attributionControl: true
        });

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors | PT Geoarta Sinar Mandala'
        }).addTo(map);

        // ============================================================
        // 8. BUAT MARKER DENGAN POPUP
        // ============================================================
        let totalData = 0;

        dataPengalaman.forEach(async (item) => {
            totalData++;
            const kategori = getCategory(item.pekerjaan);
            const warna = colorMap[kategori] || "#718096";
            
            // Bangun HTML popup
            let imageHTML = '';
            
            if (item.foto && item.foto !== "") {
                const imageUrl = getImageUrl(item.foto);
                
                if (imageUrl) {
                    const exists = await checkImageExists(imageUrl);
                    if (exists) {
                        imageHTML = `
                            <div class="popup-img-container">
                                <img src="${imageUrl}" alt="${item.pekerjaan}" loading="lazy" 
                                     onerror="this.style.display='none'; this.nextElementSibling.style.display='flex';">
                                <div class="img-placeholder" style="display:none; position:absolute; inset:0; flex-direction:column;">
                                    <span class="icon">🖼️</span>
                                    <span>Gambar gagal dimuat</span>
                                </div>
                            </div>
                        `;
                    } else {
                        imageHTML = `
                            <div class="popup-img-container">
                                <div class="img-placeholder">
                                    <span class="icon">📁</span>
                                    <span>File ID belum diisi</span>
                                    <br><small style="font-size:9px; color:#a0aec0;">${item.foto.substring(0, 30)}...</small>
                                </div>
                            </div>
                        `;
                    }
                } else {
                    imageHTML = `
                        <div class="popup-img-container">
                            <div class="img-placeholder">
                                <span class="icon">⚠️</span>
                                <span>File tidak ditemukan</span>
                                <br><small style="font-size:9px; color:#a0aec0;">${item.foto.substring(0, 30)}...</small>
                            </div>
                        </div>
                    `;
                }
            } else {
                imageHTML = `
                    <div class="popup-img-container">
                        <div class="img-placeholder">
                            <span class="icon">🖼️</span>
                            <span>Dokumentasi belum tersedia</span>
                        </div>
                    </div>
                `;
            }

            const popupContent = `
                <div class="popup-card">
                    <div class="popup-header">
                        <strong>${item.pekerjaan}</strong>
                        <span class="badge-year">${item.tahun}</span>
                    </div>
                    <div class="popup-body">
                        <div class="popup-info">
                            <strong>📍 Lokasi:</strong> ${item.lokasi}<br>
                            <strong>🏢 Instansi:</strong> ${item.instansi}<br>
                            <strong>📂 Kategori:</strong> ${kategori}
                        </div>
                    </div>
                    ${imageHTML}
                </div>
            `;

            const marker = L.marker([item.lat, item.lng], {
                icon: createPinIcon(warna),
                title: `${item.pekerjaan} - ${item.lokasi}`
            })
            .addTo(map)
            .bindPopup(popupContent, {
                maxWidth: 320,
                className
