# tT<!doctype html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Selfie Camera (Legal)</title>
  <style>
    body{font-family:Arial; background:#111; color:#fff; text-align:center; padding:18px}
    video, canvas{width:100%; max-width:420px; border-radius:12px; margin-top:12px; background:#000}
    button{padding:12px 18px; font-size:16px; margin:8px; cursor:pointer}
    .row{display:flex; gap:10px; justify-content:center; flex-wrap:wrap}
    small{opacity:.8}
  </style>
</head>
<body>
  <h2>📸 سيلفي كاميرا</h2>
  <small>سيُطلب إذن الكاميرا. لن يتم حفظ/إرسال شيء تلقائياً.</small>

  <video id="v" autoplay playsinline></video>

  <div class="row">
    <button id="start">تشغيل الكاميرا</button>
    <button id="snap" disabled>التقاط صورة</button>
    <button id="download" disabled>تنزيل الصورة</button>
    <button id="stop" disabled>إيقاف الكاميرا</button>
  </div>

  <canvas id="c"></canvas>

  <script>
    const v = document.getElementById('v');
    const c = document.getElementById('c');
    const ctx = c.getContext('2d');

    const startBtn = document.getElementById('start');
    const snapBtn = document.getElementById('snap');
    const dlBtn = document.getElementById('download');
    const stopBtn = document.getElementById('stop');

    let stream = null;
    let lastBlobUrl = null;

    async function startCamera(){
      // يفتح الكاميرا الأمامية (سيلفي) إن توفرت
      stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: "user" },
        audio: false
      });
      v.srcObject = stream;
      snapBtn.disabled = false;
      stopBtn.disabled = false;
      startBtn.disabled = true;
    }

    function takePhoto(){
      c.width = v.videoWidth;
      c.height = v.videoHeight;
      ctx.drawImage(v, 0, 0);

      // تجهيز تنزيل
      c.toBlob((blob) => {
        if (lastBlobUrl) URL.revokeObjectURL(lastBlobUrl);
        lastBlobUrl = URL.createObjectURL(blob);
        dlBtn.disabled = false;
      }, "image/jpeg", 0.92);
    }

    function downloadPhoto(){
      const a = document.createElement('a');
      a.href = lastBlobUrl;
      a.download = "selfie.jpg";
      document.body.appendChild(a);
      a.click();
      a.remove();
    }

    function stopCamera(){
      if (stream){
        stream.getTracks().forEach(t => t.stop());
        stream = null;
      }
      v.srcObject = null;
      startBtn.disabled = false;
      snapBtn.disabled = true;
      dlBtn.disabled = true;
      stopBtn.disabled = true;
      if (lastBlobUrl) { URL.revokeObjectURL(lastBlobUrl); lastBlobUrl = null; }
    }

    startBtn.onclick = () => startCamera().catch(() =>
      alert("لازم تفتح الرابط عبر HTTPS وتضغط سماح للكاميرا.")
    );
    snapBtn.onclick = takePhoto;
    dlBtn.onclick = downloadPhoto;
    stopBtn.onclick = stopCamera;
  </script>
</body>
</html>
