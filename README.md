# HelpEKG
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ER ECG Master - Full Clinical Diagnosis</title>
    <style>
        :root { --primary: #c0392b; --secondary: #2c3e50; --success: #27ae60; --danger: #e74c3c; --info: #2980b9; --warning: #f39c12; }
        body { font-family: 'Sarabun', Arial, sans-serif; background: #f0f2f5; padding: 10px; color: #333; line-height: 1.5; }
        .card { max-width: 1100px; margin: auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 5px 25px rgba(0,0,0,0.15); }
        h2 { text-align: center; color: var(--primary); margin-bottom: 20px; border-bottom: 2px solid var(--primary); padding-bottom: 10px; }
        
        .section-title { background: var(--secondary); color: white; padding: 8px 15px; margin: 20px 0 10px; border-radius: 5px; font-weight: bold; display: flex; justify-content: space-between; align-items: center; }
        .grid-inputs { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 10px; margin-bottom: 20px; background: #fff5f5; padding: 15px; border-radius: 10px; border: 1px solid #ffcccc; }
        .grid-axis { display: grid; grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); gap: 10px; background: #ebf8ff; padding: 15px; border-radius: 10px; border: 1px solid #bee3f8; }
        
        label { font-weight: bold; font-size: 0.85em; display: block; margin-bottom: 5px; color: #444; }
        input, select { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; font-size: 16px; }
        
        .pattern-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 10px; }
        .item { display: flex; align-items: center; justify-content: space-between; background: #fdfdfd; padding: 10px; border-radius: 8px; border: 1px solid #ddd; transition: 0.2s; }
        .item:hover { border-color: var(--info); background: #f0f7ff; }
        .item-label { cursor: pointer; display: flex; align-items: center; width: 100%; font-weight: bold; font-size: 0.85em; }
        .item-label input { width: auto; margin-right: 10px; transform: scale(1.3); }
        
        .guide-icon { background: var(--info); color: white; border: none; border-radius: 50%; width: 22px; height: 22px; cursor: pointer; font-size: 12px; font-weight: bold; flex-shrink: 0; }
        
        .analyze-btn { width: 100%; padding: 20px; margin-top: 25px; background: #27ae60; color: white; font-size: 22px; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; box-shadow: 0 4px 0 #1e8449; }
        .analyze-btn:active { transform: translateY(2px); box-shadow: none; }

        #result { margin-top: 20px; padding: 20px; border-radius: 10px; display: none; border-left: 12px solid; }
        .headline-box { font-size: 1.5em; font-weight: 900; border-bottom: 3px solid rgba(0,0,0,0.1); padding-bottom: 10px; margin-bottom: 15px; letter-spacing: 0.5px; }
        
        .danger { background: #fff5f5; color: #c53030; border-color: #c53030; }
        .warning { background: #fffaf0; color: #c05621; border-color: #c05621; }
        .normal { background: #f0fff4; color: #2f855a; border-color: #2f855a; }

        /* Modal Guide */
        .modal { display: none; position: fixed; z-index: 2000; left: 0; top: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); overflow-y: auto; }
        .modal-content { background: white; margin: 5% auto; padding: 25px; border-radius: 12px; width: 90%; max-width: 600px; position: relative; }
        .close { position: absolute; right: 20px; top: 15px; font-size: 30px; cursor: pointer; }
    </style>
</head>
<body>

<div class="card">
    <h2>ตัวช่วยในการอ่าน EKG (รุ่น 2 ต้องผ่านหมด ✌🏻) </h2>

    <div class="section-title">1. วัดค่าและสัญญาณชีพ (Vitals & Intervals)</div>
    <div class="grid-inputs">
        <div><label>Heart Rate (bpm)</label><input type="number" id="hr" placeholder="60-100"></div>
        <div><label>PR Interval (sec)</label><input type="number" step="0.01" id="pr" placeholder="0.12-0.20"></div>
        <div><label>QRS Duration (sec)</label><input type="number" step="0.01" id="qrs" placeholder="< 0.12"></div>
        <div><label>S(V1) + R(V5/V6) (mm)</label><input type="number" id="lvh_val" placeholder="LVH if > 35"></div>
 <div><label>S ใน V1 (mm)</label><input type="number" id="sv1" placeholder="สำหรับ LVH"></div>
        <div><label>R ใน V5 หรือ V6 (mm)</label><input type="number" id="rv5" placeholder="สำหรับ LVH"></div>
        <div><label>R ใน V1 (mm)</label><input type="number" id="rv1" placeholder="สำหรับ RVH (> 7mm)"></div>
    </div>

    <div class="section-title">2. แกนหัวใจ (Heart Axis)</div>
    <div class="grid-axis">
        <div><label>Lead I</label><select id="l1"><option value="pos">+</option><option value="neg">-</option></select></div>
        <div><label>Lead II</label><select id="l2"><option value="pos">+</option><option value="neg">-</option></select></div>
        <div><label>aVF</label><select id="avf"><option value="pos">+</option><option value="neg">-</option></select></div>
    </div>

    <div class="section-title">3. ความผิดปกติ (Rhythm & Clinical Findings)</div>
    <div class="pattern-grid">
        <div class="item"><label class="item-label"><input type="checkbox" id="afib"> AF / A-Flutter</label><button class="guide-icon" onclick="showGuide('afib')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="svt"> SVT</label><button class="guide-icon" onclick="showGuide('svt')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="vt"> VT / VF</label><button class="guide-icon" onclick="showGuide('vt')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="avb2"> 2nd Degree AVB</label><button class="guide-icon" onclick="showGuide('avb2')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="avb3"> 3rd Degree (Complete) AVB</label><button class="guide-icon" onclick="showGuide('avb3')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="lbbb"> LBBB</label><button class="guide-icon" onclick="showGuide('lbbb')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="rbbb"> RBBB</label><button class="guide-icon" onclick="showGuide('rbbb')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="stemi"> STEMI (Acute MI)</label><button class="guide-icon" onclick="showGuide('stemi')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="wellens"> Wellens' Syndrome</label><button class="guide-icon" onclick="showGuide('wellens')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="hyperk"> Hyperkalemia (Tall T)</label><button class="guide-icon" onclick="showGuide('hyperk')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="pe"> PE (S1Q3T3)</label><button class="guide-icon" onclick="showGuide('pe')">i</button></div>
        <div class="item"><label class="item-label"><input type="checkbox" id="brugada"> Brugada Syndrome</label><button class="guide-icon" onclick="showGuide('brugada')">i</button></div>
    </div>

    <button class="analyze-btn" onclick="analyze()">⚡ แปลผลแบบ Clinical Summary</button>

    <div id="result">
        <div id="headline" class="headline-box"></div>
        <div id="details"></div>
    </div>
</div>

<div id="guideModal" class="modal">
    <div class="modal-content">
        <span class="close" onclick="closeModal()">&times;</span>
        <h3 id="guideTitle" style="color:var(--primary)"></h3>
        <div id="guideBody"></div>
    </div>
</div>

<script>
const guides = {
    afib: ["AF / A-Flutter", "• Irregularly Irregular rhythm<br>• No P waves (AF) หรือ Sawtooth waves (Flutter)<br>• ประเมิน CHA2DS2-VASc"],
    svt: ["SVT", "• Regular, Narrow QRS tachycardia<br>• HR มัก > 150 bpm<br>• มองไม่เห็น P wave"],
    vt: ["VT / VF", "• VT: Wide QRS, Regular, Fast (🚨 เช็ค Pulse ด่วน!)<br>• VF: Chaotic waves (🚨 เริ่ม CPR/Defib ทันที)"],
    avb2: ["2nd Degree AV Block", "• Mobitz I (Wenckebach): PR ยาวขึ้นเรื่อยๆ จน QRS หาย<br>• Mobitz II: PR คงที่ แต่มี QRS หายไปเป็นระยะ (อันตราย!)"],
    avb3: ["3rd Degree (Complete) AVB", "• P wave กับ QRS เต้นแยกกันโดยสิ้นเชิง (P-P สม่ำเสมอ, R-R สม่ำเสมอ)<br>• Atrial rate เร็วกว่า Ventricular rate"],
    stemi: ["STEMI", "• ST-segment elevation > 1mm ในอย่างน้อย 2 leads ต่อเนื่องกัน<br>• หรือมี New LBBB ร่วมกับอาการเจ็บหน้าอก"],
    wellens: ["Wellens' Syndrome", "• Deeply inverted หรือ Biphasic T-waves ใน V2-V3<br>• บ่งบอกถึง Proximal LAD stenosis ขั้นรุนแรง"],
    hyperk: ["Hyperkalemia", "• Peaked T-waves (Tall, narrow, pointed)<br>• ระยะต่อมา QRS จะกว้างขึ้น จนกลายเป็น Sine wave"],
    pe: ["Pulmonary Embolism", "• S1Q3T3 (S ใน L1, Q ใน L3, T invert ใน L3)<br>• มักพบ Sinus Tachycardia ร่วมด้วย"],
    brugada: ["Brugada Syndrome", "• Coved ST elevation > 2mm ใน V1-V2 ตามด้วย Inverted T-wave<br>• เสี่ยงต่อ Sudden Cardiac Death"],
    lbbb: ["LBBB", "• QRS > 0.12s<br>• Wide/Notched R wave ใน Lead V6 (M-shape)"],
    rbbb: ["RBBB", "• QRS > 0.12s<br>• rsR' pattern ใน Lead V1 (Rabbit ears)"]
};

function showGuide(key) {
    document.getElementById('guideTitle').innerText = guides[key][0];
    document.getElementById('guideBody').innerHTML = guides[key][1];
    document.getElementById('guideModal').style.display = "block";
}

function closeModal() { document.getElementById('guideModal').style.display = "none"; }

function analyze() {
    let hr = document.getElementById("hr").value || "?";
    let pr = parseFloat(document.getElementById("pr").value);
    let qrs = parseFloat(document.getElementById("qrs").value);
    let lvh_val = parseFloat(document.getElementById("lvh_val").value);
    let l1 = document.getElementById("l1").value, l2 = document.getElementById("l2").value, avf = document.getElementById("avf").value;

    let diag = [];
    let isUrgent = false;

    // Axis Logic
    let axisStr = "Normal Axis";
    if (l1 === 'pos' && avf === 'neg') axisStr = (l2 === 'neg') ? "LAD" : "Normal Axis";
    else if (l1 === 'neg' && avf === 'pos') axisStr = "RAD";
    else if (l1 === 'neg' && avf === 'neg') axisStr = "Extreme Axis";

    // Diagnosis Logic
    if (document.getElementById("stemi").checked) { diag.push("STEMI"); isUrgent = true; }
    if (document.getElementById("vt").checked) { diag.push("VT/VF"); isUrgent = true; }
    if (document.getElementById("avb3").checked) { diag.push("3rd Deg AVB"); isUrgent = true; }
    if (document.getElementById("svt").checked) { diag.push("SVT"); isUrgent = true; }
    if (document.getElementById("wellens").checked) { diag.push("Wellens'"); isUrgent = true; }
    if (document.getElementById("hyperk").checked) { diag.push("Hyperkalemia"); isUrgent = true; }
    if (document.getElementById("afib").checked) diag.push("AF/A-Flutter");
    if (document.getElementById("lbbb").checked) diag.push("LBBB");
    if (document.getElementById("rbbb").checked) diag.push("RBBB");
    if (document.getElementById("pe").checked) diag.push("PE (S1Q3T3)");
    
    // Auto-check from values
    if (pr > 0.20 && !document.getElementById("avb3").checked) diag.push("1st Deg AVB");
    if (lvh_val >= 35) diag.push("LVH");
    if (hr < 60 && diag.length === 0) diag.push("Sinus Bradycardia");
    if (hr > 100 && diag.length === 0) diag.push("Sinus Tachycardia");
    if (diag.length === 0) diag.push("NSR");

    let rhythmType = document.getElementById("afib").checked ? "Irregular" : "Regular";

    // Headline Construction
    let mainHeadline = `${diag.join(" + ")} | ${rhythmType} ${axisStr} | Rate: ${hr} Bpm`;

    // Display
    let box = document.getElementById("result");
    box.style.display = "block";
    box.className = isUrgent ? "danger" : (diag.includes("NSR") ? "normal" : "warning");
    document.getElementById("headline").innerText = mainHeadline;
    
    let detailsHTML = "<b>ข้อควรระวัง/แนะนำ:</b><ul>";
    if (isUrgent) detailsHTML += "<li>🚨 <b>EMERGENCY:</b> ภาวะวิกฤต เตรียมรถ Emergency และแจ้งทีมแพทย์ทันที</li>";
    if (diag.includes("Hyperkalemia")) detailsHTML += "<li>• พิจารณาให้ Ca Gluconate หาก QRS เริ่มกว้าง</li>";
    if (diag.includes("Wellens'")) detailsHTML += "<li>• <b>ห้ามทำ Exercise Stress Test!</b> ส่งสวนหัวใจโดยด่วน</li>";
    detailsHTML += "</ul>";
    document.getElementById("details").innerHTML = detailsHTML;
}
</script>
</body>
</html>
