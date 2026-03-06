# Tải file bị chặn tải về (View Only) từ Google Drive/Download View-Only Google Drive Files

## 🚀 🇻🇳 Cách sử dụng
1. Mở file Google Drive cần tải. **Cuộn đến cuối file để load toàn bộ các trang (rất quan trọng, nếu không chỉ tải được vài trang đầu)**.
2. Mở **Developer Tools** (F12 trên Windows) và chuyển đến tab **Console**.
3. Dán toàn bộ nội dung script bên dưới và dán vào console, nhấn Enter.
4. File PDF sẽ tự động tải xuống sau khi hoàn tất.

*⚠️ Lưu ý: file sẽ được tải về dưới dạng ảnh nên dung lượng sẽ hơi lớn so với một file văn bản thông thường.*

## 🚀 EN How to Use
1. Open the Google Drive file you want to download. **Make sure to scroll all the way to the end to load every page. This is crucial – if you don't, only the first few pages will be captured.**.
2. Press F12 to open **Developer Tools** and go to the **Console** tab.
3. Paste the entire script below into the Console and press Enter.
4. Wait for the processing to finish – the PDF file will be downloaded automatically.

*⚠️ Note: The generated PDF is composed of page images, so its size will be larger than a regular text‑based PDF.*

### 📝 Script
```
(function() {
  console.log("📦 Đang tải...");

  const script = document.createElement("script");
  script.onload = async function() {
    const { jsPDF } = window.jspdf;
    const images = Array.from(document.getElementsByTagName('img'));
    const validImages = images.filter(img =>
      img.src.startsWith('blob:https://drive.google.com/')
    );

    console.log(`${validImages.length}.`);

    if (validImages.length === 0) {
      console.error("");
      return;
    }

    const convertImage = (img, index) => new Promise((resolve) => {
      if (img.complete && img.naturalWidth > 0) {
        process(img, index, resolve);
      } else {
        const timer = setTimeout(() => {
          resolve(null);
        }, 10000);

        img.onload = () => {
          clearTimeout(timer);
          process(img, index, resolve);
        };
        img.onerror = () => {
          clearTimeout(timer);
          resolve(null);
        };
      }
    });

    const process = (img, index, resolve) => {
      try {
        const width = img.naturalWidth;
        const height = img.naturalHeight;

        if (width === 0 || height === 0) {
          return resolve(null);
        }

        const canvas = document.createElement('canvas');
        canvas.width = width;
        canvas.height = height;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0, width, height);

        const imgData = canvas.toDataURL('image/jpeg', 0.95);
        resolve({
          data: imgData,
          width,
          height,
          orientation: width > height ? 'l' : 'p'
        });
      } catch (err) {
        resolve(null);
      }
    };

    const results = await Promise.all(validImages.map(convertImage));
    const successful = results.filter(r => r !== null);

    if (successful.length === 0) {
      return;
    }

    let pdf = null;
    for (let i = 0; i < successful.length; i++) {
      const { data, width, height, orientation } = successful[i];

      if (i === 0) {
        pdf = new jsPDF({
          orientation: orientation,
          unit: 'px',
          format: [width, height]
        });
      } else {
        pdf.addPage([width, height], orientation);
      }

      pdf.addImage(data, 'JPEG', 0, 0, width, height, undefined, 'SLOW');
    }

    const metaName = document.querySelector('meta[itemprop="name"]')?.content;
    let filename = metaName || document.title || 'download.pdf';
    if (!filename.toLowerCase().endsWith('.pdf')) filename += '.pdf';

    pdf.save(filename, { returnPromise: true }).then(() => {
      document.body.removeChild(script);
    });
  };

  const scriptURL = 'https://unpkg.com/jspdf@latest/dist/jspdf.umd.min.js';
  let trustedURL = scriptURL;
  if (window.trustedTypes && trustedTypes.createPolicy) {
    try {
      const policy = trustedTypes.createPolicy('pdfGenerator', {
        createScriptURL: input => input
      });
      trustedURL = policy.createScriptURL(scriptURL);
    } catch (e) {
      console.warn(' TrustedTypes không áp dụng được.');
    }
  }
  script.src = trustedURL;
  document.body.appendChild(script);
})();
```
### 🪡 Nguồn tham khảo/Credits
Script này được dựa trên code gốc từ repository  [Google-Drive-View-Only-PDF-Script-Downloader](https://github.com/zavierferodova/Google-Drive-View-Only-PDF-Script-Downloader) của tác giả [@zavierferodova](https://github.com/zavierferodova), để tăng tốc độ tải file nhanh hơn.

This script is based on the original code from the repository [Google-Drive-View-Only-PDF-Script-Downloader](https://github.com/zavierferodova/Google-Drive-View-Only-PDF-Script-Downloader) by [@zavierferodova](https://github.com/zavierferodova), to further increase file downloading speed.
