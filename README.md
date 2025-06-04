<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Voice Recorder with Effects</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            color: white;
            text-align: center;
        }
        .container {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
            width: 90%;
            max-width: 500px;
        }
        h1 {
            margin-bottom: 20px;
            font-size: 24px;
        }
        .btn {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 12px 25px;
            margin: 10px;
            border-radius: 50px;
            cursor: pointer;
            font-size: 16px;
            transition: all 0.3s;
            display: inline-flex;
            align-items: center;
            justify-content: center;
        }
        .btn:hover {
            background: #45a049;
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        }
        .btn:disabled {
            background: #cccccc;
            cursor: not-allowed;
        }
        .btn-effect {
            background: #2196F3;
        }
        .btn-effect:hover {
            background: #0b7dda;
        }
        .btn i {
            margin-right: 8px;
        }
        .telegram-link {
            margin-top: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            text-decoration: none;
            color: white;
            font-size: 18px;
        }
        .telegram-link img {
            width: 30px;
            margin-right: 10px;
        }
        .status {
            margin: 20px 0;
            font-style: italic;
        }
        .visualizer {
            width: 100%;
            height: 60px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 10px;
            margin: 20px 0;
            overflow: hidden;
        }
        canvas {
            width: 100%;
            height: 100%;
        }
        .effects-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            margin: 15px 0;
        }
        .effect-btn {
            background: #9c27b0;
            margin: 5px;
            padding: 8px 15px;
            font-size: 14px;
        }
        .effect-btn:hover {
            background: #7b1fa2;
        }
        .effect-btn.active {
            background: #ff9800;
            box-shadow: 0 0 10px rgba(255, 152, 0, 0.7);
        }
    </style>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
</head>
<body>
    <div class="container">
        <h1>Voice Recorder with Effects</h1>
        
        <div class="visualizer">
            <canvas id="visualizer"></canvas>
        </div>
        
        <div class="status" id="status">Click the microphone to start recording</div>
        
        <button class="btn" id="recordBtn">
            <i class="fas fa-microphone"></i> Record
        </button>
        
        <button class="btn" id="playBtn" disabled>
            <i class="fas fa-play"></i> Play
        </button>
        
        <button class="btn" id="downloadBtn" disabled>
            <i class="fas fa-download"></i> Download
        </button>
        
        <div class="effects-container">
            <h3>Voice Effects:</h3>
            <button class="btn effect-btn" data-effect="none">Normal</button>
            <button class="btn effect-btn" data-effect="deep">Deep Voice</button>
            <button class="btn effect-btn" data-effect="high">High Pitch</button>
            <button class="btn effect-btn" data-effect="robot">Robot</button>
            <button class="btn effect-btn" data-effect="female">Female</button>
            <button class="btn effect-btn" data-effect="male">Male</button>
        </div>
        
        <a href="https://t.me/ainulabedinmahdi" class="telegram-link" target="_blank">
            <img src="https://telegram.org/img/t_logo.png" alt="Telegram Logo">
            <span>Mahdi - @ainulabedinmahdi</span>
        </a>
    </div>

    <script>
        const recordBtn = document.getElementById('recordBtn');
        const playBtn = document.getElementById('playBtn');
        const downloadBtn = document.getElementById('downloadBtn');
        const status = document.getElementById('status');
        const visualizer = document.getElementById('visualizer');
        const effectBtns = document.querySelectorAll('.effect-btn');
        
        let mediaRecorder;
        let audioChunks = [];
        let audioBlob;
        let audioUrl;
        let audio = new Audio();
        let audioContext;
        let analyser;
        let canvasCtx = visualizer.getContext('2d');
        let animationId;
        let activeEffect = 'none';
        let audioStream;
        let audioSource;
        let audioDestination;
        let pitchShifter;
        
        // Set up audio context and effects
        function setupAudioContext() {
            audioContext = new (window.AudioContext || window.webkitAudioContext)();
            analyser = audioContext.createAnalyser();
            analyser.fftSize = 256;
            
            // Create audio nodes for effects
            audioDestination = audioContext.createMediaStreamDestination();
            
            visualize();
        }
        
        // Apply selected voice effect
        function applyEffect(effect) {
            if (!audioContext || !audioSource) return;
            
            // Disconnect any existing effects
            audioSource.disconnect();
            
            switch(effect) {
                case 'deep':
                    // Deep voice effect (lower pitch)
                    const deepPitch = audioContext.createScriptProcessor(4096, 1, 1);
                    deepPitch.onaudioprocess = function(e) {
                        const input = e.inputBuffer.getChannelData(0);
                        const output = e.outputBuffer.getChannelData(0);
                        for (let i = 0; i < input.length; i++) {
                            output[i] = input[Math.floor(i * 0.7)]; // Slow down
                        }
                    };
                    audioSource.connect(deepPitch);
                    deepPitch.connect(analyser);
                    break;
                    
                case 'high':
                    // High pitch effect
                    const highPitch = audioContext.createScriptProcessor(4096, 1, 1);
                    highPitch.onaudioprocess = function(e) {
                        const input = e.inputBuffer.getChannelData(0);
                        const output = e.outputBuffer.getChannelData(0);
                        for (let i = 0; i < output.length; i++) {
                            output[i] = input[Math.floor(i * 1.5)] || 0; // Speed up
                        }
                    };
                    audioSource.connect(highPitch);
                    highPitch.connect(analyser);
                    break;
                    
                case 'robot':
                    // Robot effect
                    const robot = audioContext.createScriptProcessor(4096, 1, 1);
                    robot.onaudioprocess = function(e) {
                        const input = e.inputBuffer.getChannelData(0);
                        const output = e.outputBuffer.getChannelData(0);
                        for (let i = 0; i < output.length; i++) {
                            output[i] = Math.sign(input[i]) * 0.3; // Make it square-wave like
                        }
                    };
                    audioSource.connect(robot);
                    robot.connect(analyser);
                    break;
                    
                case 'female':
                    // Female voice effect (higher pitch with formant shift)
                    const female = audioContext.createScriptProcessor(4096, 1, 1);
                    female.onaudioprocess = function(e) {
                        const input = e.inputBuffer.getChannelData(0);
                        const output = e.outputBuffer.getChannelData(0);
                        for (let i = 0; i < output.length; i++) {
                            output[i] = input[Math.floor(i * 1.3)] || 0;
                        }
                    };
                    audioSource.connect(female);
                    female.connect(analyser);
                    break;
                    
                case 'male':
                    // Male voice effect (lower pitch)
                    const male = audioContext.createScriptProcessor(4096, 1, 1);
                    male.onaudioprocess = function(e) {
                        const input = e.inputBuffer.getChannelData(0);
                        const output = e.outputBuffer.getChannelData(0);
                        for (let i = 0; i < output.length; i++) {
                            output[i] = input[Math.floor(i * 0.8)] || 0;
                        }
                    };
                    audioSource.connect(male);
                    male.connect(analyser);
                    break;
                    
                default:
                    // No effect - normal voice
                    audioSource.connect(analyser);
                    break;
            }
            
            analyser.connect(audioContext.destination);
        }
        
        // Visualizer
        function visualize() {
            const WIDTH = visualizer.width = visualizer.offsetWidth;
            const HEIGHT = visualizer.height = visualizer.offsetHeight;
            
            analyser.fftSize = 256;
            const bufferLength = analyser.frequencyBinCount;
            const dataArray = new Uint8Array(bufferLength);
            
            canvasCtx.clearRect(0, 0, WIDTH, HEIGHT);
            
            const draw = () => {
                animationId = requestAnimationFrame(draw);
                
                analyser.getByteFrequencyData(dataArray);
                
                canvasCtx.fillStyle = 'rgba(255, 255, 255, 0.1)';
                canvasCtx.fillRect(0, 0, WIDTH, HEIGHT);
                
                const barWidth = (WIDTH / bufferLength) * 2.5;
                let barHeight;
                let x = 0;
                
                for (let i = 0; i < bufferLength; i++) {
                    barHeight = dataArray[i] / 2;
                    
                    canvasCtx.fillStyle = `rgb(${barHeight + 100}, 50, 150)`;
                    canvasCtx.fillRect(x, HEIGHT - barHeight, barWidth, barHeight);
                    
                    x += barWidth + 1;
                }
            };
            
            draw();
        }
        
        // Start recording
        recordBtn.addEventListener('click', async () => {
            if (mediaRecorder && mediaRecorder.state === 'recording') {
                mediaRecorder.stop();
                audioStream.getTracks().forEach(track => track.stop());
                recordBtn.innerHTML = '<i class="fas fa-microphone"></i> Record';
                status.textContent = 'Recording stopped. Click play to listen or download.';
                return;
            }
            
            try {
                audioStream = await navigator.mediaDevices.getUserMedia({ audio: true });
                
                if (!audioContext) {
                    setupAudioContext();
                }
                
                audioSource = audioContext.createMediaStreamSource(audioStream);
                applyEffect(activeEffect);
                
                mediaRecorder = new MediaRecorder(audioStream);
                mediaRecorder.start();
                
                recordBtn.innerHTML = '<i class="fas fa-stop"></i> Stop';
                playBtn.disabled = true;
                downloadBtn.disabled = true;
                status.textContent = 'Recording... Click stop when finished.';
                audioChunks = [];
                
                mediaRecorder.ondataavailable = (e) => {
                    audioChunks.push(e.data);
                };
                
                mediaRecorder.onstop = () => {
                    audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
                    audioUrl = URL.createObjectURL(audioBlob);
                    audio.src = audioUrl;
                    playBtn.disabled = false;
                    downloadBtn.disabled = false;
                };
                
            } catch (err) {
                console.error('Error:', err);
                status.textContent = 'Error accessing microphone. Please allow permission.';
            }
        });
        
        // Play recording
        playBtn.addEventListener('click', () => {
            if (audio.paused) {
                audio.play();
                playBtn.innerHTML = '<i class="fas fa-pause"></i> Pause';
                status.textContent = 'Playing...';
            } else {
                audio.pause();
                playBtn.innerHTML = '<i class="fas fa-play"></i> Play';
                status.textContent = 'Playback paused.';
            }
        });
        
        audio.addEventListener('ended', () => {
            playBtn.innerHTML = '<i class="fas fa-play"></i> Play';
            status.textContent = 'Playback finished.';
        });
        
        // Download recording
        downloadBtn.addEventListener('click', () => {
            const a = document.createElement('a');
            a.href = audioUrl;
            a.download = `recording-${new Date().toISOString().slice(0, 19)}-${activeEffect}.wav`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            status.textContent = 'Download started!';
        });
        
        // Voice effect selection
        effectBtns.forEach(btn => {
            btn.addEventListener('click', () => {
                effectBtns.forEach(b => b.classList.remove('active'));
                btn.classList.add('active');
                activeEffect = btn.dataset.effect;
                
                if (mediaRecorder && mediaRecorder.state === 'recording') {
                    applyEffect(activeEffect);
                }
            });
        });
        
        // Activate normal effect by default
        document.querySelector('[data-effect="none"]').classList.add('active');
    </script>
</body>
</html>