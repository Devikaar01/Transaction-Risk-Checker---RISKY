/** @jsxImportSource https://esm.sh/react@18.2.0 */
import React, { useState } from "https://esm.sh/react@18.2.0";
import { createRoot } from "https://esm.sh/react-dom@18.2.0/client";

function App() {
  const [amount, setAmount] = useState('');
  const [recipient, setRecipient] = useState('');
  const [status, setStatus] = useState('');

  const submitTransaction = async (e: React.FormEvent) => {
    e.preventDefault();
    setStatus('🔄 Checking transaction...');
    
    try {
      const response = await fetch('/assess', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ amount: parseFloat(amount), recipient })
      });

      if (!response.ok) throw new Error("Server Error");

      const result = await response.json();
      setStatus(result.message);
    } catch (error) {
      console.error("Transaction check failed:", error);
      setStatus('❌ Transaction assessment failed. Try again.');
    }
  };

  return (
    <div style={{
      maxWidth: '400px', 
      margin: 'auto', 
      padding: '20px', 
      fontFamily: 'Arial, sans-serif', 
      border: '1px solid #ddd',
      borderRadius: '8px',
      boxShadow: '0px 4px 8px rgba(0, 0, 0, 0.1)'
    }}>
      <h1 style={{ textAlign: 'center', color: '#ff4500' }}>🚨 Transaction Risk Checker 💸</h1>
      
      <form onSubmit={submitTransaction} style={{ display: 'flex', flexDirection: 'column', gap: '10px' }}>
        <label>Amount ($)</label>
        <input 
          type="number" 
          value={amount} 
          onChange={(e) => setAmount(e.target.value)}
          required 
          style={{ padding: '8px', borderRadius: '5px', border: '1px solid #ccc' }}
        />
        
        <label>Recipient</label>
        <input 
          type="text" 
          value={recipient} 
          onChange={(e) => setRecipient(e.target.value)}
          required 
          style={{ padding: '8px', borderRadius: '5px', border: '1px solid #ccc' }}
        />
        
        <button type="submit" style={{
          padding: '10px', 
          backgroundColor: '#ff4500', 
          color: 'white', 
          border: 'none', 
          borderRadius: '5px',
          cursor: 'pointer'
        }}>
          Check Transaction
        </button>
      </form>

      {status && <p style={{ textAlign: 'center', marginTop: '10px', fontWeight: 'bold' }}>{status}</p>}
    </div>
  );
}

function client() {
  createRoot(document.getElementById("root")).render(<App />);
}
if (typeof document !== "undefined") { client(); }

let flaggedTransactions: { amount: number, recipient: string, date: string }[] = [];

export default async function server(request: Request): Promise<Response> {
  if (request.method === 'POST' && new URL(request.url).pathname === '/assess') {
    const { email } = await import("https://esm.town/v/std/email");
    const data = await request.json();
    
    const isHighRisk = assessTransactionRisk(data);
    
    if (isHighRisk) {
      flaggedTransactions.push({
        amount: data.amount, 
        recipient: data.recipient,
        date: new Date().toISOString()
      });

      await email({
        subject: "⚠️ High-Risk Transaction Detected",
        text: `Risky transaction detected:\nAmount: $${data.amount}\nRecipient: ${data.recipient}`,
      });
    }

    return new Response(JSON.stringify({
      message: isHighRisk 
        ? "🚨 High-risk transaction! Notification sent." 
        : "✅ Transaction appears safe."
    }), {
      headers: { 'Content-Type': 'application/json' }
    });
  }

  if (request.method === 'GET' && new URL(request.url).pathname === '/logs') {
    return new Response(JSON.stringify(flaggedTransactions), {
      headers: { 'Content-Type': 'application/json' }
    });
  }

  return new Response(`
    <html>
      <head>
        <title>Transaction Risk Checker</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
      </head>
      <body>
        <div id="root"></div>
        <script type="module" src="${import.meta.url}"></script>
      </body>
    </html>
  `, {
    headers: { 'Content-Type': 'text/html' }
  });
}

function assessTransactionRisk(transaction: { amount: number, recipient: string }): boolean {
  const RISK_THRESHOLD = 5000;
  const HIGH_RISK_RECIPIENTS = ['unknown', 'suspicious', 'scam'];

  // Fetch user's IP geolocation (mocked here)
  const userLocation = getUserGeolocation();
  const isForeignTransaction = userLocation !== "Trusted Country";

  // Assign a risk score
  let riskScore = 0;
  if (transaction.amount > RISK_THRESHOLD) riskScore += 50;
  if (HIGH_RISK_RECIPIENTS.some(r => transaction.recipient.toLowerCase().includes(r))) riskScore += 30;
  if (isForeignTransaction) riskScore += 20;

  return riskScore >= 50;
}

function getUserGeolocation(): string {
  return "Unknown Country"; // Mock function, replace with an API if needed
}

