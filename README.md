<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GRP Dirsek İmalat v5.0 (Pro)</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root {
            --bg-color: #f4f7f6;
            --panel-bg: #2c3e50;
            --panel-text: #ecf0f1;
            --accent: #e74c3c;
            --accent-hover: #c0392b;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            margin: 0; padding: 0; height: 100vh;
            display: flex; overflow: hidden;
        }

        /* SOL PANEL */
        .sidebar {
            width: 320px;
            background-color: var(--panel-bg);
            color: var(--panel-text);
            display: flex; flex-direction: column;
            padding: 20px;
            box-shadow: 4px 0 10px rgba(0,0,0,0.2);
            z-index: 100;
            overflow-y: auto;
        }

        .logo {
            font-size: 1.4rem; font-weight: bold; margin-bottom: 20px;
            border-bottom: 2px solid var(--accent); padding-bottom: 10px;
            display: flex; align-items: center; gap: 10px;
        }

        .input-group { margin-bottom: 15px; }
        .input-group label { display: block; font-size: 0.85rem; margin-bottom: 5px; color: #bdc3c7; }
        .input-group input, .input-group select {
            width: 100%; padding: 10px; border: none; border-radius: 4px;
            background: #34495e; color: white; font-size: 0.95rem;
        }
        .input-group input:focus { outline: 2px solid var(--accent); background: #3e5871; }

        .row { display: flex; gap: 10px; }
        .col { flex: 1; }

        button {
            width: 100%; padding: 12px; border: none; border-radius: 4px;
            cursor: pointer; font-size: 1rem; font-weight: bold;
            transition: 0.2s; margin-top: 10px; display: flex; align-items: center; justify-content: center; gap: 8px;
        }
        .btn-update { background-color: var(--accent); color: white; }
        .btn-update:hover { background-color: var(--accent-hover); }
        .btn-print { background-color: #95a5a6; color: white; margin-top: 20px; }
        .btn-print:hover { background-color: #7f8c8d; }

        /* SAĞ PANEL */
        .workspace {
            flex: 1; display: flex; justify-content: center; align-items: center;
            background-color: #e0e0e0; position: relative; padding: 40px;
        }

        #paper {
            background: white;
            box-shadow: 0 0 20px rgba(0,0,0,0.3);
            /* A4 Yatay Oranı */
            aspect-ratio: 297/210;
            width: 100%; max-width: 1400px;
            height: auto;
        }
        
        svg { width: 100%; height: 100%; }

        /* YAZDIRMA */
        @media print {
            @page { size: A4 landscape; margin: 0; }
            body { background: white; }
            .sidebar { display: none; }
            .workspace { padding: 0; background: white; height: 100vh; }
            #paper { box-shadow: none; width: 100%; height: 100%; max-width: none; }
        }
    </style>
</head>
<body>

<div class="sidebar">
    <div class="logo"><i class="fas fa-drafting-compass"></i> GRP Dirsek Çizim</div>
    
    <div class="input-group">
        <label>DN (Anma Çapı)</label>
        <input type="number" id="dn" value="600" onchange="draw()">
    </div>
    
    <div class="input-group">
        <label>Açı (Derece)</label>
        <input type="number" id="angle" value="90" min="1" max="179" onchange="draw()">
    </div>

    <div class="row">
        <div class="col input-group">
            <label>L1 Giriş (mm)</label>
            <input type="number" id="l1" value="2000" onchange="draw()">
        </div>
        <div class="col input-group">
            <label>Giriş Tipi</label>
            <select id="typeIn" onchange="draw()">
                <option value="flange">Flanş</option>
                <option value="coupling">Manşon</option>
                <option value="plain">Düz</option>
            </select>
        </div>
    </div>

    <div class="row">
        <div class="col input-group">
            <label>L2 Çıkış (mm)</label>
            <input type="number" id="l2" value="2000" onchange="draw()">
        </div>
        <div class="col input-group">
            <label>Çıkış Tipi</label>
            <select id="typeOut" onchange="draw()">
                <option value="flange">Flanş</option>
                <option value="coupling">Manşon</option>
                <option value="plain">Düz</option>
            </select>
        </div>
    </div>

    <button class="btn-update" onclick="draw()"><i class="fas fa-sync"></i> Güncelle</button>
    <button class="btn-print" onclick="window.print()"><i class="fas fa-print"></i> Yazdır / PDF</button>
</div>

<div class="workspace">
    <div id="paper">
        <svg id="svgCanvas" viewBox="0 0 1000 700"></svg>
    </div>
</div>

<script>
    const SVG_NS = "http://www.w3.org/2000/svg";

    function createSVG(tag, attrs) {
        const el = document.createElementNS(SVG_NS, tag);
        for (let k in attrs) el.setAttribute(k, attrs[k]);
        return el;
    }
    function toRad(deg) { return deg * Math.PI / 180; }
    function toDeg(rad) { return rad * 180 / Math.PI; }

    function draw() {
        const svg = document.getElementById('svgCanvas');
        svg.innerHTML = '';

        // 1. GİRDİLER
        const DN = parseFloat(document.getElementById('dn').value) || 600;
        const AngleDeg = parseFloat(document.getElementById('angle').value) || 90;
        const L1 = parseFloat(document.getElementById('l1').value) || 0;
        const L2 = parseFloat(document.getElementById('l2').value) || 0;
        const TypeIn = document.getElementById('typeIn').value;
        const TypeOut = document.getElementById('typeOut').value;
        
        // Segment Kuralı
        let Segments = 4;
        if(AngleDeg <= 31) Segments = 2;
        else if(AngleDeg <= 61) Segments = 3;

        const R_Factor = 1.5;
        const R = DN * R_Factor;
        const rOuter = R + DN/2;
        const rInner = R - DN/2;

        // 2. GEOMETRİ HESABI
        // Merkez (0,0). Giriş soldan (-L1) gelip (0,-R) noktasına bağlanır.
        // Başlangıç Açısı: 270 (-90) derece. Teğet: Yatay.
        // Dönüş: Saat yönü tersi (+).
        
        const startRad = -Math.PI / 2;
        const endRad = startRad + toRad(AngleDeg);

        let ptsCenter=[], ptsOuter=[], ptsInner=[];

        for(let i=0; i<=Segments; i++) {
            let t = i/Segments;
            let ang = startRad + t * (endRad - startRad);
            ptsCenter.push({x: Math.cos(ang)*R, y: Math.sin(ang)*R, ang: ang});
            ptsOuter.push({x: Math.cos(ang)*rOuter, y: Math.sin(ang)*rOuter});
            ptsInner.push({x: Math.cos(ang)*rInner, y: Math.sin(ang)*rInner});
        }

        // L1 (Giriş - Sola)
        const pInCenter = { x: ptsCenter[0].x - L1, y: ptsCenter[0].y };
        const pInOuter = { x: ptsOuter[0].x - L1, y: ptsOuter[0].y };
        const pInInner = { x: ptsInner[0].x - L1, y: ptsInner[0].y };

        // L2 (Çıkış - Teğet Yönü)
        const lastAng = ptsCenter[Segments].ang;
        const vx = -Math.sin(lastAng);
        const vy = Math.cos(lastAng);
        
        const pOutCenter = { x: ptsCenter[Segments].x + vx * L2, y: ptsCenter[Segments].y + vy * L2 };
        const pOutOuter = { x: ptsOuter[Segments].x + vx * L2, y: ptsOuter[Segments].y + vy * L2 };
        const pOutInner = { x: ptsInner[Segments].x + vx * L2, y: ptsInner[Segments].y + vy * L2 };

        // 3. BOYUTLANDIRMA VE FONT AYARI (KRİTİK)
        // Bounding Box
        let allX = [pInOuter.x, pOutOuter.x, 0];
        let allY = [pInOuter.y, pOutOuter.y, 0];
        ptsOuter.forEach(p => { allX.push(p.x); allY.push(p.y); });

        const minX = Math.min(...allX);
        const maxX = Math.max(...allX);
        const minY = Math.min(...allY);
        const maxY = Math.max(...allY);
        
        const drawW = maxX - minX;
        const drawH = maxY - minY;
        const maxDim = Math.max(drawW, drawH);

        // Dinamik Font Boyutu: Çizimin en büyük kenarının %2.5'i.
        // DN çok küçük olsa bile yazılar büyük kalmalı.
        const fs = Math.max(maxDim * 0.025, DN * 0.15); 
        const strokeW = fs * 0.08;

        // 4. TABLO YERLEŞİMİ (ÜST BOŞLUK)
        // "Üst Boşluk" genellikle çizimin sağ üst tarafıdır (Giriş solda, çıkış aşağıda ise).
        // Tabloyu maxX yakınına, minY hizasına koyalım.
        
        const tableW = Math.max(drawW * 0.35, fs * 25); // Tablo genişliğini artırdım
        const tableH = fs * 13;

        // Tablo Konumu: 
        // Eğer dirsek sağa/aşağı dönüyorsa boşluk Sağ-Üsttedir.
        // Tablonun X'i: Çizimin sağına hizalı veya biraz içeride.
        // Tablonun Y'si: Çizimin tepesiyle hizalı.
        
        let tableX = maxX - tableW;
        // Eğer çizim çok darsa tablo sola taşabilir, kontrol et
        if (tableX < minX + drawW/2) tableX = minX + drawW + fs*5; // Çizimin sağına at

        let tableY = minY;

        // ViewBox'ı Tabloyu da kapsayacak şekilde güncelle
        const finalMinX = Math.min(minX, tableX);
        const finalMaxX = Math.max(maxX, tableX + tableW);
        const finalMinY = Math.min(minY, tableY);
        const finalMaxY = Math.max(maxY, tableY + tableH);
        
        const margin = fs * 5;
        const vW = (finalMaxX - finalMinX) + margin*2;
        const vH = (finalMaxY - finalMinY) + margin*2;

        svg.setAttribute('viewBox', `${finalMinX - margin} ${finalMinY - margin} ${vW} ${vH}`);

        // 5. DEFS
        const defs = createSVG('defs', {});
        defs.innerHTML = `
            <marker id="arrow" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
                <path d="M0,0 L0,6 L6,3 z" fill="#000" />
            </marker>
             <pattern id="hatch" width="20" height="20" patternTransform="rotate(45 0 0)" patternUnits="userSpaceOnUse">
                <line x1="0" y1="0" x2="0" y2="20" style="stroke:#ccc; stroke-width:1" />
            </pattern>
        `;
        svg.appendChild(defs);

        // 6. ÇİZİM
        // Gövde
        let d = `M ${pInOuter.x} ${pInOuter.y}`; 
        d += ` L ${ptsOuter[0].x} ${ptsOuter[0].y}`;
        for(let i=1; i<=Segments; i++) d += ` L ${ptsOuter[i].x} ${ptsOuter[i].y}`;
        d += ` L ${pOutOuter.x} ${pOutOuter.y}`;
        d += ` L ${pOutInner.x} ${pOutInner.y}`;
        d += ` L ${ptsInner[Segments].x} ${ptsInner[Segments].y}`;
        for(let i=Segments-1; i>=0; i--) d += ` L ${ptsInner[i].x} ${ptsInner[i].y}`;
        d += ` L ${pInInner.x} ${pInInner.y}`;
        d += " Z";

        svg.appendChild(createSVG('path', {
            d: d, fill: "url(#hatch)", stroke: "black", "stroke-width": strokeW
        }));

        // Eksen
        let cd = `M ${pInCenter.x} ${pInCenter.y} L ${ptsCenter[0].x} ${ptsCenter[0].y}`;
        for(let i=1; i<=Segments; i++) cd += ` L ${ptsCenter[i].x} ${ptsCenter[i].y}`;
        cd += ` L ${pOutCenter.x} ${pOutCenter.y}`;
        svg.appendChild(createSVG('path', {
            d: cd, fill: "none", stroke: "red", "stroke-dasharray": `${fs},${fs/2}`, "stroke-width": strokeW*0.6
        }));

        // Kaynak Çizgileri
        for(let i=0; i<=Segments; i++) {
            svg.appendChild(createSVG('line', {
                x1: ptsOuter[i].x, y1: ptsOuter[i].y, x2: ptsInner[i].x, y2: ptsInner[i].y,
                stroke: "#444", "stroke-width": strokeW*0.5
            }));
        }

        // Flanş/Manşon
        function drawFitting(cx, cy, ang, type, isStart) {
            if(type === 'plain') return;
            const g = createSVG('g', {transform: `translate(${cx},${cy}) rotate(${toDeg(ang)})`});
            const fw = Math.max(DN*0.15, fs*0.8);
            const fh = DN*1.45;
            const xOff = isStart ? -fw : 0;

            if(type === 'flange') {
                g.appendChild(createSVG('rect', {
                    x: xOff, y: -fh/2, width: fw, height: fh,
                    fill: "#ecf0f1", stroke: "black", "stroke-width": strokeW
                }));
            } else if(type === 'coupling') {
                g.appendChild(createSVG('rect', {
                    x: -DN*0.3, y: -DN*0.6, width: DN*0.6, height: DN*1.2,
                    fill: "#3498db", "fill-opacity":"0.3", stroke: "black", "stroke-width": strokeW
                }));
            }
            svg.appendChild(g);
        }
        drawFitting(pInCenter.x, pInCenter.y, 0, TypeIn, true);
        drawFitting(pOutCenter.x, pOutCenter.y, lastAng + Math.PI/2, TypeOut, false);

        // 7. ÖLÇÜLENDİRME (DAHA OKUNAKLI)
        function dim(x1, y1, x2, y2, txt, offset, color="black", isBold=false) {
            const dx = x2-x1, dy = y2-y1;
            const len = Math.sqrt(dx*dx+dy*dy);
            const nx = -dy/len, ny = dx/len;
            const off = fs * 3 * offset; // DN yerine Font Size'a göre offset veriyoruz (Çakışmayı önler)
            
            const ox1 = x1+nx*off, oy1 = y1+ny*off;
            const ox2 = x2+nx*off, oy2 = y2+ny*off;
            
            const g = createSVG('g', {});
            // İnce referans çizgileri
            g.appendChild(createSVG('line', {x1:x1,y1:y1,x2:ox1,y2:oy1, stroke:"#7f8c8d", "stroke-width":strokeW/3}));
            g.appendChild(createSVG('line', {x1:x2,y1:y2,x2:ox2,y2:oy2, stroke:"#7f8c8d", "stroke-width":strokeW/3}));
            // Ana ok çizgisi
            g.appendChild(createSVG('line', {
                x1:ox1,y1:oy1,x2:ox2,y2:oy2, stroke:color, "stroke-width":strokeW*0.8,
                "marker-start":"url(#arrow)", "marker-end":"url(#arrow)"
            }));
            
            const mx = (ox1+ox2)/2, my = (oy1+oy2)/2;
            let ang = Math.atan2(dy,dx)*180/Math.PI;
            if(ang>90 || ang<-90) ang+=180;
            
            // Arka plan (Okunabilirlik için beyaz kontur)
            const t = createSVG('text', {
                x:mx, y:my, "text-anchor":"middle", "dy":-fs*0.4,
                "font-size":fs, "font-family":"Arial", "font-weight": isBold?"bold":"normal", fill:color,
                transform:`rotate(${ang},${mx},${my})`
            });
            t.textContent = txt;
            g.appendChild(t);
            svg.appendChild(g);
        }

        if(L1>0) dim(pInCenter.x, pInCenter.y, ptsCenter[0].x, ptsCenter[0].y, `L1=${L1}`, 1.5, "black", true);
        if(L2>0) dim(ptsCenter[Segments].x, ptsCenter[Segments].y, pOutCenter.x, pOutCenter.y, `L2=${L2}`, -1.5, "black", true);

        // Çap
        dim(pInOuter.x, pInOuter.y, pInInner.x, pInInner.y, `DN${DN}`, 0.8, "blue", true);
        
        // Açı
        svg.appendChild(createSVG('text', {
            x: 0, y: R*0.3, "text-anchor":"middle", "font-size":fs*1.5, fill:"#2980b9", "font-weight":"bold"
        })).textContent = `${AngleDeg}°`;

        // 8. BÜYÜK VE ÜSTTE TABLO
        const tG = createSVG('g', {transform: `translate(${tableX}, ${tableY})`});
        
        // Arka plan
        tG.appendChild(createSVG('rect', {
            x:0, y:0, width:tableW, height:tableH, fill:"white", stroke:"black", "stroke-width":strokeW*1.5
        }));

        const headerH = fs * 2.2;
        const rowH = fs * 1.6;

        // Başlık
        tG.appendChild(createSVG('rect', {x:0, y:0, width:tableW, height:headerH, fill:"#ecf0f1", stroke:"black", "stroke-width":strokeW}));
        const title = createSVG('text', {
            x:tableW/2, y:headerH/2, "text-anchor":"middle", "dy":fs*0.3,
            "font-size":fs*1.1, "font-weight":"bold", "font-family":"Arial"
        });
        title.textContent = "GRP İMALAT PLANI";
        tG.appendChild(title);

        const rows = [
            ["Parametre", "Değer"],
            ["Anma Çapı", `DN ${DN}`],
            ["Açı / Yarıçap", `${AngleDeg}° / R=${(DN*R_Factor).toFixed(0)}`],
            ["Segment", `${Segments} Parça`],
            ["Toplam L", `${(L1+L2+(toRad(AngleDeg)*R)).toFixed(0)} mm`],
            ["Malzeme/Basınç", "GRP / PN10"],
            ["Tarih", new Date().toLocaleDateString("tr-TR")]
        ];

        let curY = headerH;
        rows.forEach((row, i) => {
            tG.appendChild(createSVG('line', {x1:0, y1:curY, x2:tableW, y2:curY, stroke:"black", "stroke-width":strokeW/0.8}));
            
            // Satır Arka Planı (Opsiyonel okunabilirlik)
            if(i%2===0 && i>0) tG.appendChild(createSVG('rect', {x:0, y:curY, width:tableW, height:rowH, fill:"#f9f9f9", "fill-opacity":"0.5"}));

            const isHead = i===0;
            const fontS = isHead ? fs*0.9 : fs;
            const weight = isHead ? "bold" : "normal";
            
            const t1 = createSVG('text', {x:fs, y:curY+rowH*0.7, "font-size":fontS, "font-weight":weight, "font-family":"Arial"});
            t1.textContent = row[0];
            tG.appendChild(t1);
            
            const t2 = createSVG('text', {x:tableW*0.5, y:curY+rowH*0.7, "font-size":fontS, "font-weight":weight, "font-family":"Arial"});
            t2.textContent = row[1];
            tG.appendChild(t2);

            curY += rowH;
        });
        
        // Dikey Ayraç
        tG.appendChild(createSVG('line', {x1:tableW*0.48, y1:headerH, x2:tableW*0.48, y2:tableH, stroke:"black", "stroke-width":strokeW/2}));

        svg.appendChild(tG);
    }

    window.onload = draw;
</script>

</body>
</html>
