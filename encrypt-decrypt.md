---
layout: page
title: Text Encrypt/Decrypt Tool
permalink: /encrypt-decrypt/
---

<!-- Font Awesome CDN untuk ikon -->
<link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css" rel="stylesheet">

# Text Encrypt / Decrypt Tool

Encrypt and decrypt text using various methods. All processing is done in your browser.

<label>
  <input type="checkbox" id="darkToggle" onchange="toggleDarkMode()"> Dark Mode
</label>

## Input Text (Max 1000 characters)

<textarea id="input" rows="5" maxlength="1000" style="width: 100%;" placeholder="Enter your text here..."></textarea>

## Choose Method

<select id="method">
  <option value="base64">Base64</option>
  <option value="caesar">Caesar Cipher</option>
  <option value="rot13">ROT13</option>
  <option value="binary">Binary</option>
  <option value="md5">MD5 (Encrypt Only)</option>
</select>

<div id="shift-container" style="display:none; margin-top:10px;">
  <label>Shift (Caesar): <input type="number" id="shift" value="3" min="1" max="25"/></label>
</div>

## Action

<button onclick="encrypt()">Encrypt</button>
<button onclick="decrypt()">Decrypt</button>
<button onclick="copyOutput()">Copy Output</button>
<button onclick="downloadOutput()">Download Output</button>

<div id="status"></div>

## Output

<pre id="output" style="background:#f0f0f0;padding:10px;"></pre>

<!-- CryptoJS for MD5 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>

<script>
  const methodSelect = document.getElementById('method');
  const shiftContainer = document.getElementById('shift-container');
  const status = document.getElementById('status');

  methodSelect.addEventListener('change', () => {
    shiftContainer.style.display = methodSelect.value === 'caesar' ? 'block' : 'none';
    clearStatus();
  });

  function encrypt() {
    const text = document.getElementById('input').value;
    const method = methodSelect.value;
    let result = '';

    if (!text) {
      showStatus("Please enter text to encrypt.", true);
      return;
    }

    switch (method) {
      case 'base64':
        result = btoa(text);
        break;
      case 'caesar':
        const shift = parseInt(document.getElementById('shift').value) || 0;
        result = caesarCipher(text, shift);
        break;
      case 'rot13':
        result = rot13(text);
        break;
      case 'binary':
        result = text.split('').map(c => c.charCodeAt(0).toString(2).padStart(8, '0')).join(' ');
        break;
      case 'md5':
        result = CryptoJS.MD5(text).toString();
        break;
    }

    document.getElementById('output').textContent = result;
    showStatus("Text encrypted successfully.", false);
  }

  function decrypt() {
    const text = document.getElementById('input').value;
    const method = methodSelect.value;
    let result = '';

    if (!text) {
      showStatus("Please enter text to decrypt.", true);
      return;
    }

    try {
      switch (method) {
        case 'base64':
          result = atob(text);
          break;
        case 'caesar':
          const shift = parseInt(document.getElementById('shift').value) || 0;
          result = caesarCipher(text, 26 - shift);
          break;
        case 'rot13':
          result = rot13(text);
          break;
        case 'binary':
          result = text.split(' ').map(b => String.fromCharCode(parseInt(b, 2))).join('');
          break;
        case 'md5':
          result = 'MD5 is a one-way hash and cannot be decrypted.';
          break;
      }
    } catch {
      showStatus("Decryption failed. Invalid input or format.", true);
      return;
    }

    document.getElementById('output').textContent = result;
    showStatus("Text decrypted successfully.", false);
  }

  function caesarCipher(str, shift) {
    return str.replace(/[a-z]/gi, c => {
      const base = c <= 'Z' ? 65 : 97;
      return String.fromCharCode((c.charCodeAt(0) - base + shift) % 26 + base);
    });
  }

  function rot13(str) {
    return str.replace(/[a-zA-Z]/g, c => {
      const base = c <= 'Z' ? 65 : 97;
      return String.fromCharCode((c.charCodeAt(0) - base + 13) % 26 + base);
    });
  }

  function copyOutput() {
    const text = document.getElementById('output').textContent;
    if (!text) {
      showStatus("Nothing to copy.", true);
      return;
    }
    navigator.clipboard.writeText(text).then(() => {
      showStatus("Output copied to clipboard.", false);
    });
  }

  function downloadOutput() {
    const text = document.getElementById('output').textContent;
    const method = methodSelect.value;
    if (!text) {
      showStatus("Nothing to download.", true);
      return;
    }

    const filename = `output-${method}.txt`;
    const blob = new Blob([text], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);

    const link = document.createElement('a');
    link.href = url;
    link.download = filename;
    link.click();

    URL.revokeObjectURL(url);
    showStatus("Output downloaded.", false);
  }

  function toggleDarkMode() {
    document.body.classList.toggle("dark-mode");
  }

  function showStatus(msg, isError = false) {
    status.textContent = msg;
    status.style.color = isError ? "red" : "green";
    if (isError) {
      status.innerHTML = `<i class="fas fa-times status-icon"></i>${msg}`;
    } else {
      status.innerHTML = `<i class="fas fa-check status-icon"></i>${msg}`;
    }
  }

  function clearStatus() {
    status.textContent = '';
  }
</script>
