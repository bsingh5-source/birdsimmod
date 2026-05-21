<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LS Investigation: For the Birds Automated Simulator</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            color: #333;
            padding: 20px;
            max-width: 1000px;
            margin: 0 auto;
        }
        h1 {
            color: #2c3e50;
            border-bottom: 2px solid #34495e;
            padding-bottom: 10px;
            margin-bottom: 5px;
        }
        p { margin-top: 5px; color: #555; }
        
        .container {
            display: flex;
            gap: 20px;
            flex-wrap: wrap;
            margin-top: 20px;
        }
        .left-panel {
            flex: 1.2;
            min-width: 400px;
        }
        .right-panel {
            background-color: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            align-items: center;
            min-width: 340px;
            flex: 0.8;
        }
        
        .setup-box, .results-box, .history-box {
            background-color: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        label {
            font-weight: bold;
            display: block;
            margin-top: 15px;
            margin-bottom: 5px;
        }
        select, input, button {
            width: 100%;
            padding: 12px;
            font-size: 16px;
            border-radius: 4px;
            border: 1px solid #ccc;
            box-sizing: border-box;
        }
        button {
            background-color: #27ae60;
            color: white;
            font-weight: bold;
            border: none;
            margin-top: 20px;
            cursor: pointer;
            transition: background 0.2s, transform 0.1s;
        }
        button:hover { background-color: #219653; }
        button:disabled { background-color: #bdc3c7; cursor: not-allowed; }
        
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 10px;
            text-align: center;
            font-size: 14px;
        }
        th { background-color: #ecf0f1; color: #2c3e50; }
        
        .status {
            font-weight: bold;
            font-size: 15px;
            margin-top: 15px;
            padding: 10px;
            border-radius: 4px;
            text-align: center;
        }
        .success { background-color: #d4edda; color: #155724; border-left: 5px solid #28a745; }
        .fail { background-color: #f8d7da; color: #721c24; border-left: 5px solid #dc3545; }
        
        canvas {
            border: 2px solid #34495e;
            background-color: #eef2f3;
            border-radius: 4px;
        }
        .history-table-wrapper {
            max-height: 250px;
            overflow-y: auto;
            margin-top: 10px;
        }
    </style>
</head>
<body>

    <h1>LS Investigation — For the Birds (Engineering Simulator)</h1>
    <p>Test your safety designs. Click "Run 3 Trials" to automatically drop the object three consecutive times and gather calculations.</p>

    <div class="container">
        <div class="left-panel">
            <div class="setup-box">
                <h3>[Experimental Configurations]</h3>
                
                <label for="birdSelect">Select Target Organism (Table 1):</label>
                <select id="birdSelect" onchange="onSettingsChange()">
                    <option value="chickadee">Black-capped Chickadee (12g | Drop Height: 1.0m)</option>
                    <option value="cardinal">Northern Cardinal (45g | Drop Height: 1.2m)</option>
                    <option value="robin">American Robin (75g | Drop Height: 1.5m)</option>
                </select>

                <div id="physicsReadout" style="margin-top:8px; color:#555; font-family: monospace; font-size: 13px; background: #eef2f3; padding: 8px; border-radius: 4px;">
                    Matrix initializing...
                </div>

                <label for="meshHeight">Independent Variable - Height of mesh from floor (cm):</label>
                <input type="number" id="meshHeight" min="0" max="25" step="0.5" value="0" oninput="onSettingsChange()">

                <button id="runBtn" onclick="startAutomatedPipeline()">Run 3 Trials</button>
            </div>

            <div id="results" class="results-box" style="display: none;">
                <h3>Current Snapshot Data Log</h3>
                <table>
                    <thead>
                        <tr>
                            <th style="width: 22%;">Trial 1</th>
                            <th style="width: 22%;">Trial 2</th>
                            <th style="width: 22%;">Trial 3</th>
                            <th style="font-weight: bold;">Average Deformation</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td id="t1">-</td>
                            <td id="t2">-</td>
                            <td id="t3">-</td>
                            <td id="tAvg" style="font-weight:bold; color: #2c3e50;">-</td>
                        </tr>
                    </tbody>
                </table>
                <div id="statusMessage" class="status"></div>
            </div>
        </div>

        <div class="right-panel">
            <h3 style="margin-top: 0; margin-bottom: 10px;">Drop Zone Visualizer</h3>
            <canvas id="simCanvas" width="340" height="460"></canvas>
            <div style="margin-top: 10px; font-size: 12px; color: #666; text-align: center; line-height: 1.4;">
                <strong>Measuring Tape Scale:</strong> Objects drop dynamically from their true packet scale height (100cm, 120cm, or 150cm marks).
            </div>
        </div>
    </div>

    <div class="history-box">
        <h3 style="margin-top: 0;">Global Experimental History Log</h3>
        <p style="font-size: 13px; color: #666; margin-bottom: 10px;">Use this ledger to track, cross-reference, and compare trends across multiple configurations.</p>
        <div class="history-table-wrapper">
            <table>
                <thead>
                    <tr>
                        <th>Run</th>
                        <th>Bird Species</th>
                        <th>Mass</th>
                        <th>Drop Ht.</th>
                        <th>Mesh Ht.</th>
                        <th>Trial 1</th>
                        <th>Trial 2</th>
                        <th>Trial 3</th>
                        <th>Avg. Def.</th>
                        <th>Status Outcome</th>
                    </tr>
                </thead>
                <tbody id="historyBody">
                    <tr>
                        <td colspan="10" style="color: #95a5a6; font-style: italic; padding: 20px; text-align: center;">No history logs saved yet. Run a complete 3-trial set to drop a snapshot.</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>

    <script>
        const canvas = document.getElementById("simCanvas");
        const ctx = canvas.getContext("2d");

        // Real-world Variable Metrics Matrix
        const BIRD_DATA = {
            chickadee: { name: "Black-capped Chickadee", mass: 12, dropHeight: 1.0, minMesh: 6.0, maxDef: 2.5, color: "#e67e22", radius: 9 },
            cardinal: { name: "Northern Cardinal", mass: 45, dropHeight: 1.2, minMesh: 10.0, maxDef: 4.2, color: "#e74c3c", radius: 13 },
            robin: { name: "American Robin", mass: 75, dropHeight: 1.5, minMesh: 15.0, maxDef: 5.8, color: "#3498db", radius: 16 }
        };

        // UI Canvas Metrics
        const FLOOR_Y = 410;
        const RULER_X = 50; 
        const SCALE = 2.3; // 2.3 pixels per cm of height space

        // Operational Environment States
        let animationId = null;
        let isAnimating = false;
        let currentTrialIndex = 0;
        let simulatedTrials = [0, 0, 0];
        let globalRunCounter = 1;

        // Kinematic Properties
        let birdY = 0;
        let birdVelocity = 0;
        let currentMeshStretch = 0;

        // Execute background template on initial window boot
        onSettingsChange();

        function onSettingsChange() {
            if (isAnimating) return;

            const birdKey = document.getElementById("birdSelect").value;
            const bird = BIRD_DATA[birdKey];
            
            let meshHeight = parseFloat(document.getElementById("meshHeight").value);
            if (isNaN(meshHeight) || meshHeight < 0) meshHeight = 0;
            if (meshHeight > 25) { meshHeight = 25; document.getElementById("meshHeight").value = 25; }

            // Energy readout presentation calculation (Joules = m * g * h)
            const energyJoules = ((bird.mass / 1000) * 9.8 * bird.dropHeight).toFixed(4);
            document.getElementById("physicsReadout").innerHTML = 
                `ENERGY DISPLACEMENT: ${energyJoules} Joules | REQ CLEARANCE: ${bird.minMesh} cm`;

            // Position ball at its correct scaled physical height marker
            const startHeightCm = bird.dropHeight * 100;
            const initialBirdY = FLOOR_Y - (startHeightCm * SCALE);
            
            drawCanvas(meshHeight, bird, initialBirdY, 0);
        }

        // Rigorous non-linear physics parsing engine
        function calculatePhysicalDeformation(meshHeight, bird) {
            if (meshHeight >= bird.minMesh) {
                return 0.0;
            } else {
                let fractionOfImpact = 1.0 - (meshHeight / bird.minMesh);
                // Non-linear deformation scaling with energy square roots
                let baseDef = bird.maxDef * Math.sqrt(fractionOfImpact);
                
                const varianceOptions = [-0.2, -0.1, 0.0, 0.1, 0.2];
                let randomVariance = varianceOptions[Math.floor(Math.random() * varianceOptions.length)];
                return Math.max(0, Math.round((baseDef + randomVariance) * 10) / 10);
            }
        }

        // Automated execution control orchestrator
        function startAutomatedPipeline() {
            let meshHeight = parseFloat(document.getElementById("meshHeight").value);
            if (isNaN(meshHeight) || meshHeight < 0) meshHeight = 0;
            if (meshHeight > 25) meshHeight = 25;

            const birdKey = document.getElementById("birdSelect").value;
            const bird = BIRD_DATA[birdKey];

            // Lock input channels completely during calculation runs
            isAnimating = true;
            toggleControls(true);

            // Pre-calculate array values for all 3 trials
            simulatedTrials = [
                calculatePhysicalDeformation(meshHeight, bird),
                calculatePhysicalDeformation(meshHeight, bird),
                calculatePhysicalDeformation(meshHeight, bird)
            ];

            // Reset dashboard fields
            document.getElementById("t1").innerText = "-";
            document.getElementById("t2").innerText = "-";
            document.getElementById("t3").innerText = "-";
            document.getElementById("tAvg").innerText = "-";
            document.getElementById("statusMessage").style.display = "none";
            document.getElementById("results").style.display = "block";

            currentTrialIndex = 0;
            executeSingleDropStep(meshHeight, bird);
        }

        function executeSingleDropStep(meshHeight, bird) {
            const startHeightCm = bird.dropHeight * 100;
            birdY = FLOOR_Y - (startHeightCm * SCALE);
            birdVelocity = 0;
            currentMeshStretch = 0;
            let netImpactRegistered = false;

            let expectedDeformation = simulatedTrials[currentTrialIndex];
            const gravityStep = 0.25; // Continuous frame acceleration rate

            function animationLoop() {
                let meshBaseY = FLOOR_Y - (meshHeight * SCALE);

                if (!netImpactRegistered) {
                    birdVelocity += gravityStep;
                    birdY += birdVelocity;

                    if (meshHeight > 0 && birdY >= meshBaseY) {
                        netImpactRegistered = true;
                    } else if (birdY >= FLOOR_Y - bird.radius) {
                        birdY = FLOOR_Y - bird.radius;
                        endSingleTrial();
                        return;
                    }
                } else {
                    // Contact phase: compute net strain displacement vectors
                    birdY += birdVelocity;
                    currentMeshStretch = birdY - meshBaseY;

                    if (expectedDeformation === 0) {
                        birdVelocity *= 0.65; // Net mesh damping friction resistance
                        if (birdVelocity < 0.3) {
                            birdVelocity = 0;
                            setTimeout(endSingleTrial, 300);
                            return;
                        }
                    } else {
                        // Structural collapse tracking path
                        if (birdY >= FLOOR_Y - bird.radius) {
                            birdY = FLOOR_Y - bird.radius;
                            currentMeshStretch = FLOOR_Y - meshBaseY;
                            endSingleTrial();
                            return;
                        }
                    }
                }

                drawCanvas(meshHeight, bird, birdY, currentMeshStretch);
                animationId = requestAnimationFrame(animationLoop);
            }

            function endSingleTrial() {
                cancelAnimationFrame(animationId);
                
                // Write active data instantly to current trial UI column
                document.getElementById(`t${currentTrialIndex + 1}`).innerText = expectedDeformation + " cm";
                currentTrialIndex++;

                if (currentTrialIndex < 3) {
                    // Sequence pause buffer before triggering next consecutive automated drop
                    setTimeout(() => {
                        executeSingleDropStep(meshHeight, bird);
                    }, 500);
                } else {
                    // Wrap execution stack arrays completely
                    finalizeDataSet(meshHeight, bird);
                }
            }

            animationLoop();
        }

        function finalizeDataSet(meshHeight, bird) {
            isAnimating = false;
            toggleControls(false);

            let t1 = simulatedTrials[0];
            let t2 = simulatedTrials[1];
            let t3 = simulatedTrials[2];
            let avg = Math.round(((t1 + t2 + t3) / 3) * 10) / 10;

            document.getElementById("tAvg").innerText = avg + " cm";

            const statusDiv = document.getElementById("statusMessage");
            if (avg === 0) {
                statusDiv.innerText = "SUCCESS: Safe deployment clearance window preserved. Zero floor impacts.";
                statusDiv.className = "status success";
            } else {
                statusDiv.innerText = `CRITICAL DEFORMATION: Insufficient declaration height. Floor collision footprint calculated.`;
                statusDiv.className = "status fail";
            }
            statusDiv.style.display = "block";

            // Push snapshots to history matrix database structure
            commitToHistoryLog(bird, meshHeight, t1, t2, t3, avg);
            onSettingsChange();
        }

        function commitToHistoryLog(bird, meshHeight, t1, t2, t3, avg) {
            const body = document.getElementById("historyBody");
            if (globalRunCounter === 1) body.innerHTML = ""; // purge default string rows

            const row = document.createElement("tr");
            const wasSafe = (avg === 0);
            row.style.backgroundColor = wasSafe ? "#f3faf5" : "#fff9f9";

            row.innerHTML = `
                <td><strong>#${globalRunCounter}</strong></td>
                <td>${bird.name}</td>
                <td>${bird.mass}g</td>
                <td>${bird.dropHeight}m</td>
                <td>${meshHeight} cm</td>
                <td>${t1} cm</td>
                <td>${t2} cm</td>
                <td>${t3} cm</td>
                <td style="font-weight:bold;">${avg} cm</td>
                <td style="font-weight:bold; color: ${wasSafe ? '#1e7e34' : '#bd2130'}">${wasSafe ? 'SURVIVED' : 'MORTALITY'}</td>
            `;
            
            body.insertBefore(row, body.firstChild);
            globalRunCounter++;
        }

        function drawCanvas(meshHeight, bird, currentBirdY, currentStretch) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 1. Draw Impact Platform Base
            ctx.fillStyle = "#95a5a6";
            ctx.fillRect(0, FLOOR_Y, canvas.width, canvas.height - FLOOR_Y);
            ctx.fillStyle = "#34495e";
            ctx.fillRect(0, FLOOR_Y, canvas.width, 5); // Glass layer split mark

            // 2. Draw Measuring Tape (0cm to 160cm scaled index ruler)
            ctx.strokeStyle = "#7f8c8d";
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(RULER_X, 20);
            ctx.lineTo(RULER_X, FLOOR_Y);
            ctx.stroke();

            for (let cm = 0; cm <= 160; cm += 10) {
                let yTick = FLOOR_Y - (cm * SCALE);
                if (yTick < 20) break;

                ctx.beginPath();
                if (cm % 20 === 0) {
                    ctx.moveTo(RULER_X - 15, yTick);
                    ctx.lineTo(RULER_X, yTick);
                    ctx.strokeStyle = "#2c3e50";
                    ctx.lineWidth = 2;
                    ctx.stroke();
                    
                    ctx.fillStyle = "#2c3e50";
                    ctx.font = "bold 10px Arial";
                    ctx.fillText(cm + " cm", RULER_X - 44, yTick + 3);
                } else {
                    ctx.moveTo(RULER_X - 8, yTick);
                    ctx.lineTo(RULER_X, yTick);
                    ctx.strokeStyle = "#bdc3c7";
                    ctx.lineWidth = 1;
                    ctx.stroke();
                }
            }

            // 3. Draw Protective Safety Weave Netting
            let meshBaseY = FLOOR_Y - (meshHeight * SCALE);
            if (meshHeight > 0) {
                let midX = (canvas.width + RULER_X) / 2;
                let currentMidY = meshBaseY + currentStretch;

                ctx.strokeStyle = "#d35400";
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.moveTo(RULER_X + 5, meshBaseY);
                ctx.lineTo(midX, currentMidY);
                ctx.lineTo(canvas.width - 10, meshBaseY);
                ctx.stroke();

                // Crosshatched safety appearance lines
                ctx.strokeStyle = "rgba(211, 84, 0, 0.2)";
                ctx.lineWidth = 1;
                for (let offset = -14; offset <= 14; offset += 7) {
                    ctx.beginPath();
                    ctx.moveTo(RULER_X + 5, meshBaseY + offset);
                    ctx.lineTo(midX, currentMidY + offset);
                    ctx.lineTo(canvas.width - 10, meshBaseY + offset);
                    ctx.stroke();
                }
            }

            // 4. Draw Scaled Ball Object (Organism payload simulation)
            let birdX = (canvas.width + RULER_X) / 2;
            ctx.fillStyle = bird.color;
            ctx.beginPath();
            ctx.arc(birdX, currentBirdY, bird.radius, 0, Math.PI * 2);
            ctx.fill();
            ctx.strokeStyle = "#2c3e50";
            ctx.lineWidth = 2;
            ctx.stroke();

            // Simple vector directional eye feature
            ctx.fillStyle = "#fff";
            ctx.beginPath();
            ctx.arc(birdX + (bird.radius * 0.3), currentBirdY + (bird.radius * 0.1), bird.radius * 0.2, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = "#000";
            ctx.beginPath();
            ctx.arc(birdX + (bird.radius * 0.3), currentBirdY + (bird.radius * 0.1), bird.radius * 0.08, 0, Math.PI * 2);
            ctx.fill();

            // Workspace surface labeling
            ctx.fillStyle = "#fff";
            ctx.font = "bold 11px Arial";
            ctx.fillText("CLAY PACKET DEFORMATION PLATFORM", RULER_X + 25, FLOOR_Y + 25);
        }

        function toggleControls(setting) {
            document.getElementById("runBtn").disabled = setting;
            document.getElementById("birdSelect").disabled = setting;
            document.getElementById("meshHeight").disabled = setting;
        }
    </script>
</body>
</html>
