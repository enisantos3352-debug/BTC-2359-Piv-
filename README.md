
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Robô Ponto Zero - Web Simulator</title>
    <style>
        body { font-family: Arial, sans-serif; background: #121212; color: #fff; text-align: center; padding: 20px; }
        canvas { background: #1e1e1e; border: 1px solid #333; margin-top: 20px; }
        .info { font-size: 18px; margin: 10px; }
        .red { color: #ff4444; font-weight: bold; }
        .yellow { color: #ffeb3b; font-weight: bold; }
        .green { color: #00C853; font-weight: bold; }
    </style>
</head>
<body>

    <h2>Robo Ponto Zero - Análise de Volatilidade</h2>
    <div class="info">Máxima (Paredão): <span id="maxVal" class="red">0.00</span></div>
    <div class="info">Centro (Linha Amarela): <span id="centroVal" class="yellow">0.00</span></div>
    <div class="info">Mínima (Paredão): <span id="minVal" class="red">0.00</span></div>
    <div class="info">Preço Atual: <span id="precoAtual" class="green">0.00</span></div>

    <canvas id="grafico" width="800" height="400"></canvas>

    <script>
        async function carregarDados() {
            try {
                // Pega dados de 1 minuto do Bitcoin na Binance
                let resposta = await fetch('https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=1m&limit=200');
                let dados = await resposta.json();

                let precosHigh = [];
                let precosLow = [];
                let precosClose = [];

                dados.forEach(vela => {
                    precosHigh.push(parseFloat(vela[2]));
                    precosLow.push(parseFloat(vela[3]));
                    precosClose.push(parseFloat(vela[4]));
                });

                let maximaGlobal = Math.max(...precosHigh);
                let minimaGlobal = Math.min(...precosLow);
                let centroGlobal = (maximaGlobal + minimaGlobal) / 2;
                let precoAtual = precosClose[precosClose.length - 1];

                // Atualiza os textos na tela
                document.getElementById('maxVal').innerText = maximaGlobal.toFixed(2);
                document.getElementById('minVal').innerText = minimaGlobal.toFixed(2);
                document.getElementById('centroVal').innerText = centroGlobal.toFixed(2);
                document.getElementById('precoAtual').innerText = precoAtual.toFixed(2);

                desenharGrafico(precosClose, maximaGlobal, minimaGlobal, centroGlobal);

            } catch (erro) {
                console.log("Erro ao buscar dados:", erro);
            }
        }

        function desenharGrafico(precos, max, min, centro) {
            let canvas = document.getElementById('grafico');
            let ctx = canvas.getContext('2d');
            let largura = canvas.width;
            let altura = canvas.height;

            ctx.clearRect(0, 0, largura, altura);

            let margem = max - min;
            if (margem === 0) margem = 1;

            function mapearY(preco) {
                return altura - ((preco - min) / margem) * (altura - 40) - 20;
            }

            // Desenha as velas / linha de preço
            ctx.beginPath();
            ctx.strokeStyle = '#00C853';
            ctx.lineWidth = 2;
            for (let i = 0; i < precos.length; i++) {
                let x = (i / (precos.length - 1)) * largura;
                let y = mapearY(precos[i]);
                if (i === 0) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);
            }
            ctx.stroke();

            // Linha da Máxima (Vermelha)
            let yMax = mapearY(max);
            ctx.beginPath();
            ctx.strokeStyle = '#ff4444';
            ctx.setLineDash([5, 5]);
            ctx.moveTo(0, yMax);
            ctx.lineTo(largura, yMax);
            ctx.stroke();

            // Linha da Mínima (Vermelha)
            let yMin = mapearY(min);
            ctx.beginPath();
            ctx.moveTo(0, yMin);
            ctx.lineTo(largura, yMin);
            ctx.stroke();

            // Linha do Centro (Amarela)
            let yCentro = mapearY(centro);
            ctx.beginPath();
            ctx.strokeStyle = '#ffeb3b';
            ctx.setLineDash([]);
            ctx.lineWidth = 3;
            ctx.moveTo(0, yCentro);
            ctx.lineTo(largura, yCentro);
            ctx.stroke();
        }

        // Atualiza a cada 5 segundos
        carregarDados();
        setInterval(carregarDados, 5000);
    </script>

</body>
</html>

