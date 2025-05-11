---
layout: page
title: QR Code Generator & Scanner
permalink: /qr-generator/
---

## QR Code Generator

<textarea id="qr-input" rows="4" style="width:100%" placeholder="Enter text or URL here..."></textarea>

<label>
  Size:
  <input type="number" id="qr-size" value="256" min="100" max="1024" /> px
</label>
<br>
<label>
  Foreground color:
  <input type="color" id="qr-color" value="#000000" />
</label>
<br>
<label>
  Background color:
  <input type="color" id="qr-bg" value="#ffffff" />
</label>
<br><br>

<button onclick="generateQR()">Generate QR</button>
<button onclick="downloadQR()">Download PNG</button>
<button onclick="clearQR()">Clear</button>

<div id="qr-container" style="margin-top:20px;"></div>

<hr>

## QR Code Scanner

<button onclick="startScanner()">Start Scanner</button>
<button onclick="stopScanner()">Stop Scanner</button>
<p><strong>Last Result:</strong> <span id="scan-result">-</span></p>

<div id="reader" style="width:100%; max-width:400px;"></div>

<h3>Scan History</h3>
<ul id="scan-history"></ul>
<button onclick="clearHistory()">Clear History</button>
<button onclick="exportHistory()">Export History</button>

<!-- QRCode.js -->
<script src="https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js"></script>

<!-- Html5 QR Code Scanner -->
<script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>

<script>
  let qr;
  let html5QrCode;
  let historyList = JSON.parse(localStorage.getItem("qrScanHistory")) || [];

  function generateQR() {
    const container = document.getElementById("qr-container");
    const text = document.getElementById("qr-input").value;
    const size = parseInt(document.getElementById("qr-size").value);
    const color = document.getElementById("qr-color").value;
    const bg = document.getElementById("qr-bg").value;

    if (!text.trim()) {
      alert("Please enter some text.");
      return;
    }

    container.innerHTML = "";
    qr = new QRCode(container, {
      text: text,
      width: size,
      height: size,
      colorDark: color,
      colorLight: bg,
      correctLevel: QRCode.CorrectLevel.H,
    });
  }

  function downloadQR() {
    if (!qr) {
      alert("Generate a QR code first.");
      return;
    }
    const canvas = document.querySelector("#qr-container canvas");
    if (canvas) {
      const link = document.createElement("a");
      link.download = "qr-code.png";
      link.href = canvas.toDataURL("image/png");
      link.click();
    } else {
      alert("Failed to find QR canvas.");
    }
  }

  function clearQR() {
    document.getElementById("qr-container").innerHTML = "";
    qr = null;
  }

  function startScanner() {
    const resultElement = document.getElementById("scan-result");
    html5QrCode = new Html5Qrcode("reader");

    Html5Qrcode.getCameras().then(devices => {
      // Otomatis pilih kamera belakang jika ada
      const backCam = devices.find(device =>
        /back|rear|environment/i.test(device.label)
      ) || devices[0];

      if (backCam) {
        html5QrCode.start(
          backCam.id,
          { fps: 10, qrbox: 250 },
          qrCodeMessage => {
            resultElement.textContent = qrCodeMessage;
            addToHistory(qrCodeMessage);
            stopScanner();
          },
          errorMessage => {
            console.warn("QR scan error:", errorMessage);
          }
        );
      } else {
        alert("No camera available.");
      }
    }).catch(err => {
      alert("Camera access denied or not supported.");
    });
  }

  function stopScanner() {
    if (html5QrCode) {
      html5QrCode.stop().then(() => {
        html5QrCode.clear();
        html5QrCode = null;
      }).catch(err => console.error("Stop scanner error:", err));
    }
  }

  function addToHistory(text) {
    if (!text || historyList.includes(text)) return;
    historyList.push(text);
    localStorage.setItem("qrScanHistory", JSON.stringify(historyList));
    updateHistoryUI();
  }

  function updateHistoryUI() {
    const ul = document.getElementById("scan-history");
    ul.innerHTML = "";
    historyList.forEach(entry => {
      const li = document.createElement("li");
      li.textContent = entry;
      ul.appendChild(li);
    });
  }

  function clearHistory() {
    if (confirm("Are you sure you want to clear history?")) {
      historyList = [];
      localStorage.removeItem("qrScanHistory");
      updateHistoryUI();
    }
  }

  function exportHistory() {
    if (!historyList.length) {
      alert("No history to export.");
      return;
    }
    const blob = new Blob([historyList.join("\n")], { type: "text/plain" });
    const link = document.createElement("a");
    link.href = URL.createObjectURL(blob);
    link.download = "qr-scan-history.txt";
    link.click();
  }

  // On load
  updateHistoryUI();
</script>
