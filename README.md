import React, { useState } from 'react';
import './App.css';

// --- Types ---
interface Chemical {
  id: string;
  name: string;
  formula: string;
  type: 'acid' | 'base' | 'neutral';
  pH: number;
  color: string;
}

interface Indicator {
  id: string;
  name: string;
  description: string;
  icon: string;
}

// --- Databases ---
const CHEMICALS: Chemical[] = [
  // --- ACIDS ---
  { id: 'hcl', name: 'Hydrochloric Acid', formula: 'HCl', type: 'acid', pH: 1.0, color: '#f87171' },
  { id: 'h2so4', name: 'Sulfuric Acid', formula: 'H₂SO₄', type: 'acid', pH: 0.5, color: '#ef4444' },
  { id: 'hno3', name: 'Nitric Acid', formula: 'HNO₃', type: 'acid', pH: 1.2, color: '#dc2626' },
  { id: 'ch3cooh', name: 'Acetic Acid (Vinegar)', formula: 'CH₃COOH', type: 'acid', pH: 2.8, color: '#fca5a5' },
  { id: 'citric', name: 'Citric Acid (Lemon Juice)', formula: 'C₆H₈O₇', type: 'acid', pH: 2.2, color: '#fde047' },
  { id: 'h3po4', name: 'Phosphoric Acid (Soda)', formula: 'H₃PO₄', type: 'acid', pH: 2.5, color: '#fb923c' },
  { id: 'h2co3', name: 'Carbonic Acid', formula: 'H₂CO₃', type: 'acid', pH: 4.2, color: '#fef08a' },
  { id: 'h3bo3', name: 'Boric Acid', formula: 'H₃BO₃', type: 'acid', pH: 5.1, color: '#fef9c3' },

  // --- NEUTRAL COMPOUNDS ---
  { id: 'water', name: 'Distilled Water', formula: 'H₂O', type: 'neutral', pH: 7.0, color: '#38bdf8' },
  { id: 'nacl', name: 'Sodium Chloride (Salt)', formula: 'NaCl', type: 'neutral', pH: 7.0, color: '#a5f3fc' },
  { id: 'glucose', name: 'Glucose Solution', formula: 'C₆H₁₂O₆', type: 'neutral', pH: 7.0, color: '#cbd5e1' },
  { id: 'kcl', name: 'Potassium Chloride', formula: 'KCl', type: 'neutral', pH: 7.0, color: '#e2e8f0' },
  { id: 'c2h5oh', name: 'Ethanol Solution', formula: 'C₂H₅OH', type: 'neutral', pH: 7.1, color: '#93c5fd' },

  // --- BASES ---
  { id: 'baking_soda', name: 'Sodium Bicarbonate', formula: 'NaHCO₃', type: 'base', pH: 8.4, color: '#bfdbfe' },
  { id: 'mgoh2', name: 'Magnesium Hydroxide', formula: 'Mg(OH)₂', type: 'base', pH: 10.5, color: '#818cf8' },
  { id: 'nh3', name: 'Ammonia Solution', formula: 'NH₃', type: 'base', pH: 11.6, color: '#a7f3d0' },
  { id: 'caoh2', name: 'Calcium Hydroxide (Limewater)', formula: 'Ca(OH)₂', type: 'base', pH: 12.4, color: '#6ee7b7' },
  { id: 'naoh', name: 'Sodium Hydroxide', formula: 'NaOH', type: 'base', pH: 13.5, color: '#c084fc' },
  { id: 'koh', name: 'Potassium Hydroxide', formula: 'KOH', type: 'base', pH: 13.8, color: '#e879f9' }
];

const INDICATORS: Indicator[] = [
  { id: 'ph_meter', name: 'Digital pH Probe', description: 'Displays exact pH value on digital screen', icon: '📟' },
  { id: 'litmus_red', name: 'Red Litmus Paper', description: 'Turns Blue in bases, stays Red in acids', icon: '🔴' },
  { id: 'litmus_blue', name: 'Blue Litmus Paper', description: 'Turns Red in acids, stays Blue in bases', icon: '🔵' },
  { id: 'universal', name: 'Universal Indicator', description: 'Shows full rainbow color spectrum for pH 0-14', icon: '🌈' },
  { id: 'pheno', name: 'Phenolphthalein', description: 'Turns vibrant pink in bases (pH > 8.2)', icon: '🧪' },
  { id: 'cabbage', name: 'Red Cabbage Extract', description: 'Natural indicator changing from Red to Green/Yellow', icon: '🥬' },
  { id: 'methyl_orange', name: 'Methyl Orange', description: 'Red in strong acids, yellow in neutrals and bases', icon: '🟠' }
];

export default function App() {
  // Authentication State
  const [isLoggedIn, setIsLoggedIn] = useState<boolean>(false);
  const [loginMethod, setLoginMethod] = useState<'google' | 'phone'>('google');
  const [phoneNumber, setPhoneNumber] = useState<string>('');

  // Navigation State: 'home' | 'main_lab' | 'indicators_lab'
  const [currentView, setCurrentView] = useState<'home' | 'main_lab' | 'indicators_lab'>('home');

  // Main Lab Workbench State
  const [mainBeaker, setMainBeaker] = useState<Chemical[]>([]);
  const [mainLogs, setMainLogs] = useState<string[]>(['Laboratory initialized. All chemicals unlocked.']);
  const [isBeakerHovered, setIsBeakerHovered] = useState<boolean>(false);

  // Indicators Lab State
  const [indChemical, setIndChemical] = useState<Chemical | null>(null);
  const [indIndicator, setIndIndicator] = useState<Indicator | null>(null);
  const [isIndBeakerHovered, setIsIndBeakerHovered] = useState<boolean>(false);

  // Login Handler
  const handleLogin = (e?: React.FormEvent) => {
    if (e) e.preventDefault();
    setIsLoggedIn(true);
    setCurrentView('home');
  };

  // Main Lab Drag & Drop
  const handleMainDragStart = (e: React.DragEvent, chem: Chemical) => {
    e.dataTransfer.setData('application/json', JSON.stringify({ source: 'chemical', item: chem }));
  };

  const handleMainDrop = (e: React.DragEvent) => {
    e.preventDefault();
    setIsBeakerHovered(false);
    const data = e.dataTransfer.getData('application/json');
    if (!data) return;

    const parsed = JSON.parse(data);
    if (parsed.source === 'chemical') {
      const chem: Chemical = parsed.item;
      if (mainBeaker.length >= 4) {
        setMainLogs(prev => [`⚠️ Beaker is full! Clean beaker to reset.`, ...prev]);
        return;
      }
      const updated = [...mainBeaker, chem];
      setMainBeaker(updated);
      setMainLogs(prev => [`Added ${chem.name} (${chem.formula})`, ...prev]);

      // Check Reaction
      const hasAcid = updated.some(c => c.type === 'acid');
      const hasBase = updated.some(c => c.type === 'base');
      if (hasAcid && hasBase) {
        setMainLogs(prev => [`⚡ Reaction Occurred: Acid + Base Neutralization! Salt & Water formed.`, ...prev]);
      }
    }
  };

  // Indicators Lab Drag & Drop
  const handleIndDragStart = (e: React.DragEvent, item: Chemical | Indicator, type: 'chemical' | 'indicator') => {
    e.dataTransfer.setData('application/json', JSON.stringify({ source: type, item }));
  };

  const handleIndDrop = (e: React.DragEvent) => {
    e.preventDefault();
    setIsIndBeakerHovered(false);
    const data = e.dataTransfer.getData('application/json');
    if (!data) return;

    const parsed = JSON.parse(data);
    if (parsed.source === 'chemical') {
      setIndChemical(parsed.item);
    } else if (parsed.source === 'indicator') {
      setIndIndicator(parsed.item);
    }
  };

  // Compute Indicator Color / Result
  const getIndicatorResult = () => {
    if (!indChemical || !indIndicator) return { color: 'transparent', text: 'Drag a chemical AND an indicator here' };

    const ph = indChemical.pH;

    switch (indIndicator.id) {
      case 'ph_meter':
        return {
          color: '#020617',
          text: `pH Value: ${ph.toFixed(1)} (${ph < 7 ? 'Acidic' : ph > 7 ? 'Alkaline / Basic' : 'Neutral'})`
        };
      case 'litmus_red':
        return {
          color: ph > 7 ? '#3b82f6' : '#ef4444',
          text: ph > 7 ? 'Red Litmus turned BLUE (Base)' : 'Red Litmus stayed RED (Acid/Neutral)'
        };
      case 'litmus_blue':
        return {
          color: ph < 7 ? '#ef4444' : '#3b82f6',
          text: ph < 7 ? 'Blue Litmus turned RED (Acid)' : 'Blue Litmus stayed BLUE (Base/Neutral)'
        };
      case 'universal':
        if (ph < 3) return { color: '#ef4444', text: 'Strong Acid (Universal Red)' };
        if (ph < 6) return { color: '#f97316', text: 'Weak Acid (Universal Orange/Yellow)' };
        if (ph <= 7.5) return { color: '#22c55e', text: 'Neutral (Universal Green)' };
        if (ph < 11) return { color: '#06b6d4', text: 'Weak Base (Universal Blue)' };
        return { color: '#a855f7', text: 'Strong Base (Universal Purple)' };
      case 'pheno':
        return {
          color: ph >= 8.2 ? '#ec4899' : '#e2e8f0',
          text: ph >= 8.2 ? 'Turned Vibrant Pink (Basic)' : 'Remained Colorless (Acid/Neutral)'
        };
      case 'cabbage':
        if (ph < 3) return { color: '#ef4444', text: 'Bright Red' };
        if (ph < 7) return { color: '#a855f7', text: 'Purple / Violet' };
        if (ph === 7) return { color: '#3b82f6', text: 'Blue' };
        if (ph < 11) return { color: '#10b981', text: 'Green' };
        return { color: '#facc15', text: 'Yellow' };
      case 'methyl_orange':
        return {
          color: ph <= 3.1 ? '#ef4444' : ph <= 4.4 ? '#f97316' : '#facc15',
          text: ph <= 3.1 ? 'Red (pH ≤ 3.1)' : ph <= 4.4 ? 'Orange (pH 3.1 - 4.4)' : 'Yellow (pH > 4.4)'
        };
      default:
        return { color: 'transparent', text: '' };
    }
  };

  return (
    <div className="app-container">
      {/* GLOBAL NAVBAR WITH RETURN BUTTONS */}
      <header className="navbar">
        <div className="nav-brand" onClick={() => setCurrentView('home')}>
          🧪 <span>Experimentalitmus</span>
        </div>

        <div className="nav-controls">
          {isLoggedIn && currentView !== 'home' && (
            <button className="btn nav-btn" onClick={() => setCurrentView('home')}>
              🏠 Return to Home
            </button>
          )}

          {isLoggedIn ? (
            <button className="btn logout-btn" onClick={() => setIsLoggedIn(false)}>
              🔒 Logout
            </button>
          ) : (
            <span className="login-status-badge">Guest Session</span>
          )}
        </div>
      </header>

      {/* VIEW 1: HOME & LOGIN INTERFACE */}
      {currentView === 'home' && (
        <main className="home-view">
          {!isLoggedIn ? (
            <div className="login-card">
              <h2>Welcome to Virtual Lab</h2>
              <p>Sign in to access interactive chemical experiments.</p>

              <div className="login-tabs">
                <button
                  className={`tab ${loginMethod === 'google' ? 'active' : ''}`}
                  onClick={() => setLoginMethod('google')}
                >
                  Google Login
                </button>
                <button
                  className={`tab ${loginMethod === 'phone' ? 'active' : ''}`}
                  onClick={() => setLoginMethod('phone')}
                >
                  Phone Number
                </button>
              </div>

              {loginMethod === 'google' ? (
                <button className="btn google-btn" onClick={() => handleLogin()}>
                  <img src="https://www.svgrepo.com/show/475656/google-color.svg" alt="Google" />
                  Sign in with Google
                </button>
              ) : (
                <form onSubmit={handleLogin} className="phone-form">
                  <input
                    type="tel"
                    placeholder="Enter Phone Number"
                    value={phoneNumber}
                    onChange={(e) => setPhoneNumber(e.target.value)}
                    required
                  />
                  <button type="submit" className="btn submit-btn">Send OTP & Sign In</button>
                </form>
              )}
            </div>
          ) : (
            <div className="hero-section">
              <div className="hero-content">
                <span className="badge">REALISTIC VIRTUAL SIMULATOR</span>
                <h1>Explore Chemistry Without Limits</h1>
                <p>
                  Access all acids, bases, and neutral compounds instantly. 
                  Conduct reactions in the main lab or analyze solutions in the indicators workstation.
                </p>

                <div className="hero-action-buttons">
                  <button className="btn primary-start-btn" onClick={() => setCurrentView('main_lab')}>
                    ⚗️ Start Experiment (Main Lab)
                  </button>
                  <button className="btn secondary-start-btn" onClick={() => setCurrentView('indicators_lab')}>
                    🌈 Go to Indicators Lab
                  </button>
                </div>
              </div>
            </div>
          )}
        </main>
      )}

      {/* VIEW 2: MAIN LAB WORKBENCH */}
      {currentView === 'main_lab' && (
        <main className="lab-view-layout">
          {/* LEFT SIDE: ALL UNLOCKED CHEMICALS */}
          <aside className="sidebar left-sidebar">
            <h3>🧪 Chemicals Catalog</h3>
            <p className="sidebar-hint">Drag any chemical into the central beaker</p>
            <div className="inventory-list">
              {CHEMICALS.map((chem) => (
                <div
                  key={chem.id}
                  className="chemical-card"
                  draggable
                  onDragStart={(e) => handleMainDragStart(e, chem)}
                  onClick={() => {
                    setMainBeaker(prev => [...prev.slice(-3), chem]);
                    setMainLogs(prev => [`Added ${chem.name}`, ...prev]);
                  }}
                >
                  <div className="chem-dot" style={{ backgroundColor: chem.color }} />
                  <div className="chem-info">
                    <strong>{chem.name}</strong>
                    <small>{chem.formula} • pH {chem.pH}</small>
                  </div>
                  <span className="drag-icon">🖐️</span>
                </div>
              ))}
            </div>
          </aside>

          {/* CENTER AREA: REACTION BENCH */}
          <section className="center-stage">
            <div className="stage-header">
              <h2>Main Laboratory Reaction Bench</h2>
              <button className="btn clear-btn" onClick={() => setMainBeaker([])}>Clean Beaker</button>
            </div>

            <div
              className={`beaker-dropzone ${isBeakerHovered ? 'hovered' : ''}`}
              onDragOver={(e) => { e.preventDefault(); setIsBeakerHovered(true); }}
              onDragLeave={() => setIsBeakerHovered(false)}
              onDrop={handleMainDrop}
            >
              <div className="beaker-glass">
                <div
                  className="liquid-fill"
                  style={{
                    height: `${(mainBeaker.length / 4) * 85}%`,
                    backgroundColor: mainBeaker.length > 0 ? mainBeaker[mainBeaker.length - 1].color : 'transparent'
                  }}
                />
              </div>
              <p className="drop-hint">
                {mainBeaker.length === 0 ? 'Drop Chemicals Here' : `${mainBeaker.length} Compound(s) Mixed`}
              </p>
            </div>

            <div className="terminal-log">
              <h4>Reaction Log:</h4>
              <div className="log-entries">
                {mainLogs.map((log, index) => (
                  <div key={index} className="log-line">› {log}</div>
                ))}
              </div>
            </div>
          </section>

          {/* RIGHT SIDE: SMALL VISIBLE BUTTON TO OPEN INDICATORS LAB */}
          <aside className="sidebar right-sidebar compact-nav-sidebar">
            <div className="switch-card">
              <span className="icon-large">🌈</span>
              <h3>Indicators Lab</h3>
              <p>Test pH levels using litmus paper, cabbage extract, and universal indicators.</p>
              <button className="btn switch-lab-btn" onClick={() => setCurrentView('indicators_lab')}>
                Open Indicators Lab ➡️
              </button>
            </div>
          </aside>
        </main>
      )}

      {/* VIEW 3: INDICATORS LAB */}
      {currentView === 'indicators_lab' && (
        <main className="lab-view-layout">
          {/* LEFT SIDE: CHEMICALS */}
          <aside className="sidebar left-sidebar">
            <button className="btn return-lab-btn" onClick={() => setCurrentView('main_lab')}>
              ⬅️ Back to Main Lab
            </button>
            <h3>🧪 Step 1: Chemicals</h3>
            <p className="sidebar-hint">Drag a chemical to the testing stage</p>
            <div className="inventory-list">
              {CHEMICALS.map((chem) => (
                <div
                  key={chem.id}
                  className={`chemical-card ${indChemical?.id === chem.id ? 'selected' : ''}`}
                  draggable
                  onDragStart={(e) => handleIndDragStart(e, chem, 'chemical')}
                  onClick={() => setIndChemical(chem)}
                >
                  <div className="chem-dot" style={{ backgroundColor: chem.color }} />
                  <div className="chem-info">
                    <strong>{chem.name}</strong>
                    <small>{chem.formula}</small>
                  </div>
                </div>
              ))}
            </div>
          </aside>

          {/* CENTER AREA: TESTING WORKBENCH */}
          <section className="center-stage">
            <div className="stage-header">
              <h2>pH & Indicator Testing Station</h2>
              <button className="btn clear-btn" onClick={() => { setIndChemical(null); setIndIndicator(null); }}>
                Reset Test
              </button>
            </div>

            <div
              className={`indicator-dropzone ${isIndBeakerHovered ? 'hovered' : ''}`}
              onDragOver={(e) => { e.preventDefault(); setIsIndBeakerHovered(true); }}
              onDragLeave={() => setIsIndBeakerHovered(false)}
              onDrop={handleIndDrop}
            >
              <div className="testing-slots">
                <div className="slot">
                  <small>Chemical Selected</small>
                  <strong>{indChemical ? `${indChemical.name} (${indChemical.formula})` : 'None (Drag Here)'}</strong>
                </div>

                <div className="plus-sign">+</div>

                <div className="slot">
                  <small>Indicator Selected</small>
                  <strong>{indIndicator ? indIndicator.name : 'None (Drag Here)'}</strong>
                </div>
              </div>

              {/* Reaction Output Display */}
              <div
                className="result-box"
                style={{ backgroundColor: getIndicatorResult().color }}
              >
                <div className="result-text">{getIndicatorResult().text}</div>
              </div>
            </div>
          </section>

          {/* RIGHT SIDE: ALL INDICATORS */}
          <aside className="sidebar right-sidebar">
            <h3>🌈 Step 2: Indicators</h3>
            <p className="sidebar-hint">Drag an indicator to test the chemical</p>
            <div className="inventory-list">
              {INDICATORS.map((ind) => (
                <div
                  key={ind.id}
                  className={`indicator-card ${indIndicator?.id === ind.id ? 'selected' : ''}`}
                  draggable
                  onDragStart={(e) => handleIndDragStart(e, ind, 'indicator')}
                  onClick={() => setIndIndicator(ind)}
                >
                  <span className="ind-icon">{ind.icon}</span>
                  <div className="ind-info">
                    <strong>{ind.name}</strong>
                    <small>{ind.description}</small>
                  </div>
                </div>
              ))}
            </div>
          </aside>
        </main>
      )}
    </div>
  );
}
