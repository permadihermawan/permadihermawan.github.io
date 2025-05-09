---
layout: page
title: Background Remover
permalink: /background-remover/
---

# Image Background Remover

Upload your image below to remove the background instantly.

<form id="upload-form">
  <input type="file" id="image-upload" accept="image/*" />
  <button type="submit">Remove Background</button>
</form>

<div id="result" style="margin-top: 20px;"></div>

<script>
  const form = document.getElementById('upload-form');
  const imageInput = document.getElementById('image-upload');
  const resultDiv = document.getElementById('result');

  form.addEventListener('submit', async function (e) {
    e.preventDefault();

    const file = imageInput.files[0];
    if (!file) return;

    const formData = new FormData();
    formData.append('image_file', file);

    // Replace 'YOUR_API_KEY' with your actual remove.bg API key
    const response = await fetch('https://api.remove.bg/v1.0/removebg', {
      method: 'POST',
      headers: {
        'X-Api-Key': 'TooBJACiM2bYsPMCGYVuwuRs'
      },
      body: formData
    });

    if (response.ok) {
      const blob = await response.blob();
      const url = URL.createObjectURL(blob);
      resultDiv.innerHTML = `<img src="${url}" alt="Background removed image" style="max-width: 100%;">`;
    } else {
      const error = await response.text();
      resultDiv.innerHTML = `<p>Error: ${error}</p>`;
    }
  });
</script>
