# Yourstore_calcultor
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار المطورة</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #1e1e1e;
      color: white;
      direction: rtl;
      padding: 30px;
    }
    .logo {
      text-align: center;
      margin-bottom: 20px;
    }
    .logo img {
      max-height: 100px;
    }
    select, input {
      padding: 8px;
      margin: 5px 0;
      width: 100%;
      border: none;
      border-radius: 4px;
      box-sizing: border-box;
      background-color: #333;
      color: white;
    }
    input:disabled {
      background-color: #555;
      cursor: not-allowed;
      color: #aaa;
    }
    .section {
      background-color: #252526;
      padding: 15px;
      border-radius: 8px;
      margin-bottom: 20px;
      border: 1px solid #333;
    }
    .addon-section {
        background-color: #2a2d2e;
        padding: 10px;
        border-radius: 6px;
        margin-top: 10px;
    }
    label {
        font-weight: bold;
        color: #00e676;
    }
    button {
      background-color: #007acc;
      color: white;
      padding: 12px 22px;
      border: none;
      margin-top: 10px;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: #005a9e;
    }
    .btn-secondary { background-color: #555; }
    .btn-secondary:hover { background-color: #777; }
    .btn-danger { background-color: #c0392b; }
    .btn-danger:hover { background-color: #a52f22; }

    .results {
      background: #2b2b2b;
      padding: 20px;
      margin-top: 20px;
      border-radius: 8px;
    }
    .result-item {
      border-bottom: 1px solid #555;
      padding: 15px 0;
      line-height: 1.8;
    }
    .result-item:last-child {
      border-bottom: none;
    }
    .summary {
      font-weight: bold;
      color: #00e676;
      padding-top: 15px;
      border-top: 2px solid #00e676;
      margin-top: 15px;
      font-size: 1.1em;
    }
  </style>
</head>
<body>

<div class="logo">
  <img src="https://i.imgur.com/ih5zjVj.png" alt="شعار الشركة">
</div>

<div class="section">
  <label for="mainCategory">الفئة الرئيسية:</label>
  <select id="mainCategory" onchange="loadSubTypes()"></select>
</div>

<div class="section">
  <label for="subType">النوع الفرعي:</label>
  <select id="subType" onchange="updateUIBasedOnType()"></select>
</div>

<div class="section">
  <label for="height">الطول (متر):</label>
  <input type="number" id="height" step="0.01" value="1" />
  
  <label for="width">العرض (متر):</label>
  <input type="number" id="width" step="0.01" value="1" />

  <label for="quantity">الكمية:</label>
  <input type="number" id="quantity" value="1" />
</div>

<!-- قسم الإضافات الديناميكي -->
<div class="section" id="addonsHost"></div>

<button onclick="calculate()">➕ أضف للنتائج</button>
<button onclick="clearResults()" class="btn-danger">🗑️ مسح النتائج</button>
<button onclick="saveAsWord()" class="btn-secondary">💾 حفظ كـ Word</button>

<div class="results" id="results"></div>

<script>
// ثابت سعر الشحن لكل CBM
const SHIPPING_RATE = 48;

// هيكل البيانات الشامل
const productData = {
    "نوافذ": {
        "دبل جلاس دبل فريم ثابتة": { price: 34, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس دبل فريم حركة": { price: 73, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس دبل فريم حركتين": { price: 92, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس سنجل فريم ثابتة": { price: 26, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس سنجل فريم حركة": { price: 46, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس سنجل فريم حركتين": { price: 58, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "سنجل جلاس سنجل فريم ثابتة": { price: 20, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "سنجل جلاس سنجل فريم حركة": { price: 43, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "سنجل جلاس سنجل فريم حركتين": { price: 47, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "نوافذ السلايدنج": { price: 10, cbm: 0.13, method: 'per_meter', special: 'add_10', addons: ['curtain'] },
        "النوافذ الكهربائية": { price: 102, cbm: 0.13, method: 'per_meter' },
        "سكاي لايت بدون مكينة": { price: 56, cbm: 0.13, method: 'per_meter' },
        "سكاي لايت مع مكينة": { price: 145, cbm: 0.13, method: 'per_meter' },
        "كارتن وول ثقيل": { price: 56, cbm: 0.15, method: 'per_meter' },
        "كارتن وول خفيف": { price: 45, cbm: 0.15, method: 'per_meter' },
    },
    "أبواب": {
        "باب المدخل - زينك": { price: 66, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "باب المدخل - ستينلس ستيل": { price: 120, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "باب المدخل - كاست المنيوم": { price: 168, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "WPC - فارغ": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC - مع خشب": { price: 50, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC - مع حشوة ضد الصوت": { price: 60, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC - مع فريم ألمينيوم": { price: 67, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "ألمنيوم - فارغ": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "ألمنيوم - مع خشب": { price: 75, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "ألمنيوم - فل": { price: 85, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "ألمنيوم - باب مخفي": { price: 110, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "ألمنيوم - باب خارجي": { price: 61, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "دورات مياه - النوع الجديد": { price: 55, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "دورات مياه - النوع القديم": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "دورات مياه - مخفي زجاجي": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
    },
    "أبواب سحب": {
        "داخلي - زجاج": { price: 38, cbm: 0.15, method: 'per_meter', addons: ['curtain'] },
        "داخلي - متين": { price: 41, cbm: 0.15, method: 'per_meter' },
        "خارجي - جزء مفتوح": { price: 55, cbm: 0.15, method: 'per_meter' },
        "خارجي - جزئين مفتوحات": { price: 58, cbm: 0.15, method: 'per_meter' },
        "WPC - سلايد": { price: 61, cbm: 0.15, method: 'per_meter' },
    },
    "أبواب فولدنج": {
        "داخلي": { price: 39, cbm: 0.15, method: 'per_meter', addons: ['curtain'] },
        "خارجي": { price: 56, cbm: 0.15, method: 'per_meter' },
    },
    "شتر خارجي": {
        "شتر رول": { price: 28, cbm: 0.20, method: 'per_meter' },
    },
    "أبواب حدائق": {
        "كاست المنيوم": { price: 91, cbm: 0.20, method: 'per_meter' },
    },
    "حواجز": {
        "حاجز درج": { price: 43, cbm: 0.05, method: 'per_meter' },
        "حاجز حمام": { price: 32, cbm: 0.05, method: 'per_meter' },
    }
};

const addonPrices = {
    curtain: 26, net: { 'باب': 39, 'فولدنج': 18, 'سلايد': 14 }, telbisa: 10
};

let resultsList = [];

function initializeApp() {
    const mainCat = document.getElementById("mainCategory");
    mainCat.innerHTML = `<option value="">-- اختر الفئة --</option>`;
    Object.keys(productData).forEach(cat => mainCat.innerHTML += `<option value="${cat}">${cat}</option>`);
    loadSubTypes();
}

function loadSubTypes() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subType = document.getElementById("subType");
    subType.innerHTML = "";
    if (mainCatVal && productData[mainCatVal]) {
        Object.keys(productData[mainCatVal]).forEach(sub => subType.innerHTML += `<option value="${sub}">${sub}</option>`);
    }
    updateUIBasedOnType();
}

function updateUIBasedOnType() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subVal = document.getElementById("subType").value;
    
    if (!mainCatVal || !subVal) {
        document.getElementById("addonsHost").innerHTML = "";
        return;
    }

    const data = productData[mainCatVal][subVal];
    const heightInput = document.getElementById("height"), widthInput = document.getElementById("width");
    const addonsHost = document.getElementById("addonsHost");
    addonsHost.innerHTML = ""; 

    heightInput.disabled = false;
    widthInput.disabled = false;

    if (data.method === 'per_unit') {
        heightInput.value = data.std_h;
        widthInput.value = data.std_w;
    }
    
    if (data.addons) {
        if (data.addons.includes('curtain')) addonsHost.innerHTML += `<div class="addon-section"><label><input type="checkbox" id="addCurtain" /> إضافة ستارة داخلية (26 ريال للمتر)</label></div>`;
        if (data.addons.includes('net')) addonsHost.innerHTML += `<div class="addon-section"><label for="netType">إضافة شبك:</label><select id="netType"><option value="">-- بدون --</option><option value="باب">باب (${addonPrices.net['باب']} ريال للمتر)</option><option value="فولدنج">فولدنج (${addonPrices.net['فولدنج']} ريال للمتر)</option><option value="سلايد">سلايد (${addonPrices.net['سلايد']} ريال للمتر)</option></select></div>`;
        if (data.addons.includes('telbisa')) {
             addonsHost.innerHTML += `<div class="addon-section"><label><input type="checkbox" id="addTelbisa" /> إضافة تلبيسة للسقف</label><input type="number" id="telbisaLength" placeholder="طول التلبيسة بالمتر" style="display:none;" value="1" step="0.1" /></div>`;
             document.getElementById('addTelbisa').onchange = e => document.getElementById('telbisaLength').style.display = e.target.checked ? 'block' : 'none';
        }
    }
}

function calculate() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subVal = document.getElementById("subType").value;
    if (!mainCatVal || !subVal) { alert("يرجى اختيار الفئة والنوع."); return; }
    
    const data = productData[mainCatVal][subVal];
    const qty = parseInt(document.getElementById("quantity").value) || 1;
    const h = parseFloat(document.getElementById("height").value) || 0;
    const w = parseFloat(document.getElementById("width").value) || 0;
    if (h === 0 || w === 0) { alert("الأبعاد يجب أن تكون أكبر من صفر."); return; }
    
    const area = h * w;
    let basePrice = 0, sizePenalty = 0;

    if (data.method === 'per_meter') basePrice = data.price * area;
    else if (data.method === 'per_unit') {
        basePrice = data.price;
        const stdArea = data.std_h * data.std_w;
        if (area > stdArea) sizePenalty = Math.ceil((area - stdArea) / 0.1) * 2;
    }

    let itemPrice = basePrice + sizePenalty;
    let addonsDetails = "";
    
    if (document.getElementById("addCurtain")?.checked) {
        const cost = addonPrices.curtain * area; itemPrice += cost; addonsDetails += `<li>ستارة: ${cost.toFixed(2)}</li>`;
    }
    const netType = document.getElementById("netType")?.value;
    if (netType) {
        const cost = (area * 0.5) * addonPrices.net[netType]; itemPrice += cost; addonsDetails += `<li>شبك ${netType}: ${cost.toFixed(2)}</li>`;
    }
    let telbisaLength = 0;
    if (document.getElementById("addTelbisa")?.checked) {
        telbisaLength = parseFloat(document.getElementById("telbisaLength").value) || 0;
        if (telbisaLength > 0) {
            const cost = Math.ceil(telbisaLength) * addonPrices.telbisa; itemPrice += cost; addonsDetails += `<li>تلبيسة سقف (${telbisaLength}م): ${cost.toFixed(2)}</li>`;
        }
    }
    if (data.special === 'add_10') { itemPrice += 10; addonsDetails += `<li>إضافة خاصة: 10.00</li>`; }

    let shippingCBM = (telbisaLength > 0) ? ((h + telbisaLength) * w) * data.cbm : area * data.cbm;
    const shippingCost = shippingCBM * SHIPPING_RATE;

    const totalItemCost = itemPrice + shippingCost;
    resultsList.push({
        sub: subVal, h, w, qty,
        details: { basePrice, sizePenalty, addonsHTML: addonsDetails, shippingPerItem: shippingCost },
        singleItemPrice: totalItemCost,
        final: totalItemCost * qty
    });
    renderResults();
}

function renderResults() {
    const container = document.getElementById("results");
    container.innerHTML = "";
    let sum = 0;
    resultsList.forEach(r => {
        sum += r.final;
        const d = r.details;
        const totalAddonsPrice = (r.singleItemPrice - d.basePrice - d.sizePenalty - d.shippingPerItem) * r.qty;
        container.innerHTML += `
            <div class="result-item">
                <b>النوع:</b> ${r.sub} (الكمية: ${r.qty})<br>
                <b>المقاس:</b> ${r.h} × ${r.w} متر<br>
                <ul>
                    <li>السعر الأساسي الإجمالي: ${(d.basePrice * r.qty).toFixed(2)}</li>
                    ${d.sizePenalty > 0 ? `<li>إجمالي تكلفة زيادة الحجم: ${(d.sizePenalty * r.qty).toFixed(2)}</li>` : ''}
                    ${totalAddonsPrice > 0 ? `<li>إجمالي الإضافات: ${totalAddonsPrice.toFixed(2)}</li>` : ''}
                    <li>🚚 إجمالي الشحن: ${(d.shippingPerItem * r.qty).toFixed(2)}</li>
                </ul>
                <b>🏷️ سعر الحبة الواحدة: ${r.singleItemPrice.toFixed(2)} ريال</b><br>
                <b>💵 السعر الإجمالي: ${r.final.toFixed(2)} ريال</b>
            </div>`;
    });

    if (resultsList.length > 0) {
        const comm = sum * 0.04;
        const total = sum + comm;
        container.innerHTML += `
            <div class="summary">
                المجموع الفرعي: ${sum.toFixed(2)}<br>
                عمولة المكتب (4%): ${comm.toFixed(2)}<br>
                <b>💰 الناتج النهائي: ${total.toFixed(2)} ريال</b><br><br>
                <span style="color:#ccc; font-size: 0.9em;">* الأسعار شاملة الشحن<br>** الأسعار لا تشمل التركيب</span>
            </div>`;
    }
}

function clearResults() {
    resultsList = [];
    document.getElementById("results").innerHTML = "";
    document.getElementById("mainCategory").value = "";
    document.getElementById("height").value = "1";
    document.getElementById("width").value = "1";
    document.getElementById("quantity").value = "1";
    loadSubTypes(); 
}

// --- دالة الحفظ في ملف وورد (تم تعديلها بالكامل) ---
function saveAsWord(){
    if (resultsList.length === 0) {
        alert("لا توجد نتائج لحفظها.");
        return;
    }

    // بناء محتوى HTML للجدول بشكل منفصل
    const logoHtml = `<div style="text-align: center; margin-bottom: 20px;"><img src="https://i.imgur.com/ih5zjVj.png" alt="شعار الشركة" style="max-height: 100px;"></div>`;
    
    let tableHtml = `<table border="1" width="100%" style="border-collapse: collapse; font-family: Arial, sans-serif; text-align: center;">
                        <thead>
                            <tr style="background-color: #f2f2f2;">
                                <th style="padding: 8px;">النوع</th>
                                <th style="padding: 8px;">الكمية</th>
                                <th style="padding: 8px;">المقاس (متر)</th>
                                <th style="padding: 8px;">تفاصيل السعر (للقطعة)</th>
                                <th style="padding: 8px;">سعر الحبة</th>
                                <th style="padding: 8px;">السعر الإجمالي</th>
                            </tr>
                        </thead>
                        <tbody>`;
    
    let sum = 0;
    resultsList.forEach(r => {
        sum += r.final;
        const d = r.details;
        let detailsCellHtml = `<ul style="text-align: right; margin: 0; padding-right: 15px;">
                                <li>أساسي: ${d.basePrice.toFixed(2)}</li>
                                ${d.sizePenalty > 0 ? `<li>زيادة حجم: ${d.sizePenalty.toFixed(2)}</li>` : ''}
                                ${d.addonsHTML.replace(/<li>/g, '<li style="margin-right: -15px;">')}
                                <li>شحن: ${d.shippingPerItem.toFixed(2)}</li>
                               </ul>`;
        
        tableHtml += `<tr>
                        <td style="padding: 8px;">${r.sub}</td>
                        <td style="padding: 8px;">${r.qty}</td>
                        <td style="padding: 8px;">${r.h} × ${r.w}</td>
                        <td style="padding: 8px;">${detailsCellHtml}</td>
                        <td style="padding: 8px;">${r.singleItemPrice.toFixed(2)}</td>
                        <td style="padding: 8px;">${r.final.toFixed(2)}</td>
                      </tr>`;
    });
    tableHtml += `</tbody>`;

    const comm = sum * 0.04;
    const total = sum + comm;

    tableHtml += `<tfoot style="font-weight: bold;">
                    <tr>
                        <td colspan="5" style="text-align: left; padding: 8px;">المجموع الفرعي</td>
                        <td style="padding: 8px;">${sum.toFixed(2)} ريال</td>
                    </tr>
                    <tr>
                        <td colspan="5" style="text-align: left; padding: 8px;">عمولة المكتب (4%)</td>
                        <td style="padding: 8px;">${comm.toFixed(2)} ريال</td>
                    </tr>
                    <tr style="background-color: #e0f8e0;">
                        <td colspan="5" style="text-align: left; padding: 8px;">الناتج النهائي</td>
                        <td style="padding: 8px;">${total.toFixed(2)} ريال</td>
                    </tr>
                  </tfoot>`;

    tableHtml += `</table>`;
    
    const notes = `<div style="margin-top: 20px; font-size: 12px; font-family: Arial, sans-serif;">
                    <p>* الأسعار شاملة الشحن<br>** الأسعار لا تشمل التركيب</p>
                   </div>`;

    // تجميع كل أجزاء HTML
    const header = `<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'><head><meta charset='utf-8'><title>عرض سعر</title></head><body dir="rtl">`;
    const footer = "</body></html>";
    const sourceHTML = header + logoHtml + tableHtml + notes + footer;

    // إنشاء وتنزيل الملف
    const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(sourceHTML);
    const fileDownload = document.createElement("a");
    document.body.appendChild(fileDownload);
    fileDownload.href = source;
    fileDownload.download = 'عرض_سعر.doc';
    fileDownload.click();
    document.body.removeChild(fileDownload);
}

initializeApp();
</script>

</body>
</html>
