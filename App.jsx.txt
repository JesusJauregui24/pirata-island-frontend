import React, { useState, useEffect } from 'react';

const API_URL = 'https://pirata-island-backend.onrender.com/api';

export default function App() {
  const [player, setPlayer] = useState(null);
  const [loading, setLoading] = useState(true);
  const [battleLog, setBattleLog] = useState('Explora la isla para cazar piratas.');

  // Simulamos el ID de Telegram del usuario al entrar
  useEffect(() => {
    fetch(`${API_URL}/auth`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ telegramId: 'user_12345', username: 'Nakama_Test' })
    })
      .then(res => res.json())
      .then(data => {
        setPlayer(data);
        setLoading(false);
      })
      .catch(err => console.error(err));
  }, []);

  const handleFight = async () => {
    if (!player || player.energy < 10) return;
    const res = await fetch(`${API_URL}/fight`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ telegramId: player.telegramId })
    });
    const data = await res.json();
    if (data.success) {
      setPlayer(data.player);
      setBattleLog(`¡Derrotaste a un pirata de la Marina y ganaste ${data.earnedBeli} Beli! ⚔️`);
    }
  };

  const handleClaim = async () => {
    const res = await fetch(`${API_URL}/claim`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ telegramId: player.telegramId })
    });
    const data = await res.json();
    if (data.success) {
      setPlayer(prev => ({ ...prev, cryptoTokens: data.cryptoTokens }));
      setBattleLog('¡Has recolectado tus ganancias de cripto de las Frutas del Diablo! 💎');
    }
  };

  if (loading) return <div className="bg-slate-950 text-white h-screen flex items-center justify-center font-bold">Cargando Pirata Island... 🏴‍☠️</div>;

  return (
    <div className="bg-slate-950 text-white min-h-screen p-4 flex flex-col justify-between font-sans">
      {/* Cabecera de Estadísticas */}
      <div className="bg-slate-900 border border-amber-500/30 rounded-xl p-4 shadow-lg">
        <h1 className="text-xl font-black text-amber-400 text-center tracking-wider mb-2">🏴‍☠️ PIRATA ISLAND 🌴</h1>
        <div className="grid grid-cols-2 gap-2 text-sm text-center">
          <div className="bg-slate-800 p-2 rounded-lg border border-slate-700">
            <p className="text-slate-400 text-xs">Beli (Moneda)</p>
            <p className="font-bold text-amber-300">🪙 {player.beli}</p>
          </div>
          <div className="bg-slate-800 p-2 rounded-lg border border-slate-700">
            <p className="text-slate-400 text-xs">$DBLI (Crypto)</p>
            <p className="font-bold text-cyan-400">💎 {player.cryptoTokens.toFixed(2)}</p>
          </div>
        </div>
      </div>

      {/* Área Central / Mapa de la Isla */}
      <div className="my-auto text-center bg-gradient-to-b from-indigo-950 to-slate-900 border border-indigo-500/30 rounded-2xl p-6 shadow-2xl">
        <div className="text-6xl mb-4 animate-bounce">🏝️⚔️</div>
        <p className="text-sm text-indigo-200 mb-6 bg-slate-950/50 p-3 rounded-lg border border-indigo-500/20">{battleLog}</p>
        
        <div className="flex flex-col gap-3">
          <button 
            onClick={handleFight}
            className="w-full bg-gradient-to-r from-amber-500 to-orange-600 hover:from-amber-600 hover:to-orange-700 text-slate-950 font-black py-3 px-6 rounded-xl shadow-lg transition transform active:scale-95">
            Caminar y Atacar Piratas 🗡️ (-10 Energía)
          </button>
          
          <button 
            onClick={handleClaim}
            className="w-full bg-gradient-to-r from-cyan-500 to-blue-600 hover:from-cyan-600 hover:to-blue-700 text-slate-950 font-black py-3 px-6 rounded-xl shadow-lg transition transform active:scale-95">
            Recoger Cripto de las Frutas 🍈
          </button>
        </div>
      </div>

      {/* Pie de página / Info de Frutas */}
      <div className="bg-slate-900 border border-slate-800 rounded-xl p-3 text-xs text-slate-400 text-center">
        Frutas Activas: <span className="text-amber-400 font-semibold">{player.fruits.length} equipadas</span>
      </div>
    </div>
  );
}