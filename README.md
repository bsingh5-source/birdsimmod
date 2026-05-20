<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LS Investigation: For the Birds Engineering Simulator</title>
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
            background-color: #2980b9;
            color: white;
            font-weight: bold;
            border: none;
            margin-top: 20px;
            cursor: pointer;
            transition: background 0.2s, transform 0.1s;
        }
        button:hover { background-color: #2471a3; }
        button:active { transform: scale(0.98); }
        button:disabled { background-color: #bdc3c7; cursor: not-allowed; transform: none; }
        
        .btn-run { background-color: #27ae60; }
        .btn-run:hover { background-color: #219653; }
        
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
    <p>A mathematically rigorous physics model tracking Kinetic Energy, Mesh Negative Work, and Impact Deflection.</p>

    <div class="container">
        <div class="left-panel">
            <div class="setup-box">
                <h3>[Experimental Configurations]</h3>
                
                <label for="birdSelect">Select Target Organism (Table 1):</label>
                <select id="birdSelect" onchange="handleVariableChange()">
                    <option value="chickadee">Black-capped Chickadee (12g | Drop Height: 1.0m)</option>
                    <option value="cardinal">Northern Cardinal (45g | Drop Height: 1.2m)</option>
                    <option value="robin">American Robin (75g | Drop Height: 1.5m)</option>
                </select>

                <div id="physicsReadout" style="margin-top:8px; color:#555; font-family: monospace; font-size: 13px; background: #eef2f3; padding: 8px; border-radius: 4px;">
                    Loading System Matrix...
                </div>

                <label for="meshHeight">Independent Variable - Height of mesh from floor (cm):</label>
                <input type="number" id="meshHeight" min="0" max="25" step="0.5" value="0" oninput="handleVariableChange()">

                <button id="actionBtn" class="btn-run" onclick="triggerTrialExecution()">Run Trial 1</button>
            </div>

            <div id="results" class="results-box">
                <h3>Current Configuration Matrix</h3>
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
                <div id="statusMessage" class="status" style="display:none;"></div>
            </div>
        </div>

        <div class="right-panel">
            <h3 style="margin-top: 0; margin-bottom: 10px;">Drop Zone Visualizer</h3>
            <canvas id="simCanvas" width="340" height="460"></canvas>
            <div style="margin-top: 10px; font-size: 12px; color: #666; text-align: center; line-height: 1.4;">
                <strong>Measuring Tape Scale:</strong> Calibrated in Centimeters (cm) from the impact floor floor up to a maximum 160 cm window clearance limit.
            </div>
        </div>
    </div>

    <div class="history-box">
        <h3 style="margin-top: 0;">Global Experimental History Log</h3>
        <p style="font-size: 13px; color: #666; margin-bottom: 10px;">Use this data ledger to compare parameters across different sizes and configurations to fulfill Packet Questions 3, 9, and 10.</p>
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
                        <td colspan="10" style="color: #95a5a6; font-style: italic; padding: 20px;">No historical data records logged yet. Run 3 complete trials to commit a snapshot.</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>

    <script>
        const canvas = document.getElementById("simCanvas");
        const ctx = canvas.getContext("2d");

        // Rigorous Engineering Database
        const BIRD_DATA = {
            chickadee: { name: "Black-capped Chickadee", mass: 12, dropHeight: 1.0, minMesh: 6.0, maxDef: 2.5, color: "#e67e22", radius: 9 },
            cardinal: { name: "Northern Cardinal", mass: 45, dropHeight: 1.2, minMesh: 10.0, maxDef: 4.2, color: "#e74c3c", radius: 13 },
            robin: { name: "American Robin", mass: 75, dropHeight: 1.5, minMesh: 15.0, maxDef: 5.8, color: "#3498db", radius: 16 }
        };

        // UI Canvas Metrics
        const FLOOR_Y = 420;
        const RULER_X = 50; 
        const SCALE = 2.3; // 2.3 pixels per cm of vertical clearance height

        // State Machine Tracking Variables
        let currentTrialIndex = 0; // Tracks manual sequence (0, 1, 2)
        let simulatedTrials = [null, null, null];
        let isAnimating = false;
        let globalRunCounter = 1;

        // Kinematic/Animation variables
        let birdY = 0;
        let birdVelocity = 0;
        let currentMeshStretch = 0;
        let animationId = null;

        // Initial setup sequence
        handleVariableChange();

        // 1. ARCHITECTURE MODEL: True Physics & Material Work Energy Calculations
        function calculatePhysicalDeformation(meshHeight, bird) {
            // KE = mass (kg) * g * height (m)
            const initialKineticEnergy = (bird.mass / 1000) * 9.8 * bird.dropHeight; 
            
            // Negative Work performed by mesh tension scaling non-linearly with deployment clearance height
            const workCapacity = initialKineticEnergy * (meshHeight / bird.minMesh);
            const residualEnergy = Math.max(0, initialKineticEnergy - workCapacity);
            
            if (residualEnergy <= 0 || meshHeight >= bird.minMesh) {
                return 0.0;
            }

            // Material deflection is proportional to the square root of terminal energy displacement
            let calculatedDeformation = bird.maxDef * Math.sqrt(residualEnergy / initialKineticEnergy);
            
            // Inject structural laboratory variance modeling
            const varianceOptions = [-0.2, -0.1, 0.0, 0.1, 0.2];
            let randomVariance = varianceOptions[Math.floor(Math.random() * varianceOptions.length)];
            
            return Math.max(0, Math.round((calculatedDeformation + randomVariance) * 10) / 10);
        }

        // 2. CONTROLLER: State Protection & Input Reading
        function handleVariableChange() {
            if (isAnimating) return;

            const birdKey = document.getElementById("birdSelect").value;
            const bird = BIRD_DATA[birdKey];
            
            let meshHeight = parseFloat(document.getElementById("meshHeight").value);
            if (isNaN(meshHeight) || meshHeight < 0) meshHeight = 0;
            if (meshHeight > 25) { meshHeight = 25; document.getElementById("meshHeight").value = 25; }

            // Dynamic Physics Matrix Readout
            const energyJoules = ((bird.mass / 1000) * 9.8 * bird.dropHeight).toFixed(4);
            document.getElementById("physicsReadout").innerHTML = 
                `SYS ENERGY (KE): ${energyJoules} J | APP REQ: ${bird.minMesh} cm Clear Window`;

            // State Pollution Prevention: If mid-experiment and configuration parameters change, wipe and reset local cache
            if (currentTrialIndex > 0) {
                currentTrialIndex = 0;
                simulatedTrials = [null, null, null];
                document.getElementById("t1").innerText = "-";
                document.getElementById("t2").innerText = "-";
                document.getElementById("t3").innerText = "-";
                document.getElementById("tAvg").innerText = "-";
                document.getElementById("statusMessage").style.display = "none";
            }
            
            document.getElementById("actionBtn").innerText = `Run Trial 1`;
            document.getElementById("actionBtn").disabled = false;

            // Render scaled baseline position inside canvas view
            const startHeightCm = bird.dropHeight * 100; // translate meters to cm scale
            const initialBirdY = FLOOR_Y - (startHeightCm * SCALE);
            drawCanvas(meshHeight, bird, initialBirdY, 0);
        }

        // 3. CONTROLLER: Manual Step Execution Machine
        function triggerTrialExecution() {
            if (isAnimating) return;

            const birdKey = document.getElementById("birdSelect").value;
            const bird = BIRD_DATA[birdKey];
            const meshHeight = parseFloat(document.getElementById("meshHeight").value) || 0;

            // Calculate the immutable result snapshot for this active iteration step
            let resultDeformation = calculatePhysicalDeformation(meshHeight, bird);
            simulatedTrials[currentTrialIndex] = resultDeformation;

            // Lock UI elements to preserve variable control constraints
            isAnimating = true;
            toggleControls(true);

            // Trigger Animation Pipeline
            executeDropAnimation(meshHeight, bird, resultDeformation);
        }

        function executeDropAnimation(meshHeight, bird, finalDeformation) {
            const startHeightCm = bird.dropHeight * 100;
            birdY = FLOOR_Y - (startHeightCm * SCALE);
            birdVelocity = 0;
            currentMeshStretch = 0;
            let netImpactRegistered = false;

            const gravityStep = 0.22; // Kinematic velocity integration constant per frame

            function animationLoop() {
                let meshBaseY = FLOOR_Y - (meshHeight * SCALE);

                if (!netImpactRegistered) {
                    birdVelocity += gravityStep;
                    birdY += birdVelocity;

                    if (meshHeight > 0 && birdY >= meshBaseY) {
                        netImpactRegistered = true;
                    } else if (birdY >= FLOOR_Y - bird.radius) {
                        birdY = FLOOR_Y - bird.radius;
                        wrapSingleTrial();
                        return;
                    }
                } else {
                    // Compression Phase inside the elastic netting boundaries
                    birdY += birdVelocity;
                    currentMeshStretch = birdY - meshBaseY;

                    if (finalDeformation === 0) {
                        // Structural arrest: kinetic energy safely absorbed by mesh
                        birdVelocity *= 0.7; // Kinetic damping factor
                        if (birdVelocity < 0.3) {
                            birdVelocity = 0;
                            setTimeout(wrapSingleTrial, 300);
                            return;
                        }
                    } else {
                        // Catastrophic failure: mesh strains past thresholds, hitting glass line
                        if (birdY >= FLOOR_Y - bird.radius) {
                            birdY = FLOOR_Y - bird.radius;
                            currentMeshStretch = FLOOR_Y - meshBaseY;
                            wrapSingleTrial();
                            return;
                        }
                    }
                }

                drawCanvas(meshHeight, bird, birdY, currentMeshStretch);
                animationId = requestAnimationFrame(animationLoop);
            }

            function wrapSingleTrial() {
                cancelAnimationFrame(animationId);
                isAnimating = false;

                // Log data snapshot to current configuration display table
                document.getElementById(`t${currentTrialIndex + 1}`).innerText = finalDeformation + " cm";

                currentTrialIndex++;

                if (currentTrialIndex < 3) {
                    // Update trigger prompts for next active sequence step
                    document.getElementById("actionBtn").innerText = `Run Trial ${currentTrialIndex + 1}`;
                    toggleControls(false); // unlock button to let user manually click next trial
                    document.getElementById("birdSelect").disabled = true; // keep core variables locked
                    document.getElementById("meshHeight").disabled = true;
                } else {
                    // End of 3-Trial Cycle: Process structural evaluations
                    finalizeExperimentalSet(meshHeight, bird);
                }
            }

            animationLoop();
        }

        function finalizeExperimentalSet(meshHeight, bird) {
            let sum = simulatedTrials[0] + simulatedTrials[1] + simulatedTrials[2];
            let averageDeformation = Math.round((sum / 3) * 10) / 10;
            
            document.getElementById("tAvg").innerText = averageDeformation + " cm";

            // Status message updating
            const statusDiv = document.getElementById("statusMessage");
            let outcomeText = "";
            let statusClass = "";

            if (averageDeformation === 0) {
                outcomeText = "SUCCESS: Netting successfully absorbed velocity. Zero clay deformation recorded.";
                statusClass = "status success";
            } else {
                outcomeText = `MORTALITY DETECTED: Energy transfer caused a ${averageDeformation} cm structural deformation footprint.`;
                statusClass = "status fail";
            }
            statusDiv.innerText = outcomeText;
            statusDiv.className = statusClass;
            statusDiv.style.display = "block";

            // 4. ARCHITECTURE VIEW: Push results snapshot to Permanent History Database Table
            commitToGlobalHistory(bird, meshHeight, averageDeformation, averageDeformation === 0 ? "SURVIVED" : "MORTALITY");

            // Reset step index parameters for subsequent system iterations
            currentTrialIndex = 0;
            simulatedTrials = [null, null, null];
            
            document.getElementById("actionBtn").innerText = "Reset for New Run";
            toggleControls(false);
        }

        function commitToGlobalHistory(bird, meshHeight, avgDef, outcome) {
            const historyBody = document.getElementById("historyBody");
            
            // Clear out placeholder message row if first entry
            if (globalRunCounter === 1) {
                historyBody.innerHTML = "";
            }

            const row = document.createElement("tr");
            if (outcome === "SURVIVED") {
                row.style.backgroundColor = "#f3faf5";
            } else {
                row.style.backgroundColor = "#fff9f9";
            }

            row.innerHTML = `
                <td><strong>#${globalRunCounter}</strong></td>
                <td>${bird.name}</td>
                <td>${bird.mass}g</td>
                <td>${bird.dropHeight}m</td>
                <td>${meshHeight} cm</td>
                <td>${document.getElementById("t1").innerText}</td>
                <td>${document.getElementById("t2").innerText}</td>
                <td>${document.getElementById("t3").innerText}</td>
                <td style="font-weight:bold;">${avgDef} cm</td>
                <td style="font-weight:bold; color: ${outcome === 'SURVIVED' ? '#1e7e34' : '#bd2130'}">${outcome}</td>
            `;
            
            // Prepend row to display newest iterations at top
            historyBody.insertBefore(row, historyBody.firstChild);
            globalRunCounter++;
        }

        // 5. ARCHITECTURE VIEW: Canvas Rendering Engine
        function drawCanvas(meshHeight, bird, currentBirdY, currentStretch) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw Window/Impact Floor Interface
            ctx.fillStyle = "#95a5a6";
            ctx.fillRect(0, FLOOR_Y, canvas.width, canvas.height - FLOOR_Y);
            ctx.fillStyle = "#34495e";
            ctx.fillRect(0, FLOOR_Y, canvas.width, 5); // Structural glass plate frame boundary

            // Draw Metric Measuring Tape
            ctx.strokeStyle = "#7f8c8d";
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(RULER_X, 20);
            ctx.lineTo(RULER_X, FLOOR_Y);
            ctx.stroke();

            // Calibrate tick intervals up to a 160cm range model window
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

            // Draw Structural Safety Netting
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

                // Crosshatched industrial safety appearance
                ctx.strokeStyle = "rgba(211, 84, 0, 0.25)";
                ctx.lineWidth = 1;
                for (let offset = -15; offset <= 15; offset += 6) {
                    ctx.beginPath();
                    ctx.moveTo(RULER_X + 5, meshBaseY + offset);
                    ctx.lineTo(midX, currentMidY + offset);
                    ctx.lineTo(canvas.width - 10, meshBaseY + offset);
                    ctx.stroke();
                }
            }

            // Draw Kinematic Bird Object
            let
