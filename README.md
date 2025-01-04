<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Music & Vocal Mixer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f3f3f3;
            text-align: center;
            padding: 20px;
        }
        h1 {
            color: #333;
        }
        .upload-section {
            margin: 20px 0;
        }
        input[type="file"] {
            margin: 10px 0;
        }
        button {
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        button:hover {
            background-color: #45a049;
        }
    </style>
</head>
<body>
    <h1>Music & Vocal Mixer</h1>
    <div class="upload-section">
        <h3>Upload Music Track:</h3>
        <input type="file" id="musicFile" accept="audio/*">
    </div>
    <div class="upload-section">
        <h3>Upload Your Vocal Track:</h3>
        <input type="file" id="vocalFile" accept="audio/*">
    </div>
    <button onclick="mixTracks()">Play Mixed Track</button>

    <h3>Preview:</h3>
    <audio id="audioPlayer" controls></audio>

    <script>
        function mixTracks() {
            const musicFile = document.getElementById('musicFile').files[0];
            const vocalFile = document.getElementById('vocalFile').files[0];

            if (!musicFile || !vocalFile) {
                alert('Please upload both music and vocal tracks.');
                return;
            }

            const musicURL = URL.createObjectURL(musicFile);
            const vocalURL = URL.createObjectURL(vocalFile);

            const audioPlayer = document.getElementById('audioPlayer');
            audioPlayer.src = musicURL;
            audioPlayer.play();

            const audioContext = new (window.AudioContext || window.webkitAudioContext)();

            Promise.all([
                fetch(musicURL).then(response => response.arrayBuffer()).then(data => audioContext.decodeAudioData(data)),
                fetch(vocalURL).then(response => response.arrayBuffer()).then(data => audioContext.decodeAudioData(data))
            ]).then(([musicBuffer, vocalBuffer]) => {
                const mixedBuffer = audioContext.createBuffer(
                    musicBuffer.numberOfChannels,
                    Math.min(musicBuffer.length, vocalBuffer.length),
                    audioContext.sampleRate
                );

                for (let channel = 0; channel < mixedBuffer.numberOfChannels; channel++) {
                    const musicData = musicBuffer.getChannelData(channel);
                    const vocalData = vocalBuffer.getChannelData(channel);
                    const mixedData = mixedBuffer.getChannelData(channel);

                    for (let i = 0; i < mixedData.length; i++) {
                        mixedData[i] = musicData[i] + vocalData[i];
                    }
                }

                const mixedSource = audioContext.createBufferSource();
                mixedSource.buffer = mixedBuffer;
                mixedSource.connect(audioContext.destination);
                mixedSource.start();
            }).catch(error => console.error('Error mixing tracks:', error));
        }
    </script>
</body>
</html>
