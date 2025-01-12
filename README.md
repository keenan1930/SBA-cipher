<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>凯撒密码解密工具</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f0f0f0;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .input-section {
            display: grid;
            grid-template-columns: 1fr auto;
            gap: 10px;
            align-items: start;
            margin-bottom: 20px;
        }
        .cipher-input {
            grid-column: 1 / -1;
        }
        .letter-stats {
            display: flex;
            gap: 10px;
            align-items: center;
            background-color: #f5f5f5;
            padding: 10px;
            border-radius: 4px;
            margin-bottom: 10px;
        }
        .letter-input {
            width: 60px !important;
            text-align: center;
            font-size: 16px;
        }
        .result-card {
            border: 1px solid #ddd;
            padding: 15px;
            margin: 10px 0;
            border-radius: 4px;
            background-color: white;
            transition: all 0.3s ease;
        }
        .result-card.active {
            border-color: #4CAF50;
            background-color: #f9fff9;
        }
        .full-text {
            background-color: #f8f8f8;
            padding: 15px;
            margin-top: 10px;
            border-radius: 4px;
            white-space: pre-wrap;
            word-break: break-all;
            display: none;
            border: 1px solid #eee;
            max-height: 300px;
            overflow-y: auto;
        }
        textarea {
            width: 100%;
            padding: 10px;
            margin: 5px 0;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
            min-height: 100px;
            font-size: 14px;
        }
        input {
            width: 100%;
            padding: 8px;
            margin: 5px 0;
            box-sizing: border-box;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin: 5px 0;
            white-space: nowrap;
        }
        button:hover {
            background-color: #45a049;
        }
        .preview-text {
            font-family: monospace;
            margin: 5px 0;
        }
        .stats {
            color: #666;
            margin: 5px 0;
        }
        .toggle-button {
            background-color: #2196F3;
        }
        .toggle-button:hover {
            background-color: #1976D2;
        }
        .count-result {
            margin-left: 10px;
            padding: 5px 10px;
            background-color: #e8f5e9;
            border-radius: 4px;
            white-space: nowrap;
        }
        @media (max-width: 600px) {
            body {
                padding: 10px;
            }
            .container {
                padding: 10px;
            }
            .input-section {
                grid-template-columns: 1fr;
            }
            .letter-stats {
                flex-direction: column;
            }
            .letter-input {
                width: 100% !important;
            }
            .count-result {
                margin-left: 0;
                margin-top: 5px;
                text-align: center;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>凯撒密码解密工具</h1>
        
        <div class="input-section">
            <div class="cipher-input">
                <textarea id="ciphertext" placeholder="请输入需要解密的密文"></textarea>
            </div>
            
            <div class="letter-stats">
                <input type="text" id="letterInput" class="letter-input" maxlength="1" placeholder="字母">
                <button onclick="countLetter()">统计字母</button>
                <div id="countResult" class="count-result"></div>
            </div>

            <button onclick="decryptText()">解密文本</button>
        </div>

        <div id="decryptResults"></div>
    </div>

    <script>
        let currentResults = [];

        function decrypt(ciphertext, shift) {
            let plaintext = "";
            for (let char of ciphertext) {
                if (/[a-zA-Z]/.test(char)) {
                    const isUpper = char === char.toUpperCase();
                    const base = isUpper ? 'A'.charCodeAt(0) : 'a'.charCodeAt(0);
                    const decrypted = String.fromCharCode((char.charCodeAt(0) - base - shift + 26) % 26 + base);
                    plaintext += decrypted;
                } else {
                    plaintext += char;
                }
            }
            return plaintext;
        }

        function analyzeFrequency(text) {
            const freq = {};
            const textLower = text.toLowerCase();
            const totalChars = textLower.split('').filter(c => /[a-z]/.test(c)).length;
            
            if (totalChars === 0) return {};
            
            for (let char of 'abcdefghijklmnopqrstuvwxyz') {
                const count = (textLower.match(new RegExp(char, 'g')) || []).length;
                freq[char] = totalChars > 0 ? count / totalChars : 0;
            }
            return freq;
        }

        function toggleFullText(shift) {
            const currentCard = document.getElementById(`card-${shift}`);
            const currentText = document.getElementById(`full-text-${shift}`);
            
            if (currentText.style.display === 'none') {
                currentCard.classList.add('active');
                currentText.style.display = 'block';
            } else {
                currentCard.classList.remove('active');
                currentText.style.display = 'none';
            }
        }

        function decryptText() {
            const ciphertext = document.getElementById('ciphertext').value.trim();
            if (!ciphertext) {
                alert('错误：密文不能为空');
                return;
            }

            const resultsDiv = document.getElementById('decryptResults');
            resultsDiv.innerHTML = '<h3>解密结果（按匹配度排序）：</h3>';
            currentResults = [];

            for (let shift = 1; shift <= 25; shift++) {
                const decryptedText = decrypt(ciphertext, shift);
                const freq = analyzeFrequency(decryptedText);
                
                let score = 0;
                'etaoinshrdlcumwfgypbvkjxqz'.split('').forEach((letter, i) => {
                    const weight = 26 - i;
                    if (freq[letter]) {
                        score += freq[letter] * weight;
                    }
                });
                
                currentResults.push({
                    shift,
                    text: decryptedText,
                    score,
                    freq
                });
            }

            currentResults.sort((a, b) => b.score - a.score);

            currentResults.forEach(result => {
                const card = document.createElement('div');
                card.id = `card-${result.shift}`;
                card.className = 'result-card';
                
                const commonLetters = Object.entries(result.freq)
                    .sort((a, b) => b[1] - a[1])
                    .slice(0, 3)
                    .map(([letter]) => letter);

                const preview = result.text.slice(0, 50) + (result.text.length > 50 ? '...' : '');

                card.innerHTML = `
                    <div class="stats">
                        <strong>偏移值：${result.shift}</strong> | 
                        匹配得分：${result.score.toFixed(3)} | 
                        最常见字母：${commonLetters.join(', ')}
                    </div>
                    <div class="preview-text">预览：${preview}</div>
                    <button class="toggle-button" onclick="toggleFullText(${result.shift})">
                        显示/隐藏完整文本
                    </button>
                    <div id="full-text-${result.shift}" class="full-text"></div>
                `;
                
                resultsDiv.appendChild(card);
                
                const fullTextDiv = document.getElementById(`full-text-${result.shift}`);
                fullTextDiv.textContent = result.text;
            });
        }

        function countLetter() {
            const text = document.getElementById('ciphertext').value;
            const letter = document.getElementById('letterInput').value.toLowerCase();
            
            if (!letter || letter.length !== 1 || !/[a-z]/i.test(letter)) {
                alert('请输入单个字母！');
                return;
            }

            const count = (text.toLowerCase().match(new RegExp(letter, 'g')) || []).length;
            document.getElementById('countResult').innerHTML = `
                '${letter}' 出现 ${count} 次
            `;
        }
    </script>
</body>
</html>
