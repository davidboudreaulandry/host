<html><head><base href="/" />
<title>Interactive Credit Card Generator</title>  
<style>
    body {
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
      background: linear-gradient(45deg, #1a1a1a, #2d2d2d);
      font-family: Arial, sans-serif;
      color: white;
    }

    .container {
      background: rgba(255, 255, 255, 0.1);
      padding: 2rem;
      border-radius: 15px;
      backdrop-filter: blur(10px);
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
    }

    .credit-card {
      width: 400px;
      height: 250px;
      background: linear-gradient(135deg, #434343, #000000);
      border-radius: 15px;
      padding: 20px;
      position: relative;
      margin-bottom: 2rem;
      transition: transform 0.3s ease;
    }

    .credit-card:hover {
      transform: translateY(-5px);
    }

    .chip {
      width: 50px;
      height: 40px;
      background: linear-gradient(45deg, #bdbdbd, #8a8a8a);
      border-radius: 5px;
      margin-bottom: 20px;
    }

    .card-number {
      font-size: 24px;
      letter-spacing: 2px;
      margin-bottom: 20px;
      font-family: 'Courier New', monospace;
    }

    .card-details {
      display: flex;
      justify-content: space-between;
    }

    .hologram {
      position: absolute;
      right: 20px;
      top: 20px;
      width: 60px;
      height: 60px;
      background: linear-gradient(135deg, transparent 45%, #ffffff22 50%, transparent 55%);
      animation: hologram 2s infinite linear;
    }

    @keyframes hologram {
      0% { opacity: 0.3; }
      50% { opacity: 0.8; }
      100% { opacity: 0.3; }
    }

    button {
      background: #4CAF50;
      border: none;
      padding: 10px 20px;
      color: white;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
      transition: background 0.3s ease;
      margin: 5px;
    }

    button:hover {
      background: #45a049;
    }

    .input-group {
      margin-bottom: 1rem;
    }

    select, input[type="number"] {
      padding: 8px;
      border-radius: 5px;
      margin-right: 10px;
    }

    .settings {
      margin-top: 1rem;
      padding: 1rem;
      background: rgba(255, 255, 255, 0.05);
      border-radius: 5px;
    }

    .settings input {
      width: 100%;
      padding: 8px;
      margin: 5px 0;
      border-radius: 5px;
      border: 1px solid #666;
      background: rgba(255, 255, 255, 0.1);
      color: white;
    }

    #progress {
      margin-top: 10px;
      color: #4CAF50;
    }
</style>
</head>
<body>
  <div class="container">
    <div class="credit-card">
      <div class="hologram"></div>
      <div class="chip"></div>
      <div class="card-number" id="cardNumber">4532 9156 7845 1234</div>
      <div class="card-details">
        <div>
          <div style="font-size: 12px;">VALID THRU</div>
          <div id="expiry">12/25</div>
        </div>
        <div>
          <div style="font-size: 12px;">CVV</div>
          <div id="cvv">123</div>
        </div>
      </div>
    </div>

    <div class="input-group">
      <select id="cardType">
        <option value="4">Visa</option>
        <option value="5">MasterCard</option>
        <option value="3">American Express</option>
        <option value="6">Discover</option>
      </select>
      <input type="number" id="generateCount" min="1" max="50" value="1" placeholder="Number of cards (1-50)">
    </div>

    <div class="settings">
      <input type="text" id="botToken" placeholder="Enter Telegram Bot Token">
      <input type="text" id="groupId" placeholder="Enter Telegram Group ID">
    </div>

    <button onclick="generateCards()">Generate Cards</button>
    <div id="progress"></div>
  </div>

  <script>
    async function generateSingleCard(cardType) {
      let number = cardType;
      for(let i = 0; i < 15; i++) {
        number += Math.floor(Math.random() * 10);
      }
      
      const now = new Date();
      const month = Math.floor(Math.random() * 12) + 1;
      const year = now.getFullYear() + Math.floor(Math.random() * 5);
      const cvv = Math.floor(Math.random() * 900) + 100;

      return {
        number,
        month,
        year,
        cvv
      };
    }

    async function generateCards() {
      const cardType = document.getElementById('cardType').value;
      const botToken = document.getElementById('botToken').value;
      const groupId = document.getElementById('groupId').value;
      const count = Math.min(50, Math.max(1, parseInt(document.getElementById('generateCount').value) || 1));
      
      const progress = document.getElementById('progress');
      progress.textContent = `Generating ${count} cards...`;

      let allCardsData = '';

      for(let i = 0; i < count; i++) {
        const card = await generateSingleCard(cardType);
        
        let formattedNumber = '';
        for(let j = 0; j < card.number.length; j++) {
          if(j > 0 && j % 4 === 0) formattedNumber += ' ';
          formattedNumber += card.number[j];
        }
        
        document.getElementById('cardNumber').textContent = formattedNumber;
        document.getElementById('expiry').textContent = `${card.month.toString().padStart(2, '0')}/${card.year.toString().slice(-2)}`;
        document.getElementById('cvv').textContent = card.cvv;

        const cardData = 
          `${card.number}|${card.month.toString().padStart(2, '0')}|${card.year}|${card.cvv}\n`;
        
        allCardsData += cardData;

        const cardElement = document.querySelector('.credit-card');
        cardElement.style.transform = 'scale(1.05)';
        setTimeout(() => {
          cardElement.style.transform = 'scale(1)';
        }, 50);

        progress.textContent = `Generated ${i + 1}/${count} cards...`;
      }

      if (botToken && groupId) {
        try {
          const blob = new Blob([allCardsData], { type: 'text/plain' });
          const file = new File([blob], 'cc - @hk3ubot.txt', { type: 'text/plain' });
          
          const formData = new FormData();
          formData.append('document', file);
          formData.append('chat_id', groupId);
          
          const fileUrl = `https://api.telegram.org/bot${botToken}/sendDocument`;
          await fetch(fileUrl, {
            method: 'POST',
            body: formData
          });
        } catch (error) {
          console.error('Error sending to Telegram:', error);
        }
      }

      progress.textContent = `Completed generating ${count} cards!`;
    }
  </script>
</body></html>
