<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Earth Wire Safety Simulation - Enhanced Contact</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            margin: 0;
        }

        h1 { color: #333; text-align: center; margin-bottom: 10px; }
        p { color: #666; max-width: 600px; text-align: center; margin-bottom: 20px; }

        .container {
            background: white;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            max-width: 800px;
            width: 100%;
        }

        canvas {
            background-color: #fff;
            border: 2px solid #ddd;
            border-radius: 8px;
            display: block;
            margin: 0 auto;
            width: 100%;
            max-width: 600px;
            height: 400px;
        }

        .controls {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: 20px;
            flex-wrap: wrap;
        }

        .control-group {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        button {
            padding: 12px 24px;
            font-size: 16px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.2s;
            font-weight: bold;
            margin-top: 5px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }

        button:active { transform: translateY(2px); }

        .btn-fault { background-color: #ff4d4d; color: white; }
        .btn-fault.active { background-color: #b30000; border: 2px solid #800000; }

        .btn-earth { background-color: #4CAF50; color: white; }
        .btn-earth.active { background-color: #1b5e20; border: 2px solid #0d3b10; }

        .status-panel {
            margin-top: 20px;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
            font-weight: bold;
            font-size: 1.1em;
            min-height: 60px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .status-safe { background-color: #e8f5e9; color: #2e7d32; border: 1px solid #c8e6c9; }
        .status-danger { 
            background-color: #ffebee; color: #c62828; border: 1px solid #ffcdd2; 
            animation: pulse 0.8s infinite;
        }
        .status-protected { background-color: #e3f2fd; color: #1565c0; border: 1px solid #bbdefb; }

        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(255, 0, 0, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(255, 0, 0, 0); }
            100% { box-shadow: 0 0 0 0 rgba(255, 0, 0, 0); }
        }

        .legend {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-top: 15px;
            font-size: 0.9em;
            flex-wrap: wrap;
            border-top: 1px solid #eee;
            padding-top: 10px;
        }
        .legend-item { display: flex; align-items: center; gap: 5px; }
        .dot { width: 12px; height: 12px; border-radius: 50%; }
    </style>
</head>
<body>

    <div class="container">
        <h1>Electrical Safety Simulator</h1>
        <p>Observe how current flows when a person touches a faulty appliance.</p>
        
        <canvas id="simCanvas" width="600" height="400"></canvas>

        <div class="status-panel status-safe" id="statusText">
            System Normal. Appliance is safe.
        </div>

        <div class="controls">
            <div class="control-group">
                <label>Step 1: Earth Wire</label>
                <button id="btnEarth" class="btn-earth active" onclick="toggleEarth()">Connected</button>
            </div>
            <div class="control-group">
                <label>Step 2: Electrical Fault</label>
                <button id="btnFault" class="btn-fault" onclick="toggleFault()">Create Leakage</button>
            </div>
        </div>

        <div class="legend">
            <div class="legend-item"><div class="dot" style="background: #8B4513;"></div> Live Wire</div>
            <div class="legend-item"><div class="dot" style="background: #32CD32;"></div> Earth Wire</div>
            <div class="legend-item"><div class="dot" style="background: #FFD700;"></div> Normal Current</div>
            <div class="legend-item"><div class="dot" style="background: red;"></div> Shock Current</div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('simCanvas');
        const ctx = canvas.getContext('2d');
        const statusText = document.getElementById('statusText');
        const btnEarth = document.getElementById('btnEarth');
        const btnFault = document.getElementById('btnFault');

        // State variables
        let isEarthConnected = true;
        let isFaultActive = false;
        
        // Electron arrays
        let electrons = [];
        let shockElectrons = [];
        let earthElectrons = [];

        // Coordinates for drawing
        const APPLIANCE = { x: 280, y: 120, w: 160, h: 160 };
        const PERSON = { x: 500, y: 200 }; // Head position
        const HAND = { x: 380, y: 150 }; // Where hand touches appliance

        // Animation loop
        function animate() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 1. Draw Static Elements
            drawGround();
            drawMains();
            drawAppliance();
            
            // 2. Draw Wires
            drawWires();

            // 3. Draw Person (Dynamic based on shock state)
            const isGettingShocked = isFaultActive && !isEarthConnected;
            drawPerson(isGettingShocked);

            // 4. Handle Electron Logic
            if (!isFaultActive) {
                animateNormalFlow();
            } else {
                if (isEarthConnected) {
                    animateEarthProtectionFlow();
                } else {
                    animateShockFlow();
                }
            }

            updateStatus();
            requestAnimationFrame(animate);
        }

        function drawMains() {
            ctx.fillStyle = '#eee';
            ctx.fillRect(40, 60, 80, 140);
            ctx.strokeStyle = '#999';
            ctx.strokeRect(40, 60, 80, 140);
            ctx.fillStyle = '#333';
            ctx.font = 'bold 14px Arial';
            ctx.fillText("Mains", 55, 50);
        }

        function drawAppliance() {
            // Metal Casing body
            ctx.fillStyle = '#B0C4DE'; // LightSteelBlue
            ctx.fillRect(APPLIANCE.x, APPLIANCE.y, APPLIANCE.w, APPLIANCE.h);
            
            // Casing Border
            ctx.strokeStyle = '#4682B4';
            ctx.lineWidth = 4;
            ctx.strokeRect(APPLIANCE.x, APPLIANCE.y, APPLIANCE.w, APPLIANCE.h);

            // Label
            ctx.fillStyle = '#333';
            ctx.font = 'bold 14px Arial';
            ctx.fillText("Metal Appliance", APPLIANCE.x + 25, APPLIANCE.y - 10);

            // Internal Motor/Heater symbol
            ctx.fillStyle = '#fff';
            ctx.fillRect(APPLIANCE.x + 60, APPLIANCE.y + 60, 40, 40);
            ctx.strokeRect(APPLIANCE.x + 60, APPLIANCE.y + 60, 40, 40);
        }

        function drawGround() {
            ctx.fillStyle = '#5D4037';
            ctx.fillRect(0, 350, 600, 50);
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 16px Arial';
            ctx.fillText("GROUND / EARTH", 20, 380);
        }

        function drawWires() {
            ctx.lineWidth = 4;

            // Neutral (Blue) - Bottom wire
            ctx.strokeStyle = 'blue';
            ctx.beginPath();
            ctx.moveTo(120, 180); // From Mains
            ctx.lineTo(APPLIANCE.x + 80, 180); // To internal load center
            ctx.stroke();

            // Live (Brown) - Middle wire
            ctx.strokeStyle = '#8B4513';
            ctx.beginPath();
            ctx.moveTo(120, 140); // From Mains
            
            if (isFaultActive) {
                // Fault: Wire touches the metal casing wall
                ctx.lineTo(APPLIANCE.x + 10, 140); 
                drawSpark(APPLIANCE.x + 10, 140);
            } else {
                // Normal: Wire goes to internal load
                ctx.lineTo(APPLIANCE.x + 80, 140);
            }
            ctx.stroke();

            // Internal Connection (Load)
            if (!isFaultActive) {
                ctx.strokeStyle = '#333';
                ctx.beginPath();
                ctx.moveTo(APPLIANCE.x + 80, 140);
                ctx.lineTo(APPLIANCE.x + 80, 180);
                ctx.stroke();
            }

            // Earth (Green) - Top wire
            if (isEarthConnected) {
                ctx.strokeStyle = '#32CD32';
                ctx.setLineDash([8, 4]); // Dashed for striped look
                ctx.beginPath();
                ctx.moveTo(120, 100); // From Mains
                ctx.lineTo(APPLIANCE.x + 40, 100); // To Casing Top
                ctx.stroke();
                
                // Screw connection symbol on casing
                ctx.fillStyle = '#32CD32';
                ctx.beginPath();
                ctx.arc(APPLIANCE.x + 40, 100, 5, 0, Math.PI*2);
                ctx.fill();
            } else {
                // Broken Earth
                ctx.strokeStyle = '#32CD32';
                ctx.setLineDash([8, 4]);
                ctx.beginPath();
                ctx.moveTo(120, 100);
                ctx.lineTo(200, 100); // Stops halfway
                ctx.stroke();
                
                // Red X
                ctx.setLineDash([]);
                ctx.strokeStyle = 'red';
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.moveTo(210, 90); ctx.lineTo(230, 110);
                ctx.moveTo(230, 90); ctx.lineTo(210, 110);
                ctx.stroke();
            }
            ctx.setLineDash([]); // Reset
        }

        function drawPerson(isShocked) {
            let shakeX = 0;
            let shakeY = 0;
            
            // Vibration effect if shocked
            if (isShocked) {
                shakeX = (Math.random() - 0.5) * 6;
                shakeY = (Math.random() - 0.5) * 6;
            }

            const px = PERSON.x + shakeX;
            const py = PERSON.y + shakeY;

            ctx.strokeStyle = '#333';
            ctx.lineWidth = 3;
            ctx.fillStyle = '#FFE0BD'; // Skin color

            // 1. Arm (Back arm - drawn first so it's behind body)
            // Not drawing back arm to keep it simple and clear

            // 2. Legs
            ctx.beginPath();
            ctx.moveTo(px, py + 80); // Hip
            ctx.lineTo(px - 20, 350); // Left foot on ground
            ctx.lineTo(px - 35, 350); // Foot
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(px, py + 80); // Hip
            ctx.lineTo(px + 20, 350); // Right foot on ground
            ctx.lineTo(px + 35, 350); // Foot
            ctx.stroke();

            // 3. Body/Torso
            ctx.fillStyle = '#1a237e'; // Blue shirt
            ctx.fillRect(px - 15, py + 20, 30, 70);

            // 4. Head
            ctx.beginPath();
            ctx.arc(px, py, 20, 0, Math.PI * 2);
            ctx.fillStyle = '#FFE0BD';
            ctx.fill();
            ctx.stroke();

            // Face
            ctx.beginPath();
            if (isShocked) {
                // Shocked face
                ctx.arc(px, py + 5, 8, 0, Math.PI * 2); // Open mouth
                ctx.fillStyle = '#000';
                ctx.fill();
                // Eyes
                ctx.fillText("x", px - 8, py - 2);
                ctx.fillText("x", px + 2, py - 2);
            } else {
                // Happy face
                ctx.arc(px, py + 5, 10, 0, Math.PI, false); // Smile
                ctx.stroke();
                // Eyes
                ctx.fillStyle = '#000';
                ctx.beginPath(); ctx.arc(px - 7, py - 5, 2, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.arc(px + 7, py - 5, 2, 0, Math.PI*2); ctx.fill();
            }

            // 5. The Arm touching the appliance (Crucial part)
            ctx.strokeStyle = '#FFE0BD'; // Skin arm
            ctx.lineWidth = 12; // Thick arm
            ctx.beginPath();
            ctx.moveTo(px - 10, py + 30); // Shoulder
            ctx.lineTo(HAND.x, HAND.y); // Hand position on machine
            ctx.stroke();

            // 6. The Hand (Circle ON the machine)
            ctx.fillStyle = '#FFE0BD';
            ctx.beginPath();
            ctx.arc(HAND.x, HAND.y, 12, 0, Math.PI * 2);
            ctx.fill();
            ctx.strokeStyle = '#d4a07d'; // Darker skin outline
            ctx.lineWidth = 2;
            ctx.stroke();

            // Label for the hand
            ctx.fillStyle = '#555';
            ctx.font = 'italic 12px Arial';
            ctx.fillText("Hand touching casing", HAND.x - 40, HAND.y - 20);
            
            // Arrow pointing to hand
            ctx.beginPath();
            ctx.moveTo(HAND.x + 10, HAND.y - 15);
            ctx.lineTo(HAND.x, HAND.y - 5);
            ctx.stroke();
        }

        function drawSpark(x, y) {
            ctx.fillStyle = '#FFD700';
            ctx.beginPath();
            const size = 5 + Math.random() * 10;
            ctx.arc(x, y, size, 0, Math.PI * 2);
            ctx.fill();
        }

        // --- Electron Logic ---

        function spawnElectron(type) {
            if (type === 'normal') {
                electrons.push({ x: 120, y: 140, pathStep: 0 });
            } else if (type === 'shock') {
                shockElectrons.push({ x: 120, y: 140, pathStep: 0 });
            } else if (type === 'earth') {
                earthElectrons.push({ x: 120, y: 140, pathStep: 0 });
            }
        }

        function drawElectron(e, color) {
            ctx.fillStyle = color;
            ctx.beginPath();
            ctx.arc(e.x, e.y, 5, 0, Math.PI * 2);
            ctx.fill();
        }

        function animateNormalFlow() {
            if (Math.random() < 0.15) spawnElectron('normal');

            for (let i = 0; i < electrons.length; i++) {
                let e = electrons[i];
                // Path: Mains(120,140) -> Load(360,140) -> Down(360,180) -> Back(120,180)
                
                if (e.y === 140 && e.x < APPLIANCE.x + 80) e.x += 3;
                else if (e.x >= APPLIANCE.x + 80 && e.y < 180) e.y += 3;
                else if (e.y >= 180 && e.x > 120) { e.y = 180; e.x -= 3; }

                drawElectron(e, '#FFD700');
                if (e.x <= 120 && e.y === 180) electrons.splice(i, 1);
            }
            shockElectrons = []; earthElectrons = [];
        }

        function animateShockFlow() {
            if (Math.random() < 0.3) spawnElectron('shock');

            for (let i = 0; i < shockElectrons.length; i++) {
                let e = shockElectrons[i];
                
                // 1. Mains to Fault Point
                if (e.pathStep === 0) {
                    e.x += 5;
                    if (e.x >= APPLIANCE.x + 10) e.pathStep = 1;
                }
                // 2. Across Metal Casing to Hand
                else if (e.pathStep === 1) {
                    // Interpolate towards hand
                    let dx = HAND.x - e.x;
                    let dy = HAND.y - e.y;
                    let dist = Math.sqrt(dx*dx + dy*dy);
                    e.x += (dx/dist) * 5;
                    e.y += (dy/dist) * 5;
                    if (dist < 5) e.pathStep = 2;
                }
                // 3. Hand to Shoulder
                else if (e.pathStep === 2) {
                    let shoulderX = PERSON.x - 10;
                    let shoulderY = PERSON.y + 30;
                    let dx = shoulderX - e.x;
                    let dy = shoulderY - e.y;
                    let dist = Math.sqrt(dx*dx + dy*dy);
                    e.x += (dx/dist) * 5;
                    e.y += (dy/dist) * 5;
                    if (dist < 5) e.pathStep = 3;
                }
                // 4. Down Body to Ground
                else if (e.pathStep === 3) {
                    e.y += 5; // Down torso/legs
                    // Slight drift to feet
                    if (e.y > 280) e.x -= 0.5; 
                    if (e.y > 350) shockElectrons.splice(i, 1);
                }

                drawElectron(e, 'red');
            }
            electrons = []; earthElectrons = [];
        }

        function animateEarthProtectionFlow() {
            if (Math.random() < 0.6) spawnElectron('earth'); // Fast flow

            for (let i = 0; i < earthElectrons.length; i++) {
                let e = earthElectrons[i];
                
                // 1. Mains to Fault
                if (e.y === 140 && e.x < APPLIANCE.x + 10) {
                    e.x += 10; // Very fast
                }
                // 2. Up Casing to Earth Connection
                else if (e.x >= APPLIANCE.x + 10 && e.y > 100) {
                    // Move diagonally up to earth screw
                    let targetX = APPLIANCE.x + 40;
                    let targetY = 100;
                    e.x += (targetX - e.x) * 0.2;
                    e.y += (targetY - e.y) * 0.2;
                    if (e.y <= 105) e.y = 100;
                }
                // 3. Back to Mains via Earth Wire
                else if (e.y <= 100 && e.x > 120) {
                    e.y = 100;
                    e.x -= 10;
                }

                drawElectron(e, '#32CD32');
                if (e.x <= 120 && e.y === 100) earthElectrons.splice(i, 1);
            }
            electrons = []; shockElectrons = [];
        }

        function updateStatus() {
            statusText.className = 'status-panel';
            
            if (!isFaultActive) {
                statusText.classList.add('status-safe');
                statusText.innerHTML = "System Normal.<br>Current flows safely inside the appliance.";
            } else {
                if (isEarthConnected) {
                    statusText.classList.add('status-protected');
                    statusText.innerHTML = "SAFE: EARTH WIRE ACTIVE.<br>Current takes the low-resistance path through the Earth wire, bypassing the person.";
                } else {
                    statusText.classList.add('status-danger');
                    statusText.innerHTML = "DANGER: ELECTRIC SHOCK!<br>Current flows from the casing, INTO THE HAND, through the body, to the ground.";
                }
            }
        }

        function toggleEarth() {
            isEarthConnected = !isEarthConnected;
            btnEarth.textContent = isEarthConnected ? "Connected" : "Disconnected";
            btnEarth.className = isEarthConnected ? "btn-earth active" : "btn-earth";
        }

        function toggleFault() {
            isFaultActive = !isFaultActive;
            btnFault.textContent = isFaultActive ? "Repair Leakage" : "Create Leakage";
            btnFault.className = isFaultActive ? "btn-fault active" : "btn-fault";
        }

        // Start
        animate();

    </script>
</body>
</html>
