# FALL
import React, { useState, useCallback } from 'react';
import useGameStore from './store/useGameStore';

// Component Imports
import TitleScreen from './components/TitleScreen';
import WorldMap from './components/WorldMap';
import WardLobby from './components/WardLobby';
import BattleScreen from './components/BattleScreen';
import VictoryScreen from './components/VictoryScreen';
import DiagnosisDex from './components/DiagnosisDex';
import BadgesScreen from './components/BadgesScreen';

export default function App() {
  // Global State from Zustand
  const { 
    hp, 
    casesCompleted, 
    streak, 
    mentorUsedCount, 
    unlockBadge, 
    completeCase 
  } = useGameStore();

  // Local Navigation State [cite: 778-782]
  const [screen, setScreen] = useState("title");
  const [selectedWard, setSelectedWard] = useState(null);
  const [activeCase, setActiveCase] = useState(null);
  const [battleResults, setBattleResults] = useState(null);

  // Navigation Helper [cite: 783]
  const navigate = (target) => setScreen(target);

  // Achievement Logic [cite: 790-794]
  const checkAchievements = useCallback((score) => {
    if (casesCompleted + 1 >= 1) unlockBadge("first_blood"); [cite: 791]
    if (streak + 1 >= 3) unlockBadge("streak_3"); [cite: 792]
    if (score === 3) unlockBadge("perfect"); [cite: 792]
    if (mentorUsedCount >= 3) unlockBadge("mentor"); [cite: 793]
  }, [casesCompleted, streak, mentorUsedCount, unlockBadge]);

  // Handle Battle Completion [cite: 784-799]
  const handleBattleComplete = (score, phaseResults) => {
    checkAchievements(score);
    setBattleResults({ score, phaseResults });
    navigate("victory");
  };

  return (
    <div className="max-w-[440px] mx-auto min-h-screen bg-black relative shadow-2xl overflow-hidden shadow-cyan-900/20">
      {/* 1. Title Screen [cite: 805] */}
      {screen === "title" && (
        <TitleScreen onStart={() => navigate("world")} />
      )}

      {/* 2. World Map [cite: 806] */}
      {screen === "world" && (
        <WorldMap 
          onWardSelect={(ward) => { setSelectedWard(ward); navigate("ward"); }}
          onOpenDex={() => navigate("dex")}
          onOpenBadges={() => navigate("badges")}
        />
      )}

      {/* 3. Ward Lobby [cite: 807] */}
      {screen === "ward" && selectedWard && (
        <WardLobby 
          ward={selectedWard} 
          onCaseSelect={(c) => { setActiveCase(c); navigate("battle"); }}
          onBack={() => navigate("world")}
        />
      )}

      {/* 4. Battle Screen [cite: 808-814] */}
      {screen === "battle" && activeCase && (
        <BattleScreen 
          caseData={activeCase} 
          ward={selectedWard}
          onComplete={handleBattleComplete}
          onBack={() => navigate("ward")}
        />
      )}

      {/* 5. Victory Screen [cite: 815-820] */}
      {screen === "victory" && activeCase && battleResults && (
        <VictoryScreen 
          caseData={activeCase}
          ward={selectedWard}
          phaseResults={battleResults.phaseResults}
          onContinue={() => navigate("ward")}
        />
      )}

      {/* 6. Overlays [cite: 821-824] */}
      {screen === "dex" && (
        <DiagnosisDex onClose={() => navigate("world")} />
      )}
      {screen === "badges" && (
        <BadgesScreen onClose={() => navigate("world")} />
      )}

      {/* Global Scanline Effect [cite: 338] */}
      <div className="pointer-events-none fixed inset-0 z-50 bg-[linear-gradient(rgba(18,16,16,0)_50%,rgba(0,0,0,0.1)_50%),linear-gradient(90deg,rgba(255,0,0,0.02),rgba(0,255,0,0.01),rgba(0,0,255,0.02))] bg-[length:100%_2px,3px_100%]"></div>
    </div>
  );
}
