<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Çizim Sayfası</title>
    <style>
        body {
            background-color: #000;
            color: #fff;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }

        #canvasContainer {
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: flex-start; /* Sol tarafa hizalama */
        }

        #canvas {
            border: 1px solid #fff;
            background-color: #fff; /* Çizim tahtasının arkaplan rengi */
        }

        label, select {
            color: #fff;
            background-color: #000;
            margin-right: 10px;
            margin-bottom: 5px;
            padding: 5px;
        }

        #downloadBtn {
            color: #fff;
            background-color: #000;
            padding: 8px;
            border: none;
            cursor: pointer;
            margin-top: 10px;
            display: block; /* Butonları blok eleman yapma */
        }

        #undoBtn,
        #redoBtn,
        #clearBtn {
            color: #fff;
            background-color: #000;
            padding: 8px;
            border: none;
            cursor: pointer;
            margin-top: 10px;
            display: block; /* Butonları blok eleman yapma */
        }

        #undoBtn,
        #redoBtn,
        #clearBtn {
            margin-left: 5px; /* Aralarında boşluk bırakma miktarı ayarlanabilir */
        }

        #undoBtn:disabled,
        #redoBtn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        #downloadOptions {
            display: none;
            text-align: center;
            margin-top: 10px;
        }

        #downloadOptions button {
            margin: 5px;
        }

        .textbox-container {
            width: 300px;
            height: 300px;
            border: none; /* Kenar çizgilerini kaldırma */
            padding: 20px;
            box-sizing: border-box;
            margin-top: 20px; /* Çizim tablosundan biraz aşağıya kaydırma */
        }

        .textbox {
            width: calc(100% - 20px); /* Sağ ve sol kenarlardan 10px'lik boşluk bırakır */
            height: 100%;
            border: none;
            outline: none;
            resize: none;
            font-size: 16px;
            margin-left: 10px; /* Yazı yazma kutusunu sağa kaydırma */
            background-color: #fff; /* Yazı yazma kutusunun içini beyaz yapma */
            color: #000; /* Yazı rengini siyah yapma */
        }
    </style>
</head>
<body>

    <div id="canvasContainer">
        <label for="penSize">Kalem Boyutu:</label>
        <select id="penSize" onchange="changePenSize()">
            <option value="2">İnce</option>
            <option value="5" selected>Orta</option>
            <option value="10">Kalın</option>
        </select>
        <br>
        <label for="penColor">Renk:</label>
        <select id="penColor" onchange="changePenColor()">
            <option value="#000000">Siyah</option>
            <option value="#FF0000">Kırmızı</option>
            <option value="#FFFF00">Sarı</option>
            <option value="#FFFFFF">Beyaz</option>
            <option value="#8B4513">Kahverengi</option>
            <option value="#FFA500">Turuncu</option>
            <option value="#800080">Mor</option>
            <option value="#FFC0CB">Pembe</option>
            <option value="#0000FF">Mavi</option>
        </select>

        <canvas id="canvas" width="800" height="600"></canvas>

        <div id="downloadOptions">
            <button onclick="downloadCanvas('image/jpeg')">JPG Olarak İndir</button>
            <button onclick="downloadCanvas('image/png')">PNG Olarak İndir</button>
        </div>

        <button id="downloadBtn" onclick="showDownloadOptions()">Resmi İndir</button>
    </div>

    <div class="textbox-container">
        <textarea class="textbox" placeholder="Buraya metin yazabilirsiniz..."></textarea>
        <div id="textControls">
            <button id="undoBtn" onclick="undo()" disabled>Geri Al</button>
            <button id="redoBtn" onclick="redo()" disabled>Yeniden Yap</button>
            <button id="clearBtn" onclick="clearCanvas()">Hepsini Temizle</button>
        </div>
    </div>

    <script>
        var canvas = document.getElementById("canvas");
        var context = canvas.getContext("2d");
        var isDrawing = false;
        var isErasing = false;
        var drawingHistory = [];
        var undoHistory = [];
        var penSize = 5; // Başlangıçta orta kalınlık
        var penColor = "#000000"; // Başlangıçta siyah renk

        canvas.addEventListener("mousedown", handleMouseDown);
        canvas.addEventListener("mouseup", handleMouseUp);
        canvas.addEventListener("mousemove", handleMouseMove);
        canvas.addEventListener("contextmenu", handleContextMenu);

        function handleMouseDown(e) {
            e.preventDefault();
            isDrawing = true;
            if (e.button === 2) {
                isErasing = true;
            }
            handleMouseMove(e);
        }

        function handleMouseUp() {
            isDrawing = false;
            isErasing = false;
            context.beginPath();
            saveDrawing();
        }

        function handleMouseMove(e) {
            if (!isDrawing) return;

            context.lineWidth = penSize;
            context.lineCap = "round";

            if (isErasing) {
                context.strokeStyle = "#FFFFFF";
            } else {
                context.strokeStyle = penColor;
            }

            var rect = canvas.getBoundingClientRect();
            var x = e.clientX - rect.left;
            var y = e.clientY - rect.top;

            context.lineTo(x, y);
            context.stroke();
            context.beginPath();
            context.moveTo(x, y);

            // Çizim sırasında yeni bir çizim yapıldığında undo geçmişini temizle
            undoHistory = [];
        }

        function handleContextMenu(e) {
            e.preventDefault();
            isErasing = !isErasing;
        }

        function clearCanvas() {
            context.clearRect(0, 0, canvas.width, canvas.height);
            drawingHistory = [];
            undoHistory = [];
            updateUndoRedoButtons();
        }

        function clearTextbox() {
            var textbox = document.querySelector('.textbox');
            textbox.value = '';
            updateUndoRedoButtons();
        }

        function saveDrawing() {
            var tempCanvas = document.createElement("canvas");
            tempCanvas.width = canvas.width;
            tempCanvas.height = canvas.height;
            var tempContext = tempCanvas.getContext("2d");
            tempContext.fillStyle = "#fff"; // Geçici canvas'ın arka plan rengi
            tempContext.fillRect(0, 0, tempCanvas.width, tempCanvas.height);
            tempContext.drawImage(canvas, 0, 0);
            drawingHistory.push(tempCanvas);
            updateUndoRedoButtons();
        }

        function changePenSize() {
            penSize = document.getElementById("penSize").value;
        }

        function changePenColor() {
            penColor = document.getElementById("penColor").value;
        }

        function showDownloadOptions() {
            var optionsDiv = document.getElementById("downloadOptions");
            optionsDiv.style.display = "block";
        }

        function downloadCanvas(format) {
            var link = document.createElement('a');
            link.href = drawingHistory[drawingHistory.length - 1].toDataURL(format);
            if (format === 'image/jpeg') {
                link.download = 'resim.jpg';
            } else if (format === 'image/png') {
                link.download = 'resim.png';
            }
            link.click();
            var optionsDiv = document.getElementById("downloadOptions");
            optionsDiv.style.display = "none";
        }

        function undo() {
            if (drawingHistory.length > 1) {
                undoHistory.push(drawingHistory.pop());
                context.clearRect(0, 0, canvas.width, canvas.height);
                context.drawImage(drawingHistory[drawingHistory.length - 1], 0, 0);
                updateUndoRedoButtons();
            }
        }

        function redo() {
            if (undoHistory.length > 0) {
                drawingHistory.push(undoHistory.pop());
                context.clearRect(0, 0, canvas.width, canvas.height);
                context.drawImage(drawingHistory[drawingHistory.length - 1], 0, 0);
                updateUndoRedoButtons();
            }
        }

        function updateUndoRedoButtons() {
            var undoBtn = document.getElementById("undoBtn");
            var redoBtn = document.getElementById("redoBtn");
            var clearBtn = document.getElementById("clearBtn");

            undoBtn.disabled = drawingHistory.length <= 1;
            redoBtn.disabled = undoHistory.length === 0;
            clearBtn.disabled = drawingHistory.length === 0 && undoHistory.length === 0;
        }
    </script>

</body>
</html>
