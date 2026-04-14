<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Card Printer - 2 A4 Pages</title>
    <style>
        @page { size: A4; margin: 0; }
        body { font-family: sans-serif; background: #ddd; margin: 0; padding: 20px; text-align: center; }
        
        /* Control Panel */
        .controls { 
            background: white; padding: 20px; border-radius: 10px; 
            margin-bottom: 20px; display: inline-block; 
            box-shadow: 0 4px 10px rgba(0,0,0,0.2); width: 700px; 
            text-align: left; position: sticky; top: 10px; z-index: 100;
        }
        .controls h3 { margin: 0 0 15px 0; text-align: center; color: #333; }
        
        .grid-inputs { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .card-input-row { 
            background: #f8fafc; padding: 10px; border-radius: 8px; 
            border: 1px solid #cbd5e1; display: flex; flex-direction: column; gap: 5px;
        }
        .card-input-row label { font-weight: bold; font-size: 14px; border-bottom: 1px solid #ddd; padding-bottom: 3px; }
        .input-group { display: flex; justify-content: space-between; align-items: center; }
        .input-group input { width: 140px; font-size: 11px; }
        
        .clear-btn { padding: 2px 8px; cursor: pointer; border: 1px solid #f87171; background: #fee2e2; color: #dc2626; border-radius: 4px; font-size: 11px; }

        .bottom-ctrl { display: flex; align-items: center; justify-content: space-between; margin-top: 15px; border-top: 1px solid #eee; pt: 10px; }
        .sliders { display: flex; gap: 15px; font-size: 14px; font-weight: bold; }
        button.print-btn { padding: 12px 30px; background: #2563eb; color: white; border: none; cursor: pointer; font-size: 16px; border-radius: 6px; font-weight: bold; }

        /* A4 Page Layout */
        .page {
            width: 210mm; height: 297mm;
            background: white; margin: 20px auto;
            padding: 10mm 5mm;
            display: grid;
            grid-template-columns: 3.6in 3.6in;
            grid-template-rows: repeat(4, 2.3in);
            column-gap: 8mm; row-gap: 8mm;
            justify-content: center;
            page-break-after: always;
            box-sizing: border-box;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }

        .card-slot {
            width: 3.6in; height: 2.3in;
            border: 1px dashed #cbd5e1;
            overflow: hidden; display: flex; align-items: center; justify-content: center;
            position: relative; background: #fafafa;
        }
        
        .card-slot.filled { border: 0.1mm solid #000; background: white; }
        .card-slot img { width: 100%; height: 100%; object-fit: stretch; filter: contrast(var(--con)) brightness(var(--bri)); }

        .card-slot::before { content: attr(data-label); position: absolute; color: #ccc; font-size: 14px; }
        .card-slot.filled::before { display: none; }

        @media print {
            .controls { display: none; }
            body { padding: 0; background: none; }
            .page { margin: 0; box-shadow: none; border: none; }
            .card-slot { border: none !important; background: none; }
            .card-slot.filled { border: 0.1mm solid #000 !important; }
            .card-slot::before { display: none; }
        }
    </style>
</head>
<body style="--con: 1.1; --bri: 1.0;">

    <div class="controls">
        <h3>🖨️ Multi-Page Card Printer (8 Cards)</h3>
        
        <div class="grid-inputs" id="inputs-panel"></div>

        <div class="bottom-ctrl">
            <div class="sliders">
                <div>Con: <input type="range" id="con" min="1" max="2" step="0.1" value="1.1"></div>
                <div>Bri: <input type="range" id="bri" min="0.8" max="1.5" step="0.1" value="1.0"></div>
            </div>
            <button class="print-btn" onclick="window.print()">PRINT 2 PAGES</button>
        </div>
    </div>

    <div class="page" id="page1"></div>
    <div class="page" id="page2"></div>

    <script>
        const inputsPanel = document.getElementById('inputs-panel');
        const page1 = document.getElementById('page1');
        const page2 = document.getElementById('page2');

        // Brightness/Contrast
        document.querySelectorAll('input[type=range]').forEach(s => {
            s.oninput = (e) => document.body.style.setProperty('--' + e.target.id, e.target.value);
        });

        // Create 8 Slots for 2 Pages
        for(let i = 1; i <= 8; i++) {
            // Add Inputs
            inputsPanel.innerHTML += `
                <div class="card-input-row" id="row-${i}">
                    <div style="display:flex; justify-content:space-between">
                        <label>Card ${i}</label>
                        <button class="clear-btn" onclick="clearRow(${i})">Clear</button>
                    </div>
                    <div class="input-group">
                        <span>Front:</span> <input type="file" accept="image/*" onchange="loadImage(this, 'f${i}')">
                    </div>
                    <div class="input-group">
                        <span>Back:</span> <input type="file" accept="image/*" onchange="loadImage(this, 'b${i}')">
                    </div>
                </div>
            `;

            // Assign to Page 1 or Page 2
            const targetPage = i <= 4 ? page1 : page2;
            targetPage.innerHTML += `
                <div class="card-slot" id="f${i}" data-label="Front ${i}"></div>
                <div class="card-slot" id="b${i}" data-label="Back ${i}"></div>
            `;
        }

        function loadImage(inputEle, slotId) {
            const file = inputEle.files[0];
            const slot = document.getElementById(slotId);
            if(file) {
                const url = URL.createObjectURL(file);
                slot.innerHTML = `<img src="${url}">`;
                slot.classList.add('filled');
            }
        }

        function clearRow(num) {
            document.querySelectorAll(`#row-${num} input`).forEach(i => i.value = "");
            const f = document.getElementById('f'+num);
            const b = document.getElementById('b'+num);
            [f, b].forEach(s => { s.innerHTML = ""; s.classList.remove('filled'); });
        }
    </script>
</body>
</html>
