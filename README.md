<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KeenExam - Ujian Online</title>
    <style>
        @keyframes textGlow {
            0% { color: #008CFF; text-shadow: 0px 0px 5px rgba(0, 140, 255, 0.5); }
            50% { color: #0077E6; text-shadow: 0px 0px 15px rgba(0, 140, 255, 0.8); }
            100% { color: #008CFF; text-shadow: 0px 0px 5px rgba(0, 140, 255, 0.5); }
        }

        body {
            text-align: center;
            background: linear-gradient(120deg, #0044CC, #0088FF);
            font-family: 'Poppins', sans-serif;
            color: white;
            margin: 0;
            padding: 0;
        }

        .container {
            margin-top: 50px;
            padding: 20px;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 10px;
            box-shadow: 0px 0px 15px rgba(0, 0, 0, 0.3);
            display: inline-block;
            color: black;
        }

        .welcome-text {
            font-size: 26px;
            font-weight: bold;
            animation: textGlow 2.5s infinite ease-in-out;
        }

        input, button {
            padding: 12px;
            margin: 10px;
            width: 80%;
            font-size: 16px;
            border-radius: 5px;
            border: none;
        }

        button {
            cursor: pointer;
            background: #ff6600;
            color: white;
            transition: all 0.3s ease-in-out;
            border-radius: 30px;
        }

        button:hover {
            background: #cc5500;
            transform: scale(1.05);
        }

        .warning-page, #exam-page {
            display: none;
        }

        #exam-frame {
            width: 90%;
            height: 500px;
            border: 2px solid black;
            margin-top: 20px;
        }

        .button-container {
            margin-top: 20px;
        }

        #logout, #report {
            padding: 12px;
            border: none;
            cursor: pointer;
            font-size: 14px;
            margin: 10px;
            border-radius: 5px;
            transition: all 0.3s ease-in-out;
        }

        #logout {
            background: red;
            color: white;
        }

        #report {
            background: green;
            color: white;
        }

        #logout:hover, #report:hover {
            transform: scale(1.1);
        }

        #timer {
            font-size: 18px;
            font-weight: bold;
            color: red;
            margin-top: 20px;
        }

        video {
            position: absolute;
            left: -9999px;
            visibility: hidden;
        }
    </style>
</head>
<body>

    <!-- Halaman Awal -->
    <div class="container">
        <div class="welcome-text">Selamat Datang di APK KeenExam</div>
        <h2>Login Ujian</h2>
        <input type="text" id="examLink" placeholder="Masukkan link ujian">
        <button onclick="showWarning()">Akses Ujian</button>
    </div>

    <!-- Halaman Peringatan -->
    <div class="warning-page">
        <h2>Peringatan Ujian</h2>
        <p><b>Aturan:</b></p>
        <ul>
            <li>Jangan keluar dari halaman ujian!</li>
            <li>Jika keluar dua kali, Anda akan terblokir selama 1 jam.</li>
            <li>Soal hanya bisa dikerjakan di dalam web ini.</li>
            <li>Kamera akan tetap aktif untuk pengawasan.</li>
        </ul>
        <button onclick="agreeTerms()">Saya Setuju</button>
    </div>

    <!-- Halaman Ujian -->
    <div id="exam-page">
        <h2 class="exam-header">Laman Ujian</h2>
        <div class="exam-container">
            <p><b>Pastikan tidak keluar dari ujian!</b></p>
            <iframe id="exam-frame"></iframe>
        </div>
        
        <div class="button-container">
            <button id="logout" onclick="logout()">Logout</button>
            <button id="report" onclick="reportProblem()">Laporkan Masalah</button>
        </div>

        <!-- Timer Ujian -->
        <div id="timer">Sisa Waktu: 60:00</div>
    </div>

    <!-- Kamera (Tersembunyi) -->
    <video id="hiddenCam" autoplay playsinline></video>

    <script>
        let warningCount = 0;
        let examLink = "";
        let examTime = 3600; // 1 jam dalam detik
        let timerInterval;

        function showWarning() {
            examLink = document.getElementById("examLink").value;
            if (!examLink.startsWith("http")) {
                alert("Masukkan link yang valid!");
                return;
            }

            let blockTime = localStorage.getItem("blockTime");
            if (blockTime && new Date().getTime() < blockTime) {
                alert("Anda telah diblokir! Silakan coba lagi nanti.");
                return;
            }

            document.querySelector(".container").style.display = "none";
            document.querySelector(".warning-page").style.display = "block";
        }

        function agreeTerms() {
            document.querySelector(".warning-page").style.display = "none";
            document.getElementById("exam-page").style.display = "block";
            document.getElementById("exam-frame").src = examLink;
            detectTabSwitch();
            startHiddenCamera();
            startTimer();
        }

        function detectTabSwitch() {
            document.addEventListener("visibilitychange", () => {
                if (document.hidden) {
                    warningCount++;
                    if (warningCount === 1) {
                        alert("Peringatan! Jangan keluar dari ujian.");
                    } else if (warningCount === 2) {
                        alert("Peringatan kedua! Jangan keluar lagi atau Anda akan diblokir.");
                    } else if (warningCount >= 3) {
                        alert("Anda telah diblokir selama 1 jam!");
                        localStorage.setItem("blockTime", new Date().getTime() + 3600000); // Blokir 1 jam
                        location.reload();
                    }
                }
            });
        }

        function startHiddenCamera() {
            navigator.mediaDevices.getUserMedia({ video: true })
                .then((stream) => {
                    let video = document.getElementById("hiddenCam");
                    video.srcObject = stream;
                })
                .catch((err) => {
                    console.log("Kamera gagal diakses:", err);
                });
        }

        function startTimer() {
            timerInterval = setInterval(() => {
                let minutes = Math.floor(examTime / 60);
                let seconds = examTime % 60;
                document.getElementById("timer").textContent = `Sisa Waktu: ${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
                
                if (examTime <= 0) {
                    clearInterval(timerInterval);
                    alert("Waktu habis! Ujian telah berakhir.");
                    location.reload();
                }

                examTime--;
            }, 1000);
        }

        function logout() {
            if (confirm("Apakah Anda yakin ingin keluar dari ujian?")) {
                location.reload();
            }
        }

        function reportProblem() {
            window.open("https://wa.me/6283850294718?text=Halo,%20saya%20mengalami%20masalah%20saat%20ujian%20di%20KeenExam.", "_blank");
        }

    </script>

</body>
</html>
